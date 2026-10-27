# Books and fragments

[Documentation index](../README.md)

Health and mana fragments affect upgrade counters and their reactions. Attack and defense books affect bonus-stat counters. The sections below cover each type and its available pickup identifiers.

## Health fragments

### Game representation

- Pickup object name: `GPE_UpgradeCollectibleHealth`.
- Stat class: `HealthUpgradeFragmentTotalCount`.
- Persistent source identity: `SavableEntityBase.Id` or `guid`.

These pickups change a stat directly. Inventory-only observation or suppression does not capture their full effect.

```text
ActivableEntity.ActivateInternal
→ pickup activates and becomes used
→ stat changes
→ upgrade reaction runs where applicable
```

### Detecting collection

Observe `ActivableEntity.ActivateInternal` and identify the pickup through its save ID. Inspect `activated` and the `silent` argument to distinguish normal player interaction from restored/previously used objects. Silent activation alone must not be assumed to suppress every stat effect.

### Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.


### Reading and granting the stat

Find the active player's `ExperienceComponent.linkedEntity`, then its `StatsStateComponent`. Reacquire these after a load. Read the stat with:

```csharp
var stat = stats.GetStatObject(Il2CppType.Of<HealthUpgradeFragmentTotalCount>());
float current = stat.GetBaseValue();
```

To apply one increment:

```csharp
var parameters = new AffectStatParameters
{
    targetStat = new HealthUpgradeFragmentTotalCount(),
    addStatIfDoesNotExist = true,
    affectType = AffectStatType.IncrementOne,
    affectValue = 1f,
};
parameters.affectStatReaction = new AffectStatHealthUpgradeCountReaction(
    AffectStatHealthUpgradeCountReaction.HealthAdaptationType.KeepRatioIfNotComplete);
AffectStatFeature.AffectStat(playerEntity, parameters);
```

Perform changes on Unity's main thread after the player and stats exist. Reread the value before retrying an operation that threw; a later reaction/UI failure can occur after the stat changed.

### Fragment reaction and replacement

`AffectStatHealthUpgradeCountReaction` handles the upgrade reaction associated with the fragment counter, including merging fragments into maximum-stat upgrades. A direct counter assignment alone does not reproduce that full behavior.

Relevant observation points are `BeforeStatModification` and `AfterStatModification` on this reaction class.

A possible replacement strategy is to capture the original counter, allow the pickup to become used, restore the counter with `StatObject.SetFloat`, and suppress the unwanted after-modification reaction. Scope interception to the actual pickup so deliberate grants are not reverted. Verify the resulting maximum stat as well as the fragment counter.
### Persistence

Collection, original benefit delivery, and replacement delivery are separate events. Keep the world's consumed/opened state while suppressing only the intended reward effect. A second observation of the same source must not automatically grant a second reward.

On save/load, reacquire runtime objects and distinguish restored activation from new interaction. Verify both the consumed state and actual inventory/stat/XP state through normal save/reload. Do not assume an immediate disk save after an interaction.

The catalog below contains available source identifiers, not a guarantee of complete coverage or identical content in every game build. Descriptive labels are organizational labels; use actual save IDs, scene context, and game item IDs for identification. Unknown fields remain blank rather than being inferred.

### Source catalog

