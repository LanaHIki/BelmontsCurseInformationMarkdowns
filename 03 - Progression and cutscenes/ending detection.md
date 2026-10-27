# Ending detection

[Documentation index](../README.md)

## Detection point

Observe `CutscenePlayer.Begin` after it returns. The started player's `_cutsceneID` identifies the cutscene. Loaded `CutsceneData` assets expose `ID` and `name`, allowing the identifier to be resolved to a readable asset name.

```text
CutscenePlayer.Begin
→ read _cutsceneID
→ classify identifier or resolved CutsceneData.name
→ emit an ending-started event
```

This detects the beginning of the ending scene. It does not require waiting for credits, and does not prove that the scene finished or that completion was saved to disk.

## Ending identifiers

These identifiers have been observed in recorded gameplay:

| Ending | Cutscene asset name | Cutscene ID |
|---|---|---|
| Normal | `Master_Quarters_Dracula_Self_Phase_NormalEnding` | `3802bc32-bf30-45d3-aab0-8cd4499f1afa` |
| True | `Master_Quarters_Dracula_Self_Phase_TrueEnding` | `d35c4dbe-ffcb-462d-a225-ee37565c21be` |
| Bad | `Fake_MasterQuarter_BadEnding` | `03a10330-92d2-4763-952c-7a2a9b85f340` |

Asset identifiers and names are build-dependent; validate them if the game's content changes.

## Basic classifier

```csharp
enum EndingKind { None, Bad, Normal, True }

static EndingKind ClassifyEnding(string id, string name)
{
    if (id == "3802bc32-bf30-45d3-aab0-8cd4499f1afa"
        || name == "Master_Quarters_Dracula_Self_Phase_NormalEnding")
        return EndingKind.Normal;

    if (id == "d35c4dbe-ffcb-462d-a225-ee37565c21be"
        || name == "Master_Quarters_Dracula_Self_Phase_TrueEnding")
        return EndingKind.True;

    if (id == "03a10330-92d2-4763-952c-7a2a9b85f340"
        || name == "Fake_MasterQuarter_BadEnding")
        return EndingKind.Bad;

    return EndingKind.None;
}
```

The enum and classifier are example application code, not game APIs. Exact identifiers avoid confusing entrance or route-transition scenes with an ending.

## Resolving names

```csharp
static string FindCutsceneName(string id)
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

A name lookup can fail if the asset is not available in the loaded-object search. Classification by a known ID still works in that case. Cache successful ID/name mappings when observing many scenes; avoid scanning every frame.

## Observation example

```csharp
[HarmonyPostfix]
[HarmonyPatch(typeof(CutscenePlayer), nameof(CutscenePlayer.Begin))]
static void AfterCutsceneBegin(CutscenePlayer __instance)
{
    if (__instance == null) return;
    string id = __instance._cutsceneID;
    string name = FindCutsceneName(id);
    EndingKind ending = ClassifyEnding(id, name);
    if (ending == EndingKind.None) return;

    // Notify the consumer that this ending has started.
    // Deduplicate according to the consumer's session/save policy.
}
```

Observation does not require skipping the scene, changing watched-state queries, granting items, or setting progression flags.

## Preceding scenes are not completion

Normal and true endings can be preceded by:

| Scene | ID |
|---|---|
| `Master_Quarters_Dracula_Self_Phase_Two_Outro` | `fd057f30-f4a9-4145-afb0-475c40b1f1b6` |

The shared outro does not distinguish normal from true ending. Wait for the ending-specific scene.

The recorded bad-ending route includes:

```text
Castle_Clock_Tower_GoToBadEnding
→ Fake_Master_Quarter_Bad_Ending_Entrance
→ Fake_MasterQuarter_BadEnding
```

Intermediate gameplay/loading occurs between these markers. The first two markers are route/entrance scenes, not the final ending:

| Scene | ID |
|---|---|
| `Castle_Clock_Tower_GoToBadEnding` | `9bc1061f-18f5-4ca2-9e99-13bc5547121c` |
| `Fake_Master_Quarter_Bad_Ending_Entrance` | `846bce3a-307c-4f90-8a7c-7e3952fec84f` |

Do not classify every name containing `BadEnding` as completion. Exact final-scene matching is clearer than broad substring matching.

## Started versus finished versus persisted

| Observation | Meaning |
|---|---|
| Ending scene begins | The ending presentation has started |
| `CutsceneManager.OnCutscenePlayerFinished` for that player | The cutscene manager reports that playback finished |
| Ending-related state survives save/reload | Persistence has been separately verified |

When observing finish events, associate them with the same `CutscenePlayer` instance that began the ending. Finishing a different scene is not an ending-complete signal. Forced ending/skipping can also produce finish handling, so finish alone does not prove uninterrupted playback.

## Event deduplication

An ending can be encountered again after replay or reloading. Deduplication is a consumer policy:

- Per playback instance for scene telemetry.
- Per session for a one-time notification.
- Per save and ending type for a persistent completion tracker.

Do not assume an external record of an observed ending means the current game save contains completion state. No specific persisted ending flag or save-completion API is established here.


