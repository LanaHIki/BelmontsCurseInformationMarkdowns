# Save/load lifecycle

[Documentation index](../README.md)

## Lifecycle overview

Loading a save restores persistent data and rebuilds or updates the playable runtime. Save-data restoration, player availability, room readiness, and completed spawn placement are separate milestones.

```text
Save load begins
→ saved progression is read
→ player components and scene objects become available
→ entrance/spawn setup places the character
→ gameplay state resumes
→ deferred changes can be applied and verified
```

This is a practical readiness model, not a verified ordering of every internal engine callback. A single callback does not establish that the entire load has finished.

## Save-data restoration observation point

`ProgressionComponent.ReadPlayerSave` is an available method for observing progression restoration. A postfix observer runs after that method returns:

```csharp
[HarmonyPostfix]
[HarmonyPatch(typeof(ProgressionComponent), nameof(ProgressionComponent.ReadPlayerSave))]
static void AfterProgressionRead()
{
    // Invalidate cached runtime references as needed.
    // Schedule readiness checks; avoid assuming the world is fully loaded.
}
```

The callback identifies a progression-data read. It is not, by itself, a full-load-complete event or proof that inventory, stats, room objects, and placement are all ready.

## Reacquiring the player

Use the active `VaniaMovesComponent` in a valid scene:

```csharp
VaniaMovesComponent FindPlayer()
{
    foreach (var obj in Resources.FindObjectsOfTypeAll(
                 Il2CppType.Of<VaniaMovesComponent>()))
    {
        var candidate = obj.TryCast<VaniaMovesComponent>();
        if (candidate != null
            && candidate.gameObject.scene.IsValid()
            && candidate.gameObject.activeInHierarchy)
            return candidate;
    }
    return null;
}
```

Runtime references can become invalid across scene changes or loads. A managed reference being non-null alone is insufficient; accessing its underlying Unity object can also fail after destruction. Revalidate cached objects and search again when necessary.

From the active player:

| Required object | Lookup |
|---|---|
| Inventory | Parent `InventoryComponent`, or one under the player's root; read `inventory` |
| Experience | Parent `ExperienceComponent`, or one under the player's root |
| Player entity | `ExperienceComponent.linkedEntity` |
| Stats | `StatsStateComponent` from the player entity |
| Progression | `ProgressionComponent` from the player entity |

Entity components may be exposed through `GetComponentData` or `GetComponent`; check the result before use.

## Room availability

`CurrentRoomInstance.currentRoom` identifies a loaded room. Its `gameObject.scene.name` provides the scene name; its `worldRoomData` provides room metadata.

```text
Find CurrentRoomInstance
→ check currentRoom exists
→ inspect scene / worldRoomData
```

A missing room during transition is a reason to defer room-dependent operations. An available room alone does not prove that the player has finished spawning.

## Gameplay readiness

Useful state APIs:

```csharp
var stateManager = GameStateManager.GetInstance();
if (stateManager != null)
{
    bool gameplay = stateManager.IsGameplayState();
    var activeState = stateManager.GetActiveStateId();
}
```

Prefer `IsGameplayState()` over assuming the numeric meaning of an active state ID. A missing manager or unreadable state should be treated as unknown readiness.

Before a deferred mutation, check the objects needed for that operation:

1. Active player exists in a valid scene.
2. Required inventory/entity/stat component exists.
3. Gameplay state is appropriate.
4. Destination/current room is available when relevant.
5. Conflicting item acquisition, cutscene, or placement work has finished.

Run game mutations on Unity's main thread. Recheck readiness when executing queued work, not only when initially scheduling it.

## Acquisition sequences

Item acquisition has its own lifecycle:

```csharp
var acquisition = ItemAcquisitionGlobalController.GetInstance();
bool busy = acquisition != null
    && !acquisition.IsAcquisitionSequenceFinished();
```

Starting another acquisition sequence during an existing one can conflict with its UI and callbacks. Inventory mutation and opening an acquisition screen are separate operations. Defer grants when the required game systems are unavailable or an incompatible sequence is active.

## Stat restoration and deferred mutation

Do not apply stat changes solely because progression reading has returned. Wait for the player entity and its `StatsStateComponent`, then inspect the restored value before changing it.

```text
Wait for stable runtime
→ obtain current StatsStateComponent
→ read current value
→ calculate intended change
→ apply through the stat API
→ reread result
```

A fixed delay can supplement readiness checks but is not a game guarantee. No universal load duration is established here.

An operation can change a stat before a later reaction or UI callback throws. Verify the value before retrying to avoid applying the same increment twice.

## Inventory restoration

To inspect ownership after loading:

```csharp
var item = itemsManager.FindItem(gameItemId);
if (item != null && inventory != null)
{
    bool owned = inventory.HasItemInInventoryOrEquipped(item);
}
```

Use the inventory belonging to the newly validated player. Loading an older save can restore older ownership and progression state. Do not assume that a successful runtime grant was persisted unless a later save/load confirms it.

Ownership checks are useful for unique items. They do not establish quantities or justify replaying consumable, XP, or stat increments.

## Level transitions and spawn placement

Level transitions also rebuild playable state. Relevant spawn APIs include:

- `MainCharacterSpawnLogic.SetupBiomeTransition`
- `MainCharacterSpawnLogic.PlaceMainCharacter`
- `MainCharacterSpawnLogic.SetForcePosition(position, null)`
- `MainCharacterSpawnLogic.ClearForcePosition()`

```text
Transition setup
→ entrance configuration
→ player placement
→ movement/gameplay resumes
```

Entrance setup can reset a requested position, and movement/spawn logic can override a direct transform assignment. Coordinate changes during loading therefore may not stick.

For deliberate spawn overrides, placement can be controlled through the spawn system and verified after gameplay resumes. Clear temporary forced positions afterward: leaving one active can affect subsequent respawns. This concerns placement during transitions; it is not a save-slot loading API.

## Persistent activation and progression

- `SavableEntityBase.Id` or `guid` identifies persistent world objects.
- `ActivableEntity.activated` exposes activation state.
- `ActivableEntity.ActivateInternal` has a `silent` argument; restoration-related activation can be silent.
- `ProgressionComponent` exposes `HasProgressionKey`, `SetProgressionKey`, and `UnsetProgressionKey`.

Silent activation must not be interpreted as a new player collection. It also does not guarantee that no gameplay effect occurs. Avoid replaying rewards when observing restored object state.

A position jump or room change after load, death, or teleport is not necessarily a player-traversed room connection. Reset traversal observation baselines after these discontinuities.

## Scope of known APIs

The APIs above cover observing progression reads, checking readiness, finding restored components, and handling placement. They do not establish the complete save serializer, an exact global callback order, a method for loading an arbitrary save slot, or a method for forcing a disk save. Those entry points are not established here.


