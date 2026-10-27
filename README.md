# Disclaimer and information
So this repo was made to document information from a vibe coded version of Castlevania Belmont's Curse. Codex was used to organize information while being looked over.

None of this information should be regraded as the truth. Like seriously shit might just be hallucinated etc. The only reason this was made was to be leads towards better things. Things to investigate and look at while finding the truth for implementation by humans. None of this was verified directly. Especially not the code(it's all illustrative please just don't copy paste it)

Like seriously it's vibe coded claude into codex summary of the code with no context.

I hope it helps a little with speeding up development.
(and hopefully not slow things down)


# Game documentation

Standalone game API notes and catalogs. Start with the topic groups below; every Markdown document is listed here.

## Suggested starting points

- **Grant something:** Items and abilities.
- **Identify or replace a reward:** Rewards and pickups.
- **Change story state:** Progression and cutscenes.
- **Move between destinations:** Travel and map.
- **Read live game state:** Runtime and lifecycle.
- **Look up exact identifiers:** Catalogs.

## Status notes

- Code examples are illustrative unless their document says otherwise.
- Arcana task/talent runtime behavior has not been investigated; the document distinguishes extracted assets from save-file observations.
- Exact map-completion aggregation and statue XP suppression remain unconfirmed.
- Runtime inspection is retained as a reference. These notes do not schedule further investigation.

## Items and abilities

| Document | Contents |
|---|---|
| [Item, ability, and stat granting](01%20-%20Items%20and%20abilities/ability%20grant.md) | Inventory grants, direct movement-ability controls, and stat APIs. |
| [Experience granting](01%20-%20Items%20and%20abilities/experience%20granting.md) | Fixed and level-scaled XP grants through ExperienceComponent. |
| [Arcana tasks and talents](01%20-%20Items%20and%20abilities/arcana%20tasks%20and%20talents.md) | Task/talent asset groups and GUIDs. Runtime behavior remains uninvestigated; state observations came from save files. |

## Rewards and pickups

| Document | Contents |
|---|---|
| [Reward sources and replacement mechanics](02%20-%20Rewards%20and%20pickups/check%20replacement.md) | Overview and per-category code examples for reward detection and suppression. |
| [Chests](02%20-%20Rewards%20and%20pickups/chests.md) | Chest activation, inventory reward interception, quick-equip considerations, and source IDs. |
| [Boss and cutscene rewards](02%20-%20Rewards%20and%20pickups/boss%20rewards.md) | Scripted boss reward grants, encounter attribution, and grouped rewards. |
| [Unique rewards, altars, and special chests](02%20-%20Rewards%20and%20pickups/unique%20rewards%20and%20altars.md) | Special givers, state-machine relationships, and unique reward sources. |
| [Books and fragments](02%20-%20Rewards%20and%20pickups/books%20and%20fragments.md) | Health/mana fragments and attack/defense books: stats, reactions, and source catalogs. |
| [EXP statues](02%20-%20Rewards%20and%20pickups/exp%20statues.md) | Statue activation, source IDs, and detection examples; original XP suppression is unconfirmed. |
| [Clues](02%20-%20Rewards%20and%20pickups/clues.md) | Clue ownership, acquisition events, identifiers, and candidate-room mappings. |

## Progression and cutscenes

| Document | Contents |
|---|---|
| [Progression flags](03%20-%20Progression%20and%20cutscenes/progression%20flags.md) | Key APIs, character steps, identity flags, and the three-gem Catacombs door. |
| [Cutscene control](03%20-%20Progression%20and%20cutscenes/cutscene%20control.md) | Playback tracking, watched-state queries, force-ending, and video handling. |
| [Ending detection](03%20-%20Progression%20and%20cutscenes/ending%20detection.md) | Bad, normal, and true ending identifiers and cutscene-start detection. |

## Travel and map

