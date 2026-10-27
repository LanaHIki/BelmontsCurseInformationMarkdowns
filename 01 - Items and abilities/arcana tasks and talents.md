# Arcana tasks and talents

[Documentation index](../README.md)

> **Runtime behavior has not been investigated yet.** Task/talent state and point-accounting observations came from save-file inspection, not a verified runtime API. The asset names, GUIDs, and groupings came from extracted game data. This document is a catalog of known identifiers; it does not provide verified runtime methods for reading task progress, completing tasks, spending points, or unlocking talents.

## Separate parts of Arcana state

Arcana ownership, task progress, task completion, task visibility, and talent unlocks are different state layers.

```text
Arcana item ownership
+ gameplay task conditions and progress
→ completed tasks
→ talent-point availability
→ talent selection / unlocked talent state
```

The exact event handlers, task thresholds, point-allocation rules, and runtime talent-activation methods are not established here. The tables below contain the known asset identifiers and groupings.

## Known catalog size

The available task/talent catalog contains:

- 16 task/talent groups.
- 44 grouped task entries.
- 43 grouped talent entries.

These counts describe the available catalog, not a guarantee that every asset or future game build is covered. Additional task-like assets can exist outside the groups. In particular, assets described as BloodSpikes `LongTermBleedingKeyEvent` and InvocationStone `GargoyleKillingGargoyle` were identified as ungrouped; their exact identifiers and gameplay roles are not established here. Do not add them to the grouped completion total without verification.

## Identifier namespaces and naming differences

| Identifier | Example | Role |
|---|---|---|
| Arcana item ID | `BoomerangCross` | Spell item ownership |
| Group asset name | `BoomerangCross_TaskTalentGroup` | Groups task/talent entries |
| Task asset name | `BoomerangCross_Task_ChainKill` | Specific task definition |
| Task GUID | `cf586d0902bdb784db7a061351979bcf` | Task asset identity |
| Talent asset name | `BoomerangCross_Talent_DoubleCross` | Specific talent definition |
| Talent GUID | `ed8d72a5c9eb81849b58bf3a50f6f556` | Talent asset identity |

Do not derive a GUID from its name or assume a consistent prefix. Known differences include:

- `HangedMan_TaskTalentGroup` contains `RiseAndFall_*` tasks/talents; the Arcana item `RiseAndFall` is Chronomantic Dive.
- `TargetingChargingWheel_TaskTalentGroup` uses singular `Wheel`, while task/talent names and the item ID use `TargetingChargingWheels`.
- `CursedFireBall_TaskTalentGroup` contains tasks beginning `CurseFireBall` but talent names beginning `CursedFireBall`.
- `MassPurification_TaskTalentGroup` is present, but its exact relationship to the available Arcana item list is not established.
- Absolution has item ID `RageIncarnation`; a corresponding group with that prefix is not present in the available catalog. Do not equate it to `MassPurification` without further evidence.

## Catalog interpretation

The tables preserve exact asset names and GUIDs. A name can suggest a task objective or talent effect, but does not establish thresholds, duration, damage multiplier, prerequisites, or localized wording.

Tasks and talents listed together belong to the same available group. Row order is not proof that task N unlocks talent N. Do not infer one-to-one edges from the lists or assume completing a particular task automatically activates a specific talent.

## Group summary

| Group asset | Associated Arcana label | Tasks | Talents |
|---|---|---|---|
| BloodSpikes_TaskTalentGroup | Blood Spikes | 3 | 3 |
| BoomerangCross_TaskTalentGroup | Holy Cross | 3 | 3 |
| CursedFireBall_TaskTalentGroup | Cursed Flame | 2 | 1 |
| Detainment_TaskTalentGroup | Chains of Justice | 3 | 3 |
| FireSeismicWave_TaskTalentGroup | Infernal Pyre | 3 | 3 |
| FreezingMist_TaskTalentGroup | Freezing Path | 2 | 2 |
| GiantFist_TaskTalentGroup | Fist of Talos | 3 | 3 |
| GlassArmor_TaskTalentGroup | Stained-Glass Shield | 3 | 3 |
| InvocationStone_TaskTalentGroup | Masterpiece | 3 | 3 |
| LaserScytheSlash_TaskTalentGroup | Soul Harvest | 3 | 3 |
| LifeDrain_TaskTalentGroup | Life Drain | 3 | 3 |
| MassPurification_TaskTalentGroup | Not established | 3 | 3 |
| PetrifyingRay_TaskTalentGroup | Petrifying Gaze | 2 | 2 |
| PocketMirror_TaskTalentGroup | Mirror Walk | 3 | 3 |
| HangedMan_TaskTalentGroup | Chronomantic Dive (RiseAndFall naming) | 2 | 2 |
| TargetingChargingWheel_TaskTalentGroup | Wheel of Chaos (TargetingChargingWheels naming) | 3 | 3 |

