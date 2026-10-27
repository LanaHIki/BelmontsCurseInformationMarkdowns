# Runtime inspection

[Documentation index](../README.md)

## Scope

Runtime inspection reads the currently loaded Unity objects and player state. It can reveal room identity, object hierarchy, persistent IDs, equipment, abilities, stats, progression, and active game state without changing them.

Loaded-object searches are not a complete world database: unloaded rooms and assets may be absent. Names and proximity are discovery clues, not proof of an object's function or reachability.

The snippets below illustrate game-facing C# calls through Unity IL2CPP interop. They are not a complete plugin and have not been compiled or runtime-tested here. Relevant namespaces include `UnityEngine`, `Il2CppInterop.Runtime`, `MotherBase.EntityFramework`, and `MotherBase.RoomDesigner`, with `System` and collection/LINQ namespaces where used.

## Finding the player

```csharp
static VaniaMovesComponent FindPlayer()
{
    foreach (var obj in Resources.FindObjectsOfTypeAll(
                 Il2CppType.Of<VaniaMovesComponent>()))
    {
        var player = obj.TryCast<VaniaMovesComponent>();
        if (player != null && player.gameObject.scene.IsValid()
            && player.gameObject.activeInHierarchy)
            return player;
    }
    return null;
}
```

Validate that the selected component belongs to the intended playable character if multiple active candidates exist. Revalidate cached references after death, save loads, and transitions. An IL2CPP/Unity object may have been destroyed even if a managed wrapper remains accessible.

## Position and scene

```csharp
var player = FindPlayer();
if (player != null)
{
    Vector3 position = player.transform.position;
    string scene = player.gameObject.scene.name;
    Debug.Log($"Player: {position}; object scene: {scene}");
}
```

Position is world-space. The player's owning scene is not necessarily the authoritative current room, especially if player objects persist across scene loads. Use room state for room attribution.

## Current room metadata

```csharp
foreach (var obj in Resources.FindObjectsOfTypeAll(
             Il2CppType.Of<CurrentRoomInstance>()))
{
    var instance = obj.TryCast<CurrentRoomInstance>();
    var room = instance?.currentRoom;
    if (room == null) continue;

    string scene = room.gameObject.scene.name;
    var data = room.worldRoomData;
    Debug.Log($"Room scene: {scene}");
    if (data != null)
        Debug.Log($"World room: {data.instanceName}; ID: {data.id}");
}
```

Do not assume the first result is authoritative when several instances are loaded. Correlate with player position and active game state. A missing room can be transient during loading.

## Player inventory and entity

```csharp
var player = FindPlayer();
if (player == null) return;

var inventoryComponent = player.GetComponentInParent(
        Il2CppType.Of<InventoryComponent>(), true)
    ?? player.transform.root.GetComponentInChildren(
        Il2CppType.Of<InventoryComponent>(), true);
var inventory = inventoryComponent?.TryCast<InventoryComponent>()?.inventory;

var experienceComponent = player.GetComponentInParent(
        Il2CppType.Of<ExperienceComponent>(), true)
    ?? player.transform.root.GetComponentInChildren(
        Il2CppType.Of<ExperienceComponent>(), true);
var experience = experienceComponent?.TryCast<ExperienceComponent>();
var entity = experience?.linkedEntity;
```

This is a method-body fragment. A missing component means its state cannot currently be inspected; do not interpret it as an empty inventory or zero stats.

## Equipped weapon and Arcana

```csharp
if (inventory != null)
{
    string weaponId = inventory.GetEquippedWeapon()?.itemID;
    string arcanaId = inventory.GetEquippedSpell()?.itemID;
    Debug.Log($"Weapon: {weaponId ?? "none"}; Arcana: {arcanaId ?? "none"}");
}
```

These are internal item IDs, not localized display names. An equipped item and an item owned in the inventory are different concepts.

To check ownership of a known item:

```csharp
var item = itemsManager.FindItem("GiantFist");
if (item != null && inventory != null)
    Debug.Log(inventory.HasItemInInventoryOrEquipped(item));
```

