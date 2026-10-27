# EXP statues

[Documentation index](../README.md)

## Mechanics

Known statue object name: `GPE_DestructibleSourceExperience(Clone)`.

```text
Statue broken / activated
→ persistent destruction state
→ experience reward / orbs
```

The activation and save ID provide a collection-detection point. Detecting or recording destruction does not suppress experience. Replacing the XP itself requires identifying and intercepting its experience/orb-producing path; the exact suppression call is not established here.

## Persistent source identity

`SavableEntityBase` provides persistent object identifiers:

- Use `Id` when present; otherwise inspect `guid`.
- The savable component can be on a parent or child of the reward-giving component.
- `component.gameObject.scene.name` identifies the scene.
- Object names and hierarchy paths help distinguish reward-giving children from nearby state machines.

A scene name alone is not a unique pickup identifier. Multiple sources can exist in one room.

## Persistence

Collection, original benefit delivery, and replacement delivery are separate events. Keep the world's consumed/opened state while suppressing only the intended reward effect. A second observation of the same source must not automatically grant a second reward.

On save/load, reacquire runtime objects and distinguish restored activation from new interaction. Verify both the consumed state and actual inventory/stat/XP state through normal save/reload. Do not assume an immediate disk save after an interaction.

The catalog below contains available source identifiers, not a guarantee of complete coverage or identical content in every game build. Descriptive labels are organizational labels; use actual save IDs, scene context, and game item IDs for identification. Unknown fields remain blank rather than being inferred.

## Example: detect a known statue activation

This example observes one known statue using its persistent save ID. Add other statue IDs from the catalog to cover more sources. Identification uses save IDs rather than object-name guesses.

The prefix captures whether a new non-silent activation is being attempted. The postfix checks that the object became activated before notifying the consumer. It leaves the original activation and XP behavior unchanged.

```csharp
using System;
using System.Collections.Generic;
using HarmonyLib;
using Il2CppInterop.Runtime;
using UnityEngine;

[HarmonyPatch(typeof(ActivableEntity), "ActivateInternal")]
internal static class StatueActivationObserver
{
    private static readonly HashSet<string> StatueIds = new(StringComparer.Ordinal)
    {
        // Prison Tower EXP Statue 1
        "53987f65-dfec-4448-9df5-79b483a5cca7",
    };

    // Example consumer notification: save ID and scene name.
    internal static event Action<string, string> StatueActivated;

    private sealed class PendingActivation
    {
        internal string SaveId;
        internal string Scene;
    }

    [HarmonyPrefix]
    private static void BeforeActivation(
        ActivableEntity __instance, bool silent,
        out PendingActivation __state)
    {
        __state = null;
        if (__instance == null || silent || __instance.activated)
            return;

        string id = FindSaveId(__instance);
        if (id == null || !StatueIds.Contains(id))
            return;

        __state = new PendingActivation
        {
            SaveId = id,
            Scene = __instance.gameObject.scene.name,
        };
    }

    [HarmonyPostfix]
    private static void AfterActivation(
        ActivableEntity __instance, PendingActivation __state)
    {
        if (__state == null || __instance == null || !__instance.activated)
            return;

        Debug.Log($"Statue activated: {__state.SaveId} in {__state.Scene}");
        StatueActivated?.Invoke(__state.SaveId, __state.Scene);
    }

    private static string FindSaveId(Component component)
    {
        var savable = component.GetComponentInParent(
                Il2CppType.Of<SavableEntityBase>(), true)
            ?.TryCast<SavableEntityBase>()
            ?? component.GetComponentInChildren(
                Il2CppType.Of<SavableEntityBase>(), true)
            ?.TryCast<SavableEntityBase>();

        if (savable == null) return null;
        return string.IsNullOrEmpty(savable.Id) ? savable.guid : savable.Id;
    }
}
```

Install the patch once from the plugin's initialization using:

```csharp
Harmony.CreateAndPatchAll(
    typeof(StatueActivationObserver), "example.statue.activation");
```

This is illustrative code, not a runtime-tested standalone plugin. It assumes `activated` is set before `ActivateInternal` returns; verify that timing for the relevant game build. A deferred activation would require a later state check. Handle destroyed Unity objects and exceptions appropriately in a production observer.

`StatueActivated` reports activation, not the amount of XP received. An event consumer can record the source or schedule another reward, but this example does not suppress XP, write a save, or implement persistent reward deduplication. Scope any completion records to the intended game save/run so replaying another save is handled correctly. Other patches can also alter activation behavior, so validate the hook alongside them.

## Source catalog

