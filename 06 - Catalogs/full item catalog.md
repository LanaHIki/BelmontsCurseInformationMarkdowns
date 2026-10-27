# Full item catalog

[Documentation index](../README.md)

## Scope

This catalog covers the available game item asset records: 168 records representing 167 distinct internal item IDs. It includes collectible items, clues, costumes, abstract weapon bases, deprecated entries, and a placeholder. It is the full available item catalog, not a guarantee that every asset in every game build is included or obtainable during ordinary play.

The records contain 12 explicitly abstract weapon entries and 2 explicitly deprecated charm entries. Unknown metadata is left unknown. Items with missing names are not assigned invented display names.

## Runtime lookup

```csharp
ItemData item = itemsManager.FindItem(gameItemId);
if (item != null)
{
    string id = item.itemID;
    var type = item.itemType;
}
```

Resolve `ItemsManager` from loaded runtime objects. `FindItem` uses the internal item ID, not necessarily the display name or asset name. A failed lookup can mean unavailable data or an identifier mismatch; it is not proof that an asset never exists.

Ordinary inventory granting uses `inventory.AddItemToInventory(item, false)`. Abstract, deprecated, placeholder, and duplicate-ID assets should not be treated as ordinary grant targets without validation. Use the dedicated movement or stat APIs where appropriate; see [ability grant.md](../01%20-%20Items%20and%20abilities/ability%20grant.md).

## Categories and numeric item types

The category labels below describe observed records; they are not a declaration of the game's exact enum member names.

| Numeric itemType | Category | Asset records | Distinct item IDs |
|---|---|---|---|
| 1 | Weapons | 46 | 46 |
| 2 | Arcana | 17 | 16 |
| 3 | Charms, eye relics, and placeholder | 48 | 48 |
| 4 | Movement abilities | 5 | 5 |
| 5 | Clues | 40 | 40 |
| 6 | Quest keys | 5 | 5 |
| 7 | Costumes | 7 | 7 |

## Reading status labels

- **Concrete record:** explicitly not abstract and not deprecated in available metadata; this does not establish ordinary gameplay availability.
- **Abstract:** `abstractItem = 1`; a base/category asset rather than an established collectible grant target.
- **Deprecated:** explicitly marked deprecated.
- **Placeholder:** item ID `UniqueItemID`, with display name `Lorem Ipsum`.
- **Metadata incomplete:** abstract/deprecated fields were not included in the available record.
- **Duplicate item ID:** more than one asset has the same ID; preserve asset identity when inspecting behavior.

The eye relics have numeric item type 3 alongside charms. Their quest use does not imply itemType 6. Costumes have type 7; no dedicated runtime costume-selection API is established here.

## Duplicate ID: Soul Harvest

`LaserScytheSlash` occurs on two assets:

- `LaserScytheSlash_SpellData`
- `LaserScytheSlash_Doppelgagner_SpellData`

Both have the display name Soul Harvest. `FindItem("LaserScytheSlash")` alone does not establish which asset variant is returned or whether the variants behave identically. The spelling `Doppelgagner` is preserved exactly from the asset name.