| Source label | Room | Object | Save ID | Original item ID |
|---|---|---|---|---|
| Streets of Paris - Prison Tower Health Fragment | StreetsPoorV2_Room01_uid82 | GPE_UpgradeCollectibleHealth | 417d3b09-286b-4b4b-93ea-0330065ca0b9 |  |
| Streets of Paris - Court of Justice Health Fragment | StreetsPoorV2_Room34_uid127 | GPE_UpgradeCollectibleHealth | 74d46fed-5392-4998-b785-cec733806181 |  |
| Streets of Paris - Undercity Health Fragment | StreetsPoorV2_Room40_uid133 | GPE_UpgradeCollectibleHealth | 6badac03-879a-47bc-9c77-f86742d2700a |  |
| Innocents' Cemetery - Ossuary of the Innocents Health Fragment | Cemetery2_Room20_uid108 | GPE_UpgradeCollectibleHealth | 8b74a0e5-d9f3-46f7-b9a7-bbb7788644f4 |  |
| Innocents' Cemetery - Ancient Sewers Health Fragment | Cemetery2_Room38_uid128 | GPE_UpgradeCollectibleHealth | 35ee7e9f-6b7b-4b6c-8ecd-2a08c7698e0c |  |
| Innocents' Cemetery - Mausoleum of the Kings Health Fragment | Cemetery_MVP_Room_090_uid140 | GPE_UpgradeCollectibleHealth | ad6dbad9-707a-411c-aef1-91fd7491854e |  |
| Catacombs - Blood Pits Health Fragment | Catacombs_MVP_Room05_uid113 | GPE_UpgradeCollectibleHealth | 6b1a1926-954d-4741-93f1-5acde8647dac |  |
| Catacombs - Catacomb Entrance Health Fragment | Catacombs_MVP_Room20_uid129 | GPE_UpgradeCollectibleHealth | c14acf24-831d-48c5-9699-65c7dff659ec |  |
| Catacombs - Circle of Greed Health Fragment | Catacombs_MVP_Room37_uid147 | GPE_UpgradeCollectibleHealth | 497e7a2e-e00c-411b-a0c5-35b7eddc55ea |  |
| Notre-Dame - Tree of Saints Health Fragment | NotreDame_MVP_Room_030_uid91 | GPE_UpgradeCollectibleHealth | 595df51f-9203-4f3d-b1b0-88d5d7bddd44 |  |
| Notre-Dame - La Flèche Health Fragment | NotreDame_MVP_Room_050_uid93 | GPE_UpgradeCollectibleHealth | bedec489-66bc-4977-a46b-2a2952f69f28 |  |
| Notre-Dame - The Forest Health Fragment | NotreDame_MVP_Room_255_uid106 | GPE_UpgradeCollectibleHealth | f0976546-b509-435f-91db-8920983d6e23 |  |
| Main Hall - Armoury Health Fragment | MainHall_MVP_Room_041_uid238 | GPE_UpgradeCollectibleHealth | b50f6d00-82d4-4fe4-a0a3-0bb05f58950e |  |
| Main Hall - Mirror Rooms Health Fragment | MainHall_MVP_Room_18_uid263 | GPE_UpgradeCollectibleHealth | 21ca0ab9-ccc7-401b-a5d8-933ea0122c4c |  |
| Main Hall - Balcony Health Fragment | MainHall_MVP_Room_23_uid271 | GPE_UpgradeCollectibleHealth | 79fd4332-26d1-47ba-a564-107ef1274022 |  |
| Library - Poetry Shelves Health Fragment | Library_MVP_Room_02_uid95 | GPE_UpgradeCollectibleHealth | f04f51b1-2008-4bca-81ff-b22239046666 |  |
| Library - Planetarium Health Fragment | Library_MVP_Room_06_uid99 | GPE_UpgradeCollectibleHealth | 61950d86-4da8-4ded-a7b4-f3e197554ec4 |  |
| Library - Library Foyer Health Fragment | Library_MVP_Room_09_uid102 | GPE_UpgradeCollectibleHealth | b8967e4b-8d9f-4387-870f-264c04a1ce24 |  |
| Clock Tower - Eastern Steps Health Fragment | ClockTower_MVP_Room07_uid105 | GPE_UpgradeCollectibleHealth | 6b662085-d63b-4efd-a8fd-2be39d573ad9 |  |
| Clock Tower - The Gauntlet Health Fragment | ClockTower_MVP_Room38_uid138 | GPE_UpgradeCollectibleHealth | bdbb6a93-2619-4748-9119-6b8d762288df |  |
| Clock Tower - Serpent Pinion Health Fragment | ClockTower_MVP_Room67_uid171 | GPE_UpgradeCollectibleHealth | 3cff1de9-a6de-43b7-8112-8e18ff5ced90 |  |
| Hall of Mirrors - Steps of Misdirection Health Fragment | HallOfMirrors_MVP_Room_04_uid102 | GPE_UpgradeCollectibleHealth | afb0e99a-4d53-46be-ac4b-114518b4d4d4 |  |
| Hall of Mirrors - Looping Hallways Health Fragment | HallOfMirrors_MVP_Room_24_uid123 | GPE_UpgradeCollectibleHealth | 930f2d5f-e57d-4ee2-9f77-ed2a029a88a1 |  |
| Hall of Mirrors - Nightmare Atrium Health Fragment | HallOfMirrors_MVP_Room_34_uid134 | GPE_UpgradeCollectibleHealth | a82b02ce-2968-46b0-b93e-b1f1aec57aaa |  |

