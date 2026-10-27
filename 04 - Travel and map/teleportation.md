# Teleportation and custom spawn placement

[Documentation index](../README.md)

Teleportation moves the player to world-space coordinates. It is separate from selecting a mirror destination or unlocking a mirror.

| Operation | Game mechanism |
|---|---|
| Move inside a loaded level | `TestCommands.TeleportToPosition(x, y)` coroutine |
| Set placement during level loading | `MainCharacterSpawnLogic.SetForcePosition` and player-placement handling |
| Load a different level/entrance | `BiomeTransitor.StartBiomeTransition(playerEntity)`; documented in [mirror travel.md](mirror%20travel.md) |

## Required runtime objects

- Active `VaniaMovesComponent`: player component used to start the teleport coroutine.
- `ExperienceComponent.linkedEntity`: player entity, useful for checking readiness.
- `MainCharacterSpawnLogic`: owns placement during transitions and respawns.
- `CurrentRoomInstance.currentRoom`: current room and scene.
- `GameStateManager`: exposes `IsGameplayState()`.

Find loaded components with `Resources.FindObjectsOfTypeAll(Il2CppType.Of<T>())` and convert with `TryCast<T>()`. Revalidate or reacquire them after scene changes. Run changes on Unity's main thread.

## Teleporting inside a loaded level

```csharp
var routine = TestCommands.TeleportToPosition(targetX, targetY);
player.StartCoroutine(routine);
```

Here `player` is the active `VaniaMovesComponent` and coordinates are world-space X/Y. A small vertical offset can help avoid embedding the character in the floor, but must suit the destination geometry.

Assigning `player.transform.position` directly can be overwritten by the character movement or spawn systems. The teleport coroutine uses game-owned movement handling instead.

The coroutine is not a substitute for loading the correct level. Confirm that the intended destination scene/biome is present first.

## Broken mirrors and custom spawn placement

Some Hall of Mirrors destinations share a level entrance rather than owning a dedicated entrance.

```text
Resolve shared Hall of Mirrors level/entrance
→ choose target mirror coordinates
→ request forced spawn
→ start biome transition
→ preserve position through entrance setup and player placement
→ wait for stable gameplay
→ clear forced spawn
→ verify resulting position
```

Relevant APIs:

```csharp
spawnLogic.SetForcePosition(targetPosition, null);
spawnLogic.ClearForcePosition();
```

`MainCharacterSpawnLogic.SetupBiomeTransition` can reset entrance positioning. `MainCharacterSpawnLogic.PlaceMainCharacter` receives the target position used for placement. An interception can reapply the forced position after setup and override the placement argument when a custom destination is active:

```csharp
// Example prefix body for MainCharacterSpawnLogic.PlaceMainCharacter:
static void BeforePlacement(ref Vector3 targetPosition)
{
    if (requestedSpawnPosition.HasValue)
        targetPosition = requestedSpawnPosition.Value;
}
```

`requestedSpawnPosition` is application-managed state, not a game field. Keep it scoped to the intended transition. Clear the force after successful placement and on cancellation/failure; otherwise a later death or respawn can reuse the unwanted position.

## Readiness and arrival

For an in-level teleport, confirm that the destination level is already loaded, the active player exists, and gameplay is ready. A coroutine cannot load an arbitrary destination level merely from its X/Y coordinates.

```csharp
var manager = GameStateManager.GetInstance();
bool ready = manager != null && manager.IsGameplayState();
```

Wait for scene loading, spawn setup, and conflicting gameplay sequences to settle. Recheck the current room and player after a transition. Fixed delays alone are insufficient to prove readiness.

After teleportation or spawning, reread `player.transform.position` and compare it with the requested target. Account for any intentional vertical offset. Verify again after movement resumes to detect a position that was overwritten.

A transform assignment can be reversed by movement or spawn logic. Use the game's teleport or spawn mechanism appropriate to the operation. Clean up temporary forced positions on success, cancellation, and timeout so later deaths do not reuse an unintended destination.

## Coordinates and persistence

Coordinates are world-space X/Y values within a particular level. The same coordinates in another level describe a different place. Confirm destination geometry before placement; a vertical offset must suit the floor and nearby collision.

Moving the character does not itself unlock a mirror, activate a save point, or prove that a disk save occurred. Verify persistent location changes separately through normal save/reload behavior.


