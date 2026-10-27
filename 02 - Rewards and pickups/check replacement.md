# Reward sources and replacement mechanics

[Documentation index](../README.md)

## Basic structure

Game reward collection has separate parts:

```text
World interaction or scripted event
→ source activation / reward sequence
→ inventory addition, stat change, experience effect, or progression flag
→ persistent collected/progression state
```

A replacement system can observe the source, preserve its collected state, suppress the original benefit, and grant another reward. This is an interception strategy; replacement is not an automatic game feature.

## Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.

## Chests

Relevant game methods:

- `ItemGiver.GiveItem`: reward handoff.
- `ActivableEntity.ActivateInternal`: activation and used-state handling.
- `Inventory.AddItemToInventory`: inventory mutation.

```text
Chest opens
→ giver hands out its configured reward
→ inventory receives ItemData
→ chest remains opened/used in persistent state
```

Replacement strategy:

```text
Identify chest by save ID
→ remember collection
→ allow opening/used-state behavior
→ suppress original inventory addition
→ grant replacement separately
```

An inventory overload available for interception has these parameter types:

```csharp
Inventory.AddItemToInventory(
    ItemData,
    Il2CppSystem.Action<ItemData>,
    bool,
    bool)
```

Suppressing inventory addition does not necessarily suppress the acquisition screen. Its displayed reward and quick-equip behavior require separate handling:

- `ItemAcquisitionController.TryQuickEquip`
- `ItemAcquisitionController.CanQuickEquip`
- `BooleanProviderItemAcquisitionControllerCanQuickEquip.GetValue`

Preventing only inventory addition can leave another route to equipping the original displayed reward.

## Boss and cutscene rewards

Scripted rewards can reach `Inventory.AddItemToInventory` without a uniquely identified world pickup.

```text
Boss defeat / reward cutscene
→ original reward ItemData
→ itemToAdd.itemID
→ inventory addition
```

Original item IDs can help identify rewards. For example, Holy Cross uses `BoomerangCross`. An item ID identifies the reward asset, not necessarily a unique encounter or grant instance; combine it with encounter/scene context when needed.

A single boss can award multiple items or abilities through separate steps. Ability reward scenes may be skipped when that ability is already owned. Encounter completion and each individual reward-grant callback therefore need not be equivalent.

For replacement, account for all rewards belonging to the encounter and suppress each original benefit as appropriate. Grouping rewards is a replacement-system policy, not a built-in game identifier.

## Unique rewards, altars, and special chests

The actual giver can be a sibling of the persistent state machine controlling a reward. Consequently, the giver's nearest save ID may differ from the state machine's identifier.

Useful identification data:

1. Persistent save ID.
2. Scene plus specific object/hierarchy identity.
3. Configured original item ID and encounter context.

Objects named with the `LdBrick_StateMachine` prefix can participate in unique reward sequences. Spawned pickups may hand off their reward through inventory addition after the original interaction.

## Health/mana fragments and attack/defense books

These rewards change stats directly. An inventory-only interception does not cover them.

```text
ActivableEntity.ActivateInternal
→ pickup becomes activated/used
→ stat modification
→ fragment upgrade reaction where applicable
```

`ActivableEntity` exposes `activated`, and `ActivateInternal` has a `silent` argument. Silent activation can be associated with loading/restoring used objects, but it must not be assumed to prevent every stat effect.

| Pickup | Stat |
|---|---|
| Health Fragment | `HealthUpgradeFragmentTotalCount` |
| Mana Fragment | `ManaUpgradeFragmentTotalCount` |
| Attack Book | `PowerAttackBonus` |
| Defense Book | `PowerDefenseBonus` |

For fragment replacement, relevant methods are `BeforeStatModification` and `AfterStatModification` on:

- `AffectStatHealthUpgradeCountReaction`
- `AffectStatManaUpgradeCountReaction`

A possible suppression strategy is to capture the original counter, restore it with `StatObject.SetFloat`, and prevent the upgrade reaction from processing the unwanted increment. This preserves the pickup's consumed state while withholding the benefit. Guard this interception so it does not also undo deliberate replacement grants.