## Mana fragments

### Game representation

- Pickup object name: `GPE_UpgradeCollectibleMana`.
- Stat class: `ManaUpgradeFragmentTotalCount`.
- Persistent source identity: `SavableEntityBase.Id` or `guid`.

These pickups change a stat directly. Inventory-only observation or suppression does not capture their full effect.

```text
ActivableEntity.ActivateInternal
→ pickup activates and becomes used
→ stat changes
→ upgrade reaction runs where applicable
```

### Detecting collection

Observe `ActivableEntity.ActivateInternal` and identify the pickup through its save ID. Inspect `activated` and the `silent` argument to distinguish normal player interaction from restored/previously used objects. Silent activation alone must not be assumed to suppress every stat effect.

### Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.


### Reading and granting the stat

Find the active player's `ExperienceComponent.linkedEntity`, then its `StatsStateComponent`. Reacquire these after a load. Read the stat with:

```csharp
var stat = stats.GetStatObject(Il2CppType.Of<ManaUpgradeFragmentTotalCount>());
float current = stat.GetBaseValue();
```

To apply one increment:

```csharp
var parameters = new AffectStatParameters
{
    targetStat = new ManaUpgradeFragmentTotalCount(),
    addStatIfDoesNotExist = true,
    affectType = AffectStatType.IncrementOne,
    affectValue = 1f,
};
parameters.affectStatReaction = new AffectStatManaUpgradeCountReaction(
    AffectStatManaUpgradeCountReaction.ManaAdaptationType.KeepRatioIfNotComplete);
AffectStatFeature.AffectStat(playerEntity, parameters);
```

Perform changes on Unity's main thread after the player and stats exist. Reread the value before retrying an operation that threw; a later reaction/UI failure can occur after the stat changed.

### Fragment reaction and replacement

`AffectStatManaUpgradeCountReaction` handles the upgrade reaction associated with the fragment counter, including merging fragments into maximum-stat upgrades. A direct counter assignment alone does not reproduce that full behavior.

Relevant observation points are `BeforeStatModification` and `AfterStatModification` on this reaction class.

A possible replacement strategy is to capture the original counter, allow the pickup to become used, restore the counter with `StatObject.SetFloat`, and suppress the unwanted after-modification reaction. Scope interception to the actual pickup so deliberate grants are not reverted. Verify the resulting maximum stat as well as the fragment counter.
### Persistence

Collection, original benefit delivery, and replacement delivery are separate events. Keep the world's consumed/opened state while suppressing only the intended reward effect. A second observation of the same source must not automatically grant a second reward.

On save/load, reacquire runtime objects and distinguish restored activation from new interaction. Verify both the consumed state and actual inventory/stat/XP state through normal save/reload. Do not assume an immediate disk save after an interaction.

The catalog below contains available source identifiers, not a guarantee of complete coverage or identical content in every game build. Descriptive labels are organizational labels; use actual save IDs, scene context, and game item IDs for identification. Unknown fields remain blank rather than being inferred.

### Source catalog

