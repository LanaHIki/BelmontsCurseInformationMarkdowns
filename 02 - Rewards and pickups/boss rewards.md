# Boss and cutscene rewards

[Documentation index](../README.md)

## Mechanics

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

## Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.

## Persistence

Collection, original benefit delivery, and replacement delivery are separate events. Keep the world's consumed/opened state while suppressing only the intended reward effect. A second observation of the same source must not automatically grant a second reward.

On save/load, reacquire runtime objects and distinguish restored activation from new interaction. Verify both the consumed state and actual inventory/stat/XP state through normal save/reload. Do not assume an immediate disk save after an interaction.

The catalog below contains available source identifiers, not a guarantee of complete coverage or identical content in every game build. Descriptive labels are organizational labels; use actual save IDs, scene context, and game item IDs for identification. Unknown fields remain blank rather than being inferred.

## Source catalog

| Source label | Room | Object | Save ID | Original item ID |
|---|---|---|---|---|
| Streets of Paris - The Fallen (Leon) Boss Loot 1 |  |  |  | BoomerangCross |
| Streets of Paris - The Fallen (Leon) Boss Loot 2 |  |  |  | CatacombsGateKeyLeon |
| Streets of Paris - The Fallen (Leon) Ability |  |  |  | WhipPendulum |
| Streets of Paris - The Warrior (Joan) Boss Loot 1 |  |  |  | FireSeismicWave |
| Streets of Paris - The Warrior (Joan) Boss Loot 2 |  |  |  | CatacombsGateKeyJeanne |
| Streets of Paris - The Warrior (Joan) Ability |  |  |  | VaniaLight |
| Innocents' Cemetery - The Enchantress (Medusa) Boss Loot 1 |  |  |  | PetrifyingRay |
| Innocents' Cemetery - The Enchantress (Medusa) Boss Loot 2 |  |  |  | CatacombsGateKeyMedusa |
| Main Hall - The Artist (Brauner) Boss Loot 1 |  |  |  | InvocationStone |
| Main Hall - The Artist (Brauner) Boss Loot 2 |  |  |  |  |
| Library - The Occultist (Carmilla) Boss Loot 1 |  |  |  | BloodSpikes |
| Library - The Occultist (Carmilla) Boss Loot 2 |  |  |  |  |
| Clock Tower - The Builder (Talos) Boss Loot 1 |  |  |  | GiantFist |
| Clock Tower - The Builder (Talos) Ability |  |  |  | LightCrash |
| Notre-Dame - The Shadow (Death) Boss Loot 1 |  |  |  | LaserScytheSlash |
| Notre-Dame - The Shadow (Death) Ability |  |  |  | DoubleJump |
| Catacombs - The Whisperer (Ferryman) Reward |  |  |  | FreezingMist |
| Notre-Dame - The Judge (Saint Louis) Reward | StreetRich_Boss_StLouis_uid114 |  |  | Detainment |
| Notre-Dame - The Self (Doppelganger) Reward | NotreDame_MVP_BossRoom_Doppelganger_uid144 |  |  | RageIncarnation |
| Clock Tower - Isaac Boss Loot | ClockTower_MVP_Room47_uid150 |  |  | RiseAndFall |
| Hall of Mirrors - Succubus Boss Loot | Gym_Boss_SuccubusAgnes_uid144 |  |  | LifeDrain |
| Catacombs - The Miser (Mammon) Boss Loot | MillersBridge_13_BossRoom_uid105 |  |  | TargetingChargingWheels |