| Source label | Room | Object | Save ID | Original item ID |
|---|---|---|---|---|
| Streets of Paris - Prison Tower EXP Statue 1 | StreetsPoorV2_Room01_uid82 | GPE_DestructibleSourceExperience(Clone) | 53987f65-dfec-4448-9df5-79b483a5cca7 |  |
| Streets of Paris - Prison Tower EXP Statue 2 | StreetsPoorV2_Room01_uid82 | GPE_DestructibleSourceExperience(Clone) | 66006d2c-04c6-4e36-af87-73a3dfb4eb1e |  |
| Streets of Paris - Prison Tower EXP Statue 3 | StreetsPoorV2_Room01_uid82 | GPE_DestructibleSourceExperience(Clone) | 923fdc1e-7292-4d55-8003-e2711e800545 |  |
| Streets of Paris - Prison Tower EXP Statue 4 | StreetsPoorV2_Room01_uid82 | GPE_DestructibleSourceExperience(Clone) | e942fa5d-0488-4e0d-ae1d-4e45ec8178a8 |  |
| Streets of Paris - Prison Tower EXP Statue 5 | StreetsPoorV2_Room01_uid82 | GPE_DestructibleSourceExperience(Clone) | ea0097ee-9e6b-4c0a-8c8d-7bb9c1ff7e1b |  |
| Streets of Paris - Court of Miracles EXP Statue 1 | StreetsPoorV2_Room02_uid84 | GPE_DestructibleSourceExperience(Clone) | 555a7663-4483-4a3a-872a-fdb8a8b9f0f1 |  |
| Streets of Paris - Court of Miracles EXP Statue 2 | StreetsPoorV2_Room02_uid84 | GPE_DestructibleSourceExperience(Clone) | c716add1-a38b-4073-a73d-0fc52920a134 |  |
| Streets of Paris - Court of Miracles EXP Statue 3 | StreetsPoorV2_Room02_uid84 | GPE_DestructibleSourceExperience(Clone) | f5907889-5ef6-46ce-a6db-c4a503f1a15b |  |
| Streets of Paris - Sewer Reservoir EXP Statue 1 | StreetsPoorV2_Room03_uid81 | GPE_DestructibleSourceExperience(Clone) | 9f7bce16-12ef-420b-8a8b-dfe270bd30ad |  |
| Streets of Paris - Bridge of the Millers EXP Statue | StreetsPoorV2_Room05_uid85 | GPE_DestructibleSourceExperience(Clone) | c7be09c2-6b68-4564-85c9-42a19c7b2969 |  |
| Streets of Paris - Burned District EXP Statue 1 | StreetsPoorV2_Room07_uid88 | GPE_DestructibleSourceExperience(Clone) | 597663e3-c1d0-472c-b261-ddbc7ec518a5 |  |
| Streets of Paris - Street of the Tomb Masons EXP Statue 1 | StreetsPoorV2_Room10_uid92 | GPE_DestructibleSourceExperience(Clone) | 0a7125ae-2542-4503-b568-2f80871fd596 |  |
| Streets of Paris - Street of the Tomb Masons EXP Statue 2 | StreetsPoorV2_Room10_uid92 | GPE_DestructibleSourceExperience(Clone) | 57179e39-6eae-4e89-8129-bbb55904bc8c |  |
| Streets of Paris - Street of the Tomb Masons EXP Statue 3 | StreetsPoorV2_Room10_uid92 | GPE_DestructibleSourceExperience(Clone) | d0d3670d-635a-4acf-adca-78710a11c803 |  |
| Streets of Paris - Street of the Tomb Masons EXP Statue 4 | StreetsPoorV2_Room10_uid92 | GPE_DestructibleSourceExperience(Clone) | d2ddc58a-78fa-484d-9930-840420c97ee8 |  |
| Streets of Paris - Upper Tower EXP Statue | StreetsPoorV2_Room14_uid97 | GPE_DestructibleSourceExperience(Clone) | f4c9968b-b24c-4c79-9aa7-35d66c4d63d4 |  |
| Streets of Paris - Burned District EXP Statue 2 | StreetsPoorV2_Room21_uid108 | GPE_DestructibleSourceExperience(Clone) | 8575c536-0cd0-4ab4-be74-f54737b82a1c |  |
| Streets of Paris - Sewer Reservoir EXP Statue 2 | StreetsPoorV2_Room26_uid118 | GPE_DestructibleSourceExperience(Clone) | 18e72541-2cf2-4a2a-920f-6886b485c5ec |  |
| Streets of Paris - Path of the Rat EXP Statue | StreetsPoorV2_Room30_uid122 | GPE_DestructibleSourceExperience(Clone) | 674c83b8-32f8-4a42-980a-70dbed320359 |  |
| Streets of Paris - Abandoned Sewers EXP Statue 1 | StreetsPoorV2_Room35_uid128 | GPE_DestructibleSourceExperience(Clone) | 4fb8f0a4-e8c3-4525-8ad3-25b0823c2252 |  |
| Streets of Paris - Abandoned Sewers EXP Statue 2 | StreetsPoorV2_Room36_uid129 | GPE_DestructibleSourceExperience(Clone) | 5f56882c-cb56-44f0-9b4e-a3fd3c0037a8 |  |
| Streets of Paris - Guard Level EXP Statue | StreetsPoorV2_Room37_uid130 | GPE_DestructibleSourceExperience(Clone) | aba6f28f-451d-40cb-902a-6eda424f6f80 |  |
| Streets of Paris - Undercity EXP Statue | StreetsPoorV2_Room40_uid133 | GPE_DestructibleSourceExperience(Clone) | 96b2d923-2b89-43c4-bd54-bf9ee80b3261 |  |
| Streets of Paris - Central Sewer Drain EXP Statue | StreetsPoorV2_Room41_uid134 | GPE_DestructibleSourceExperience(Clone) | e23b0855-d836-4c5c-a175-d4482daa6385 |  |
| Streets of Paris - Ruined Market EXP Statue | StreetsPoorV2_Room42_uid135 | GPE_DestructibleSourceExperience(Clone) | 4d7e5328-0822-454e-8b0e-9cd1faaf9e06 |  |
| Streets of Paris - Seer's Hovel EXP Statue | StreetsPoor_RenderTarget_06_uid104 | GPE_DestructibleSourceExperience(Clone) | cb20a780-92e2-465f-86a1-5d424514a12d |  |
| Streets of Paris - Morgue EXP Statue 1 | StreetsPoor_Room21_uid59 | GPE_DestructibleSourceExperience(Clone) | 08a4bd18-a5e6-49fa-90b6-02f33b1a2ceb |  |
| Streets of Paris - Morgue EXP Statue 2 | StreetsPoor_Room21_uid59 | GPE_DestructibleSourceExperience(Clone) | ee22f04f-200f-4135-b95e-c2af72df262e |  |
| Innocents' Cemetery - Fountain of the Innocents EXP Statue | Cemetery2_Room19_uid151 | GPE_DestructibleSourceExperience(Clone) | 1c7d9a8c-1f25-41a0-9718-6ebb56f05f59 |  |
| Innocents' Cemetery - Pauper's Grave EXP Statue | Cemetery2_Room22_uid110 | GPE_DestructibleSourceExperience(Clone) | df9e1cd9-cdac-4e61-bab2-2b86f03cf31a |  |
| Innocents' Cemetery - Ruins Entrance EXP Statue | Cemetery2_Room23_uid111 | GPE_DestructibleSourceExperience(Clone) | c783f148-c2b7-43aa-aaef-53d4c8217443 |  |
| Innocents' Cemetery - Roman Baths EXP Statue 1 | Cemetery2_Room25_uid113 | GPE_DestructibleSourceExperience(Clone) | 0882b8a8-50ee-44ba-aec8-eccb793914ef |  |
| Innocents' Cemetery - Roman Baths EXP Statue 2 | Cemetery2_Room25_uid113 | GPE_DestructibleSourceExperience(Clone) | 5ca4acbe-a733-420e-99fb-11917589f355 |  |
| Innocents' Cemetery - Hot Springs EXP Statue | Cemetery2_Room28_uid116 | GPE_DestructibleSourceExperience(Clone) | fcbd9e89-630f-4c8a-a592-0cd8d211b564 |  |
| Innocents' Cemetery - Serpents' Lair EXP Statue | Cemetery2_Room30_uid118 | GPE_DestructibleSourceExperience(Clone) | 93368305-b607-4886-88d9-26f34295eb79 |  |
| Innocents' Cemetery - Memorial for the Dead EXP Statue 1 | Cemetery2_Room31_uid119 | GPE_DestructibleSourceExperience(Clone) | a6ca0548-2c4e-4d8e-8bdf-14266aa03f6c |  |
| Innocents' Cemetery - Memorial for the Dead EXP Statue 2 | Cemetery2_Room31_uid119 | GPE_DestructibleSourceExperience(Clone) | d0af476f-d8f4-4b84-9b3a-df73ade3037a |  |
| Innocents' Cemetery - Memorial for the Dead EXP Statue 3 | Cemetery2_Room31_uid119 | GPE_DestructibleSourceExperience(Clone) | e59e4117-9dc1-4d0b-b1f9-284293ac4903 |  |
| Innocents' Cemetery - Cemetery Entrance EXP Statue | Cemetery2_Room35_uid125 | GPE_DestructibleSourceExperience(Clone) | 40d2d03b-4733-4a5b-ae46-17905328a223 |  |
| Innocents' Cemetery - Sepulcher of the Knight EXP Statue | Cemetery2_Room36_uid126 | GPE_DestructibleSourceExperience(Clone) | fc2e14c8-2619-43bc-bf34-55ed1f2f1744 |  |
| Innocents' Cemetery - Ancient Sewers EXP Statue | Cemetery2_Room37_uid127 | GPE_DestructibleSourceExperience(Clone) | c4f63fa9-38df-466f-b591-42b101994499 |  |
| Innocents' Cemetery - Overgrown Crypt EXP Statue 1 | Cemetery_MVP_Room_160_uid148 | GPE_DestructibleSourceExperience(Clone) | 4db4561a-6517-4431-9890-1791416f5434 |  |
| Innocents' Cemetery - Overgrown Crypt EXP Statue 2 | Cemetery_MVP_Room_160_uid148 | GPE_DestructibleSourceExperience(Clone) | 6875b388-73cc-4ee4-bbf5-1bd490a16169 |  |
| Catacombs - Gallery of Teeth EXP Statue 1 | Catacombs_MVP_Room02_uid109 | GPE_DestructibleSourceExperience(Clone) | c9d5b27a-4b01-495e-ba1b-dddf35b77d7b |  |
| Catacombs - Gallery of Teeth EXP Statue 2 | Catacombs_MVP_Room02_uid109 | GPE_DestructibleSourceExperience(Clone) | fec16fee-8373-4765-9755-8309cb7d6b3e |  |
| Catacombs - Chamber of Whispers EXP Statue | Catacombs_MVP_Room04_uid111 | GPE_DestructibleSourceExperience(Clone) | 296840f2-86d0-4b60-91d3-a7cc478e3a50 |  |
| Catacombs - Blood Pits EXP Statue 1 | Catacombs_MVP_Room05_uid113 | GPE_DestructibleSourceExperience(Clone) | 27d2c4c6-8b7a-4e9f-a752-3d0f3a1f2ef6 |  |
| Catacombs - Blood Pits EXP Statue 2 | Catacombs_MVP_Room05_uid113 | GPE_DestructibleSourceExperience(Clone) | 7e22b906-9d6f-449f-baee-9c52eb1b3fba |  |
| Catacombs - Jaws of Limbo EXP Statue | Catacombs_MVP_Room17_uid126 | GPE_DestructibleSourceExperience(Clone) | 3981d690-d60e-4807-bc2b-c86df65c236e |  |
| Catacombs - Catacomb Entrance EXP Statue 1 | Catacombs_MVP_Room18_uid127 | GPE_DestructibleSourceExperience(Clone) | b8adc7fe-1d9e-4ab1-b8bd-6dce087a87d8 |  |
| Catacombs - Catacomb Entrance EXP Statue 2 | Catacombs_MVP_Room19_uid128 | GPE_DestructibleSourceExperience(Clone) | 6efee410-2503-46b5-8e00-e4fb781f9b95 |  |
| Catacombs - Catacomb Entrance EXP Statue 3 | Catacombs_MVP_Room19_uid128 | GPE_DestructibleSourceExperience(Clone) | e711fa33-c2a1-414f-9c01-e64480e0fd17 |  |
| Catacombs - Charnel Galleries EXP Statue | Catacombs_MVP_Room25_uid134 | GPE_DestructibleSourceExperience(Clone) | ca9e2eae-0d5f-4184-9140-96294731c7a7 |  |
| Catacombs - Path of the Abyss EXP Statue | Catacombs_MVP_Room26_uid136 | GPE_DestructibleSourceExperience(Clone) | c8b83024-c633-46ca-b84c-0a2765cc0519 |  |
| Catacombs - Collapsed Path of Orpheus EXP Statue 1 | Catacombs_MVP_Room28_uid138 | GPE_DestructibleSourceExperience(Clone) | 4a403b9a-4133-47aa-95e9-663cb33f75eb |  |
| Catacombs - Collapsed Path of Orpheus EXP Statue 2 | Catacombs_MVP_Room28_uid138 | GPE_DestructibleSourceExperience(Clone) | e5b57bad-6303-479b-b700-44d7f66438aa |  |
| Catacombs - Collapsed Path of Orpheus EXP Statue 3 | Catacombs_MVP_Room28_uid138 | GPE_DestructibleSourceExperience(Clone) | eef37f79-8438-46ce-9434-2c574122cd60 |  |
| Catacombs - Realm of Old Night EXP Statue 1 | Catacombs_MVP_Room32_uid142 | GPE_DestructibleSourceExperience(Clone) | 825310e1-8737-4729-b889-bb060ede93b9 |  |
| Catacombs - Collapsed Path of Orpheus EXP Statue 4 | Catacombs_MVP_Room33_uid143 | GPE_DestructibleSourceExperience(Clone) | aa3eb442-412e-4fd3-961d-0085c0574c71 |  |
| Catacombs - Realm of Old Night EXP Statue 2 | Catacombs_MVP_Room34_uid144 | GPE_DestructibleSourceExperience(Clone) | 2f60b968-e11b-4862-b2cc-d3237da97b59 |  |
| Catacombs - Realm of Old Night EXP Statue 3 | Catacombs_MVP_Room34_uid144 | GPE_DestructibleSourceExperience(Clone) | f3f2a9b1-f55f-441c-ad30-34a5220e2467 |  |
| Catacombs - Antechamber of Sin EXP Statue 1 | Catacombs_MVP_Room35_uid145 | GPE_DestructibleSourceExperience(Clone) | 959ab0eb-5d80-402f-8d85-6fdd7b9b266b |  |
| Catacombs - Antechamber of Sin EXP Statue 2 | Catacombs_MVP_Room35_uid145 | GPE_DestructibleSourceExperience(Clone) | c60cdb8e-ccd1-4b67-b717-e014608e1c67 |  |
| Notre-Dame - Narthex EXP Statue | NotreDame_MVP_Room_010_uid89 | GPE_DestructibleSourceExperience(Clone) | ede20811-57ea-4a22-8811-5f81f25de5cb |  |
| Notre-Dame - Chapel of Patience EXP Statue | NotreDame_MVP_Room_045_uid110 | GPE_DestructibleSourceExperience(Clone) | 5b59da4a-e345-4737-a8a1-05b967830a5d |  |
| Notre-Dame - West Triforium EXP Statue | NotreDame_MVP_Room_055_uid135 | GPE_DestructibleSourceExperience(Clone) | cfd21251-04c5-4f6f-b5bf-165a01ec2402 |  |
| Notre-Dame - Notre-Dame Undercroft EXP Statue | NotreDame_MVP_Room_080_uid95 | GPE_DestructibleSourceExperience(Clone) | ee02bfdf-7504-4341-99ce-bbb1c72ed151 |  |
| Notre-Dame - Apse EXP Statue 1 | NotreDame_MVP_Room_085_uid108 | GPE_DestructibleSourceExperience(Clone) | 15799141-809e-4b94-a0a8-8cf6b04f9584 |  |
| Notre-Dame - Apse EXP Statue 2 | NotreDame_MVP_Room_085_uid108 | GPE_DestructibleSourceExperience(Clone) | 33803431-0ef9-43fd-b998-c5aa61053b8c |  |
| Notre-Dame - Tomb of the Chevaliers EXP Statue 1 | NotreDame_MVP_Room_135_uid101 | GPE_DestructibleSourceExperience(Clone) | 774d1856-b9b7-445e-90b1-06267bcaf33f |  |
| Notre-Dame - Tomb of the Chevaliers EXP Statue 2 | NotreDame_MVP_Room_135_uid101 | GPE_DestructibleSourceExperience(Clone) | d6d9f1ec-db3f-4bfa-8bac-5229d90aca49 |  |
| Notre-Dame - Transept EXP Statue | NotreDame_MVP_Room_140_uid80 | GPE_DestructibleSourceExperience(Clone) | 3f0577c4-ad06-44a1-9c06-f6e8c26d255f |  |
| Notre-Dame - Nave EXP Statue 1 | NotreDame_MVP_Room_220_uid122 | GPE_DestructibleSourceExperience(Clone) | 050c63b9-c85b-49e5-83e7-7126252cf3d8 |  |
| Notre-Dame - Nave EXP Statue 2 | NotreDame_MVP_Room_220_uid122 | GPE_DestructibleSourceExperience(Clone) | 6a862d82-1aee-4966-a302-8e6713cada13 |  |
| Notre-Dame - East Gallery EXP Statue 1 | NotreDame_MVP_Room_225_uid104 | GPE_DestructibleSourceExperience(Clone) | 55ad1f76-842f-4c37-9c2e-5e8a6b05f0a9 |  |
| Notre-Dame - East Gallery EXP Statue 2 | NotreDame_MVP_Room_225_uid104 | GPE_DestructibleSourceExperience(Clone) | 5f5ed262-b3ff-4154-ba61-6c8c7a625fb8 |  |
| Notre-Dame - Reliquary EXP Statue | NotreDame_MVP_Room_230_uid138 | GPE_DestructibleSourceExperience(Clone) | d31d259f-7738-41ba-ac79-4e37e349ccd1 |  |
| Notre-Dame - Path of the Penitent EXP Statue | NotreDame_MVP_Room_235_uid109 | GPE_DestructibleSourceExperience(Clone) | e0d89071-3f7b-407e-ac55-1df3851a653c |  |
| Notre-Dame - The Forest EXP Statue | NotreDame_MVP_Room_255_uid106 | GPE_DestructibleSourceExperience(Clone) | a6015fad-ab66-46d0-8c0e-a7eb665eb3e5 |  |
| Notre-Dame - Great Organ Room EXP Statue | NotreDame_MVP_Room_666_uid112 | GPE_DestructibleSourceExperience(Clone) | 31d26fca-54ee-4452-8864-6c80e6b51b96 |  |
| Notre-Dame - Boss Arthur EXP Statue 1 | NotreDame_MVP_Room_Boss_Arthur_uid121 | GPE_DestructibleSourceExperience(Clone) | 18e7e5e2-ffbd-4406-bbf1-5f55f13efb3b |  |
| Notre-Dame - Boss Arthur EXP Statue 2 | NotreDame_MVP_Room_Boss_Arthur_uid121 | GPE_DestructibleSourceExperience(Clone) | 56f4d960-f9a4-448b-bcb9-3d39142c05dc |  |
| Main Hall - Dungeon EXP Statue 1 | MainHall_MVP_Room_02_uid247 | GPE_DestructibleSourceExperience(Clone) | 01883714-82f2-4d42-895d-da72f9417dfc |  |
| Main Hall - Dungeon EXP Statue 2 | MainHall_MVP_Room_02_uid247 | GPE_DestructibleSourceExperience(Clone) | 14b33c76-ff04-4d14-b3b1-5b4623afdf83 |  |
| Main Hall - Dungeon EXP Statue 3 | MainHall_MVP_Room_02_uid247 | GPE_DestructibleSourceExperience(Clone) | c7617811-8d34-4b0f-9936-3c4f66986aca |  |
| Main Hall - Dungeon EXP Statue 4 | MainHall_MVP_Room_02_uid247 | GPE_DestructibleSourceExperience(Clone) | e7a2009c-d5c2-4322-968e-f524c809a7fe |  |
| Main Hall - Dungeon EXP Statue 5 | MainHall_MVP_Room_02_uid247 | GPE_DestructibleSourceExperience(Clone) | fca80df2-fb25-4319-8bd3-b10285711cc8 |  |
| Main Hall - Grand Hall EXP Statue 1 | MainHall_MVP_Room_03_uid248 | GPE_DestructibleSourceExperience(Clone) | 03b3fbdd-a0b7-4297-8724-58fd16a3fb48 |  |
| Main Hall - Grand Hall EXP Statue 2 | MainHall_MVP_Room_03_uid248 | GPE_DestructibleSourceExperience(Clone) | 61b3c330-2f53-480d-8ebf-a41d98f9c73e |  |
| Main Hall - Grand Hall EXP Statue 3 | MainHall_MVP_Room_03_uid248 | GPE_DestructibleSourceExperience(Clone) | e1d04810-1218-4d6e-9c0c-151e4fef824e |  |
| Main Hall - Grand Hall EXP Statue 4 | MainHall_MVP_Room_03_uid248 | GPE_DestructibleSourceExperience(Clone) | edaf695c-3f44-42e0-aa65-fa7bd92c5095 |  |
| Main Hall - Grand Hall EXP Statue 5 | MainHall_MVP_Room_03_uid248 | GPE_DestructibleSourceExperience(Clone) | f17a382c-62cd-4024-bae2-44b0de9d9350 |  |
| Main Hall - West Hall EXP Statue | MainHall_MVP_Room_070_uid213 | GPE_DestructibleSourceExperience(Clone) | 412261fc-1957-466b-a987-237c44f794e8 |  |
| Main Hall - Boudoirs EXP Statue | MainHall_MVP_Room_10_uid255 | GPE_DestructibleSourceExperience(Clone) | 026e7a45-e7c2-44a5-9585-53c7b72a73eb |  |
| Main Hall - Ballrooms EXP Statue 1 | MainHall_MVP_Room_15_uid260 | GPE_DestructibleSourceExperience(Clone) | 9eeb46b7-d7ab-4443-b325-ecf49538e038 |  |
| Main Hall - Ballrooms EXP Statue 2 | MainHall_MVP_Room_15_uid260 | GPE_DestructibleSourceExperience(Clone) | a62de5f3-ab2b-4d11-91b5-fa27304f9c6d |  |
| Main Hall - Torture Chambers EXP Statue 1 | MainHall_MVP_Room_16_uid261 | GPE_DestructibleSourceExperience(Clone) | d5854e72-ab78-4237-9283-041049bebdad |  |
| Main Hall - Torture Chambers EXP Statue 2 | MainHall_MVP_Room_17_uid262 | GPE_DestructibleSourceExperience(Clone) | f756fa30-7253-49f8-91c5-8d53eecc687b |  |
| Main Hall - Torture Chambers EXP Statue 3 | MainHall_MVP_Room_17_uid262 | GPE_DestructibleSourceExperience(Clone) | fb38b76e-4eb7-4710-bf36-42339e039f10 |  |
| Main Hall - Entrance Corridor EXP Statue 1 | MainHall_MVP_Room_1986_uid220 | GPE_DestructibleSourceExperience(Clone) | d7ad035a-1678-446f-819e-dd4b435e9af2 |  |
| Main Hall - Wine Cellar EXP Statue | MainHall_MVP_Room_19_uid265 | GPE_DestructibleSourceExperience(Clone) | dafd7d11-30a5-4962-9a5e-cbb9b02f8bd3 |  |
| Main Hall - Armoury EXP Statue | MainHall_MVP_Room_21_uid267 | GPE_DestructibleSourceExperience(Clone) | 81a2734e-39ad-4734-b8ec-a9de931f3b4e |  |
| Main Hall - Entrance Corridor EXP Statue 2 | MainHall_MVP_Room_560_uid196 | GPE_DestructibleSourceExperience(Clone) | b11a4325-18c5-404f-bf1f-5f4d953c6e27 |  |
| Library - Poetry Shelves EXP Statue | Library_MVP_Room_02_uid95 | GPE_DestructibleSourceExperience(Clone) | 030dccd6-643e-4deb-8578-5c9dd34717dd |  |
| Library - Library Foyer EXP Statue | Library_MVP_Room_09_uid102 | GPE_DestructibleSourceExperience(Clone) | f245f7c7-a2b2-4c78-bc53-9fce9874e6a2 |  |
| Library - Astronomer's Office EXP Statue | Library_MVP_Room_10_uid103 | GPE_DestructibleSourceExperience(Clone) | f7083568-ee44-43b9-83e8-849b9f19bae6 |  |
| Library - Trophy Room EXP Statue | Library_MVP_Room_11_uid104 | GPE_DestructibleSourceExperience(Clone) | f48353d7-c024-4564-a200-037f47ba5047 |  |
| Library - Natural Philosophy Shelves EXP Statue 1 | Library_MVP_Room_17_uid110 | GPE_DestructibleSourceExperience(Clone) | 0a517749-4fb5-41d9-bf1b-1ee4c841f7a3 |  |
| Library - Natural Philosophy Shelves EXP Statue 2 | Library_MVP_Room_17_uid110 | GPE_DestructibleSourceExperience(Clone) | 3d8ea53e-9419-405b-9a41-639dcd0d4a52 |  |
| Library - Study Room EXP Statue 1 | Library_MVP_Room_21_uid116 | GPE_DestructibleSourceExperience(Clone) | 32f6de52-355a-4d53-8416-625972afa2e4 |  |
| Library - Study Room EXP Statue 2 | Library_MVP_Room_21_uid116 | GPE_DestructibleSourceExperience(Clone) | b77d2729-94d0-4a08-897f-1161508eb414 |  |
| Library - Exhibition Room EXP Statue | Library_MVP_Room_25_uid120 | GPE_DestructibleSourceExperience(Clone) | 32fb4443-e5a2-4a05-8e71-d1240c906dc5 |  |
| Clock Tower - Serpent Pinion EXP Statue | ClockTower_MVP_Room06_uid104 | GPE_DestructibleSourceExperience(Clone) | 906517db-dccf-4c92-aa9f-a27917f85d99 |  |
| Clock Tower - Path of the Builder EXP Statue | ClockTower_MVP_Room32_uid132 | GPE_DestructibleSourceExperience(Clone) | 3011278b-ff0a-421d-b1f2-b560876e81a6 |  |
| Clock Tower - Eastern Summit EXP Statue 1 | ClockTower_MVP_Room37_uid137 | GPE_DestructibleSourceExperience(Clone) | 014e02ad-41d7-4930-b666-fc9aa484b721 |  |
| Clock Tower - Eastern Access Shaft EXP Statue 1 | ClockTower_MVP_Room41_uid141 | GPE_DestructibleSourceExperience(Clone) | 0fa55e36-b6ff-432e-b96b-d78d718d1008 |  |
| Clock Tower - Eastern Access Shaft EXP Statue 2 | ClockTower_MVP_Room41_uid141 | GPE_DestructibleSourceExperience(Clone) | 2b8ce3f1-fb79-4103-890c-999b781fe473 |  |
| Clock Tower - Eastern Access Shaft EXP Statue 3 | ClockTower_MVP_Room41_uid141 | GPE_DestructibleSourceExperience(Clone) | 3e46ebde-ee8b-4e2b-adf4-34b773aac24b |  |
| Clock Tower - Eastern Access Shaft EXP Statue 4 | ClockTower_MVP_Room41_uid141 | GPE_DestructibleSourceExperience(Clone) | ad3b0319-1a84-45e8-8e37-bd693edbaf21 |  |
| Clock Tower - Eastern Access Shaft EXP Statue 5 | ClockTower_MVP_Room41_uid141 | GPE_DestructibleSourceExperience(Clone) | bd4800c6-9735-46aa-b960-83a3e14d25ba |  |
| Clock Tower - Eastern Summit EXP Statue 2 | ClockTower_MVP_Room46_uid148 | GPE_DestructibleSourceExperience(Clone) | a3670bbc-8ee3-4fce-9843-957823a33524 |  |
| Clock Tower - Daedalus Crossing EXP Statue 1 | ClockTower_MVP_Room55_uid159 | GPE_DestructibleSourceExperience(Clone) | 4112c7d5-ef9f-48ba-a591-fc6bf219f86b |  |
| Clock Tower - Daedalus Crossing EXP Statue 2 | ClockTower_MVP_Room55_uid159 | GPE_DestructibleSourceExperience(Clone) | 62c1dc70-8755-4e6b-9548-aec3ae60318b |  |
| Clock Tower - Daedalus Crossing EXP Statue 3 | ClockTower_MVP_Room55_uid159 | GPE_DestructibleSourceExperience(Clone) | e4c5ea6e-43e2-4030-850a-303ef919c413 |  |
| Clock Tower - Clock Keeper's Atrium EXP Statue | ClockTower_MVP_Room57_uid161 | GPE_DestructibleSourceExperience(Clone) | ce12183d-05f9-4a3c-a402-bed250f91574 |  |
| Clock Tower - Mechanic's Ascent EXP Statue | ClockTower_MVP_Room59_uid163 | GPE_DestructibleSourceExperience(Clone) | 0a86ff0e-f213-438a-a392-7f47f37c67c6 |  |
| Hall of Mirrors - Boudoir of Bewilderment EXP Statue | HallOfMirrors_MVP_Room_01_uid98 | GPE_DestructibleSourceExperience(Clone) | cdbc1b48-a653-443f-8e9c-793c7d87c777 |  |
| Hall of Mirrors - Steps of Misdirection EXP Statue 1 | HallOfMirrors_MVP_Room_04_uid102 | GPE_DestructibleSourceExperience(Clone) | 6cb29664-20c8-4905-ba73-6e8ed1d63741 |  |
| Hall of Mirrors - Steps of Misdirection EXP Statue 2 | HallOfMirrors_MVP_Room_04_uid102 | GPE_DestructibleSourceExperience(Clone) | ee5fdd97-5bfd-4a7f-a3b5-27f359accb87 |  |
| Hall of Mirrors - Passage of Perplexity EXP Statue | HallOfMirrors_MVP_Room_05_uid103 | GPE_DestructibleSourceExperience(Clone) | a77da348-f8d3-48dd-b331-a2f232167215 |  |
| Hall of Mirrors - Gallery of Glass EXP Statue | HallOfMirrors_MVP_Room_07_uid105 | GPE_DestructibleSourceExperience(Clone) | 705a785a-1c8d-47bd-af5e-65ecdecec691 |  |
| Hall of Mirrors - Familiar Way EXP Statue | HallOfMirrors_MVP_Room_10_uid108 | GPE_DestructibleSourceExperience(Clone) | 2aacb1a5-e1ef-48ed-ac2b-a26397d0cdd8 |  |
| Hall of Mirrors - Dead-end Tower EXP Statue | HallOfMirrors_MVP_Room_15_uid113 | GPE_DestructibleSourceExperience(Clone) | d4a89573-3735-437f-85e1-01b972bf9f5a |  |
| Hall of Mirrors - Threshold of Vision EXP Statue | HallOfMirrors_MVP_Room_20_uid119 | GPE_DestructibleSourceExperience(Clone) | 78cdc5dd-8775-40b5-b738-b2a74bf10200 |  |
| Hall of Mirrors - Servants' Refectory EXP Statue | HallOfMirrors_MVP_Room_21_uid120 | GPE_DestructibleSourceExperience(Clone) | b9ee9dfe-cb8d-455f-b6df-506d70775e91 |  |
| Hall of Mirrors - Looping Hallways EXP Statue 1 | HallOfMirrors_MVP_Room_22_uid121 | GPE_DestructibleSourceExperience(Clone) | 14ebb88f-e97c-4720-a2b6-acd3a08a468a |  |
| Hall of Mirrors - Looping Hallways EXP Statue 2 | HallOfMirrors_MVP_Room_24_uid123 | GPE_DestructibleSourceExperience(Clone) | f90983e6-c5ba-4e4a-9d7a-8c5661ebfb99 |  |
| Hall of Mirrors - Level Deviled EXP Statue | HallOfMirrors_MVP_Room_30_uid129 | GPE_DestructibleSourceExperience(Clone) | 5aa9fbd8-bcd1-4493-8d4f-69b63b776a48 |  |
| Hall of Mirrors - Nightmare Atrium EXP Statue | HallOfMirrors_MVP_Room_34_uid134 | GPE_DestructibleSourceExperience(Clone) | f45d83fa-8827-4254-a24f-37bbe2809864 |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 1 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | 036ce76e-0bed-420b-8ad9-8b272891ac08 |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 2 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | 1b33657c-de21-429a-8a47-82cbdd8a8d91 |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 3 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | 44189cc0-d522-4f90-8795-6c8f834b64be |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 4 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | 6ded163e-8bdf-45a1-9433-940f81bd8450 |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 5 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | a530de9c-d9b4-4348-ab76-4593cdf100ac |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 6 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | b37d949d-9720-4d8a-97a3-e2112101b43c |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 7 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | bb611afe-6ca3-40a4-9851-04efa7e006ff |  |
| Hall of Mirrors - Tower of Worldly Reflection EXP Statue 8 | HallOfMirrors_MVP_Room_41_uid145 | GPE_DestructibleSourceExperience(Clone) | f67394ea-1dbe-43ab-9502-95462b7674de |  |




