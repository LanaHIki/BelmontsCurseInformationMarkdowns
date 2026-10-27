# Progression flags

[Documentation index](../README.md)

## Representation

Progression flags are `ProgressionKeyID` assets stored in the player's `ProgressionComponent`. A key has an asset `name` and a persistent `guid`. Possession of a key can control routes, encounters, mirror access, character reveals, and room completion.

```text
Player entity → ProgressionComponent → progressionKeys → ProgressionKeyID assets
```

A progression-key GUID, an inventory `itemID`, a world object's save ID, and a `CharacterID` GUID identify different things. Do not substitute one for another.

## Finding the progression component

Obtain the active player's entity through `ExperienceComponent.linkedEntity`. Then look up the progression data:

```csharp
var progression = entity.GetComponentData(Il2CppType.Of<ProgressionComponent>())
    ?.TryCast<ProgressionComponent>()
    ?? entity.GetComponent(Il2CppType.Of<ProgressionComponent>())
    ?.TryCast<ProgressionComponent>();
```

A child-component search is another possible fallback. Reacquire the entity and component after loads or transitions; missing data means the player is not ready for inspection or mutation.

## Reading flags

```csharp
foreach (var key in progression.progressionKeys)
{
    if (key == null) continue;
    string assetName = key.name;
    string guid = key.guid;
}
```

With a resolved key asset:

```csharp
bool present = progression.HasProgressionKey(key);
```

Check that the progression component, key collection, and target key exist. A failed lookup is not evidence that the flag is absent.

## Resolving key assets

Search loaded assets by GUID or exact asset name:

```csharp
ProgressionKeyID FindLoadedKey(string guid)
{
    foreach (var obj in Resources.FindObjectsOfTypeAll(Il2CppType.Of<ProgressionKeyID>()))
    {
        var key = obj.TryCast<ProgressionKeyID>();
        if (key != null && key.guid == guid)
            return key;
    }
    return null;
}
```

When the asset is not already loaded, its GUID can be resolved through Addressables:

```csharp
var handle = Addressables.LoadAssetAsync<ProgressionKeyID>(
    (Il2CppSystem.Object)(Il2CppSystem.String)guid);
var key = handle.WaitForCompletion();
```

The synchronous wait can stall during loading. Manage asset-handle lifetimes and validate the result. GUIDs and names can vary across game builds.

## Setting and removing flags

```csharp
if (!progression.HasProgressionKey(key))
    progression.SetProgressionKey(key);

// Remove only when intended:
if (progression.HasProgressionKey(key))
    progression.UnsetProgressionKey(key);
```

Use the game's methods rather than directly editing the collection. Perform mutations on Unity's main thread with valid runtime objects. A flag change can have multiple gameplay effects; it is not necessarily equivalent to completing the event that normally sets it.

## Observing changes

`ProgressionComponent.SetProgressionKey` provides an observation point:

```csharp
[HarmonyPostfix]
[HarmonyPatch(typeof(ProgressionComponent), nameof(ProgressionComponent.SetProgressionKey))]
static void AfterKeySet(ProgressionKeyID progressionKey)
{
    if (progressionKey == null) return;
    // Inspect progressionKey.name and progressionKey.guid.
}
```

This reports that the setter returned, not necessarily that a previously absent key was newly acquired. The key may already have existed. Some changes, including save restoration and certain scripted paths, may bypass the observed setter. Compare snapshots or poll `HasProgressionKey` when complete state tracking is required.

`ProgressionKeyHolder.SetProgressionKey` is another event-path observation point. It does not replace reading the player's actual progression state.

If observing both setters and polling, deduplicate events. If an observer itself sets keys, guard against recursive handling.

## Known gameplay keys

| Purpose | Asset name or family | GUID / effect |
|---|---|---|
| Eleanor character key | `MC_CharacterDataModificator_Eleanor` | `6e518ea36b23065439b86276732c7471`; separate from the mirror-travel lock |
| First Shadow confrontation | `DeathCatacombs_Defeated_ID` | `769b38ad62fa14249b54a74ef889998c`; gates mirror travel, opens the east Streets route, and changes confrontation-area state |
| Clock Tower Death Trap | `MC_Progression_DeathTrapDisarmed_ID` | `10f68e1e54b7d98498cd7ef384ba6e47`; represents disarming the trap associated with the normal/true-ending route |
| Joan defeated | Boss-defeat key | `641325fec3360bb459c879275fdee639`; relevant to Joan's Catacombs gem slot |
| Medusa defeated | `Medusa_Defeated_ID` | `ac985fb3574b3cc4eadaf3a79793112e`; relevant to Medusa's Catacombs gem slot |
| Used save mirrors | `MC_Used_SaveRoom_*` | Individual destination keys; separate from the global first Death travel lock |
| Character name/title reveal | `MC_CharacterDataModificator_*` | Reveals character identity; some keys also unlock functionality |