## Weapons

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| Unknown | DualSword | WeaponDualWieldBase_WeaponData | Abstract |
| Unknown | WeaponAndShield | WeaponAndShield_WeaponData | Abstract |
| Unknown | WeaponAndShieldBase | WeaponAndShieldBase_WeaponData | Abstract |
| Unknown | WeaponDualWield | WeaponDualWield_WeaponData | Abstract |
| Unknown | WeaponFist | WeaponFist_WeaponData | Abstract |
| Unknown | WeaponFistBase | WeaponFistBase_WeaponData | Abstract |
| Unknown | WeaponOneHanded | WeaponOneHanded_WeaponData | Abstract |
| Unknown | WeaponSpear | WeaponSpear_WeaponData | Abstract |
| Unknown | WeaponSpearBase | WeaponSpearBase_WeaponData | Abstract |
| Unknown | WeaponTwoHanded | WeaponTwoHanded_WeaponData | Abstract |
| Unknown | WeaponTwoHandedBase | WeaponTwoHandedBase_WeaponData | Abstract |
| Unknown | WeaponWhip | WeaponWhip_WeaponData | Abstract |
| Almace and Hauteclaire | WeaponDualWieldTier3 | WeaponDualWieldTier3_WeaponData | Concrete record |
| Almace and Hauteclaire | WeaponDualWieldTier3_Doppelganger | WeaponDualWieldTier3_Doppelganger_WeaponData | Concrete record |
| Alucard's Sword and Shield | WeaponAndShieldTier4 | WeaponAndShieldTier4_WeaponData | Concrete record |
| Arondight and Lancelot's Shield | WeaponAndShieldTier3 | WeaponAndShieldTier3_WeaponData | Concrete record |
| Arondight and Lancelot's Shield | WeaponAndShieldTier3_Doppelganger | WeaponAndShieldTier3_Doppelganger_WeaponData | Concrete record |
| Bastard sword | WeaponOneHandedTier2 | WeaponOneHandedTier2_WeaponData | Concrete record |
| Bernhard's Wrath | WeaponFistTier4 | WeaponFistTier4_WeaponData | Concrete record |
| Burnt Sword of Fierbois | WeaponTwoHandedTier1 | WeaponTwoHandedTier1_WeaponData | Concrete record |
| Cestus | WeaponFistTier1 | WeaponFistTier1_WeaponData | Concrete record |
| Chauve-Souris | WeaponSpearTier4 | WeaponSpearTier4_WeaponData | Concrete record |
| Claymore | WeaponTwoHandedTier2 | WeaponTwoHandedTier2_WeaponData | Concrete record |
| Durandal | WeaponTwoHandedTier3 | WeaponTwoHandedTier3_WeaponData | Concrete record |
| Durandal | WeaponTwoHandedTier3_Doppelganger | WeaponTwoHandedTier3_Doppelganger_WeaponData | Concrete record |
| Excalibur | WeaponTwoHandedTier4 | WeaponTwoHandedTier4_WeaponData | Concrete record |
| Fists of Talos | WeaponFistTier3 | WeaponFistTier3_WeaponData | Concrete record |
| Fists of Talos | WeaponFistTier3_Doppelganger | WeaponFistTier3_Doppelganger_WeaponData | Concrete record |
| Fleuret and Tower Shield | WeaponAndShieldTier2 | WeaponAndShieldTier2_WeaponData | Concrete record |
| Gungnir | WeaponSpearTier3 | WeaponSpearTier3_WeaponData | Concrete record |
| Gungnir | WeaponSpearTier3_Doppelganger | WeaponSpearTier3_Doppelganger_WeaponData | Concrete record |
| Halberd | WeaponSpearTier2 | WeaponSpearTier2_WeaponData | Concrete record |
| Jewel Knuckles | WeaponFistTier2 | WeaponFistTier2_WeaponData | Concrete record |
| Joyeuse | WeaponOneHandedTier3 | WeaponOneHandedTier3_WeaponData | Concrete record |
| Joyeuse | WeaponOneHandedTier3_Doppelganger | WeaponOneHandedTier3_Doppelganger_WeaponData | Concrete record |
| Lance | WeaponSpearTier1 | WeaponSpearTier1_WeaponData | Concrete record |
| Longsword | WeaponOneHandedTier1 | WeaponOneHandedTier1_WeaponData | Concrete record |
| Morning Star | WeaponWhipTier3 | WeaponWhipTier3_WeaponData | Concrete record |
| Pair of Longswords | WeaponDualWieldTier1 | WeaponDualWieldTier1_WeaponData | Concrete record |
| Rapiers | WeaponDualWieldTier2 | WeaponDualWieldTier2_WeaponData | Concrete record |
| Rusted Harpè and Aegis | WeaponAndShieldTier1 | WeaponAndShieldTier1_WeaponData | Concrete record |
| Thorn Whip | WeaponWhipTier2 | WeaponWhipTier2_WeaponData | Concrete record |
| Valmanway | WeaponOneHandedTier4 | WeaponOneHandedTier4_WeaponData | Concrete record |
| Vampire Killer | WeaponWhipTier4 | WeaponWhipTier4_WeaponData | Concrete record |
| War and Famine | WeaponDualWieldTier4 | WeaponDualWieldTier4_WeaponData | Concrete record |
| Whip | WeaponWhipTier1 | WeaponWhipTier1_WeaponData | Concrete record |


