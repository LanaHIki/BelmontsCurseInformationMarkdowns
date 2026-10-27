# Unique rewards, altars, and special chests

[Documentation index](../README.md)

## Mechanics

The actual giver can be a sibling of the persistent state machine controlling a reward. Consequently, the giver's nearest save ID may differ from the state machine's identifier.

Useful identification data:

1. Persistent save ID.
2. Scene plus specific object/hierarchy identity.
3. Configured original item ID and encounter context.

Objects named with the `LdBrick_StateMachine` prefix can participate in unique reward sequences. Spawned pickups may hand off their reward through inventory addition after the original interaction.

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
| Streets of Paris - Roof of the Tower Unique Reward | Room_Gym_Boss_JeanneDArc_uid69 | LdBrick_StateMachine_FierboisLooter | 470b9562-19cf-4a8b-94b7-795baa743297 |  |
| Innocents' Cemetery - Serpents' Lair Unique Reward | Cemetery2_Room30_uid118 | GPE_ChestUniqueLooter | d026f3a9-2682-4093-8a72-3764ce4f6e84 |  |
| Catacombs - The Miser (Mammon) Unique Reward | MillersBridge_13_BossRoom_uid105 | GPE_ChestUniqueLooter | 5b4dc3a0-aead-4a46-b033-05da110d4682 | FerrymanCompletionKey |
| Notre-Dame - Great Organ Room Unique Reward | NotreDame_MVP_Room_667_uid146 | GPE_ChestUniqueLooter | dabc584e-861c-416e-957d-54746affa904 |  |
| Notre-Dame - Chapel Unique Reward | NotreDame_MVP_Room_Boss_Arthur_uid121 | LdBrick_StateMachine_ExcaliburState | 6d7befec-c3fb-4ce9-bc89-118f3a0884d9 |  |
| Main Hall - Alucard's Tower Unique Reward | MainHall_MVP_Room_07_uid252 | GPE_ChestUniqueLooter | 0cdeaf3e-2e44-419f-bb04-5a746ccb782a |  |
| Library - Observatory Unique Reward | Gym_Boss_Carmilla_Room01_uid133 | GPE_ChestUniqueLooter | 0ff6c2a6-e680-4b1f-b57f-5497645cb7c8 | NotreDameKey |
| Clock Tower - Builder's Bureau Unique Reward | ClockTower_MVP_Room58_uid162 | GPE_ChestUniqueLooter | 196e7ab6-a5c9-4a58-bbf6-caa499dcaa6a |  |
| Hall of Mirrors - Level Deviled Unique Reward | HallOfMirrors_MVP_Room_30_uid129 | GPE_ChestUniqueLooter | e81eaed6-080a-4c5b-9dee-0789542fc633 |  |
| Streets of Paris - Hospital Mammon Altar (Thrilling Eye) | StreetsPoor_Room24_uid62 | GPE_MammonAltarMines |  | MammonKeyMines |
| Innocents' Cemetery - Pillaged Tombs Mammon Altar (Trembling Eye) | Cemetery_MVP_Room_170_uid150 | GPE_MammonAltarHoming |  | MammonKeyHoming |
| Catacombs - Circle of Greed Mammon Altar (Oozing Eye) | Catacombs_MVP_Room39_uid149 | GPE_MammonAltarCrossFire |  | MammonKeyCrossFire |
| Main Hall - The Sage (Eleanor): Kneel at the Mirrors |  |  |  | #MirrorKneel |
| Clock Tower - Isaac Unique Reward | ClockTower_MVP_Room47_uid150 | GPE_ChestUniqueLooter | 67b35013-4b97-4a56-8528-b014efd3993d | WeaponSpearTier4 |
| Catacombs - Altar of Bones: The Shadow Confronted |  |  |  | #ShadowConfronted |