### Eleanor

The Eleanor key is not the mirror-travel lock. Mirror travel uses the first Death key (`DeathCatacombs_Defeated_ID`), while each used-mirror key records an individual destination. Directly launching a level transition does not itself grant those prerequisites. Detailed access and destination mappings are in [mirror travel.md](../04%20-%20Travel%20and%20map/mirror%20travel.md).

### First Shadow confrontation

The flag can be updated by a scripted path without passing through the ordinary setter observation point. Polling can detect its presence.

Setting the flag early can prevent the encounter from starting. Removing it can affect the castle route. Treat encounter availability and traversal access as separate consequences of the same key.

### Clock Tower trap

The trap key is associated with `ClockTower_MVP_Room14_uid112`. Soul Harvest is associated with disarming the trap during normal gameplay. Possessing its key is not proof that every related room-completion or mechanism state has refreshed.

## Character name and title flags

The following suffixes belong to the `MC_CharacterDataModificator_` asset-name family:

| Character | Name/title suffixes |
|---|---|
| Leon | `Leon`, `TheFallen` |
| Joan | `Joan`, `TheWarrior` |
| Medusa | `Medusa`, `TheProphetess` |
| Carmilla | `Carmilla` |
| Talos | `Talos`, `TheBuilder` |
| Ferryman | `Ferryman` |
| Mammon | `Mammon`, `TheMiser` |
| Saint Louis | `SaintLouis`, `TheJudge` |
| Arthur | `Arthur`, `TheBrittleKing` |
| Isaac | `Isaac` |
| The Self | `Doppelganger` |
| Succubus | `Succubus` |
| Brauner | `Brauner`, `TheArtist` |
| Death | `Death`, `TheShadow` |

For example, `MC_CharacterDataModificator_Leon` and `MC_CharacterDataModificator_TheFallen` provide name/title identity. Introductory cutscenes can set identity flags; skipping those sequences can leave names displayed as `???` unless their intended flag changes occur separately.

Do not infer the mirror-travel lock from the Eleanor character key; use the first Death progression key.

## Character progression steps are separate

Boss state can involve `CharacterProgressionComponent` and a `CharacterID` asset in addition to progression keys.

```csharp
var step = characters.GetProgressionStep(character);
characters.SetProgressionStep(
    character, CharacterProgressionStep.Absorbed, false, true);
```

`CharacterProgressionStep.Absorbed` has observed integer value 4; `Hinted` has observed value 2. Prefer enum names. The full semantics of the two boolean arguments are unconfirmed.

| Character | CharacterID GUID | Related inventory key ID |
|---|---|---|
| Joan | `c780f2bb5d95b6d4e88196839e50ecb9` | `CatacombsGateKeyJeanne` |
| Medusa | `4e87b57ded43fb44c99aae5c306ed768` | `CatacombsGateKeyMedusa` |

The Catacombs three-gem gate checks more than inventory ownership. Boss-defeat state and character progression can also matter. A defeat key alone is insufficient: the corresponding character step also needs to reach Absorbed for the tested gem-slot interaction.

This illustrates why granting an item, setting a key, advancing a character step, and completing a world mechanism must be documented separately.

## Three-gem Catacombs door

### Location and game objects

The door is `GPE_ArcanaGates` in The Fallen's boss room. Relevant room identifiers are:

- Door room: scene prefix `StreetsPoor_Room_Boss_Leon_MainRoom`.
- Adjacent approach room: `StreetsPoorV2_Room49_uid143`.

Check the full runtime scene name, allowing for its instance suffix where applicable. Preparing state before entering the door room can matter if the interaction evaluates prerequisites during room initialization; the exact refresh timing of every slot is not established.

### Three distinct state layers