For books, the bonus counter can be restored to a known intended value with `SetFloat`. Do not rely on inventory suppression for either book type.

## EXP statues

Known statue object name: `GPE_DestructibleSourceExperience(Clone)`.

```text
Statue broken / activated
→ persistent destruction state
→ experience reward / orbs
```

The activation and save ID provide a collection-detection point. Detecting or recording destruction does not suppress experience. Replacing the XP itself requires identifying and intercepting its experience/orb-producing path; the exact suppression call is not established here.

## Clues

Clues can be represented as inventory items, such as `Clue_Leon_HiddenLever`.

```text
Interaction or cutscene
→ clue ItemData
→ Inventory.AddItemToInventory
```

An inventory interception can observe clue acquisition while allowing the clue to remain available. Whether a replacement should remove the clue is a separate design decision; observation alone does not remove it.

## Progression events

Relevant game API:

```csharp
progression.HasProgressionKey(key);
progression.SetProgressionKey(key);
progression.UnsetProgressionKey(key);
```

Keys are `ProgressionKeyID` assets. Some event paths may update progression without passing through the setter being observed, so polling state can be necessary.

### Eleanor character event

- Key: `MC_CharacterDataModificator_Eleanor`
- GUID: `6e518ea36b23065439b86276732c7471`
- This is an Eleanor character key, not the mirror-travel lock. Mirror travel is gated by the first Death key.

Observe the key setter to identify this character-state change. Do not interpret it as the mirror-travel unlock.

### First Shadow confrontation

- Key: `DeathCatacombs_Defeated_ID`
- GUID: `769b38ad62fa14249b54a74ef889998c`
- Location: Altar of Bones, `Catacombs_MVP_Room27_uid137`.
- Effects: opens the east Streets route to the castle and changes access/state around the confrontation.

Setting this key early can prevent the confrontation from starting. Its traversal and encounter effects must both be considered when changing it. A setter callback alone may not capture every change to this key.

## Replacement boundaries

| Source | Observation point | Benefit requiring separate handling |
|---|---|---|
| Chest | Giver/activation plus save ID | Inventory addition and acquisition UI |
| Boss/cutscene | Reward inventory addition and encounter context | Every item/ability awarded by the encounter |
| Unique reward / altar | Save ID, scene/object identity, reward handoff | Inventory reward or other scripted effect |
| Fragment | Activation plus save ID | Stat increment and upgrade reaction |
| Stat book | Activation plus save ID | Attack/defense bonus counter |
| EXP statue | Activation/destruction plus save ID | Experience/orb-producing effect |
| Clue | Clue inventory addition | Optional clue ownership change |
| Progression event | Key setter or state observation | Flag and all gameplay effects it controls |

Keep source completion, reward ownership, UI presentation, and progression effects separate. Recording an interaction does not automatically replace its benefit.

## Code examples by category

The examples below show game-facing interception points. They are illustrative fragments, not a complete plugin, and have not been compiled or runtime-tested here. Install each patch class once through `Harmony.CreateAndPatchAll(typeof(PatchClass), "your.unique.patch.id")`. Use `HarmonyLib`, `Il2CppInterop.Runtime`, and `UnityEngine` as required.

Application-owned callbacks such as `RecordCollection` and `QueueReplacement` are placeholders: implement them with persistent per-save deduplication and main-thread deferred granting. Avoid opening another reward screen inside an active acquisition sequence.

### Shared source lookup