| Source label | Room | Object | Save ID | Original item ID |
|---|---|---|---|---|
| Streets of Paris - Guard Storehouse Mana Fragment | StreetsPoorV2_Room09_uid90 | GPE_UpgradeCollectibleMana | 68be2bca-0caa-48a4-ab69-f1e44cdb3a76 |  |
| Streets of Paris - Street of the Tomb Masons Mana Fragment | StreetsPoorV2_Room10_uid92 | GPE_UpgradeCollectibleMana | 7e77a6a3-315d-4265-8b02-dd9eb3c4cfeb |  |
| Streets of Paris - Torture Level Mana Fragment | StreetsPoorV2_Room38_uid131 | GPE_UpgradeCollectibleMana | e5c6ddbb-ba49-4461-919b-ebf17c023177 |  |
| Innocents' Cemetery - Roman Baths Mana Fragment | Cemetery2_Room25_uid113 | GPE_UpgradeCollectibleMana | c0b35f4f-63e5-4d53-8b75-ac25809cb071 |  |
| Innocents' Cemetery - Cloister Mana Fragment | Cemetery2_Room33_uid124 | GPE_UpgradeCollectibleMana | f73608fe-de63-4958-97b7-b647d9b9b1dc |  |
| Innocents' Cemetery - Mausoleum of the Kings Mana Fragment | Cemetery_MVP_Room_080_uid139 | GPE_UpgradeCollectibleMana | c8876993-d513-4c26-894e-84e7679d5869 |  |
| Catacombs - Chamber of Whispers Mana Fragment | Catacombs_MVP_Room04_uid111 | GPE_UpgradeCollectibleMana | 1eeba497-0884-41fd-9b03-cfde5ba69d48 |  |
| Catacombs - Throat of Hell Mana Fragment | Catacombs_MVP_Room13_uid121 | GPE_UpgradeCollectibleMana | 47d3e36b-2e81-417b-bf03-ca93ce6e9dc1 |  |
| Catacombs - Hall of the Fallen Angels Mana Fragment | Catacombs_MVP_Room15_uid124 | GPE_UpgradeCollectibleMana | d1178ec2-4365-4ff8-91ef-dec062a93ccb |  |
| Notre-Dame - Tree of Saints Mana Fragment | NotreDame_MVP_Room_030_uid91 | GPE_UpgradeCollectibleMana | 59363937-40b1-4678-8022-06a777371879 |  |
| Notre-Dame - Chapel of Patience Mana Fragment | NotreDame_MVP_Room_045_uid110 | GPE_UpgradeCollectibleMana | c827605f-1ba5-4c7f-9a55-b627c8abe37b |  |
| Notre-Dame - Apse Mana Fragment | NotreDame_MVP_Room_085_uid108 | GPE_UpgradeCollectibleMana | b9c7e1c9-f9cf-429b-b3c9-02e9b3b87b75 |  |
| Main Hall - Balcony Mana Fragment | MainHall_MVP_Room_23_uid271 | GPE_UpgradeCollectibleMana | 0d19b623-5793-4e2d-a747-e1ba9bae6114 |  |
| Main Hall - Entrance Corridor Mana Fragment | MainHall_MVP_Room_560_uid196 | GPE_UpgradeCollectibleMana | 20b45d3a-d20b-4c0c-80f6-d5f26520989a |  |
| Main Hall - Wine Cellar Mana Fragment | MainHall_MVP_Room_562_uid236 | GPE_UpgradeCollectibleMana | bbbcaf47-4b46-448d-bb86-6fbdcfaad3c8 |  |
| Library - Rhetoric Shelves Mana Fragment | Library_MVP_Room_03_uid96 | GPE_UpgradeCollectibleMana | 17325079-e507-4aa0-a5d2-e507cf269684 |  |
| Library - Planetarium Mana Fragment | Library_MVP_Room_06_uid99 | GPE_UpgradeCollectibleMana | b09ac924-1c62-4128-aa80-f95ab2f90ffa |  |
| Library - Fine Arts Shelves Mana Fragment | Library_MVP_Room_18_uid111 | GPE_UpgradeCollectibleMana | 32af2be2-1ca5-43ec-9f85-5a2d9d4994c0 |  |
| Clock Tower - West Tower Base Mana Fragment | ClockTower_MVP_Room20_uid119 | GPE_UpgradeCollectibleMana | f623a393-e829-4d92-b232-7af1175030b6 |  |
| Clock Tower - Central Fuel Reservoir Mana Fragment | ClockTower_MVP_Room43_uid145 | GPE_UpgradeCollectibleMana | 793225e1-54d5-4a1f-8a2f-814045ff597f |  |
| Clock Tower - Upper Gallery Mana Fragment | ClockTower_MVP_Room44_uid146 | GPE_UpgradeCollectibleMana | 8e1f917a-a996-46c8-9479-07d407533f79 |  |
| Hall of Mirrors - Boudoir of Bewilderment Mana Fragment | HallOfMirrors_MVP_Room_01_uid98 | GPE_UpgradeCollectibleMana | ea05462f-4d38-4265-9432-dfed5de6623d |  |
| Hall of Mirrors - Chamber of  Secrecy Mana Fragment | HallOfMirrors_MVP_Room_12_uid110 | GPE_UpgradeCollectibleMana | b58542b7-603d-442c-b0f5-e1f63010ed39 |  |
| Hall of Mirrors - Ballroom of Terror Mana Fragment | HallOfMirrors_MVP_Room_26_uid125 | GPE_UpgradeCollectibleMana (1) | 79e50dc2-0a93-4d19-a642-d0173f247d28 |  |