## Arcana

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| Absolution | RageIncarnation | MassPurification_SpellData | Concrete record |
| Blood Spikes | BloodSpikes | BloodSpikes_SpellData | Concrete record |
| Chains of Justice | Detainment | Detainment_SpellData | Concrete record |
| Chronomantic Dive | RiseAndFall | RiseAndFall_SpellData | Concrete record |
| Cursed Flame | CursedFireBall | CursedFireBall_SpellData | Concrete record |
| Fist of Talos | GiantFist | GiantFist_SpellData | Concrete record |
| Freezing Path | FreezingMist | FreezingMist_SpellData | Concrete record |
| Holy Cross | BoomerangCross | BoomerangCross_SpellData | Concrete record |
| Infernal Pyre | FireSeismicWave | FireSeismicWave_SpellData | Concrete record |
| Life Drain | LifeDrain | LifeDrain_SpellData | Concrete record |
| Masterpiece | InvocationStone | InvocationStone_SpellData | Concrete record |
| Mirror Walk | PocketMirror | PocketMirror_SpellData | Concrete record |
| Petrifying Gaze | PetrifyingRay | PetrifyingRay_SpellData | Concrete record |
| Soul Harvest | LaserScytheSlash | LaserScytheSlash_Doppelgagner_SpellData | Concrete record; duplicate item ID |
| Soul Harvest | LaserScytheSlash | LaserScytheSlash_SpellData | Concrete record; duplicate item ID |
| Stained-Glass Shield | GlassArmor | GlassArmor_SpellData | Concrete record |
| Wheel of Chaos | TargetingChargingWheels | TargetingChargingWheels_SpellData | Concrete record |


