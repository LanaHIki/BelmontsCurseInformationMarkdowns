# Experience granting

[Documentation index](../README.md)

## Game API

Player experience is handled by `ExperienceComponent`.

| Member | Purpose |
|---|---|
| `ExperienceComponent.linkedEntity` | Player entity associated with the component |
| `GetCurrentLevelMaxExperience()` | Returns the current level's maximum-experience requirement value |
| `IncrementCurrentExperience(entity, amount)` | Applies an experience increment through the game system |

```text
Active player
→ ExperienceComponent
→ validate linkedEntity
→ choose XP amount
→ IncrementCurrentExperience
```

Experience is separate from inventory items, movement unlocks, fragment counters, and attack/defense book bonuses. Use the experience API rather than adding an inventory item and assuming it changes XP.

## Finding the component

```csharp
using Il2CppInterop.Runtime;
using UnityEngine;

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

static ExperienceComponent FindPlayerExperience()
{
    var player = FindPlayer();
    if (player == null) return null;

    var component = player.GetComponentInParent(
            Il2CppType.Of<ExperienceComponent>(), true)
        ?? player.transform.root.GetComponentInChildren(
            Il2CppType.Of<ExperienceComponent>(), true);
    return component?.TryCast<ExperienceComponent>();
}
```

Validate that the result belongs to the intended playable character. Reacquire cached objects after transitions or other lifecycle changes. A missing component means XP state is unavailable; it does not mean the player has zero XP.

## Fixed XP grant

```csharp
static bool GiveFixedExperience(int amount)
{
    if (amount <= 0) return false;
    var experience = FindPlayerExperience();
    if (experience == null || experience.linkedEntity == null)
        return false;

    experience.IncrementCurrentExperience(experience.linkedEntity, amount);
    return true;
}

// Example, called from an appropriate main-thread gameplay context:
// bool submitted = GiveFixedExperience(100);
```

The return value is example application status: it says the call returned, not that a particular final level or XP value was independently verified. The method itself is application code; `IncrementCurrentExperience` is the game API.

These snippets are illustrative and have not been compiled or runtime-tested here.

## Level-scaled XP grant

A reward amount can be calculated from the current level requirement:

```csharp
using System;

static int GiveLevelScaledExperience()
{
    var experience = FindPlayerExperience();
    if (experience == null || experience.linkedEntity == null)
        return 0;

    int levelRequirement = experience.GetCurrentLevelMaxExperience();
    int amount = Math.Max(10, levelRequirement / 4);
    experience.IncrementCurrentExperience(experience.linkedEntity, amount);
    return amount;
}
```

The calculation selects one quarter of the returned requirement, with integer division and a minimum of 10 XP. This is an example reward policy, not a built-in XP-item value or native reward formula.

It uses the level's requirement value, not the player's remaining XP to the next level. It does not guarantee exactly one quarter of a visible progress bar, especially near a level boundary. Recalculate for each grant if the policy should follow the player's current level.

## Timing and execution

Perform calls on Unity's main thread after the player, experience component, and linked entity are ready. Before a queued grant executes, revalidate them.

`GameStateManager.IsGameplayState()` can help avoid loading or other busy states. Inventory acquisition readiness can be checked separately through `ItemAcquisitionGlobalController.IsAcquisitionSequenceFinished()` if the surrounding feature also uses item UI. An XP increment does not require adding an inventory item or explicitly opening an acquisition screen.

An exception does not prove that no XP was added: later callbacks or presentation can fail after a mutation. Do not blindly retry an uncertain grant. Verify actual gameplay state before deciding whether to repeat it.

## Level-up behavior

Passing XP through `IncrementCurrentExperience` uses the game's experience-increment path, allowing its normal level processing rather than directly assigning a guessed XP field.

The exact level-up callbacks, excess-XP carry behavior, level cap, current-XP getter, and current-level getter are not established here. Do not invent their method names or assume that an arbitrarily large grant is equivalent to repeated ordinary rewards.

If a test needs exact verification, first identify the actual current-XP/level fields or getters, then compare before and after values and UI behavior.

## XP versus other stats

`PowerAttackBonus` and `PowerDefenseBonus` are book-related bonus counters; they are not interchangeable with experience. `HealthUpgradeFragmentTotalCount` and `ManaUpgradeFragmentTotalCount` belong to fragment upgrades.

Granting XP through the experience API does not explicitly add books or fragments. Any level-derived stat changes are managed by the game's own level-processing logic; the complete stat-growth formula is not established here.

## Source rewards and replacement

An XP grant is separate from detecting an XP source. EXP statue activation can be observed, but its exact XP/orb suppression path is not established in this document.

```text
Observe statue activation
+ manually grant XP
≠ replacement of the original statue XP
```

If the original orbs still award XP, a manual grant adds to that reward. Likewise, a successful experience call is not a source-completion record. Deduplicate source events separately if they should award XP only once.