```csharp
static string ReadSaveId(Component component)
{
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

This helper belongs in an application utility class accessible to the patch classes below. It resolves the nearest savable component, which may not identify the intended source in every hierarchy; validate each mapping.

### Chests: observe the giver

```csharp
[HarmonyPatch(typeof(ItemGiver), nameof(ItemGiver.GiveItem))]
static class ChestRewardObserver
{
    [HarmonyPrefix]
    static void BeforeGive(ItemGiver __instance)
    {
        string id = ReadSaveId(__instance);
        // Sewer Reservoir Chest
        if (id != "a9d3c2f1-add3-4cdd-9ad8-7095b3452349") return;

        RecordCollection(id);
        // Associate this exact giver/reward sequence with the inventory
        // interception below. Do not block unrelated inventory grants.
    }
}
```

The giver prefix observes an attempted reward handoff; it does not cancel opening or guarantee reward completion. For successful activation observation, capture prior `activated`/`silent` state and check the resulting state after activation, as shown for statues below.

### Inventory reward suppression: chests, bosses, and unique rewards

```csharp
[HarmonyPatch(typeof(Inventory), nameof(Inventory.AddItemToInventory),
    new[] { typeof(ItemData), typeof(Il2CppSystem.Action<ItemData>),
            typeof(bool), typeof(bool) })]
static class InventoryRewardGate
{
    [HarmonyPrefix]
    static bool BeforeAdd(ItemData itemToAdd, bool fromInit)
    {
        if (itemToAdd == null || fromInit || replacementGrantInProgress)
            return true;

        // Application-defined attribution: the item ID alone may not
        // distinguish a boss reward from another source of the same item.
        if (!TryResolvePendingSource(itemToAdd.itemID, out string sourceId))
            return true;

        RecordCollection(sourceId);
        QueueReplacement(sourceId);
        return false; // Skip this original inventory-add operation.
    }
}
```

`replacementGrantInProgress` and `TryResolvePendingSource` are application-owned state/logic. Tie attribution to the intended giver, encounter, scene, and item, and clear stale contexts. Blanket blocking by item ID can suppress legitimate grants elsewhere. Do not assume returning `false` invokes the original grant's completion callback; inspect sequence cleanup behavior.

When deliberately adding the replacement:

```csharp
replacementGrantInProgress = true;
try
{
    inventory.AddItemToInventory(replacementItem, false);
}
finally
{
    replacementGrantInProgress = false;
}
```

The bypass must be shared with the inventory interceptor. It prevents that interceptor from blocking its own replacement.

### Boss rewards: identify the reward item

```csharp
static void ObserveBossReward(ItemData item, string scene)
{
    if (item == null) return;

    // Holy Cross: one of Leon's rewards.
    if (item.itemID == "BoomerangCross"
        && IsExpectedLeonRewardContext(scene))
    {
        RecordCollection("leon-holy-cross");
        // Resolve the complete encounter's reward plan separately.
    }
}
```

Call this from the inventory observation/interception point. `IsExpectedLeonRewardContext` is a placeholder for encounter attribution. A scene name or item ID in isolation is not a universal encounter-completion signal. To suppress the original reward, return `false` through the inventory gate for the attributed grant.

An encounter can grant multiple items/abilities, and an already-owned ability can prevent its reward cutscene from playing. Group completion only when encounter evidence establishes that the relevant rewards should count; do not infer a universal boss-death event from one inventory call.

### Unique rewards and altars: inspect source and hierarchy

```csharp
static void ObserveUniqueGiver(ItemGiver giver)
{
    if (giver == null) return;
    string id = ReadSaveId(giver);
    string scene = giver.gameObject.scene.name;
    string objectName = giver.gameObject.name;

    if (TryResolveUniqueSource(id, scene, objectName, out string sourceId))
        RecordCollection(sourceId);
}
```

`TryResolveUniqueSource` must use validated source mappings. Inspect parent/sibling state machines when the nearest savable ID belongs to another object. Do not assign a reward merely because any `LdBrick_StateMachine` is present in the room. Spawned altar pickups may need attribution carried from the interaction to the eventual inventory grant. Use the inventory gate for original-item suppression.

### Health and mana fragments: observe activation

```csharp
[HarmonyPatch(typeof(ActivableEntity), "ActivateInternal")]
static class FragmentObserver
{
    [HarmonyPrefix]
    static void BeforeActivate(ActivableEntity __instance, ref bool silent)
    {
        if (__instance == null || silent || __instance.activated) return;
        string id = ReadSaveId(__instance);

        bool health = id == "417d3b09-286b-4b4b-93ea-0330065ca0b9";
        bool mana = id == "68be2bca-0caa-48a4-ab69-f1e44cdb3a76";
        if (!health && !mana) return;

        RecordCollection(id);
        BeginExpectedFragmentChange(id, health); // Application-owned scope.
        silent = true;
        // Original activation continues. Silent alone is insufficient
        // to guarantee that the fragment's stat effect is suppressed.
    }
}
```

### Health and mana fragments: intercept the stat reaction

The following schematic shares logic between the two reaction classes. The guard must identify the specific pending vanilla pickup and exclude deliberate replacement grants. The snapshot store is application-owned; scope it per expected operation/stat to avoid nested changes overwriting it.

```csharp
[HarmonyPatch]
static class FragmentReactionGate
{
    [HarmonyPrefix]
    [HarmonyPatch(typeof(AffectStatHealthUpgradeCountReaction),
        nameof(AffectStatHealthUpgradeCountReaction.BeforeStatModification))]
    static void BeforeHealth(StatsStateComponent.StatObject targetStatObject)
    {
        if (ShouldSuppressExpectedFragment(true) && targetStatObject != null)
            CaptureFragmentValue(true, targetStatObject.GetBaseValue());
    }

