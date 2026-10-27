# Room and district catalog

[Documentation index](../README.md)

Browse by area below. Each area page lists its districts and rooms, with object inventories folded under the relevant room.

## Areas

| Area | Districts | Rooms |
|---|---:|---:|
| [Catacombs](room%20district%20catalog/Catacombs.md) | 20 | 43 |
| [Clock Tower](room%20district%20catalog/ClockTower.md) | 32 | 52 |
| [Hall of Mirrors](room%20district%20catalog/HallOfMirrors.md) | 25 | 41 |
| [Innocents' Cemetery](room%20district%20catalog/InnocentsCemetery.md) | 20 | 42 |
| [Library](room%20district%20catalog/Library.md) | 20 | 37 |
| [Main Hall](room%20district%20catalog/MainHall.md) | 32 | 58 |
| [Master's Quarter](room%20district%20catalog/MasterQuarter.md) | 6 | 9 |
| [Millers' Bridge](room%20district%20catalog/MillersBridge.md) | 17 | 21 |
| [Master's Quarter - Bad Ending](room%20district%20catalog/MQBadEnding.md) | 1 | 5 |
| [Notre-Dame](room%20district%20catalog/NotreDame.md) | 33 | 41 |
| [Streets of Paris - Poor Quarter](room%20district%20catalog/StreetsPoor.md) | 30 | 49 |
| [Streets of Paris - Rich Quarter](room%20district%20catalog/StreetsRich.md) | 7 | 18 |

[Rooms without district membership (7)](room%20district%20catalog/unassigned.md)

## Shared room assignments

Two rooms appear in more than one district. Both memberships are preserved.

| Room | Memberships | Reverse lookup |
|---|---|---|
| `MillersBridge_MillersBridge_Room23_uid101` | MillersBridge_District16, MillersBridge_District17 | MillersBridge_District17 |
| `Streets_Rich_StreetsRich_Room_450_uid64` | StreetsRich_District09, StreetsRich_District07 | StreetsRich_District07 |

## Reading the catalog

- **243 districts**, **423 room identifiers**, **384 object inventories**, and **97 barrier-candidate records** are available.
- Open an area, choose a district, then expand a room to see its full identifier and object list.
- Short room labels are used for navigation; the exact full identifier remains inside each room entry.
- Counts describe extracted object-name occurrences, not guaranteed unique interactables or collectible totals.
- Missing inventory data does not mean a room is empty. Barrier visual effects do not prove a required ability.
- Membership is not a connection graph or proof that a room is reachable.
- Display identifiers are metadata; do not use them as transition or progression GUIDs.

## Runtime lookup

Use `CurrentRoomInstance.currentRoom.worldRoomData` to read `instanceName` and `id`. Read the scene name from the current room object for runtime scene context.

Map selection exposes `LevelMapContext.currentDistrictMainSelection`; `DistrictData.ContainsRoom(roomName)` tests membership. The displayed map level may differ from the current player level.

Exact room IDs can include biome prefixes or instance suffixes. Verify conversions rather than stripping identifiers blindly.