## Attack books

### Game representation

- Pickup object name: `GPE_ConsumableAtkUpBook`.
- Stat class: `PowerAttackBonus`.
- Persistent source identity: `SavableEntityBase.Id` or `guid`.

These pickups change a stat directly. Inventory-only observation or suppression does not capture their full effect.

```text
ActivableEntity.ActivateInternal
→ pickup activates and becomes used
→ stat changes
→ upgrade reaction runs where applicable
```

### Detecting collection

Observe `ActivableEntity.ActivateInternal` and identify the pickup through its save ID. Inspect `activated` and the `silent` argument to distinguish normal player interaction from restored/previously used objects. Silent activation alone must not be assumed to suppress every stat effect.

### Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.


### Reading and granting the stat

Find the active player's `ExperienceComponent.linkedEntity`, then its `StatsStateComponent`. Reacquire these after a load. Read the stat with:

```csharp
var stat = stats.GetStatObject(Il2CppType.Of<PowerAttackBonus>());
float current = stat.GetBaseValue();
```

To apply one increment:

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

Perform changes on Unity's main thread after the player and stats exist. Reread the value before retrying an operation that threw; a later reaction/UI failure can occur after the stat changed.

### Original benefit suppression

The bonus is a stat counter, not an inventory reward. A replacement system can restore it with `StatObject.SetFloat(intendedValue)` after identifying the unwanted increment. Preserve a trustworthy baseline or intended value rather than subtracting blindly on every observed callback.

Do not overwrite unrelated legitimate changes. Detecting activation, setting `silent`, and blocking inventory addition are not themselves proof that the bonus was withheld.
### Persistence

Collection, original benefit delivery, and replacement delivery are separate events. Keep the world's consumed/opened state while suppressing only the intended reward effect. A second observation of the same source must not automatically grant a second reward.

On save/load, reacquire runtime objects and distinguish restored activation from new interaction. Verify both the consumed state and actual inventory/stat/XP state through normal save/reload. Do not assume an immediate disk save after an interaction.

The catalog below contains available source identifiers, not a guarantee of complete coverage or identical content in every game build. Descriptive labels are organizational labels; use actual save IDs, scene context, and game item IDs for identification. Unknown fields remain blank rather than being inferred.

### Source catalog