    [HarmonyPrefix]
    [HarmonyPatch(typeof(AffectStatHealthUpgradeCountReaction),
        nameof(AffectStatHealthUpgradeCountReaction.AfterStatModification))]
    static bool AfterHealth(StatsStateComponent.StatObject targetStatObject)
        => RestoreExpectedFragment(true, targetStatObject);

    [HarmonyPrefix]
    [HarmonyPatch(typeof(AffectStatManaUpgradeCountReaction),
        nameof(AffectStatManaUpgradeCountReaction.BeforeStatModification))]
    static void BeforeMana(StatsStateComponent.StatObject targetStatObject)
    {
        if (ShouldSuppressExpectedFragment(false) && targetStatObject != null)
            CaptureFragmentValue(false, targetStatObject.GetBaseValue());
    }

    [HarmonyPrefix]
    [HarmonyPatch(typeof(AffectStatManaUpgradeCountReaction),
        nameof(AffectStatManaUpgradeCountReaction.AfterStatModification))]
    static bool AfterMana(StatsStateComponent.StatObject targetStatObject)
        => RestoreExpectedFragment(false, targetStatObject);

    static bool RestoreExpectedFragment(
        bool health, StatsStateComponent.StatObject stat)
    {
        if (stat == null || !ShouldSuppressExpectedFragment(health)
            || !TryConsumeFragmentSnapshot(health, out float original))
            return true;

        stat.SetFloat(original);
        return false; // Skip the original after-modification reaction.
    }
}
```

`ShouldSuppressExpectedFragment`, `CaptureFragmentValue`, and `TryConsumeFragmentSnapshot` are placeholders, not game APIs. Restore the counter before suppressing the reaction and verify maximum HP/MP as well as the fragment value. Always expire or clear the pending scope after completion, failure, or load. Withholding the original effect and granting another fragment are separate operations.

### Attack and defense books: observe and correct bonus counters

```csharp
static bool IsKnownBook(ActivableEntity pickup, out bool attack)
{
    string id = ReadSaveId(pickup);
    attack = id == "97135d6a-f214-4bae-9cc0-fe25412fb7b9";
    bool defense = id == "c35959c9-e6ac-40f9-a5ad-e64b0e73703d";
    return attack || defense;
}