`itemsManager` is a resolved `ItemsManager`, discoverable through a loaded-object search. No full inventory-enumeration API is established here.

## Movement abilities

```csharp
var item = itemsManager.FindItem("DoubleJump");
var move = item?.TryCast<VaniaMoveItemData>()?.vaniaMove;
if (player != null && move != null)
    Debug.Log($"Double Jump unlocked: {player.IsVaniaMoveUnlocked(move)}");
```

| Ability | Item ID |
|---|---|
| Arcane Whip | `WhipPendulum` |
| Holy Light | `VaniaLight` |
| Smash | `LightCrash` |
| Aerial Rush | `AerialDash` |
| Double Jump | `DoubleJump` |

Missing item data is an unknown result, not an explicitly locked ability. Reading movement unlock state is more direct than inferring it from a reward event.

## Stats and progression

```csharp
var stats = entity?.GetComponentData(Il2CppType.Of<StatsStateComponent>())
    ?.TryCast<StatsStateComponent>()
    ?? entity?.GetComponent(Il2CppType.Of<StatsStateComponent>())
    ?.TryCast<StatsStateComponent>();

var attack = stats?.GetStatObject(Il2CppType.Of<PowerAttackBonus>());
if (attack != null)
    Debug.Log($"Attack bonus counter: {attack.GetBaseValue()}");
```

`PowerAttackBonus` is a bonus counter, not necessarily the character's final calculated attack. Choose the exact stat class appropriate to the question. Other known counters include `PowerDefenseBonus`, `HealthUpgradeFragmentTotalCount`, and `ManaUpgradeFragmentTotalCount`.

Resolve `ProgressionComponent` from the entity in the same manner and enumerate `progressionKeys`:

```csharp
if (progression?.progressionKeys != null)
{
    foreach (var key in progression.progressionKeys)
        if (key != null)
            Debug.Log($"Progression: {key.name}; {key.guid}");
}
```

Character progression steps require `CharacterProgressionComponent` and `CharacterID` rather than a boolean key. These systems are covered in [progression flags.md](../03%20-%20Progression%20and%20cutscenes/progression%20flags.md).

## Nearby object scan

```csharp
static void InspectNearby(VaniaMovesComponent player, float radius = 15f)
{
    if (player == null) return;
    var origin = player.transform.position;
    var results = new List<(float Distance, GameObject Object)>();

    foreach (var obj in Resources.FindObjectsOfTypeAll(Il2CppType.Of<GameObject>()))
    {
        var go = obj.TryCast<GameObject>();
        if (go == null || !go.scene.IsValid() || !go.activeInHierarchy)
            continue;

        float distance = Vector3.Distance(origin, go.transform.position);
        if (distance <= radius)
            results.Add((distance, go));
    }

    foreach (var result in results.OrderBy(x => x.Distance).Take(25))
    {
        var go = result.Object;
        Debug.Log($"{result.Distance:F1}: {go.name}; "
            + $"scene={go.scene.name}; path={HierarchyPath(go.transform)}");
    }
}
```

The radius and output limit are example settings, not game constants. Use `Vector2.Distance` when intentionally ignoring Z. Spatial closeness does not prove connectivity; nearby objects can be in overlapping loaded rooms, behind walls, or at prefab-root positions unlike their visible geometry.

Optional discovery filters include `GPE_`, `BreakableTile`, and `SecretZone`. Filtering names can hide relevant objects; start broad and narrow deliberately. To inspect restored/disabled pickups, optionally include inactive objects while still rejecting non-scene assets.

## Hierarchy paths

```csharp
static string HierarchyPath(Transform transform)
{
    var names = new List<string>();
    for (var current = transform; current != null; current = current.parent)
        names.Add(current.name);
    names.Reverse();
    return string.Join("/", names);
}
```

A path helps identify prefab roots, giver children, cameras, triggers, and sibling state machines. It is diagnostic context rather than a guaranteed permanent identifier. Runtime clones, repeated names, and hierarchy changes can make paths ambiguous.