## Charms, eye relics, and placeholder

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| Abyssal Eye | Charm_BoostDamageOnControlledMob | Charm_BoostDamageOnControlledMob | Concrete record |
| Banshee's Wail | Charm_GetManaFromDmgReceived | Charm_GetManaFromDmgReceived | Concrete record |
| Beelzebub's Seal | Charm_BoostDamageButReduceLife | Charm_BoostDamageButReduceLife | Concrete record |
| Blackened Heart | Charm_GainHPHitMobNoPotion | Charm_GainHPHitMobNoPotion | Concrete record |
| Blood Vial | Charm_ExtraPotionAfterSave | Charm_ExtraPotionAfterSave | Concrete record |
| Crypt Key | Charm_ChestRecover | Charm_ChestRecover | Concrete record |
| Crystallized Blood | Charm_ConsummableChanceBoost | Charm_ConsummableChanceBoost | Concrete record |
| Cursed Coin | Charm_BoostDamageAfterParry | Charm_BoostDamageAfterParry | Concrete record |
| Dark Sigil | Charm_BoostDefenseOnFullLife | Charm_BoostDefenseOnFullLife | Concrete record |
| Dusk Amulet | Charm_BoostSpeedLowMana | Charm_BoostSpeedLowMana | Concrete record |
| Ebony Claw | Charm_LongerMobStagger | Charm_LongerMobStagger | Concrete record |
| Eldritch Talisman | Charm_InvulnerabilityAfterSave | Charm_InvulnerabilityAfterSave | Concrete record |
| Family's Grace | Charm_TankOneHit | Charm_TankOneHit | Concrete record |
| Forsaken Ring | Charm_BoostDamageFullLife | Charm_BoostDamageFullLife | Concrete record |
| Ghostly Pendant | Charm_BoostDefenseForTrap | Charm_BoostDefenseForTrap | Concrete record |
| Gloomstone | Charm_RevealSecret | Charm_RevealSecret | Metadata incomplete |
| Grim Token | Charm_ParryGiveMana | Charm_ParryGiveMana | Concrete record |
| Haunting Relic | Charm_BoostDefenseAfterGettingHit | Charm_BoostDefenseAfterGettingHit | Concrete record |
| Hollow Bone | Charm_BoostSpeedAfterWhip | Charm_BoostSpeedAfterWhip | Concrete record |
| Infernal Medallion | Charm_ClampMCMana | Charm_ClampMCMana | Concrete record |
| Lich's Mark | Charm_ReduceSpellManaCostSpam | Charm_ReduceSpellManaCostSpam | Concrete record |
| Lorem Ipsum | DEPRECATED_Charm_BoostXPKillMob | DEPRECATED_Charm_BoostXPKillMob | Deprecated |
| Lorem Ipsum | DEPRECATED_Charm_PotionHealBoost | DEPRECATED_Charm_PotionHealBoost | Deprecated |
| Lorem Ipsum | UniqueItemID | PlaceholderItem | Placeholder |
| Midnight Tear | Charm_NerfDmgOnLowMana | Charm_NerfDmgOnLowMana | Concrete record |
| Moonlit Shard | Charm_BoostXPKillMobFullLife | Charm_BoostXPKillMobFullLife | Concrete record |
| Mourning Bead | Charm_KillMobBoostSpeed | Charm_KillMobBoostSpeed | Concrete record |
| Necro Charm | Charm_KillMobBoostDamageInflicted | Charm_KillMobBoostDamageInflicted | Concrete record |
| Nightshade | Charm_PotionGiveMana | Charm_PotionGiveMana | Concrete record |
| Obsidian Fang | Charm_BoostDamageAfterBreak | Charm_BoostDamageAfterBreak | Concrete record |
| Oozing Eye | MammonKeyCrossFire | MammonKeyCrossFire_ItemData | Metadata incomplete |
| Pale Thorn | Charm_BoostDamageBackReduceDamageFront | Charm_BoostDamageBackReduceDamageFront | Concrete record |
| Phantom Ring | Charm_RoomChangeResetMobSpawner | Charm_RoomChangeResetMobSpawner | Metadata incomplete |
| Raven's Feather | Charm_BoostDamageWhenUnGrounded | Charm_BoostDamageWhenUnGrounded | Concrete record |
| Revenant's Shard | Charm_BoostManaKillMob | Charm_BoostManaKillMob | Concrete record |
| Sand Charm | Charm_BoostSpeedAfterSave | Charm_BoostSpeedAfterSave | Concrete record |
| Shadow Veil | Charm_BoostDefenseOnLowLife | Charm_BoostDefenseOnLowLife | Concrete record |
| Sinister Rune | Charm_BoostDamageOnMobFullLife | Charm_BoostDamageOnMobFullLife | Concrete record |
| Soul Fragment | DEPRECATED_Charm_BoostManaOnDmgLowLife | Charm_BoostManaOnDmgLowLife | Concrete record |
| Specter's Whisper | Charm_BoostDamageOnLowLife | Charm_BoostDamageOnLowLife | Concrete record |
| Thrilling Eye | MammonKeyMines | MammonKeyMines_ItemData | Metadata incomplete |
| Trembling Eye | MammonKeyHoming | MammonKeyHoming_ItemData | Metadata incomplete |
| Twilight Gem | Charm_BoostDamageAfterSpell | Charm_BoostDamageAfterSpell | Concrete record |
| Vampire's Fang | Charm_BatPet | Charm_BatPet | Concrete record |
| Vile Trinket | Charm_BoostDamageBackHit | Charm_BoostDamageBackHit | Concrete record |
| Witch's Brew | Charm_BoostDamagePerEmptyPotion | Charm_BoostDamagePerEmptyPotion | Concrete record |
| Witch's Knot | Charm_HealthPotionHealFullLife | Charm_HealthPotionHealFullLife | Concrete record |
| Wraith's Tear | Charm_KillMobRevengeHeal | Charm_KillMobRevengeHeal | Concrete record |


## Movement abilities

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| Aerial Rush | AerialDash | AerialDash_ItemData | Concrete record |
| Arcane Whip | WhipPendulum | Whip_ItemData | Concrete record |
| Double Jump | DoubleJump | DoubleJump_ItemData | Concrete record |
| Holy Light | VaniaLight | VaniaLight_ItemData | Concrete record |
| Smash | LightCrash | Stomp_ItemData | Concrete record |