static void RestoreBookBonus(
    StatsStateComponent stats, bool attack, float intendedValue)
{
    var type = attack
        ? Il2CppType.Of<PowerAttackBonus>()
        : Il2CppType.Of<PowerDefenseBonus>();
    var stat = stats.GetStatObject(type);
    if (stat != null)
        stat.SetFloat(intendedValue);
}
```

Invoke the identification from a non-silent, previously unused activation. Capture the correct intended bonus before the unwanted effect and restore after that effect has actually executed. An activation postfix is sufficient only if the effect is synchronous; verify timing. Preserve legitimate concurrent bonuses and avoid repeated subtraction. Neither inventory suppression nor `silent = true` alone proves the bonus was withheld.

### EXP statues: detect a new activation

```csharp
[HarmonyPatch(typeof(ActivableEntity), "ActivateInternal")]
static class StatueObserver
{
    [HarmonyPrefix]
    static void Before(ActivableEntity __instance, bool silent,
        out string __state)
    {
        __state = null;
        if (__instance == null || silent || __instance.activated) return;
        string id = ReadSaveId(__instance);
        if (id == "53987f65-dfec-4448-9df5-79b483a5cca7")
            __state = id; // Prison Tower EXP Statue 1
    }

    [HarmonyPostfix]
    static void After(ActivableEntity __instance, string __state)
    {
        if (__state == null || __instance == null || !__instance.activated)
            return;
        RecordCollection(__state);
        QueueReplacement(__state);
    }
}
```

This assumes activation is observable before the method returns; use a later state check if activation is deferred. It detects the source while leaving normal XP/orbs intact. No established XP-suppression API is supplied here. A bonus reward is not full replacement of the statue's experience.

### Clues: observe while preserving ownership

```csharp
[HarmonyPatch(typeof(Inventory), nameof(Inventory.AddItemToInventory),
    new[] { typeof(ItemData), typeof(Il2CppSystem.Action<ItemData>),
            typeof(bool), typeof(bool) })]
static class ClueObserver
{
    [HarmonyPrefix]
    static void BeforeAdd(ItemData itemToAdd, bool fromInit)
    {
        if (fromInit || itemToAdd == null) return;
        if (itemToAdd.itemID != "Clue_Leon_HiddenLever") return;

        RecordCollection(itemToAdd.itemID);
        // Void prefix: original inventory addition is allowed to run.
    }
}
```

This observes an attempted clue grant; confirm resulting ownership if successful acquisition is required. Deduplicate replayed grants and coordinate this observer with any inventory gate so clues are not accidentally blocked. Other clues can be recognized by their exact IDs from [clues.md](clues.md).

### Progression events: intercept an Eleanor character key

```csharp
[HarmonyPatch(typeof(ProgressionComponent),
    nameof(ProgressionComponent.SetProgressionKey))]
static class SageUnlockGate
{
    [HarmonyPrefix]
    static bool BeforeSet(ProgressionKeyID progressionKey)
    {
        if (progressionKey == null || deliberateKeyChange) return true;
        if (progressionKey.guid != "6e518ea36b23065439b86276732c7471")
            return true;

        RecordCollection("eleanor-character-event");
        return allowEleanorKey;
    }
}
```

`deliberateKeyChange` and `allowEleanorKey` are application policy state. This example intercepts the Eleanor key; it does not implement the mirror-travel lock. Returning `false` withholds the Eleanor key while allowing it separately later through `SetProgressionKey`. Guard deliberate changes against recursive observation. This hook sees attempts to set the key, not proof the player has just met Eleanor for the first time.

### Progression events: observe the Shadow flag by polling

```csharp
static void PollShadowFlag(ProgressionComponent progression,
    ProgressionKeyID shadowKey)
{
    if (progression == null || shadowKey == null) return;
    if (shadowKey.guid != "769b38ad62fa14249b54a74ef889998c") return;

    bool present = progression.HasProgressionKey(shadowKey);
    ObserveFlagSnapshot(shadowKey.guid, present);
}
```

`ObserveFlagSnapshot` is application-owned state tracking. On initialization/load, establish the baseline without treating restored presence as a newly completed fight. Polling can detect changes that bypass the observed setter, but flag changes alone do not identify why the change happened. Pair with room/encounter context.

If deliberately withholding this key, `UnsetProgressionKey(shadowKey)` removes it. Its route-opening and encounter-suppression effects make removal or early assignment consequential; consult [progression flags.md](../03%20-%20Progression%20and%20cutscenes/progression%20flags.md) before changing it.



