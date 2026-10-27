# Mirror travel

[Documentation index](../README.md)

Mirror travel selects a save-mirror destination using its used-mirror progression key and its destination level/entrance identifiers.

This document covers mirror-specific access and destinations. General asset resolution, `BiomeTransitor` configuration, loading readiness, and transition verification are documented in [level biome transitions.md](level%20biome%20transitions.md). Coordinate teleportation and forced spawn placement are documented in [teleportation.md](teleportation.md).

## Mirror travel flow

```text
Check first Death progression key
→ read used-mirror progression flags
→ choose an eligible mirror
→ resolve its destination level and entrance
→ use the level/biome transition system
→ verify arrival at the intended mirror
```

Normal mirror destinations have an entrance identified by a `BiomeTransitionID`, often named with the pattern `BiomeTransitionID_*_HallOfMirrors_NN`. Travel can use that entrance without assigning the player's transform manually.

Some broken Hall of Mirrors destinations have no individual entrance. They use a shared entrance plus custom spawn coordinates. The catalog's `Custom spawn` column distinguishes these destinations; placement is covered in [teleportation.md](teleportation.md).

## Mirror unlock state

Used save mirrors are represented by `MC_Used_SaveRoom_*` progression keys. Read the player's `ProgressionComponent.progressionKeys` and compare each `ProgressionKeyID.guid` with the destination's unlock-key GUID.

```text
Player entity
→ ProgressionComponent
→ progressionKeys
→ matching used-mirror key
→ destination has been used/unlocked
```

Alternatively, resolve the key asset and use `HasProgressionKey(key)`. A missing player or component means unlock state is unavailable, not proof that every mirror is locked.

Unlock state and the act of launching a transition are separate: a custom travel interface should explicitly enforce its intended unlock policy.

## Travel lock: first Death progression key

Mirror travel is gated by the first Death/The Shadow confrontation progression key, not by meeting Eleanor.

- Key: `DeathCatacombs_Defeated_ID`
- GUID: `769b38ad62fa14249b54a74ef889998c`
- Event: first confrontation with Death/The Shadow at the Altar of Bones.

This global travel lock is separate from `MC_Used_SaveRoom_*`, which records whether an individual destination mirror has been used.

```text
First Death progression key present
+ destination mirror's used-save-room key present
→ mirror travel is eligible under these progression prerequisites
```

Gameplay readiness and destination validity must still be checked before launching a transition. These prerequisites are distinct from entering and unsealing other Hall of Mirrors content.

`MC_CharacterDataModificator_Eleanor` (`6e518ea36b23065439b86276732c7471`) is a separate character progression key. It must not be used as the mirror-travel lock. The earlier attribution of travel access to Eleanor was incorrect.

A direct `BiomeTransitor.StartBiomeTransition(playerEntity)` call does not itself enforce a custom travel interface's eligibility rules. Check the first Death key and destination-used flag explicitly when following normal mirror-travel restrictions. Calling a transition does not grant either flag.

## Destination information

| Field | Meaning |
|---|---|
| Name | Human-readable mirror label |
| Progression GUID | Used-mirror key |
| Level GUID | Destination `LevelID` asset |
| Transition GUID | Destination `BiomeTransitionID` asset |
| Room | Scene/room identifier |
| District | Map district identifier |
| X/Y | World-space coordinates, when known |

These are different identifier namespaces. A progression GUID cannot be substituted for a level or transition GUID.

## Example destination

Central Sewer Drain, Streets of Paris:

| Property | Value |
|---|---|
| Used-mirror key GUID | `c40c8285459ea6c40b6e196480d2bc30` |
| Level GUID | `946565bcb5d5f244e991e1790575a67c` |
| Transition GUID | `2565439dc685cdd4aba5bf3284cc22a3` |
| Room | `StreetsPoor_SaveRoom02_uid78` |
| District | `StreetsPoor_District05` |
| Coordinates | X = 36, Y = -11 |

The normal entrance-based journey uses the level/transition pair; it does not require applying these coordinates manually. Identifiers and positions are build-dependent.

## Travel and persistence

Travel, unlocking a mirror, activating its save point, and writing a disk save are distinct operations. Launching a biome transition does not by itself establish that the destination's used-mirror flag was granted or a save was written.

## Destination catalog