| Source label | Room | Object | Save ID | Original item ID |
|---|---|---|---|---|
| Streets of Paris - Prison Tower Attack Book | StreetsPoorV2_Room01_uid82 | GPE_ConsumableAtkUpBook | 97135d6a-f214-4bae-9cc0-fe25412fb7b9 |  |
| Streets of Paris - Sewer Reservoir Attack Book | StreetsPoorV2_Room26_uid118 | GPE_ConsumableAtkUpBook | 48c2703e-0613-4b5c-8333-68aa0ada0c90 |  |
| Streets of Paris - Stairs to the Tower Roof Attack Book | StreetsPoorV2_Room39_uid132 | GPE_ConsumableAtkUpBook | b8700036-f9d5-4f54-870c-1879b811209c |  |
| Innocents' Cemetery - Ossuary of the Innocents Attack Book | Cemetery2_Room21_uid109 | GPE_ConsumableAtkUpBook | b79dfb07-aa98-452a-a6b4-1976d66fcb17 |  |
| Innocents' Cemetery - Overgrown Crypt Attack Book | Cemetery_MVP_Room_070_uid138 | GPE_ConsumableAtkUpBook | 2dbc4da0-0018-4dcf-abb0-a339d3d3fcca |  |
| Notre-Dame - Narthex Attack Book | NotreDame_MVP_Room_010_uid89 | GPE_ConsumableAtkUpBook | a6d55036-4b3c-47b2-9488-83e8ac3caeb7 |  |
| Notre-Dame - Tree of Saints Attack Book | NotreDame_MVP_Room_030_uid91 | GPE_ConsumableAtkUpBook | 65be63bd-f345-4efa-8716-4e6749b2af29 |  |
| Notre-Dame - Tomb of the Chevaliers Attack Book 1 | NotreDame_MVP_Room_135_uid101 | GPE_ConsumableAtkUpBook | 43ddb3e8-4a12-4138-9a40-14b0392ae96d |  |
| Notre-Dame - Tomb of the Chevaliers Attack Book 2 | NotreDame_MVP_Room_135_uid101 | GPE_ConsumableAtkUpBook (1) | f745746b-5d4f-4612-a989-9f6ce144a1fb |  |
| Main Hall - Wine Cellar Attack Book | MainHall_MVP_Room_20_uid266 | GPE_ConsumableAtkUpBook | fbc891a0-bfbf-4b01-9a79-d159c38ef4ca |  |
| Main Hall - Old Gallery Attack Book | MainHall_MVP_Room_28_uid277 | GPE_ConsumableAtkUpBook | 9cc20fc1-cb2f-42ec-b2b3-2ce90a0ed6f3 |  |
| Clock Tower - Counterweight Chamber Attack Book | ClockTower_MVP_Room24_uid123 | GPE_ConsumableAtkUpBook | 4f0622d1-230d-4197-8e5d-566fa061ba1e |  |
| Hall of Mirrors - Looping Hallways Attack Book | HallOfMirrors_MVP_Room_22_uid121 | GPE_ConsumableAtkUpBook | caeac29c-1ea3-4be9-9ae6-96adb6b3feb8 |  |
| Hall of Mirrors - Ballroom of Terror Attack Book | HallOfMirrors_MVP_Room_26_uid125 | GPE_ConsumableAtkUpBook | c57e37b5-f2f3-49ed-ae0f-39c3802860e9 |  |

## Defense books

### Game representation

- Pickup object name: `GPE_ConsumableDefUpBook`.
- Stat class: `PowerDefenseBonus`.
- Persistent source identity: `SavableEntityBase.Id` or `guid`.

These pickups change a stat directly. Inventory-only observation or suppression does not capture their full effect.

```text
ActivableEntity.ActivateInternal
→ pickup activates and becomes used
→ stat changes
→ upgrade reaction runs where applicable
```

### Detecting collection

Observe `ActivableEntity.ActivateInternal` and identify the pickup through its save ID. Inspect `activated` and the `silent` argument to distinguish normal player interaction from restored/previously used objects. Silent activation alone must not be assumed to suppress every stat effect.

### Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.


### Reading and granting the stat

Find the active player's `ExperienceComponent.linkedEntity`, then its `StatsStateComponent`. Reacquire these after a load. Read the stat with:

```csharp
var stat = stats.GetStatObject(Il2CppType.Of<PowerDefenseBonus>());
float current = stat.GetBaseValue();
```

To apply one increment:

```csharp
var parameters = new AffectStatParameters
{
    targetStat = new PowerDefenseBonus(),
    addStatIfDoesNotExist = true,
    affectType = AffectStatType.IncrementOne,
    affectValue = 1f,
};
AffectStatFeature.AffectStat(playerEntity, parameters);
```

Perform changes on Unity's main thread after the player and stats exist. Reread the value before retrying an operation that threw; a later reaction/UI failure can occur after the stat changed.

### Original benefit suppression

The bonus is a stat counter, not an inventory reward. A replacement system can restore it with `StatObject.SetFloat(intendedValue)` after identifying the unwanted increment. Preserve a trustworthy baseline or intended value rather than subtracting blindly on every observed callback.

