# Native Godot estate implementation

## Authority and placement

`scenes/estate.tscn` is a native Godot Control embedding a SubViewport, Node2D world, Camera2D and TileMapLayer. It does not use a WebView or load any of the preserved xianfu-background / xianfu-terrain paintings. Only a distant mountain-and-cloud painting is behind the native world.

Each facility occupies a real 3×3 square of 64×32 isometric terrain cells. Neighboring plot centers are six native cells apart, leaving three-cell garden/road gaps. `PLOT_CELLS` in `scripts/estate_view.gd` is the position authority. The original p0–p8 IDs are preserved. The p9–p23 additions bring the total to 24 plots, eight per region. Their costs and unlock conditions come from `GameEconomy.get_all_plots()` / `data/godot-rules.json`.

Purchasing an expansion does actually grow the TileMapLayer land area. Before purchase its location is a small marker in the cloud. Locked markers can be inspected but cannot receive a facility. All plot floor polygons, native paving/grass, edge cliffs, picking, nourishment, edit ghosts, pan and zoom go through `TileMapLayer.map_to_local` / `local_to_map`, then the SubViewport canvas transform. The background image has no buildable platforms, floor cells, paths or buildings.

## Facility art

The retained seven facility images are independent `scenes/building.tscn` instances. A triangulated Polygon2D maps their measured image-space foundation corners onto the exact shared floor diamond (left -96,0; top 0,-48; right 96,0; bottom 0,48). This is a four-sector mesh transform, not a width-relative anchor tweak. Even asymmetric input floors align with the projected terrain. The inverse transform drives alpha-aware roof/facade picking. Transparent sprite rectangles do not capture neighbors.

Facilities use native Y sorting by committed ground center. Static captions are hidden until selection. A selected rear facility fades nearby foreground occluders. Native permanent lanterns/shrubs appear at higher upgrade stages; hall and alchemy additionally use their retained stage-specific art. The original image assets are preserved unchanged.

Production effects, construction, full storage, missing materials and pending inventory are read directly from `GameEconomy.get_building_status`. They remain separate dimensions. No production flag is invented by the renderer. Move previews use independent translucent scene instances; the original remains at its committed plot until the UI transaction succeeds. Cancel never changes the state.

The three additional facility types currently have original code-authored SVG artwork in `assets/terrain/{warehouse,mine,relicworkshop}-native.svg`, with exact metadata in `extra-building-visuals.json`. These are distinct native vector compositions. They are visually less painterly than the retained seven sprites and should not be represented as matching premium raster art. The metadata file permits later approved art replacement without changing placement logic.

## Public interface

- `set_state(Dictionary)` (`configure` alias), `set_editing(bool)`
- `select_building(id)`, `focus_plot(id)`
- `set_zoom(relative_factor)`, `zoom_by(multiplier)`, `reset_camera()`
- `preview_move(id, plot_id)`, `cancel_move()`
- Signals: `building_selected(id)`, `plot_selected(id)`, `move_previewed(id, plot_id)`, `move_cancelled`

The main UI owns purchases, construction, confirmation and persistence. EstateView is read-only relative to economic state. Default input is tap-to-select, drag empty ground to pan and wheel/pinch to zoom. Building drags are permitted only in explicit edit mode. Right-click/Escape cancels the ghost. Facility-management UI is available for access even when a building is occluded.

## Verification and evidence

Executed in Godot 4.6.3 on Linux:

`godot --headless --path . --script tests/test_estate.gd`

Latest result: 1,017 checks, zero failures. Includes 441 native tile coordinate roundtrips; all 24 plot center hits; exact four-corner foundation projection and inverse alpha-picking mapping for all ten types / four stages; fixed/occupied/locked placement rejection; normal mode refusing move; preview and cancel retaining identical source state; nine viewport/zoom combinations; native Y sorting; expansion increasing actual native terrain area; and production heartbeat preserving a pending move confirmation.

Actual graphical Godot was launched on the cloud Linux desktop and observed through native screenshots. The ordinary full-game screen is `docs/ui-estate.png`. Its hall selection opened the real facility management modal. The isolated seven-facility test is `tests/estate_visual.tscn`; its refreshed capture `docs/estate-native-visual.png` shows the widened spacing and current native terrain at overview zoom 0.84. `docs/estate-native-move-preview.png` was captured after a real native mouse drag of the farm from p2 to p7 in edit mode. The original remained at p2; right-click cancellation removed the ghost and retained all original placements. These isolated captures are not evidence of ordinary-save economic progression. Native main roster and trial-battle screens were also captured as `docs/ui-roster.png` and `docs/ui-battle.png`.

The cloud graphics driver reports unsupported VSync mode, and its ALSA device is absent (Godot falls back to Dummy audio). This is not a claim of audio verification or stable frame-rate benchmarking. Actual mobile hardware and touch gestures have not been tested.

## Background asset provenance

`assets/terrain/distant-cloud-mountains.png` was generated using the built-in image-generation tool during this authorized game task, then inspected and copied into the project. No external purchase, account or image service was used. Prompt: “distant background only for a native 2D Chinese fantasy estate-building game; refined painterly far-away blue-green ink-wash mountain ridges fading into soft jade cloud sea, pale warm sunrise haze; entire center and lower two thirds soft empty cloud; no foreground land, islands, terraces, platforms, floors, paths, buildings, gates, UI or text; muted premium hand-painted shanshui texture.”

`assets/terrain/estate_tiles.svg` and three extra facilities are original code-native SVG assets. They have no external runtime dependency.