```text
Inventory contains the corresponding gem key
+ boss/character progression permits its placement
→ player places key in its slot
→ placed-slot state changes
→ completed gate mechanism opens the route
```

Inventory ownership, character progression, and the already-placed state are distinct. Giving all three inventory items alone is insufficient to guarantee that all slots accept them.

| Slot | Inventory item ID | Defeat key | CharacterID GUID |
|---|---|---|---|
| The Fallen / Leon | `CatacombsGateKeyLeon` | `Leon_Defeated_ID`; GUID not established here | Not established here |
| The Warrior / Joan | `CatacombsGateKeyJeanne` | `Jeanne_Defeated_ID`, `641325fec3360bb459c879275fdee639` | `c780f2bb5d95b6d4e88196839e50ecb9` |
| Medusa | `CatacombsGateKeyMedusa` | `Medusa_Defeated_ID`, `ac985fb3574b3cc4eadaf3a79793112e` | `4e87b57ded43fb44c99aae5c306ed768` |

The item IDs are exact internal strings. Display titles are descriptive; use the IDs for lookup.

### Character step requirement

A recorded placement test found that setting Joan's defeat key alone did not allow the slot to accept her inventory key. Her character state was `Hinted` (2), whereas completed bosses were `Absorbed` (4). Raising the corresponding character step to `CharacterProgressionStep.Absorbed`, alongside the defeat-key preparation, allowed placement in the subsequent test.

This establishes that the character step matters. It does not isolate every individual gate condition or prove that the defeat key is independently required once the step is Absorbed.

### Inspecting prerequisites

```csharp
// Resolve these assets first:
// gemItem: ItemsManager.FindItem("CatacombsGateKeyJeanne")
// boss: CharacterID with GUID c780f2bb5d95b6d4e88196839e50ecb9
// defeatedKey: ProgressionKeyID with GUID 641325fec3360bb459c879275fdee639

bool ownsKey = inventory.HasItemInInventoryOrEquipped(gemItem);
bool defeatedFlagPresent = progression.HasProgressionKey(defeatedKey);
CharacterProgressionStep step = characters.GetProgressionStep(boss);

// Inspect all three rather than treating ownsKey as sufficient.
```

Obtain `CharacterProgressionComponent` from the player entity through `GetComponentData`, `GetComponent`, or an appropriate child lookup. Resolve `CharacterID` and `ProgressionKeyID` assets by their GUIDs. Check all objects for availability before calling the methods.

### Preparing character state explicitly

The following illustrates the known API calls, not a complete door-opening function:

```csharp
bool hadDefeatKey = progression.HasProgressionKey(defeatedKey);
CharacterProgressionStep previousStep = characters.GetProgressionStep(boss);

if (!hadDefeatKey)
    progression.SetProgressionKey(defeatedKey);

if (previousStep < CharacterProgressionStep.Absorbed)
    characters.SetProgressionStep(
        boss, CharacterProgressionStep.Absorbed, false, true);

// Allow the normal slot interaction to run after state is ready.
// These calls do not themselves place the gem or open the door.
```

The boolean parameter meanings are not established here. Do not assume these calls only affect the door: changing boss progression can affect other dialogue, rewards, encounters, and story behavior.

For a temporary override, retain the original key presence and character step. Restore only changes owned by that override, and avoid reverting genuine progression that occurred in the meantime. Reacquire components and handle interrupted loads/restarts before attempting cleanup.

### Placed keys and opening the gate

Placed-slot state is separate from possession of the item. Progression data can include boss-associated placed-key entries, but exact placed-key names/GUIDs and the gate's complete open-state implementation are not established here.

Do not invent a placed-key identifier or assume that removing an inventory item means its gem was placed. Observe actual slot interaction, inspect the resulting persistent state, and confirm that the door opens. Use normal placement once prerequisites are ready unless the underlying placement API has been separately identified.

## Temporary changes and persistence

A temporary flag override should preserve the original state:

```text
Capture original presence and related character step
→ apply intended temporary override
→ perform operation
→ restore only the state changed by that override
```

Do not remove a key the player already legitimately owned. Account for genuine progression occurring while an override is active, and avoid rolling it back. Cleanup should also consider interrupted loads, deaths, and restarts.

Runtime changes participate in game state but do not prove an immediate disk save. Verify through the normal save/load cycle and reacquire components before comparing restored flags. An external record of a flag is separate from the game save.