Do not overwrite unrelated legitimate changes. Detecting activation, setting `silent`, and blocking inventory addition are not themselves proof that the bonus was withheld.
### Persistence

Collection, original benefit delivery, and replacement delivery are separate events. Keep the world's consumed/opened state while suppressing only the intended reward effect. A second observation of the same source must not automatically grant a second reward.

On save/load, reacquire runtime objects and distinguish restored activation from new interaction. Verify both the consumed state and actual inventory/stat/XP state through normal save/reload. Do not assume an immediate disk save after an interaction.

The catalog below contains available source identifiers, not a guarantee of complete coverage or identical content in every game build. Descriptive labels are organizational labels; use actual save IDs, scene context, and game item IDs for identification. Unknown fields remain blank rather than being inferred.

### Source catalog

| Source label | Room | Object | Save ID | Original item ID |
|---|---|---|---|---|
| Streets of Paris - Upper Tower Defense Book | StreetsPoorV2_Room14_uid97 | GPE_ConsumableDefUpBook | c35959c9-e6ac-40f9-a5ad-e64b0e73703d |  |
| Streets of Paris - Abandoned Sewers Defense Book | StreetsPoorV2_Room35_uid128 | GPE_ConsumableDefUpBook | 8e19a236-b4f5-4891-8538-e9a07dbdbf0f |  |
| Streets of Paris - Ruined Market Defense Book | StreetsPoorV2_Room42_uid135 | GPE_ConsumableDefUpBook | 3538b40d-eb71-4a88-9c4c-1e413762fc91 |  |
| Innocents' Cemetery - Ancient Sewers Defense Book | Cemetery2_Room38_uid128 | GPE_ConsumableDefUpBook | 4cbd0ce3-ace1-4be6-8a3a-f745cf73195d |  |
| Innocents' Cemetery - Flooded Crypt Defense Book | Cemetery2_Room41_uid131 | GPE_ConsumableDefUpBook | 16690d8a-0a6c-4257-ab47-e9d3139c5873 |  |
| Notre-Dame - Tree of Saints Defense Book | NotreDame_MVP_Room_030_uid91 | GPE_ConsumableDefUpBook | c9e1f5e9-eff8-40e4-8a97-a93239cedd8d |  |
| Notre-Dame - East Triforium Defense Book | NotreDame_MVP_Room_175_uid105 | GPE_ConsumableDefUpBook | 23138a06-19fc-4faa-8729-666a2aac5914 |  |
| Notre-Dame - Great Organ Room Defense Book | NotreDame_MVP_Room_666_uid112 | GPE_ConsumableDefUpBook | 11c2a8f8-eb45-4089-98da-810564cbabee |  |
| Main Hall - Mirror Rooms Defense Book | MainHall_MVP_Room_11_uid256 | GPE_ConsumableDefUpBook | aefb4a68-7ee6-4742-921f-4163feea1da1 |  |
| Main Hall - Servants' Quarters Defense Book | MainHall_MVP_Room_211_uid230 | GPE_ConsumableDefUpBook | ec14da01-cfc7-430a-a301-f2de899fb4f5 |  |
| Clock Tower - Eastern Summit Defense Book | ClockTower_MVP_Room46_uid148 | GPE_ConsumableDefUpBook | a9fbf106-fc13-44fb-9c9f-7e9d54e07e70 |  |
| Clock Tower - Daedalus Crossing Defense Book | ClockTower_MVP_Room55_uid159 | GPE_ConsumableDefUpBook | d0d9afd9-b18d-49af-9d13-c4f39a1921b7 |  |
| Hall of Mirrors - Passage of Perplexity Defense Book | HallOfMirrors_MVP_Room_05_uid103 | GPE_ConsumableDefUpBook | 88363dcd-eb4a-4f24-9631-003ce1c71f84 |  |
| Hall of Mirrors - Corridor of Déjà Vu Defense Book | HallOfMirrors_MVP_Room_09_uid107 | GPE_ConsumableDefUpBook | 61f2469d-3571-449d-97e1-b852db604868 |  |




