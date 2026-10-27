# Clues

[Documentation index](../README.md)

## Representation and acquisition

Tarot/codex clues can be inventory items identified by internal strings such as `Clue_Leon_HiddenLever`. Scripted interactions and cutscenes can grant them without a conventional chest or pickup object.

```text
Interaction or cutscene
→ clue ItemData
→ Inventory.AddItemToInventory
→ clue ownership / codex information
```

Detect a clue grant by inspecting `ItemData.itemID` at inventory addition. The following overload is an available observation point:

```csharp
Inventory.AddItemToInventory(
    ItemData,
    Il2CppSystem.Action<ItemData>,
    bool,
    bool)
```

Allow the original method to run when recording acquisition without removing the clue. A grant attempt is not necessarily a first acquisition: initialization, replays, and duplicate grants may also reach inventory code. Compare ownership or maintain deduplicated observations when first acquisition matters.

## Looking up clue items

```csharp
var clue = itemsManager.FindItem("Clue_Leon_HiddenLever");
if (clue != null && inventory != null)
{
    bool owned = inventory.HasItemInInventoryOrEquipped(clue);
}
```

The ordinary inventory grant API can add a resolved clue item:

```csharp
inventory.AddItemToInventory(clue, false);
```

Inventory ownership, the cutscene granting the clue, and story progression are distinct. Adding an item alone is not established to reproduce every side effect of its original interaction.

## Identifying the acquisition room

Obtain the room through `CurrentRoomInstance.currentRoom`; its `gameObject.scene.name` identifies the current scene.

```text
Observe clue grant
→ read internal item ID
→ resolve current room
→ record item ID, scene, cutscene, and relevant runtime context
```

The observed room identifies where the grant occurred, not necessarily the place described by the clue. Delayed scripted grants or transitions can complicate attribution. Retain the scene/cutscene context and distinguish observations from inferred locations.

## Candidate rooms

Room candidates can be inferred from cutscene asset placement, director objects, and scene objects associated with a clue. Multiple candidates can mean uncertainty; they do not prove that a clue is obtainable in every listed room.

A room base name may appear at runtime with an appended instance suffix:

```text
Candidate: StreetsPoorV2_Room42
Runtime:   StreetsPoorV2_Room42_uid...
```

A conservative matching rule is:

```csharp
bool SameRoom(string scene, string candidate)
{
    return scene != null && candidate != null
        && (scene == candidate
            || scene.StartsWith(candidate + "_uid", StringComparison.Ordinal));
}
```

Avoid unrestricted substring matching. Similar room names and shared cutscene assets can otherwise generate false matches.

## Confidence and ambiguity

| Evidence | Interpretation |
|---|---|
| Runtime grant observed in a scene | Observed acquisition context |
| One candidate from asset placement | Candidate, not independently verified acquisition |
| Several candidates for one clue | Unresolved placement or shared assets |
| Several clues assigned to one candidate room | Could be genuine co-location or ambiguous mapping |
| Clue-looking scene object | Discovery lead only |

A runtime observation can take priority over asset-derived candidates for a particular observed route/build. Keep observations separate rather than silently overwriting history: one observed grant does not prove that no alternative acquisition context exists.

## Scene discovery

Object names containing `clue`, `tarot`, or `codex` can help locate relevant scene content. Examples include `ClueSpiritShow`, `MC_TarotDeck_Mesh`, and clue-associated camera/director objects.

Visual effects, props, and camera names are not proof of collectible behavior. Follow the interaction/cutscene to its actual inventory grant or other state change. A clue source need not expose a usable world-pickup save ID.

## Cutscene examples

Candidate scene relationships include:

| Cutscene | Candidate room |
|---|---|
| `First_Clue_Leon` | `StreetsPoorV2_Room42` |
| `Fool_Boss_Door` | `StreetsPoorV2_Room29` |
| `Joan_Clue_001` | `StreetsPoorV2_Room21` |
| `Joan_Clue_002` | `StreetsPoorV2_Room14` |
| `Ferryman_Clue_001` | `StreetsPoor_Room24` |
| `Death_Clue_001` | `Catacombs_MVP_Room24` |
| `Catacombs_Mammon_Door_Mammon_Clue_02` | `Catacombs_MVP_Room36` |

These are candidate mappings, not proof that each scene always grants a clue there. `Fool_Boss_Door` illustrates that a clue-granting sequence need not have `Clue` in its name. Cutscene skipping can therefore remove useful information or reward effects if based only on naming patterns.

## Persistence