## BloodSpikes_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| BloodSpikes_Task_BleedKillCount | 3d75694d9a9eca748a17cdcc8c4012e9 |
| BloodSpikes_Task_FindDualWieldingWeapons | 9878548b8c3aa5048b45f8e7ec702e2c |
| BloodSpikes_Task_LongTermBleeding | f736a228c4c47c442bf92e2350fabf2c |

### Talents

| Asset name | GUID |
|---|---|
| BloodSpikes_Talent_DoubleBleedDamages | 973815755c510a0488f075e6954a5c77 |
| BloodSpikes_Talent_BleedLifeSteal | 5453444e3715f984a8155f873c47b90d |
| BloodSpikes_Talent_WeakeningBleed | fdb91094c0010a24e82a62823c561ab6 |

## BoomerangCross_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| BoomerangCross_Task_FindWhipWeapons | 59d04ac0868a2ca4781268efaf054024 |
| BoomerangCross_Task_ChainKill | cf586d0902bdb784db7a061351979bcf |
| BoomerangCross_Task_WhipAttackKillCount | 5a1c97ee47764cf44a9ff4f8ce0c7370 |

### Talents

| Asset name | GUID |
|---|---|
| BoomerangCross_Talent_DoubleCross | ed8d72a5c9eb81849b58bf3a50f6f556 |
| BoomerangCross_Talent_DistanceIncreasingDamages | 7ac0837d721f75f49b54bde40d157aa3 |
| BoomerangCross_Talent_RegainManaOnCatch | 14c7a54344f19094db1daa938b0e6dac |

## CursedFireBall_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| CurseFireBall_Task_HitCount | 1c26d19acd6d910448748388f5f044be |
| CurseFireBall_Task_SecretTask | a96860c17952b15458739e186029b33f |

### Talents

| Asset name | GUID |
|---|---|
| CursedFireBall_Talent_BigFireBall | 36fec1e5ae091bd479861e34f3940c3e |

## Detainment_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| Detainment_Task_DetainedGroup | 359078c57388f7544815f930aa6116d2 |
| Detainment_Task_DetainedKillCount | 7e2866036f07c854b836c9f540379b01 |
| Detainment_Task_UpgradeLifeMana | 0e6672336301e984c80f18777d0e2243 |

### Talents

| Asset name | GUID |
|---|---|
| Detainment_Talent_StunDetained | 97c453cf3beb15040be3a5a7844c1b2b |
| Detainment_Talent_DetainedStaggerLoop | 9a4e50dc21a418e4f8a52975d4aaa609 |
| Detainment_Talent_LowLifeKill | 2db626072b045564bb12de787dc6cb18 |

## FireSeismicWave_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| FireSeismicWave_Task_ParryCount | f2c497d34c5eabe4988d891a35a93ba8 |
| FireSeismicWave_Task_HolyPlace | 8e65503567065954b95eab71a4d7965b |
| FireSeismicWave_Task_GroupFire | 6ceb76eab60dc964f8dac40c76f39023 |

### Talents

| Asset name | GUID |
|---|---|
| FireSeismicWave_Talent_ManaRecoverOnExplosionHit | 07568814b647cbc4ab74ad4562453dca |
| FireSeismicWave_Talent_HigherProjection | aecbaf1f675a42e4f800051981844b39 |
| FireSeismicWave_Talent_ReignitionOnWeaponHit | 6b53d5052c9ad6f4ebaab98e746dce97 |

