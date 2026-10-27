# Room and map completion

[Documentation index](../README.md)

## Completion is a collection of categories

A room can be explored and have its visible pickups consumed while still reporting incomplete. Completion is evaluated through `CompletionItemType` assets that expose current and total counts for a `WorldRoomData` and player entity.

```text
CurrentRoomInstance.currentRoom.worldRoomData
+ player Entity
+ CompletionItemType asset
→ current count / total count
```

Reported categories can include items, secret zones, discovered doors, exploration zones, progression keys, and counters. Combined and district-level categories can aggregate other results.

Map coloring and percentages can reflect more than whether a chest looks opened. A purple/incomplete room does not prove a visible pickup is missing. The complete map-color decision logic and percentage formula are not established here.

## How room completion progress builds up

Room completion is represented through several category counts evaluated for a specific room and player. Each category reports how many requirements exist and how many are satisfied.

```text
WorldRoomData + player entity/state
→ each CompletionItemType calculates current / total
→ room and broader map completion reporting consumes those results
```

Illustrative example, not a measured room:

| Category | Current / total | Interpretation |
|---|---|---|
| Items | 1 / 2 | One item requirement remains |
| Secret zones | 1 / 1 | Reported secret-zone requirement satisfied |
| Exploration zones | 2 / 2 | Reported exploration requirements satisfied |
| Progression keys | 0 / 1 | One progression-related requirement remains |

### Two observable paths

**Category evaluation:** each `CompletionItemType` exposes `GetRoomCurrentCompletionCount(roomData, playerEntity)` and `GetRoomTotalCompletionCount(roomData, playerEntity)`. Different category implementations can evaluate different state, including item acquisition, discoveries, progression, or counters. These queries expose results; they do not reveal the complete internal formula.

**Counter updates:** some events pass through `ExplorerComponent.IncrementCompletionCount(worldRoom, completionItemType, incrementValue)`. This is an observable update to a particular room/category counter. It is not established that every category advances through this method; some results may be calculated from other state.

```text
Relevant gameplay event
→ inventory, discovery, progression, or counter state changes
→ category's current-count result may change
→ map reporting can reflect the updated result
```

This is a practical model, not a verified ordering of every callback or UI refresh.

### What advances and what does not necessarily advance

- Opening/consuming a chest is separate from receiving its item. A consumed chest was observed leaving the item category at `0/1` when its original inventory reward was withheld.
- Receiving an unrelated replacement item is not established to satisfy the original room's item requirement.
- Owning a progression key is not universally equivalent to satisfying the progression category. The Clock Tower trap room reported `0/1` despite its disarmed-trap key being present.
- Secret discovery, exploration, and door discovery can contribute separate requirements. Collecting every visible item therefore does not establish room completion.
- Combined and district-level categories can overlap leaf categories. Summing every reported value can double-count progress.

### Exact formula remains unestablished

No verified universal rule such as `sum(current) / sum(total)` or `all categories satisfied = complete map color` is established here. The complete category-selection, aggregation, refresh, weighting, and color-decision logic is unknown. Treat a short leaf-category count as evidence of that category's unmet requirement, rather than assuming a complete global formula.


## Relevant game APIs

| API | Purpose |
|---|---|
| `CurrentRoomInstance.currentRoom.worldRoomData` | Current room metadata |
| `WorldRoomData.instanceName` | Runtime room identifier used in room lookups |
| `WorldRoomData.id` | Room ID |
| `CompletionItemType.GetRoomCurrentCompletionCount(room, entity)` | Current count for a category |
| `CompletionItemType.GetRoomTotalCompletionCount(room, entity)` | Required total for a category |
| `CompletionItemTypeItems` | Inventory/item-related completion category |
| `CompletionItemTypeProgressionKeys` | Progression-key-related completion category |
| `ExplorerComponent.IncrementCompletionCount` | Counter-increment path |
| `CompletionItemTypeCounter` | Counter category passed to increment calls |

Asset names and concrete type names are different: record both during diagnosis instead of assuming their spelling is interchangeable.

## Obtaining room and player context

Find a validated `CurrentRoomInstance`, then read:

```csharp
var room = roomInstance.currentRoom;
WorldRoomData worldRoom = room?.worldRoomData;
```

Obtain the active player's entity through `ExperienceComponent.linkedEntity`. Reacquire objects after a load or transition. If multiple room instances exist, identify the actual current room rather than assuming the first loaded object is correct.

Queries should run on Unity's main thread with valid room/entity context. Missing context is unknown completion, not a zero count.

## Interpreting the results

| Result | Interpretation |
|---|---|
| `current < total` | This category reports unmet completion |
| `current == total`, positive total | This category's count is satisfied |
| `0/0` | No reported contribution for this category/context |
| `current > total` | Possible duplicate counting or category-specific behavior |
| Query fails | Context/category is unavailable or incompatible; no conclusion |

Do not sum every category to derive a percentage. Combined/district assets can overlap underlying categories, and multiple assets can cover related state. Aggregate and leaf-category counts can overlap.

## Items and consumed pickup state

An opened chest or activated pickup does not necessarily satisfy `CompletionItemTypeItems`. In an observed room, a consumed chest whose original reward did not enter inventory left the item category at `0/1` while other categories were satisfied.

```text
Object used/opened state
≠ inventory reward delivery
≠ item completion count
```

When implementing replacement rewards, verify the room's category after suppressing an original inventory grant. Delivering an unrelated replacement item is not established to satisfy the original room requirement.

This observation does not establish every item category implementation.

## Secret zones, exploration, and doors

Completion assets can represent secret-zone discovery, exploration zones, and discovered doors. Their state can differ from inventory state and visible geometry.

A room with all visible pickups collected may still have a discovery requirement. Inspect the category first, then the room's relevant triggers and objects. Proximity to a secret or entering a nearby scene is not proof its discovery event fired.

Exact discovery APIs and complete per-room conditions are unconfirmed.

## Progression-key completion

`CompletionItemTypeProgressionKeys` can report a missing count even when a related key is present in player progression.

Known example:

| Property | Value |
|---|---|
| Room | `ClockTower_MVP_Room14_uid112` |
| Mechanism | Clock Tower Death Trap |
| Related key | `MC_Progression_DeathTrapDisarmed_ID` |
| Key GUID | `10f68e1e54b7d98498cd7ef384ba6e47` |
| Observed mismatch | Progression-key category `0/1` while the key was already owned |

The room scene references the key through trap/exit/interaction-related objects. The cause of the mismatch is not established. Do not assume that every progression category is a direct `HasProgressionKey` count or that setting its key automatically updates completion.

## Room, district, and global map totals

Rooms belong to map/world structures, and district-level completion can aggregate room state. Room-local counts do not by themselves establish a district or global percentage.

```text
Leaf room categories
→ room completion evaluation
→ district/combined reporting
→ map display and broader totals
```

This is an organizational model, not a verified formula. Check the relevant aggregate category and room/district mapping when a district percentage remains incomplete. Do not infer global 100% from one room's categories or from having visited every visible room.

## Boundaries

The exact global percentage formula, every category implementation, the complete color legend, a universal map-refresh API, and a reliable force-100%-completion operation are not established here. No single universal completion flag is established.




