# Map interaction

[Documentation index](../README.md)

## Runtime objects

| Type/member | Purpose |
|---|---|
| `LevelMapCursor` | Active map cursor |
| `LevelMapCursor.context` | Context for the displayed map |
| `LevelMapController` | Map controller, including cursor movement |
| `LevelMapContext.level` | Displayed map's `LevelID` |
| `LevelMapContext.currentDistrictMainSelection` | District currently selected on the map |
| `LevelMapContext.staticLevelMapItems` | Static map-item collection |
| `LevelMapContext.dynamicLevelMapItems` | Dynamic map-item collection |
| `LevelMapSavePointItem` | Save-point marker |
| `LevelMapSavePointItem.context` | Context associated with that marker |
| `LevelMapHoverableStrategySavePoint` | Save-point hover behavior |
| `DistrictData.ContainsRoom(roomName)` | Tests room membership in a district |

These are runtime UI objects, not the player's current room or world position. The map can display a different level from the one the character occupies.

## Detecting an active map

An active `LevelMapCursor` is an available practical indicator that the level-map interface is active:

```csharp
using Il2CppInterop.Runtime;
using UnityEngine;

static LevelMapCursor FindActiveMapCursor()
{
    foreach (var obj in UnityEngine.Object.FindObjectsOfType(
                 Il2CppType.Of<LevelMapCursor>()))
    {
        var cursor = obj.TryCast<LevelMapCursor>();
        if (cursor != null && cursor.isActiveAndEnabled)
            return cursor;
    }
    return null;
}
```

Find an active `LevelMapController` in the same way. Validate cached wrappers and active state each update; a closed/destroyed map can invalidate them. Search at a bounded interval rather than repeatedly scanning every frame.

This is not a verified universal map-open flag for every map surface, including world-map views that may use different objects. If multiple active candidates exist, resolve the cursor/controller belonging to the intended UI instance.

## Reading displayed context

```csharp
var cursor = FindActiveMapCursor();
var context = cursor?.context;
if (context?.level != null)
{
    string levelName = context.level.name;
    string levelGuid = context.level.guid;
    string districtName = context.currentDistrictMainSelection?.name;
    Debug.Log($"Displayed level: {levelName} ({levelGuid}); district: {districtName}");
}
```

A selected district can be unavailable while context is changing. Read the selection after the UI has updated rather than assuming it changes synchronously with every cursor operation.

## Observing save-point hover

```csharp
using HarmonyLib;
using UnityEngine;

[HarmonyPatch]
static class MapSavePointObserver
{
    [HarmonyPostfix]
    [HarmonyPatch(typeof(LevelMapHoverableStrategySavePoint),
        nameof(LevelMapHoverableStrategySavePoint.OnHover))]
    static void AfterHover(LevelMapHoverableStrategySavePoint __instance)
    {
        var item = __instance?.savePointItem;
        var context = item?.context;
        Debug.Log($"Save point hover: level={context?.level?.name}; "
            + $"district={context?.currentDistrictMainSelection?.name}");
        // Retain the exact marker/context if the consumer needs selection state.
    }

    [HarmonyPostfix]
    [HarmonyPatch(typeof(LevelMapHoverableStrategySavePoint),
        nameof(LevelMapHoverableStrategySavePoint.OnUnHover))]
    static void AfterUnHover(LevelMapHoverableStrategySavePoint __instance)
    {
        // Clear only selection state owned by this marker/UI instance.
    }
}
```

Install once with `Harmony.CreateAndPatchAll`. These snippets are illustrative and have not been compiled or runtime-tested here. A hover callback identifies a marker interaction, not proof the destination is unlocked or reachable.

Clear stale hover/selection state when the map closes, changes level, or destroys the marker. If a positional fallback also maintains selection, keep its state separate from actual hover callbacks.

## Mapping a marker to a mirror destination

A practical resolution strategy combines:

```text
Marker/context displayed level GUID
+ selected district name or district room membership
→ candidate mirror destination
→ used-mirror flag check
```

For a destination record, compare:

```csharp
bool sameLevel = context.level.guid == destinationLevelGuid;
bool sameDistrict = district.name == destinationDistrictName;
bool roomInDistrict = destinationRoomName != null
    && district.ContainsRoom(destinationRoomName);

bool candidate = sameLevel && (sameDistrict || roomInDistrict);
```