| Document | Contents |
|---|---|
| [Level and biome transitions](04%20-%20Travel%20and%20map/level%20biome%20transitions.md) | Level/entrance assets, BiomeTransitor, loading readiness, and placement lifecycle. |
| [Mirror travel](04%20-%20Travel%20and%20map/mirror%20travel.md) | GPE_Mirror eligibility, per-mirror Death conditions, destination-used flags, and mirror catalog. |
| [Teleportation and custom spawn placement](04%20-%20Travel%20and%20map/teleportation.md) | Coordinate movement and forced spawn placement, separate from selecting mirror destinations. |
| [Map interaction](04%20-%20Travel%20and%20map/map%20interaction.md) | Map context, save-point hover, marker matching, and map-cursor movement. |
| [Room and map completion](04%20-%20Travel%20and%20map/room%20map%20completion.md) | Completion categories, counter behavior, and known item/progression mismatches; exact aggregation is unconfirmed. |

## Runtime and lifecycle

| Document | Contents |
|---|---|
| [Runtime inspection](05%20-%20Runtime%20and%20lifecycle/runtime%20inspection.md) | Read-only player, room, object, equipment, ability, stat, and progression inspection with examples. |
| [Save/load lifecycle](05%20-%20Runtime%20and%20lifecycle/save%20load%20lifecycle.md) | Runtime restoration observation, component readiness, and spawn/loading lifecycle. |

## Catalogs

| Document | Contents |
|---|---|
| [Full item catalog](06%20-%20Catalogs/full%20item%20catalog.md) | 168 item asset records / 167 distinct IDs, with abstract, deprecated, and duplicate records marked. |
| [Room and district catalog](06%20-%20Catalogs/room%20district%20catalog.md) | Area index for 243 districts and 423 room identifiers, with membership caveats. |

## Area catalogs

Detailed district/room pages linked from the room catalog. Room inventories can be expanded within each page.

| Document | Contents |
|---|---|
| [Catacombs](06%20-%20Catalogs/room%20district%20catalog/Catacombs.md) | Catacombs districts, rooms, and object inventories. |
| [Clock Tower](06%20-%20Catalogs/room%20district%20catalog/ClockTower.md) | Clock Tower districts, rooms, and object inventories. |
| [Hall of Mirrors](06%20-%20Catalogs/room%20district%20catalog/HallOfMirrors.md) | Hall of Mirrors districts, rooms, and object inventories. |
| [Innocents' Cemetery](06%20-%20Catalogs/room%20district%20catalog/InnocentsCemetery.md) | Innocents' Cemetery districts, rooms, and object inventories. |
| [Library](06%20-%20Catalogs/room%20district%20catalog/Library.md) | Library districts, rooms, and object inventories. |
| [Main Hall](06%20-%20Catalogs/room%20district%20catalog/MainHall.md) | Main Hall districts, rooms, and object inventories. |
| [Master's Quarter](06%20-%20Catalogs/room%20district%20catalog/MasterQuarter.md) | Master's Quarter districts, rooms, and object inventories. |
| [Millers' Bridge](06%20-%20Catalogs/room%20district%20catalog/MillersBridge.md) | Millers' Bridge districts, rooms, and object inventories. |
| [Master's Quarter - Bad Ending](06%20-%20Catalogs/room%20district%20catalog/MQBadEnding.md) | Bad-ending Master's Quarter districts and rooms. |
| [Notre-Dame](06%20-%20Catalogs/room%20district%20catalog/NotreDame.md) | Notre-Dame districts, rooms, and object inventories. |
| [Streets of Paris - Poor Quarter](06%20-%20Catalogs/room%20district%20catalog/StreetsPoor.md) | Poor-quarter Streets districts, rooms, and object inventories. |
| [Streets of Paris - Rich Quarter](06%20-%20Catalogs/room%20district%20catalog/StreetsRich.md) | Rich-quarter Streets districts, rooms, and object inventories. |
| [Rooms without district membership](06%20-%20Catalogs/room%20district%20catalog/unassigned.md) | Seven room identifiers without district membership. |

## Folder layout

```text
README.md
01 - Items and abilities/
02 - Rewards and pickups/
03 - Progression and cutscenes/
04 - Travel and map/
05 - Runtime and lifecycle/
06 - Catalogs/
  room district catalog/
```



