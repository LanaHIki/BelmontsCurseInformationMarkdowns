# Chests

[Documentation index](../README.md)

## Mechanics

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
| Streets of Paris - Sewer Reservoir Chest | StreetsPoorV2_Room03_uid81 | GPE_Chest | a9d3c2f1-add3-4cdd-9ad8-7095b3452349 |  |
| Streets of Paris - Burned District Chest | StreetsPoorV2_Room07_uid88 | GPE_Chest(Clone) | 3844c52e-0209-4507-9f8b-30b747bbbb74 |  |
| Streets of Paris - Central Sewer Drain Chest | StreetsPoorV2_Room17_uid101 | GPE_Chest(Clone) | efdbf82a-b9ae-413c-84b1-af9c567c59ad |  |
| Streets of Paris - Justice District Sewers Chest | StreetsPoorV2_Room23_uid110 | GPE_Chest(Clone) | 671678f9-4ce3-4074-90d8-1bd6fdd262dd |  |
| Streets of Paris - Guard Level Chest | StreetsPoorV2_Room37_uid130 | GPE_Chest(Clone) | 9aa11863-b1cd-4d22-aec2-d17d749b15a1 |  |
| Streets of Paris - Ruined Market Chest | StreetsPoorV2_Room44_uid137 | GPE_Chest(Clone) | d30d0afd-a6e7-46f5-ab26-f0d6163064cf |  |
| Streets of Paris - Seer's Hovel Chest | StreetsPoor_RenderTarget_06_uid104 | GPE_Chest(Clone) | 5c08221d-f5ed-4136-80da-eea21d25a289 |  |
| Innocents' Cemetery - Fountain of the Innocents Chest | Cemetery2_Room19_uid151 | GPE_Chest | a0211686-ce9f-41cb-95b9-d692569a0402 |  |
| Innocents' Cemetery - Mass Grave Chest | Cemetery2_Room27_uid115 | GPE_Chest(Clone) | 0a7c7b8f-7c9b-42ea-b68d-0f60bb4de20a |  |
| Innocents' Cemetery - Serpents' Lair Chest | Cemetery2_Room30_uid118 | GPE_Chest | 38ac5df9-406b-473f-b295-e1392d5acbf6 |  |
| Innocents' Cemetery - Cemetery Entrance Chest | Cemetery2_Room35_uid125 | GPE_Chest(Clone) | 625684f5-6389-4213-8360-5a82924adbc7 |  |
| Innocents' Cemetery - Flooded Crypt Chest | Cemetery2_Room41_uid131 | GPE_Chest(Clone) | c7e942ce-004d-4240-a380-f6fa91f5b709 |  |
| Innocents' Cemetery - Hot Springs Chest | Cemetery_MVP_Room_010_uid132 | GPE_Chest(Clone) | 09e4cf71-0b91-4861-8ad8-8f42c288ec2a |  |
| Innocents' Cemetery - Pillaged Tombs Chest | Cemetery_MVP_Room_020_uid133 | GPE_Chest(Clone) | 4e1c1aea-7845-46fc-9e7d-a07e0bec02e9 |  |
| Catacombs - Crossroads of the Dead Chest | Catacombs_MVP_Room01_uid108 | GPE_Chest(Clone) | c327c4fa-3a76-44ca-a7af-cdd63a41530e |  |
| Catacombs - Throat of Hell Chest | Catacombs_MVP_Room13_uid121 | GPE_Chest(Clone) | 97e903dd-ac70-464b-acaf-61ccd99d5b42 |  |
| Catacombs - Hall of the Fallen Angels Chest | Catacombs_MVP_Room14_uid123 | GPE_Chest | 4b26cfd3-3052-4dc6-925d-3af1faa43566 |  |
| Catacombs - Jaws of Limbo Chest | Catacombs_MVP_Room17_uid126 | GPE_Chest(Clone) | b6fb32c8-722f-423d-8c9b-ebc3f9c582ab |  |
| Catacombs - Catacomb Entrance Chest | Catacombs_MVP_Room19_uid128 | GPE_Chest(Clone) | ebad67ca-40a8-4e69-85b3-a66e71ae52a2 |  |
| Catacombs - Path of the Abyss Chest | Catacombs_MVP_Room26_uid136 | GPE_Chest(Clone) | 607fee28-4b3e-4f78-9a2f-ed4606ff8608 |  |
| Catacombs - Collapsed Path of Orpheus Chest | Catacombs_MVP_Room28_uid138 | GPE_Chest(Clone) | 326cb8ce-5826-46b8-bbcc-5a02f08ceffe |  |
| Catacombs - Antechamber of Sin Chest | Catacombs_MVP_Room35_uid145 | GPE_Chest(Clone) | cf0fec6d-eff2-40e6-b8e5-e0dc7c5fa945 |  |
| Notre-Dame - Chapel of Patience Chest | NotreDame_MVP_Room_045_uid110 | GPE_Chest(Clone) | e65e9d0e-129c-46e9-a8ca-5b770438246e |  |
| Notre-Dame - Nave Chest | NotreDame_MVP_Room_060_uid94 | GPE_Chest(Clone) | 7e35afcb-de11-40f3-a8d5-bc0b3b5a04d1 |  |
| Notre-Dame - Transept Chest | NotreDame_MVP_Room_140_uid80 | GPE_Chest(Clone) | f8bb05db-398a-4648-aa8a-d337b5c1b8b3 |  |
| Notre-Dame - Tower Entrance Chest | NotreDame_MVP_Room_185_uid99 | GPE_Chest(Clone) | e6383722-0dd7-41d9-8279-41e596ff1e3a |  |
| Notre-Dame - East Gallery Chest | NotreDame_MVP_Room_225_uid104 | GPE_Chest(Clone) | 394c0857-75a4-4877-b9c4-160875590cdf |  |
| Notre-Dame - Reliquary Chest | NotreDame_MVP_Room_230_uid138 | GPE_Chest(Clone) | 5163deb8-5140-4e87-af14-9dd67b6acf4a |  |
| Notre-Dame - Path of the Penitent Chest | NotreDame_MVP_Room_235_uid109 | GPE_Chest(Clone) | 5de4d346-96ec-42db-84ab-9d4dae01acd0 |  |
| Main Hall - Crematorium Chest | MainHall_MVP_Room_01_uid246 | GPE_Chest(Clone) | 3684c4f2-81e6-42e0-bd5f-f118a6a13fe6 |  |
| Main Hall - Servants' Quarters Chest | MainHall_MVP_Room_031_uid232 | GPE_Chest(Clone) | fd4e30eb-1e03-4de4-acea-54be984383af |  |
| Main Hall - Archives Chest | MainHall_MVP_Room_051_uid240 | GPE_Chest(Clone) | 0bb56ae2-1d5e-4543-910a-e9bbe848d210 |  |
| Main Hall - Abandoned Staircase Chest | MainHall_MVP_Room_05_uid250 | GPE_Chest(Clone) | 7f6fe1a2-a4b9-430e-852d-b5004fd8af66 |  |
| Main Hall - Watch Tower Chest | MainHall_MVP_Room_060_uid210 | GPE_Chest(Clone) | 2351b538-7c9d-4866-9484-850ca06498c5 |  |
| Main Hall - West Hall Chest | MainHall_MVP_Room_070_uid213 | GPE_Chest(Clone) | 8bbc000f-37ce-4556-b53f-14ed43085e5e |  |
| Main Hall - Abandoned Workshop Chest | MainHall_MVP_Room_080_uid245 | GPE_Chest(Clone) | 4bac8559-be53-4967-b13f-4504384f2dac |  |
| Main Hall - Wine Cellar Chest | MainHall_MVP_Room_19_uid265 | GPE_Chest(Clone) | 68971f0a-db88-444c-8da2-a7636f3aceb1 |  |
| Main Hall - Inner Courtyard Chest | MainHall_MVP_Room_25_uid273 | GPE_Chest(Clone) | f6d980ab-7c16-442f-88c4-6f262822e311 |  |
| Main Hall - Old Gallery Chest | MainHall_MVP_Room_28_uid277 | GPE_Chest(Clone) | d62cb02c-a790-4171-a79f-a35584f37dea |  |
| Library - Rhetoric Shelves Chest 1 | Library_MVP_Room_07_uid100 | GPE_Chest(Clone) | 4d3730a6-6fad-4c5e-885f-c9ccd4387de3 |  |
| Library - Curiosity Cabinet Chest | Library_MVP_Room_13_uid106 | GPE_Chest(Clone) | 890f2db6-c5b7-4487-b237-dbc6a4c180ea |  |
| Library - Condemned Shelves Chest | Library_MVP_Room_16_uid109 | GPE_Chest(Clone) | d03d14f9-abd6-4e47-b713-111794c65b73 |  |
| Library - Study Room Chest | Library_MVP_Room_21_uid116 | GPE_Chest(Clone) | 7c3c2cb4-1b70-4386-95a2-f839a78020df |  |
| Library - Fine Arts Shelves Chest | Library_MVP_Room_23_uid118 | GPE_Chest(Clone) | 31033d49-b2cd-4d3d-8520-86ed1e5467e4 |  |
| Library - Rhetoric Shelves Chest 2 | Library_MVP_Room_30_uid125 | GPE_Chest(Clone) | 1df7662b-cf39-4b45-87e3-f828d75763b6 |  |
| Library - Lisa's Laboratory LisaLab Chest | Library_MVP_Room_36_LisaLab_uid138 | GPE_Chest(Clone) | 16658d05-b2ff-4005-9cb9-63a7e15806ea |  |
| Clock Tower - Western Ascent Chest | ClockTower_MVP_Room03_uid101 | GPE_Chest(Clone) | a1d26509-8c78-4040-89d1-7c5ee9a7151d |  |
| Clock Tower - Path of the Builder Chest | ClockTower_MVP_Room25_uid124 | GPE_Chest(Clone) | 6914af90-04f4-426b-9f88-2b09597f9de2 |  |
| Clock Tower - Eastern Access Shaft Chest | ClockTower_MVP_Room45_uid147 | GPE_Chest(Clone) | 53574af8-49da-4411-980c-40ee4943c2c7 |  |
| Clock Tower - Daedalus Crossing Chest 1 | ClockTower_MVP_Room51_uid155 | GPE_Chest(Clone) | 36b9876c-581b-43d3-88f8-687fa4900718 |  |
| Clock Tower - Daedalus Crossing Chest 2 | ClockTower_MVP_Room55_uid159 | GPE_Chest(Clone) | 7a65b29e-8f64-4e54-a473-19355df1a8df |  |
| Clock Tower - Mechanic's Ascent Chest | ClockTower_MVP_Room59_uid163 | GPE_Chest(Clone) | 5d4c144f-e533-4b2e-a8da-123f032d61a6 |  |
| Clock Tower - The Way of the Fist Chest | ClockTower_MVP_TestRoom08_uid93 | GPE_Chest(Clone) | 3060719d-e139-45c3-8bb8-2acc34fa9893 |  |
| Hall of Mirrors - Delived Level Chest | HallOfMirrors_MVP_Room_02_uid100 | GPE_Chest(Clone) | d1cd7fa7-5bf3-456b-b63c-184e326aa945 |  |
| Hall of Mirrors - Dead-end Tower Chest | HallOfMirrors_MVP_Room_15_uid113 | GPE_Chest(Clone) | d888829a-0776-4d69-ba5d-52cc8fea9b4b |  |
| Hall of Mirrors - Servants' Refectory Chest | HallOfMirrors_MVP_Room_21_uid120 | GPE_Chest(Clone) | e0bc73f7-b305-4b0f-90f3-1fd0ec8f8672 |  |
| Hall of Mirrors - Steps of Misdirection Chest | HallOfMirrors_MVP_Room_28_uid127 | GPE_Chest(Clone) | 7b355d8c-ea24-48a9-9e43-21609ffe7bf6 |  |
| Master's Quarter - Sanctuary of Solitude Chest | MastersQuarters_Room_03_uid6 | GPE_Chest(Clone) | d44c614a-677b-404e-bab2-0273600efaf9 |  |


