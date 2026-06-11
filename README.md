# GTA-2W — GTA 2 Web Clone (Three.js)

Top-down arcade driving/action game in the spirit of GTA 2, built with React + Three.js (@react-three/fiber), fully data-driven.

## Run

```bash
npm install
npm run dev
```

## Controls

| Key | Action |
|---|---|
| W/S or ↑/↓ | Walk / throttle & brake-reverse |
| A/D or ←/→ | Turn / steer |
| E | Enter / exit nearest vehicle |
| Space | Fire weapon (on foot) / handbrake (in vehicle) |

Walk over the pistol pickup to arm yourself. Run over pedestrians, shoot barrels, steal cars. Score in the HUD.

## Architecture

```
src/
  config/constants.ts    Single source for TILE_SIZE, WORLD_SCALE, camera, lighting
  types/                 Strict interfaces: MapData, AssetPreset, VehicleConfig, TileType, WeaponConfig
  data/                  ALL gameplay values live here (JSON, editable without code)
    map.json             City layout (string rows + legend), spawns, object placements
    tile-types.json      Tile definitions (height, color, texture kind, solid, drivable)
    vehicle-configs.json Handling per vehicle (speed, accel, turn rate, grip, ...)
    weapon-configs.json  Weapon stats
    asset-presets.json   Per-asset transform presets (the editor edits these)
  render/                SHARED rendering used by BOTH game and editor (Golden Rule)
    SceneEnvironment     Lighting rig
    TileMapMesh          One InstancedMesh per tile type; live PNG texture swap
    AssetInstance        Transform-wrapper spawn pattern (see below)
    builtinModels        Procedural placeholder models (replace via GLB upload)
  game/                  Pure logic, React-free: World class, vehicle physics, collision, input
  scenes/                GameScene (play) and EditorScene (preview — same components)
  ui/                    HUD, AssetEditorPanel, ConfigMenu
  tiled/importTiled.ts   Tiled .tmj importer
```

### Asset spawn pattern (mandatory)

```
outer Group   <- world position/rotation from map data
  preset Group <- preset transform from asset-presets.json (editor-adjusted)
    model       <- GLTF or builtin, never mutated
```

### Golden Rule

`EditorScene` renders with the exact same `SceneEnvironment`, `TileMapMesh`, `AssetInstance`, and constants as `GameScene`. An asset positioned in the editor looks identical in the game.

## Editors

- **Asset Editor**: pick any asset, upload a GLB, position with gizmo (translate/rotate/scale) or sliders, *Save & Apply to Game* hot-applies, reset to default, export/import presets JSON.
- **Config**: upload a PNG per tile type (live InstancedMesh material swap), tune vehicle handling live, export vehicle JSON.

## Tiled import (.tmj)

- Tile layer named `ground` with a layer property `legend` = JSON mapping gid → tile type id (e.g. `{"1":"road_h","2":"sidewalk"}`).
- Object layer named `objects`; object class = asset preset id, or special: `playerSpawn`, `vehicleSpawn` (properties `presetId`, `traffic`), `pedestrianSpawn`.
- 32 px in Tiled = one tile (TILE_SIZE world units).

## Performance

One InstancedMesh per tile type, instanced projectiles, ref-based per-frame transform sync (no React re-render per frame). Measured 60 fps.
