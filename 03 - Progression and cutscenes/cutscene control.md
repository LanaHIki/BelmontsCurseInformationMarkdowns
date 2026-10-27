# Cutscene control

[Documentation index](../README.md)

## Core game types

| Type or member | Purpose |
|---|---|
| `CutsceneData.ID` | Cutscene identifier |
| `CutsceneData.name` | Readable asset name |
| `CutscenePlayer._cutsceneID` | Identifier associated with a playback instance |
| `CutscenePlayer.Begin()` | Starts playback |
| `CutscenePlayer.ForceEnd()` | Requests forced ending of that player |
| `CutsceneManager.OnCutscenePlayerFinished(player)` | Playback-finished notification |
| `CutsceneManager.HasWatchedCutscene(...)` | Watched-state query |
| `UnityEngine.Video.VideoPlayer` | Video playback component, when used by a sequence |

Control playback on Unity's main thread. Cutscene instances and assets may become unavailable across scene loads.

## Identifying the active sequence

Observe `CutscenePlayer.Begin` with a postfix and retain its instance and identifier:

```csharp
static CutscenePlayer activePlayer;
static string activeId;

[HarmonyPostfix]
[HarmonyPatch(typeof(CutscenePlayer), nameof(CutscenePlayer.Begin))]
static void AfterBegin(CutscenePlayer __instance)
{
    activePlayer = __instance;
    activeId = __instance?._cutsceneID;
}
```

The static fields here are example tracking state, not game fields. A single stored instance is a simplified observer; it does not establish that nested or overlapping playback is impossible.

Resolve a readable name through loaded `CutsceneData` assets:

```csharp
static string FindName(string id)
{
    if (string.IsNullOrEmpty(id)) return null;
    foreach (var obj in Resources.FindObjectsOfTypeAll(Il2CppType.Of<CutsceneData>()))
    {
        var data = obj.TryCast<CutsceneData>();
        if (data != null && data.ID == id)
            return data.name;
    }
    return null;
}
```

Keep the ID when a readable name cannot be resolved. Missing loaded data does not prove the cutscene does not exist. Cache successful mappings instead of repeatedly searching every frame.

## Tracking the end of playback

Observe the matching finish notification:

```csharp
[HarmonyPostfix]
[HarmonyPatch(typeof(CutsceneManager),
    nameof(CutsceneManager.OnCutscenePlayerFinished))]
static void AfterFinished(CutscenePlayer player)
{
    if (activePlayer != null && player != null
        && player.Pointer == activePlayer.Pointer)
    {
        activePlayer = null;
        activeId = null;
    }
}
```

Comparing native pointers associates the callback with the tracked IL2CPP instance. Clear stale tracking on lifecycle changes as needed. A missing finish callback can leave observer state stale, so a cached non-null player is not a reliable universal gameplay-busy flag. Use `GameStateManager.IsGameplayState()` when deciding whether unrelated gameplay operations may run.

## Watched-state queries

Two query overloads are available:

```csharp
CutsceneManager.HasWatchedCutscene(CutsceneData cutscene)
CutsceneManager.HasWatchedCutscene(string cutsceneID)
```

They return whether the scene is considered watched. Some repeat-skippable sequences consult these queries before playing.

A postfix can return `true` for a specifically selected sequence:

```csharp
[HarmonyPostfix]
[HarmonyPatch(typeof(CutsceneManager), nameof(CutsceneManager.HasWatchedCutscene),
    new[] { typeof(CutsceneData) })]
static void OverrideWatched(CutsceneData cutscene, ref bool __result)
{
    if (__result || cutscene == null) return;
    if (ShouldSkip(cutscene.ID, cutscene.name))
        __result = true;
}
```

`ShouldSkip` is an application-defined selection function. A corresponding observer can handle the string overload when necessary.

```text
Game asks whether scene was watched
→ selected query returns true
→ caller may follow its repeat/skip branch
```

This changes a query result; it does not itself write the saved watched list. It only affects paths that consult that query. It is not a universal way to stop all cutscenes, nor a guarantee that rewards and story effects will still occur.

## Force-ending current playback

```csharp
if (activePlayer != null)
    activePlayer.ForceEnd();
```

```text
Scene begins
→ capture CutscenePlayer
→ wait for an appropriate point
→ ForceEnd
→ observe playback/state cleanup
```

Force-ending a running sequence differs from avoiding playback through watched-state queries. Effects already executed may remain, while later effects may not occur. A force-end call is not proof that every reward, progression update, input lock, audio state, and cleanup operation completed as intended.

Starting, ending, and persistence are separate. Ending recognition is documented in [ending detection.md](ending%20detection.md).

## Video playback handling

A sequence can contain a video followed by dialogue or other cutscene content. Detect active Unity video playback with:

```csharp
static bool AnyVideoPlaying()
{
    foreach (var obj in Resources.FindObjectsOfTypeAll(
                 Il2CppType.Of<UnityEngine.Video.VideoPlayer>()))
    {
        var video = obj.TryCast<UnityEngine.Video.VideoPlayer>();
        if (video != null && video.isActiveAndEnabled && video.isPlaying)
            return true;
    }
    return false;
}
```

The opening sequence `Trevor_Rose_Arrive` includes a video before its text portion. Force-ending during that video can leave sound effects muted. Waiting until video playback finishes avoids ending it at that particular point, but does not establish universal skip safety.

This scan detects any active Unity `VideoPlayer`, not specifically the current scene's video. It may also miss playback performed by another subsystem. When possible, identify the actual video associated with the sequence.

## Deferred automatic ending

A reusable controller can schedule a force-end for an explicitly selected sequence:

```text
Observe Begin
→ store exact player instance and cutscene ID
→ schedule a later readiness check
→ wait while relevant video is playing
→ confirm same scene is still active
→ ForceEnd
→ clear pending request
```

Bind pending work to the intended playback instance. Cancel it when that instance finishes, is destroyed, or is replaced; a timer must not accidentally end a later unrelated cutscene. Use bounded retries/timeouts and verify gameplay recovery. A fixed delay alone is not evidence of safety.

## Reward and story side effects

Cutscenes can grant items, teach interactions, set character identity flags, and change encounter state.

Known examples:

| Sequence or effect | Implication |
|---|---|
| `Street_Poor_Justice_Louis_Intro` | Includes The Judge's Arcana absorption; its name containing `Intro` does not make it cosmetic |
| `Fool_Boss_Door` | Gives a clue even though its name does not contain `Clue` |
| Character introductions | Can set `MC_CharacterDataModificator_*` name/title keys; skipped effects can leave `???` labels |
| Ability reward sequences | May be omitted when the ability is already owned |

Prefer explicit scene-ID/name selections whose behavior has been checked. Name patterns such as `Intro`, `Arrive`, or `Vista` are discovery hints, not proof that a scene has no gameplay effects.

If an intended effect must occur independently of playback, identify and verify its actual inventory, ability, or progression operation. Do not assume that all effects can be reproduced by marking a scene watched.