The table below lists destination identifiers and available coordinates. A custom-spawn entry requires coordinate placement through a shared entrance. Labels describe destinations; they are not asset lookup keys.

| Mirror | Region | Used-key GUID | Level GUID | Entrance GUID | X/Y | Custom spawn |
|---|---|---|---|---|---|---|
| Central Sewer Drain | Streets of Paris | c40c8285459ea6c40b6e196480d2bc30 | 946565bcb5d5f244e991e1790575a67c | 2565439dc685cdd4aba5bf3284cc22a3 | 36, -11 | No |
| Prison Tower | Streets of Paris | b42866ca3afd5414fac2ffaf5064fec9 | 946565bcb5d5f244e991e1790575a67c | f1a1df6d21af0e245b64822f639f1524 | 132, 67 | No |
| Gate to the Stone Bridge | Streets of Paris | d35010d062a139149b572ef99ee0d978 | 946565bcb5d5f244e991e1790575a67c | b8e8d8c53d3ac0945970ed3178389f17 | 516, 28 | No |
| Burned District | Streets of Paris | 46959e79a9fd8d043809a714f248919d | 946565bcb5d5f244e991e1790575a67c | 594c4fd6633d0c94ca0cf425f5e3dbb6 | 228, 41 | No |
| Undercity | Streets of Paris | 783e0e913786a7246b8845542c1a9c46 | 946565bcb5d5f244e991e1790575a67c | 3f584dd20e9b22645bfbb3524050f9fb | -132, 28 | No |
| Cloister | Innocents' Cemetery | 868f690e7ed94464387603cd3822215f | e18ccfe651d904147ba60f4c72f9b146 | 5fb42b09770c44f4aa652d0a301aa895 | 396, 262 | No |
| Pauper's Grave | Innocents' Cemetery | 7ab796e23daf8134a9a7c3f02c073fb4 | e18ccfe651d904147ba60f4c72f9b146 | f9a0f0e356ba3114fad768e7b918f411 | -60, 262 | No |
| Ancient Sewers | Innocents' Cemetery | 987ce3a438083cb468dda97deb247c95 | e18ccfe651d904147ba60f4c72f9b146 | 6fd8abea310881d409ce2db9c5646913 | 228, 184 | No |
| Overgrown Crypt | Innocents' Cemetery | c7c8f2b10fe75e1499e958933c9c504f | e18ccfe651d904147ba60f4c72f9b146 | c44be841bed015143838b60f448812ce | -108, 210 | No |
| Quarry Elevator | Catacombs | cf68ec93a64807941abea099de3836e0 | afeda68d5a0c4204da3488bbf7f04900 | 16dfd77433df3384192b28d55281c723 | 204, 145 | No |
| Realm of Old Night | Catacombs | c10f47df5007eb248a55ef8542448fce | afeda68d5a0c4204da3488bbf7f04900 | 4b4bebd707697c140888494faad51f29 | 84, 28 | No |
| Notre-Dame Undercroft | Notre-Dame | fc9c3fdeb16d953418588aea4f8a0366 | 57cb4e79eb2c4e943821705d39709461 | 9f7dfd0cd60b5834aa75e8f7e68e4604 | Unknown | No |
| West Triforium | Notre-Dame | 3847d9eeab734594e83f987c81fc8911 | 57cb4e79eb2c4e943821705d39709461 | 9c3e8b05f832d8c4999e6eee27d8c39b | Unknown | No |
| East Gallery | Notre-Dame | f361f2df1203c174f8a2f414f95fbe1c | 57cb4e79eb2c4e943821705d39709461 | bf78ca4921f7b574db45ca73a609d297 | Unknown | No |
| Entrance Corridor | Main Hall | dac851a2229a18241aea2f84fe804cc0 | 4f7ce84e4688e534aa656a76e6bd9416 | eab4b4c551748244b941a310a5630dc5 | 348, 54 | No |
| Ballrooms | Main Hall | 07ee6fc6717a8a34ea34be7248b83b9c | 4f7ce84e4688e534aa656a76e6bd9416 | 33db5d7867c6f8e4b99e657b6bcb1b8e | 540, 54 | No |
| Elevator Tower | Main Hall | b62994e6f03748d4c9661e8f715c9fa2 | 4f7ce84e4688e534aa656a76e6bd9416 | 05fd8b1fb039bf64cb2d3297598de0a9 | 444, 106 | No |
| Boudoirs | Main Hall | 8724ec587f61c09458ce03fc2ac9c435 | 4f7ce84e4688e534aa656a76e6bd9416 | 0527e6487bb82af41bac4ea58ac851ca | 420, 28 | No |
| Servants' Quarters | Main Hall | 536961f2bf983944d8d1cbc24d30f99e | 4f7ce84e4688e534aa656a76e6bd9416 | a64af81d25afed34b9978b06992441f7 | 540, -102 | No |
| Torture Chambers | Main Hall | 0a25e6f7aff582b41a9afacbe2199d1e | 4f7ce84e4688e534aa656a76e6bd9416 | d426fb956ecf949448fd3cb3976bf49b | 540, -128 | No |
| Waterworks | Main Hall | 862e24f4f9b71e940b36276ef42b58b9 | 4f7ce84e4688e534aa656a76e6bd9416 | bb30a35d74a266445aeb5818d1669efa | 420, -128 | No |
| Storerooms | Main Hall | 6d7edc685d3691c4ba63a3a945caea1d | 4f7ce84e4688e534aa656a76e6bd9416 | a70100f1645b47241b0759c2804cec43 | 348, -102 | No |
| Archives | Main Hall | 2caba5d23f8e84d45adc170a0ef93175 | 4f7ce84e4688e534aa656a76e6bd9416 | b92a51ee1b8cccb4a98463f9322cf8ee | 444, -50 | No |
| Rhetoric Shelves | Library | 7f1b03df08ebc57438f7460f64b4e991 | 226c20eca175b104e8fe1a8f4fc2c879 | 576f96da3da7ce7478aac8cc55fd0c78 | Unknown | No |
| Exhibition Room | Library | 258556103daee0d44b69583432ba6df5 | 226c20eca175b104e8fe1a8f4fc2c879 | 227cba484c384844fbfa5a2cf5231462 | Unknown | No |
| Mechanic's Ascent | Clock Tower | 21e177c48b8371f459db950ec555821e | c912f1bff8fc06740902f1f67666ae86 | a8fae16093d61dc4e91ba621b9bfde93 | Unknown | No |
| Foundry Apex | Clock Tower | a32ffffeb32802b40b020df38507bc24 | c912f1bff8fc06740902f1f67666ae86 | f7fd24801d459bd4ea03d0cfda776084 | Unknown | No |
| Eastern Summit | Clock Tower | 2fd94930c290a814d94f1b3446432a4f | c912f1bff8fc06740902f1f67666ae86 | 86543cebb2e00a64886ab3f9f056856a | Unknown | No |
| Sanctuary of Solitude | Master's Quarter | 383a160a50c52464e9da03cb7c7bdd89 | 794afe09823b0e04dac2044bf092cff2 | fdd5b6b22adc06f4dbd686664cba8bea | Unknown | No |
| Threshold of Vision | Hall of Mirrors | b68c99f91c3a2064e8bdc7bd8abd2529 | b76c5d815e88a8442ad46b1c869efd5f | 117e85d816d7eba49a5fc3ccab86f406 | 156, 197 | Yes |
| Familiar Way | Hall of Mirrors | dc2fc6b4f9aa9484f87f12ee7a670176 | b76c5d815e88a8442ad46b1c869efd5f | 117e85d816d7eba49a5fc3ccab86f406 | 156, 2 | Yes |
| Nightmare Atrium | Hall of Mirrors | 076c637c5bf14f84995d8122d5980e2a | b76c5d815e88a8442ad46b1c869efd5f | 117e85d816d7eba49a5fc3ccab86f406 | 252, 184 | Yes |
| The Descent Into Memory | Hall of Mirrors | 594a83550b6c0e34c9717704f0ef1229 | b76c5d815e88a8442ad46b1c869efd5f | 117e85d816d7eba49a5fc3ccab86f406 | 156, 106 | Yes |
| Heart of the Maze | Hall of Mirrors | f2519f6b210852e49a9d54c031012c66 | b76c5d815e88a8442ad46b1c869efd5f | 117e85d816d7eba49a5fc3ccab86f406 | 156, 67 | Yes |