Clue items use normal inventory state. Confirm retention with normal save/reload and recheck ownership on the restored player inventory. An external observation log does not mean the clue is present in the current save.

A clue item and its named character's progression keys are different identifiers. In particular, `Clue_Eleanor_KneelDown` describes kneeling, but does not establish travel eligibility. The mirror-travel lock is the first Death key, `DeathCatacombs_Defeated_ID`, separate from Eleanor character state. Do not treat clue ownership as proof that an ability or interaction is unlocked.

## Clue catalog and room candidates

The following 40 entries provide clue labels, internal item IDs, and available candidate rooms. Labels/area groupings are descriptive. Room entries are asset-derived candidates unless independently confirmed in play; `Unknown` means no candidate is established here. This is not a claim that every clue in every game build is included.

| Clue | Internal item ID | Candidate rooms |
|---|---|---|
| Streets of Paris - The Fallen (Leon) Clue: A Lever for the Well | Clue_Leon_HiddenLever | StreetsPoorV2_Room42 |
| Streets of Paris - The Fallen (Leon) Clue: A Monster terrorizes Paris | Clue_Leon_BossDoor | StreetsPoorV2_Room29 |
| Streets of Paris - The Warrior (Joan) Clue: The Warrior is at the top of the Tower | Clue_JoanArc_TowerLocation | StreetsPoorV2_Room21 |
| Streets of Paris - The Warrior (Joan) Clue: Carpenter's Crane Sketch | Clue_JoanArc_TowerShortcut | StreetsPoorV2_Room14 |
| Streets of Paris - The Whisperer (Ferryman) Clue: Stolen Oboles | Clue_Ferryman_MissingCoins | StreetsPoor_Room24 |
| Streets of Paris - The Miser (Mammon) Clue: Tomb Relic | Clue_Mammon_DiaryJanuary | StreetsPoorV2_Room47 |
| Streets of Paris - The Miser (Mammon) Clue: Hospital Relic | Clue_Mammon_DiaryFebruary | StreetsPoorV2_Room47 |
| Streets of Paris - The Miser (Mammon) Clue: Catacomb Relic | Clue_Mammon_DiaryMarch | StreetsPoorV2_Room47 |
| Catacombs - The Miser (Mammon) Clue: A Door made of Flesh and Eyes | Clue_Mammon_EyesDoor | Catacombs_MVP_Room36 |
| Innocents' Cemetery - The Enchantress (Medusa) Clue: Cemetary Fountain | Clue_Medusa_CanalShortcut | Cemetery2_Room19 |
| Innocents' Cemetery - The Enchantress (Medusa) Clue: The roots of the Hollow Tree | Clue_Medusa_CemeteryLocation | Cemetery2_Room22 |
| Innocents' Cemetery - The Enchantress (Medusa) Clue: Roman Baths | Clue_Medusa_RomanBath | Cemetery2_Room34 |
| Innocents' Cemetery - The Enchantress (Medusa) Clue: Guardian's Gaze | Clue_Medusa_SpecialAttack | Cemetery2_Room26 |
| Catacombs - The Shadow (Death) Clue: Death Mark | Clue_Death_Catacombs | Catacombs_MVP_Room24 |
| Notre-Dame - The Shadow (Death) Clue: Melody Sheet for the Concerto of Notre-Dame | Clue_Death_Concerto | NotreDame_MVP_Room_666, NotreDame_MVP_Room_195, NotreDame_MVP_Room_050, NotreDame_MVP_Room_105 |
| Notre-Dame - The Shadow (Death) Clue: The Masters of Music hide from Death | Clue_Death_MaestrosLocations | NotreDame_MVP_Room_666, NotreDame_MVP_Room_195, NotreDame_MVP_Room_050, NotreDame_MVP_Room_105 |
| Notre-Dame - The Shadow (Death) Clue: Melody Sheet for the Fugue of Notre-Dame | Clue_Death_MelodiesNumber | NotreDame_MVP_Room_666, NotreDame_MVP_Room_195, NotreDame_MVP_Room_050, NotreDame_MVP_Room_105 |
| Notre-Dame - The Shadow (Death) Clue: The Corrupted Pipes of the Notre-Dame Organ | Clue_Death_OrganPipes | NotreDame_MVP_Room_140 |
| Notre-Dame - The Shadow (Death) Clue: Melody Sheet for the Sonata of Notre-Dame | Clue_Death_Sonata | NotreDame_MVP_Room_666, NotreDame_MVP_Room_195, NotreDame_MVP_Room_050, NotreDame_MVP_Room_105 |
| Main Hall - The Artist (Brauner) Clue: Alucard Guides | Clue_Brauner_AlucardPainting | MainHall_MVP_Room_050, MainHall_MVP_Room_28, MainHall_MVP_Room_1903 |
| Main Hall - The Artist (Brauner) Clue: Artist hides in the Main Hall | Clue_Brauner_MainHallLocation | MainHall_MVP_Room_050, MainHall_MVP_Room_28, MainHall_MVP_Room_1903 |
| Main Hall - The Artist (Brauner) Clue: A Strange Painting of Notre-Dame | Clue_Brauner_NotreDamePainting | MainHall_MVP_Room_1345 |
| Main Hall - The Artist (Brauner) Clue: Brauner's "Masterpiece" | Clue_Brauner_StatueInstruction | MainHall_MVP_Room_050, MainHall_MVP_Room_28, MainHall_MVP_Room_1903 |
| Main Hall - The Occultist (Carmilla) Clue: Carmilla in the Library | Clue_Carmilla_BraunerNote | MainHall_MVP_Room_1903, Library_MVP_Room_15, Library_MVP_Room_10, Library_MVP_Room_14, Library_MVP_Room_17 |
| Library - The Occultist (Carmilla) Clue: Blood Poem | Clue_Carmilla_BloodPoem | MainHall_MVP_Room_1903, Library_MVP_Room_15, Library_MVP_Room_10, Library_MVP_Room_14, Library_MVP_Room_17 |
| Library - The Occultist (Carmilla) Clue: Star Chart | Clue_Carmilla_DiaryRequest | MainHall_MVP_Room_1903, Library_MVP_Room_15, Library_MVP_Room_10, Library_MVP_Room_14, Library_MVP_Room_17 |
| Library - The Occultist (Carmilla) Clue: Organic Astrology | Clue_Carmilla_PlanetList | MainHall_MVP_Room_1903, Library_MVP_Room_15, Library_MVP_Room_10, Library_MVP_Room_14, Library_MVP_Room_17 |
| Library - The Occultist (Carmilla) Clue: Magical Brush | Clue_Carmilla_ToolSTheft | MainHall_MVP_Room_1903, Library_MVP_Room_15, Library_MVP_Room_10, Library_MVP_Room_14, Library_MVP_Room_17 |
| Clock Tower - The Builder (Talos) Clue: Builder flees to the Clock Tower | Clue_Talos_ArchitectLocation | MainHall_MVP_Room_080, ClockTower_MVP_Room55 |
| Clock Tower - The Builder (Talos) Clue: Blueprints for the Clock Tower | Clue_Talos_ArchitectShortcut | MainHall_MVP_Room_080, ClockTower_MVP_Room55 |
| Clock Tower - Isaac Clue: Isaac waits in the Clock Tower | Clue_Isaac_BossLocation | ClockTower_MVP_Room59 |
| Hall of Mirrors - The Sage (Eleanor) Clue: Eleanor Flees to the Cemetary | Clue_Eleanor_Escaping | HallOfMirrors_MVP_Room_22, HallOfMirrors_MVP_Room_04, ClockTower_MVP_Room58 |
| Hall of Mirrors - The Sage (Eleanor) Clue: Kneeling is the Way | Clue_Eleanor_KneelDown | HallOfMirrors_MVP_Room_22, HallOfMirrors_MVP_Room_04, ClockTower_MVP_Room58 |
| Hall of Mirrors - The Sage (Eleanor) Clue: A Sketch from Talos | Clue_Eleanor_SealsLocation | HallOfMirrors_MVP_Room_22, HallOfMirrors_MVP_Room_04, ClockTower_MVP_Room58 |
| Hall of Mirrors - The Sage (Eleanor) Clue: The Reflection Deceives | Clue_Eleanor_TrueWay | HallOfMirrors_MVP_Room_22, HallOfMirrors_MVP_Room_04, ClockTower_MVP_Room58 |
| Hall of Mirrors - Succubus Clue: The Temptress guards the Hall of Mirrors | Clue_Succubus_HoMLocation | HallOfMirrors_MVP_Room_16 |
| Notre-Dame - Arthur Clue: Feat of Strength | Clue_Arthur_ExcaliburStele | Unknown |
| Notre-Dame - The Judge (Saint Louis) Clue: Balancing the Scales of Justice | Clue_SaintLouis_ScaleText | Unknown |
| Notre-Dame - The Self (Doppelganger) Clue: The First Card | Clue_Doppelganger_FirstCard | Unknown |
| Notre-Dame - The Self (Doppelganger) Clue: The Prophecy | Clue_Doppelganger_Prophecy | Unknown |