## Persistent save IDs

```csharp
static string SaveIdOf(Component component)
{
    if (component == null) return null;
    var savable = component.GetComponentInParent(
            Il2CppType.Of<SavableEntityBase>(), true)
        ?.TryCast<SavableEntityBase>()
        ?? component.GetComponentInChildren(
            Il2CppType.Of<SavableEntityBase>(), true)
        ?.TryCast<SavableEntityBase>();
    if (savable == null) return null;
    return string.IsNullOrEmpty(savable.Id) ? savable.guid : savable.Id;
}
```

This searches parents before children; the returned object can belong to a container or a different nested source. Inspect the hierarchy to confirm attribution. A missing save ID does not mean the object has no gameplay state; scripted rewards may be represented through items or progression instead.

## Activation state

For an `ActivableEntity`, inspect `activated`. Observing `ActivateInternal` can also expose the `silent` argument. Distinguish new player interaction from restoration/initialization and repeated activation attempts.

The used-state flag, reward delivery, and stat or inventory effects are separate observations. Do not infer successful reward delivery solely from activation.

## Game state and acquisition readiness

```csharp
var manager = GameStateManager.GetInstance();
if (manager != null)
    Debug.Log($"State: {manager.GetActiveStateId()}; "
        + $"gameplay: {manager.IsGameplayState()}");

var acquisition = ItemAcquisitionGlobalController.GetInstance();
if (acquisition != null)
    Debug.Log($"Acquisition finished: {acquisition.IsAcquisitionSequenceFinished()}");
```

Prefer semantic queries over hardcoded numeric state meanings. A missing manager is unknown state, not proof of readiness. A game can have a valid player object while loading, dialogue, a map, or another sequence still owns gameplay flow.

## Room completion diagnostics

For a validated current `WorldRoomData` and player entity:

```csharp
foreach (var obj in Resources.FindObjectsOfTypeAll(Il2CppType.Of<CompletionItemType>()))
{
    var category = obj.TryCast<CompletionItemType>();
    if (category == null) continue;
    int current = category.GetRoomCurrentCompletionCount(worldRoom, entity);
    int total = category.GetRoomTotalCompletionCount(worldRoom, entity);
    Debug.Log($"{category.name}: {current}/{total}");
}
```

Evaluate categories independently. Composite/district counters can overlap item-level counters; do not add all reported totals blindly. Some categories can be unavailable or throw for a particular runtime context; isolate errors per category in a diagnostic tool.

## Event attribution and snapshots

Useful observation points include:

- `ItemGiver.GiveItem`: giver handoff attempt.
- `Inventory.AddItemToInventory`: inventory grant attempt.
- `ActivableEntity.ActivateInternal`: activation.
- `ProgressionComponent.SetProgressionKey`: key-setting path.
- `CutscenePlayer.Begin`: cutscene start.
- `ProgressionComponent.ReadPlayerSave`: progression restore.

A nearest pickup is only a candidate cause of an inventory grant. A recent cutscene or giver can be another candidate. Capture explicit source context when available and record uncertain attribution as uncertain.

A useful snapshot records timestamp, scene, room ID, player X/Y/Z, object name/path/save ID, internal item ID, equipment, ability unlocks, game state, and whether data was unavailable. Reset room-walk baselines after loads, deaths, and teleports so position discontinuities are not recorded as ordinary exits.

## Performance and reliability

- Run Unity inspection on the main thread.
- Trigger broad object scans on demand or at a bounded interval rather than every frame.
- Prefer targeted component searches and cached asset metadata; invalidate runtime references on lifecycle changes.
- Limit and sort diagnostic output.
- Use invariant numeric formatting and correct escaping for machine-readable logs.
- Handle destroyed objects and unavailable components per operation, without turning errors into false facts.
- Separate read-only snapshots from any mutation or replacement logic.

A runtime snapshot describes the loaded state at that moment. It does not establish the entire map, every possible event path, or persistence on disk.

