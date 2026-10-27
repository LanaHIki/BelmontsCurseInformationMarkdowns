# Item, ability, and stat granting

[Documentation index](../README.md)

## Game objects

- **Player movement component:** active `VaniaMovesComponent`.
- **Inventory:** `InventoryComponent.inventory`, located in the player's parents or root's children.
- **Item library:** `ItemsManager`.
- **Player entity:** `ExperienceComponent.linkedEntity`.
- **Stats:** `StatsStateComponent` on the player entity.

Loaded components can be found with `Resources.FindObjectsOfTypeAll(Il2CppType.Of<T>())`, then converted using `TryCast<T>()`. Select the active player in a valid scene. Perform mutations on Unity's main thread after the required objects exist. Avoid overlapping grants with an active acquisition sequence; `ItemAcquisitionGlobalController.IsAcquisitionSequenceFinished()` exposes its state.

## Weapons and Arcana

Both use inventory items:

```csharp
ItemData item = itemsManager.FindItem(gameItemId);
if (item != null && inventory != null)
    inventory.AddItemToInventory(item, false);
```

```text
Internal item ID → ItemsManager.FindItem → ItemData → Inventory.AddItemToInventory
```

| Arcana | Internal item ID |
|---|---|
| Blood Spikes | `BloodSpikes` |
| Holy Cross | `BoomerangCross` |
| Cursed Flame | `CursedFireBall` |
| Chains of Justice | `Detainment` |
| Infernal Pyre | `FireSeismicWave` |
| Freezing Path | `FreezingMist` |
| Fist of Talos | `GiantFist` |
| Stained-Glass Shield | `GlassArmor` |
| Masterpiece | `InvocationStone` |

Weapons use the same operation with their internal item IDs. Adding ownership, equipping an item, showing its acquisition screen, and completing Arcana tasks/talents are separate concerns. The call above does not explicitly perform those additional operations.

## Movement abilities

Resolve the ability item to its movement identifier:

```csharp
var item = itemsManager.FindItem(gameItemId);
var move = item?.TryCast<VaniaMoveItemData>()?.vaniaMove;
if (move != null && player != null)
{
    player.UnlockVaniaMove(move);
    // player.LockVaniaMove(move);
    bool unlocked = player.IsVaniaMoveUnlocked(move);
}
```

```text
Internal item ID → VaniaMoveItemData → VaniaMoveID → player unlock/lock
```

| Ability | Internal item ID |
|---|---|
| Arcane Whip | `WhipPendulum` |
| Holy Light | `VaniaLight` |
| Smash | `LightCrash` |
| Aerial Rush | `AerialDash` |
| Double Jump | `DoubleJump` |

Ability item assets can also be passed to `Inventory.AddItemToInventory`. Direct movement calls expose explicit enable/disable control. Unlocked movement abilities participate in game save state.

## Stats: fragments and books

Apply stat changes through the player entity:

```csharp
var parameters = new AffectStatParameters
{
    targetStat = new PowerAttackBonus(),
    addStatIfDoesNotExist = true,
    affectType = AffectStatType.IncrementOne,
    affectValue = 1f,
};
AffectStatFeature.AffectStat(playerEntity, parameters);
```

```text
Player entity → choose stat → construct AffectStatParameters → AffectStatFeature.AffectStat
```

| Grant | Stat class | Reaction |
|---|---|---|
| Attack Book | `PowerAttackBonus` | None |
| Defense Book | `PowerDefenseBonus` | None |
| Health Fragment | `HealthUpgradeFragmentTotalCount` | `AffectStatHealthUpgradeCountReaction` |
| Mana Fragment | `ManaUpgradeFragmentTotalCount` | `AffectStatManaUpgradeCountReaction` |

For health, set:

```csharp
parameters.targetStat = new HealthUpgradeFragmentTotalCount();
parameters.affectStatReaction = new AffectStatHealthUpgradeCountReaction(
    AffectStatHealthUpgradeCountReaction.HealthAdaptationType.KeepRatioIfNotComplete);
```

For mana, set:

```csharp
parameters.targetStat = new ManaUpgradeFragmentTotalCount();
parameters.affectStatReaction = new AffectStatManaUpgradeCountReaction(
    AffectStatManaUpgradeCountReaction.ManaAdaptationType.KeepRatioIfNotComplete);
```

Apply health and mana as separate operations. Their reactions handle fragment merging and resulting maximum HP/MP changes.

To inspect a stat:

```csharp
var stat = stats.GetStatObject(Il2CppType.Of<PowerAttackBonus>());
float value = stat.GetBaseValue();
```

`StatObject.SetFloat(value)` directly sets a value. For fragments, changing the counter alone is not equivalent to running the upgrade reaction. Reread the stat after applying a change before retrying: a later UI error can occur after a value has changed.