The destination fields are application-owned catalog data, not fields guaranteed on `LevelMapSavePointItem`. Obtain `district` from the validated map context.

A district match is not a guaranteed unique marker identity. If several mirrors occupy the same district, use additional marker/position information and report ambiguity rather than selecting the first candidate blindly.

Mirror eligibility requires the first Death progression key (`DeathCatacombs_Defeated_ID`) and the matching destination `MC_Used_SaveRoom_*` flag. Eleanor is not the travel lock. See [mirror travel.md](mirror%20travel.md) and [progression flags.md](../03%20-%20Progression%20and%20cutscenes/progression%20flags.md).

## Positional fallback

The level-map cursor transform can be compared against known destination world-space X/Y coordinates for the corresponding level map:

```csharp
Vector3 cursorPosition = cursor.transform.position;
float distance = Vector2.Distance(
    new Vector2(cursorPosition.x, cursorPosition.y),
    destinationWorldPosition);
```

First require the destination level GUID to match the displayed level. Then choose the nearest eligible destination within a deliberately chosen tolerance.

This is a fallback heuristic. A tolerance such as seven units is a controller choice, not a game constant. Check coordinate alignment for the current map view, zoom/context, and game build. Close markers can remain ambiguous, and unknown coordinates cannot be inferred from cursor proximity.

Do not confuse map cursor position with the playable character's position.

## Enumerating save-point markers

Both static and dynamic collections can contain save points:

```csharp
using System.Collections.Generic;

static List<LevelMapSavePointItem> FindSavePointMarkers(LevelMapContext context)
{
    var result = new List<LevelMapSavePointItem>();
    if (context == null) return result;

    foreach (var list in new[]
             { context.staticLevelMapItems, context.dynamicLevelMapItems })
    {
        if (list == null) continue;
        foreach (var item in list)
        {
            var savePoint = item?.TryCast<LevelMapSavePointItem>();
            if (savePoint != null) result.Add(savePoint);
        }
    }
    return result;
}
```

Deduplicate marker identities if a marker appears in both collections. Rebuild the list when map context changes. A loaded marker does not itself prove used-mirror eligibility.

## Moving the map cursor

```csharp
controller.TeleportCursorToLevelMapItem(savePointMarker);
```

This moves the map cursor. It does not teleport the playable character or launch a biome transition.

If no direct marker-to-destination identity is available, a controller can inspect candidates sequentially:

```text
Enumerate save-point markers on displayed map
→ move cursor to one marker
→ wait for selection context to refresh
→ read selected district/marker context
→ compare against intended destination
→ stop on a validated match or continue
```

Moving through candidates changes visible UI state. Bind the search to the current map level and cancel if the user closes/navigates the map or changes selection. Use bounded attempts and allow for frame-based updates; do not assume a fixed number of frames is universally sufficient.

This does not establish a method for switching arbitrary map levels or zooming to the world map. Moving the cursor on one map cannot reveal a marker belonging only to another displayed level.

## Selecting a destination and travelling

Map interaction and character travel should be separate operations:

```text
Hover/select marker
→ resolve and validate destination
→ request travel
→ map closes and gameplay becomes ready
→ start level/biome transition
```

A custom confirmation prompt or keyboard shortcut is application UI behavior, not a native game travel API. Revalidate destination, unlock state, player entity, and gameplay state when executing the request. Cancel expired requests rather than carrying a stale selection into later gameplay.

Transition execution is documented in [level biome transitions.md](level%20biome%20transitions.md); mirror-specific flags/destinations are in [mirror travel.md](mirror%20travel.md); coordinate movement is in [teleportation.md](teleportation.md).

## Learning destination positions from runtime events

The used-save-room key can be observed through `ProgressionComponent.SetProgressionKey`. If it corresponds to a known mirror, capture the player's position and room at the interaction.

This is a candidate location observation, not an automatic authoritative coordinate. The setter can also be invoked programmatically or during another lifecycle path. Validate that the player is actually at the intended save point and distinguish a new interaction from restored state.

## Boundaries

The runtime APIs above support map-context inspection, save-point hover observation, marker enumeration, and cursor movement. They do not establish a universal open/close-map command, arbitrary zoom/level navigation, every map-marker type, or exact screen-to-world coordinate conversion for every view.

Map completion percentages and room-color logic are separate systems documented in [room map completion.md](room%20map%20completion.md).