## FreezingMist_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| FreezingMist_Task_FreezeBreakKillCount | e587e7c596de40f4aa9cb5d17eb956c7 |
| FreezingMist_Task_GroupFreeze | 9252cd70e18ac204f89444c671ffd816 |

### Talents

| Asset name | GUID |
|---|---|
| FreezingMist_Talent_FrostWaveOnEnd | 64ffb5bbb69ce214ea2dca591cff7df3 |
| FreezingMist_Talent_IncreasedSpeedOnMist | 323a4a9d44ff6eb4ebfe8b867125ecc6 |

## GiantFist_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| GiantFist_Task_FistGroupKill | 61af9bd8957c082489bc2b5fd061e379 |
| GiantFist_Task_AirborneEnemyKill | c07c4dff0f114aa428eb1189e40567a9 |
| GiantFist_Task_StompKill | 28195e676518fc3448c7eb0062821b49 |

### Talents

| Asset name | GUID |
|---|---|
| GiantFist_Talent_ControlledEnemiesIncreasedDamages | 054ed4e8390ebcd45bdaa7b2b5bb05b2 |
| GiantFist_Talent_IncreasedSeismicWave | dc09f106ce0b3ab4c84aa4e8d115cd32 |
| GiantFist_Talent_SpikesFist | cf88a7b03457b1e40bf684b6a57cec60 |

## GlassArmor_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| GlassArmor_Task_GlassShieldSavior | 7eb73d5f26825764aa991bfdf3454497 |
| GlassArmor_Task_HauntedArmorsKillCount | c8716f87b125d4341b2b47fb4462319b |
| GlassArmor_Task_SingleAttackGroupKill | 60765778caa388943b486c3347068e84 |

### Talents

| Asset name | GUID |
|---|---|
| GlassArmor_Talent_AdditionalShield | 8e9df52a991cefa4d9584b51926d746d |
| GlassArmor_Talent_GlassShardOnHit | c361222cbc6dc0e4f80e50244ceabe57 |
| GlassArmor_Talent_ResistanceDuringWeaponAttack | 3db425c348e217145b0fee9c8612a09d |

## InvocationStone_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| InvocationStone_Task_DiveAttackKillCount | 8d1fdecc1e4e2dd49a1d5e2329c0343f |
| InvocationStone_Task_InvocationStoneFatalFall | 0f6658a45480b224ea4dee66e3681838 |
| InvocationStone_Task_MainHallSecretZonesExploration | 9047ced99b419ef4fbf0619f8c48cc21 |

### Talents

| Asset name | GUID |
|---|---|
| InvocationStone_Talent_DoubleGargoyle | df078384c9c943a4ab3f9e39a6ef311e |
| InvocationStone_Talent_HealingFireball | aa829f8f48941ae469fc64610e2be07b |
| InvocationStone_Talent_PreserveGargoyleOnKill | 832ad780c5aca6642b9e03f0fdc52fe2 |

## LaserScytheSlash_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| LaserScytheSlash_Task_NotreDamePuzzle | 2a3f2c8d9a534e647b2b9430d7c54a14 |
| LaserScytheSlash_Task_SkeletonKillCount | bac9a8b71483d514d9ad273e3f37f008 |
| LaserScytheSlash_Task_GroupScythes | 9bceaaf310bae9448a666817bbca77e3 |

### Talents

| Asset name | GUID |
|---|---|
| LaserScytheSlash_Talent_SingleHitHeal | ec88c83b40a56734da9fa1299b333d04 |
| LaserScytheSlash_Talent_ScythesAdditionalSplit | fab2657df02676e40ae9796ee4ea7a55 |
| LaserScytheSlash_Talent_ManaRecoverOnScytheKill | 35fe8503662a7dd469ad4fbd5362ec97 |

## LifeDrain_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| LifeDrain_Task_DrainDuration | 3411973f6dcba2d4cb19191582ae48ba |
| LifeDrain_Task_ExploreBiome | 670c95b801ea1d546947236606321a4e |
| LifeDrain_Task_KillCount | 200c35ff386b15d4888ab1a29c24af44 |

### Talents