## Clues

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| A Door made of Flesh and Eyes | Clue_Mammon_EyesDoor | Clue_Mammon_EyesDoor | Concrete record |
| A Lever for the Well | Clue_Leon_HiddenLever | Clue_Leon_HiddenLever | Concrete record |
| A Monster terrorizes Paris | Clue_Leon_BossDoor | Clue_Leon_BossDoor | Concrete record |
| A Sketch from Talos | Clue_Eleanor_SealsLocation | Clue_Eleanor_SealsLocation | Concrete record |
| A Strange Painting of Notre-Dame | Clue_Brauner_NotreDamePainting | Clue_Brauner_NotreDamePainting | Concrete record |
| Alucard Guides | Clue_Brauner_AlucardPainting | Clue_Brauner_AlucardPainting | Concrete record |
| Artist hides in the Main Hall | Clue_Brauner_MainHallLocation | Clue_Brauner_MainHallLocation | Concrete record |
| Balancing the Scales of Justice | Clue_SaintLouis_ScaleText | Clue_SaintLouis_ScaleText | Concrete record |
| Blood Poem | Clue_Carmilla_BloodPoem | Clue_Carmilla_BloodPoem | Concrete record |
| Blueprints for the Clock Tower | Clue_Talos_ArchitectShortcut | Clue_Talos_ArchitectShortcut | Concrete record |
| Brauner's "Masterpiece" | Clue_Brauner_StatueInstruction | Clue_Brauner_StatueInstruction | Concrete record |
| Builder flees to the Clock Tower | Clue_Talos_ArchitectLocation | Clue_Talos_ArchitectLocation | Concrete record |
| Carmilla in the Library | Clue_Carmilla_BraunerNote | Clue_Carmilla_BraunerNote | Concrete record |
| Carpenter's Crane Sketch | Clue_JoanArc_TowerShortcut | Clue_JoanArc_TowerShortcut | Concrete record |
| Catacomb Relic | Clue_Mammon_DiaryMarch | Clue_Mammon_DiaryMarch | Concrete record |
| Cemetary Fountain | Clue_Medusa_CanalShortcut | Clue_Medusa_CemetaryFountain | Concrete record |
| Death Mark | Clue_Death_Catacombs | Clue_Death_Catacombs | Concrete record |
| Eleanor Flees to the Cemetary | Clue_Eleanor_Escaping | Clue_Eleanor_Escaping | Concrete record |
| Feat of Strength | Clue_Arthur_ExcaliburStele | Clue_Arthur_ExcaliburStele | Concrete record |
| Guardian's Gaze | Clue_Medusa_SpecialAttack | Clue_Medusa_SpecialAttack | Concrete record |
| Hospital Relic | Clue_Mammon_DiaryFebruary | Clue_Mammon_DiaryFebruary | Concrete record |
| Isaac waits in the Clock Tower. | Clue_Isaac_BossLocation | Clue_Isaac_BossLocation | Concrete record |
| Kneeling is the Way | Clue_Eleanor_KneelDown | Clue_Eleanor_KneelDown | Concrete record |
| Magical Brush | Clue_Carmilla_ToolSTheft | Clue_Carmilla_ToolSTheft | Concrete record |
| Melody Sheet for the Concerto of Notre-Dame | Clue_Death_Concerto | Clue_Death_Concerto | Concrete record |
| Melody Sheet for the Fugue of Notre-Dame | Clue_Death_MelodiesNumber | Clue_Death_MelodiesNumber | Concrete record |
| Melody Sheet for the Sonata of Notre-Dame | Clue_Death_Sonata | Clue_Death_Sonata | Concrete record |
| Organic Astrology | Clue_Carmilla_PlanetList | Clue_Carmilla_PlanetList | Concrete record |
| Roman Baths | Clue_Medusa_RomanBath | Clue_Medusa_RomanBaths | Concrete record |
| Star Chart | Clue_Carmilla_DiaryRequest | Clue_Carmilla_DiaryRequest | Concrete record |
| Stolen Oboles | Clue_Ferryman_MissingCoins | Clue_Ferryman_MissingCoins | Concrete record |
| The Corrupted Pipes of the Notre-Dame Organ | Clue_Death_OrganPipes | Clue_Death_OrganPipes | Concrete record |
| The First Card | Clue_Doppelganger_FirstCard | Clue_Doppelganger_FirstCard | Concrete record |
| The Masters of Music hide from Death | Clue_Death_MaestrosLocations | Clue_Death_MaestrosLocations | Concrete record |
| The Prophecy | Clue_Doppelganger_Prophecy | Clue_Doppelganger_Prophecy | Concrete record |
| The Reflection Deceives | Clue_Eleanor_TrueWay | Clue_Eleanor_TrueWay | Concrete record |
| The roots of the Hollow Tree | Clue_Medusa_CemeteryLocation | Clue_Medusa_HollowTree | Concrete record |
| The Temptress guards the Hall of Mirrors | Clue_Succubus_HoMLocation | Clue_Succubus_HoMLocation | Concrete record |
| The Warrior is at the top of the Tower. | Clue_JoanArc_TowerLocation | Clue_JoanArc_TowerLocation | Concrete record |
| Tomb Relic | Clue_Mammon_DiaryJanuary | Clue_Mammon_DiaryJanuary | Concrete record |


