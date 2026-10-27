# Level and biome transitions

[Documentation index](../README.md)

## Scope

The level/biome transition system loads a destination `LevelID` and places the player through a `BiomeTransitionID` entrance. Mirrors are one use of this system; the transition APIs are general game functionality.

These APIs do not establish how every ordinary adjacent room change works. Movement between rooms inside an already loaded level may use other systems.

## Core objects

| Game object | Role |
|---|---|
| `LevelID` | Destination level asset |
| `BiomeTransitionID` | Destination entrance/transition asset |
| `BiomeTransitor` | Starts and coordinates a transition |
| Player `Entity` | Character passed to the transition |
| `GameStateManager` | Reports gameplay readiness and active state |
| `MainCharacterSpawnLogic` | Sets up the entrance and places the player |
| `CurrentRoomInstance` | Exposes the loaded room |

The active player's `ExperienceComponent.linkedEntity` provides the player entity. Obtain `ExperienceComponent` from the player's parents or root's children. Validate it before use and reacquire objects after a load.

```text
Choose destination level and entrance
→ resolve LevelID and BiomeTransitionID assets
→ validate gameplay readiness
→ configure BiomeTransitor
→ StartBiomeTransition(playerEntity)
→ load and spawn
→ verify destination state
```

## Resolving destination assets

`LevelID`, `BiomeTransitionID`, and `ProgressionKeyID` derive from `SavableScriptableObject` and expose `guid`.

Resolve assets by searching loaded objects first:

```csharp
foreach (var obj in Resources.FindObjectsOfTypeAll(Il2CppType.Of<LevelID>()))
{
    var level = obj.TryCast<LevelID>();
    if (level != null && level.guid == destinationLevelGuid)
    {
        // Found the destination level asset.
    }
}
```

If an asset is not loaded, its GUID can be used with Addressables:

```csharp
var handle = Addressables.LoadAssetAsync<LevelID>(
    (Il2CppSystem.Object)(Il2CppSystem.String)destinationLevelGuid);
var level = handle.WaitForCompletion();
```

Use `BiomeTransitionID` for an entrance asset. `WaitForCompletion` is synchronous and may stall while loading; manage asset handle lifetimes when building a reusable loader. Validate assets before beginning travel.

## Starting a level/biome transition

```text
Validate player and gameplay readiness
→ resolve destination LevelID and BiomeTransitionID
→ configure BiomeTransitor
→ start transition with player entity
→ await destination gameplay readiness
→ verify room and player placement
```

A transitor can be attached to a dedicated object that survives scene loading:

```csharp
var host = new GameObject("LevelTransition");
UnityEngine.Object.DontDestroyOnLoad(host);
var transitor = host.AddComponent(Il2CppType.Of<BiomeTransitor>())
    .TryCast<BiomeTransitor>();

host.transform.position = player.transform.position;
transitor.enterPosition = host.transform;
transitor.levelToLoad = destinationLevel;
transitor.biomeTransition = destinationTransition;
transitor.saveBiomeTransitionForRetry = false;

transitor.StartBiomeTransition(playerEntity);
```

A real loaded `BiomeTransitor` provides compatible exit settings that can be copied before starting:

```csharp
transitor.enterActionTrigger = existingTransitor.enterActionTrigger;
transitor.enteringDirection = existingTransitor.enteringDirection;
transitor.disableInputsOnExit = existingTransitor.disableInputsOnExit;
```

These relate to exit action/input handling. Use a real game transitor as the template, rather than a previous temporary travel object. Clean up temporary objects and avoid retaining transition state across unrelated journeys. The snippet illustrates API usage, not a complete travel controller.

## Readiness and arrival

Before travel, require an active player entity and appropriate gameplay state:

```csharp
var manager = GameStateManager.GetInstance();
bool ready = manager != null && manager.IsGameplayState();
```

Defer travel while a map/menu, dialogue, cutscene, acquisition sequence, or load owns the relevant game flow. Check actual state rather than relying only on a remembered cutscene-start flag.

After travel, reacquire runtime objects. Check:

- `CurrentRoomInstance.currentRoom` and its scene name.
- Active player and `ExperienceComponent.linkedEntity`.
- Gameplay state.
- Player position relative to the target, when coordinates apply.

A transition call returning does not mean loading finished. Positioning too early can be overwritten. A brief stable-readiness interval and a later position check can detect this; fixed waits alone do not establish readiness. Add timeouts and cleanup to queued transitions.

## Spawn placement and cleanup

`MainCharacterSpawnLogic.SetupBiomeTransition` configures entrance placement, and `PlaceMainCharacter` performs player placement. `SetForcePosition(position, null)` and `ClearForcePosition()` support explicit forced placement.

An entrance setup can overwrite a position chosen too early. Direct transform changes can also be overwritten by movement/spawn logic. Custom spawn control must remain active through the relevant placement callbacks, then be cleared after stable gameplay. Detailed coordinate placement is documented in [teleportation.md](teleportation.md).

Use a fresh temporary transition object for a new manually initiated journey, or explicitly reset all state before reuse. Clean up failed/cancelled operations and temporary spawn overrides. Do not allow a second journey to overlap an unfinished transition.

## Persistent state and travel policy

Starting a transition is distinct from unlocking a destination, changing story flags, activating a save point, or writing a disk save. Destination eligibility is a separate policy from transition execution.

`saveBiomeTransitionForRetry` is an available transitor field. The example sets it to `false`; do not infer complete save/retry behavior from this field alone. Check persistence and death/retry behavior separately when implementing a travel feature.

Mirror-specific destination-used flags, the first Death travel lock, and destination mappings are documented in [mirror travel.md](mirror%20travel.md).