| Asset name | GUID |
|---|---|
| LifeDrain_Talent_FullLifeBonusDamage | d06b08991403c0342a6ae6e5dbfbf8f0 |
| LifeDrain_Talent_InterruptProtect | f4cebfc93e60a0445bbe38e9559b0bc2 |
| LifeDrain_Talent_MultipleTarget | a5b4abdba9780c64a8d193b4a21d5f42 |

## MassPurification_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| MassPurification_Task_KillGroupMultiple | 89ee2507a0e54c3418ca494e6027e5cd |
| MassPurification_Task_ExploreBiome | 1ff5e2143642ab040bd6219c1cf8ae55 |
| MassPurification_Task_FullHealCount | fbb8c7e13f6eb774e92d8a25ec574164 |

### Talents

| Asset name | GUID |
|---|---|
| MassPurification_Talent_MoreDamageFullHealth | 1eec585dfe6f10c4dbd269abd3e9c282 |
| MassPurification_Talent_BurnOnHit | badb7ce5c8249574dbf0821c4d7a672c |
| MassPurification_Talent_HealOnKill | 56d6d74dc2381d54c92cdb3f101f99f0 |

## PetrifyingRay_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| PetrifyingRay_Task_ShieldedEnemiesPetrification | 862dd7b84a2033e46b87aab31a6a4916 |
| PetrifyingRay_Task_PetrifiedFatalFallKillCount | b298c3fe842c7d34bba95e3fd64ae88f |

### Talents

| Asset name | GUID |
|---|---|
| PetrifyingRay_Talent_ManaRecoverOnPetrifiedKill | e9517c6b7693f174a880eba310dfc9ae |
| PetrifyingRay_Talent_PersistingClosePetrification | 13468740adb68df4a8792b9c45cd9685 |

## PocketMirror_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| PocketMirror_Task_SaveRoomsDiscovery | 42cc4d2bdabd3084fb9302ed6c2dec2b |
| PocketMirror_Task_HealthRecoverCount | 6f98fa9fff5b67e468d918fd31ee1c50 |
| PocketMirror_Task_DodgeAndKillCount | d679013200fb648419def3ca63d99891 |

### Talents

| Asset name | GUID |
|---|---|
| PocketMirror_Talent_ManaRecoverWhenNoHeal | 5a4ec3aa7982c8e4093b8e182aa34f6c |
| PocketMirror_Talent_DamagingTeleportation | ee6d0c85ed77d3247852d8ab2bb2912b |
| PocketMirror_Talent_DeathDefiance | e96e85081d8dedd4f901559933e43499 |

## HangedMan_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| RiseAndFall_Task_SlowMoKillCount | d3519383b44f03248bbdd21f9508bae6 |
| RiseAndFall_Task_ClockTowerExploration | f458bad117b9c784aafb5098ea039daa |

### Talents

| Asset name | GUID |
|---|---|
| RiseAndFall_Talent_IncreasedSlowMoDuration | 7196d92f748580f4cb9d699908227b65 |
| RiseAndFall_Talent_AdditionalDamageStep | ec79a6122bc2cb6459428597657fad95 |

## TargetingChargingWheel_TaskTalentGroup

### Tasks

| Asset name | GUID |
|---|---|
| TargetingChargingWheels_Task_LongTermWheel | e649519df0f00a34fb27e9cfc66a49a7 |
| TargetingChargingWheels_Task_KillCount | 5c66aafa82cab4041bc7b81a2b552159 |
| TargetingChargingWheels_Task_LoseMana | 2f8b22964f26d564aa1cc85b1a20d08c |

### Talents

| Asset name | GUID |
|---|---|
| TargetingChargingWheels_Talent_ConsumeHealthInsteadOfMana | aca3afadc56a0ed43a0e270453595857 |
| TargetingChargingWheels_Talent_ChainIncreasedDamages | 4ae3114f63f7b4849a52af3ed6346403 |
| TargetingChargingWheels_Talent_RandomEffects | b66fc4d2e5106514c9d32d019ab4ee87 |

## Granting boundaries

Adding an Arcana item through `Inventory.AddItemToInventory` does not itself establish task completion or talent unlocks.

No verified live API for completing an arbitrary task, marking one seen, spending points, unlocking a talent, or refreshing its effect is established here. Avoid inventing methods such as `UnlockTalent` from asset names. These APIs are not established here.