## Quest keys

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| Brauner's Paintbrush | NotreDameKey | NotreDameKey_QuestItem | Concrete record |
| Ferryman's Obol | FerrymanCompletionKey | FerrymanCompletionKey_QuestItem | Concrete record |
| The Fallen's Key | CatacombsGateKeyLeon | CatacombsGateKeyLeon_QuestItem | Concrete record |
| The Prophetess's Key | CatacombsGateKeyMedusa | CatacombsGateKeyMedusa_QuestItem | Concrete record |
| The Warrior's Key | CatacombsGateKeyJeanne | CatacombsGateKeyJeanne_QuestItem | Concrete record |


## Costumes

| Display name | Internal item ID | Asset name | Status |
|---|---|---|---|
| Alucard Style Costume | Skin_MC_Alucard | Skin_MC_Alucard | Concrete record |
| Cursed's outfit | Skin_MC_FullCurse | Skin_MC_FullCurse | Concrete record |
| Doppelganger Style Costume | Skin_MC_Doppleganger | Skin_MC_Doppleganger | Concrete record |
| Rose's outfit | Skin_MC_Base | Skin_MC_Base | Concrete record |
| Sonia Style Costume | Skin_MC_Sonia | Skin_MC_Sonia | Concrete record |
| Sypha Style Costume | Skin_MC_Sypha | Skin_MC_Sypha | Concrete record |
| Trevor Style Costume | Skin_MC_Trevor | Skin_MC_Trevor | Concrete record |


## Non-inventory rewards and unlocks

The following game functions are not additional item asset records in the tables above:

| Function | Runtime representation |
|---|---|
| Health Fragment | `HealthUpgradeFragmentTotalCount` with `AffectStatHealthUpgradeCountReaction` |
| Mana Fragment | `ManaUpgradeFragmentTotalCount` with `AffectStatManaUpgradeCountReaction` |
| Attack Book | `PowerAttackBonus` |
| Defense Book | `PowerDefenseBonus` |
| XP | `ExperienceComponent.IncrementCurrentExperience` |
| Mirror-travel lock | First Death key `DeathCatacombs_Defeated_ID`, GUID `769b38ad62fa14249b54a74ef889998c` |
| First Shadow confrontation / castle route flag | `DeathCatacombs_Defeated_ID`, GUID `769b38ad62fa14249b54a74ef889998c` |

Do not pass invented item IDs such as a stat class name into `FindItem` and assume they represent inventory assets. Movement abilities do have real item assets, but their explicit runtime unlock state is exposed through `VaniaMovesComponent`.

## Related catalogs and runtime documents

- [clues.md](../02%20-%20Rewards%20and%20pickups/clues.md): clue acquisition and candidate rooms.
- [arcana tasks and talents.md](../01%20-%20Items%20and%20abilities/arcana%20tasks%20and%20talents.md): task/talent asset identities, distinct from spell ownership.
- [books and fragments.md](../02%20-%20Rewards%20and%20pickups/books%20and%20fragments.md): stat pickup APIs and source identifiers.
- [progression flags.md](../03%20-%20Progression%20and%20cutscenes/progression%20flags.md): story flags, character steps, and the Catacombs gem gate.
- [experience granting.md](../01%20-%20Items%20and%20abilities/experience%20granting.md): XP increment API.




