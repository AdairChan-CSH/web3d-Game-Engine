# UnirealEngine

A Unity-style scene editor and play-mode sandbox that runs entirely in the browser.

Plain **HTML, CSS and JavaScript** — no build step, no package manager, no server required, and **no SQL**
anywhere. Scenes live in memory, persist to `localStorage`, and import/export as JSON.

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ UnirealEngine File Edit GameObject Component Help   ▶ ⏸ ⏭ ⏹   Edit stats │
├──────────────┬───────────────────────────────────────────┬───────────────┤
│ Hierarchy    │  Scene | Game   W E R   Grid Wire Snap    │ Inspector     │
│  Ground      │                                           │  Transform    │
│  Crate A  rb │            3D WebGL viewport              │  Mesh Renderer│
│  Sun  Point  │    gizmos, grid, shadows, light cookies   │  Light / FSM  │
│──────────────│                                           │  Collider     │
│ Project      ├───────────────────────────────────────────┤  Rigidbody    │
│  models      │  Console                                  │               │
└──────────────┴───────────────────────────────────────────┴───────────────┘
```

---

## Run it

Option A — just open the file:

```text
UnirealEngine/index.html
```

Everything except `fetch()`-based helpers works from `file://`; all meshes are generated procedurally, so
there is nothing to download.

Option B — use any static file server while editing the source (optional; no build step and no server-side
runtime is required). For example, any editor live-preview extension or a local static server pointing at the
repository root works. The page itself has no server-side dependency.

The full user manual lives at `UnirealEngine/manual.html`. Open it from **Help → Open Full Manual** in the editor,
with `Ctrl+Shift+F1`, or directly in a browser. It has a live client-side search system (`Ctrl+K` or `/`) over
the guide, component tables, keyboard shortcuts, audio/video setup and the complete 84-action PlayMaker catalog.
The manual reads the action catalog from `js/playmaker.js` at runtime, so it stays in sync with the editor.

The `<script>` tags in `index.html`, `manual.html` and `tests/index.html`, and the stylesheet links they use, carry a
`?v=20` suffix. Bump that number after changing JavaScript or CSS so a browser with an older cached copy loads your
edits instead of the previous files.

## Tests

Open `UnirealEngine/tests/index.html` in a browser. The page loads the engine source directly, runs the core suite
and reports every group plus a pass/fail summary; failures also appear in the Console. Everything is plain
browser JavaScript, so there is nothing to install and no command line is involved. Use **Run again** to repeat
the suite in place. The suite currently runs **78 groups**, covering the engine, importer, animation, PlayMaker
(custom actions and scene changer), extras, texture tools, input/gamepad state, PiP camera data, media
assets/runtimes and multiplayer network systems.

## What it does

### Scene editing
- Hierarchy panel with search, tag filtering (`tag:Player`), active/disabled toggles, drag-to-reparent and
  double-click framing.
- Inspector for Transform, Mesh Renderer, Collider, Rigidbody, Camera, Script, Light, Animator, Network Object,
  Audio Source, Video Player and every PlayMaker component.
- A gameplay-facing **Game Object** card sets Active, switches the mesh between a built-in primitive and an
  imported model, and shows network ownership.
- Unity-style **Game Object Tag** picker at the top of the Inspector: browse tags with usage counts, create a
  new tag, and close the popup with its **×**, <kbd>Esc</kbd> or a click outside. Tags are free-form strings and
  serialize with the scene.
- Add primitives (cube, sphere, cylinder, cone, plane), empty objects, cameras and four light types from the
  Project panel or the GameObject menu.
- Import OBJ, STL, GLB and glTF models from the Project panel, the File menu, or by dropping files on the
  viewport. Import audio and video the same way from the File menu, Project panel or viewport drop.
- Undo/redo history (80 steps) for edits, moves, component changes, imports and deletions.
- Copy/paste/duplicate — including whole parent/child subtrees.
- Scene-wide sun, ambient and sky-gradient controls in the scene inspector.

### Tools and presets
- The **Tools** menu opens a four-tab popup for object presets, FSM templates, texture editing and input settings.
  Object Presets and FSM Templates are also available from the **GameObject** menu.
- Twelve one-click object presets: Empty, Player, Enemy/NPC, Pickup, Projectile, Platform, Trigger Zone,
  Main Camera, Second Camera (PiP), Sun, Point Light and Spot Light. Presets add the useful components, tag and
  network role for that object; camera presets also update the Main Camera flag.
- Nine editable FSM templates: Idle Log, Spin, Patrol, Chase Player, Pickup, Projectile, Health, Follow Camera and
  Network Player. Templates produce normal PlayMaker data, so every state, action and transition remains editable
  in the Inspector. An optional **Replace existing FSMs** checkbox removes the selected object's old FSMs first.
- The **Texture Editor** lists base-color textures used by models in the current scene. It can tint, grayscale,
  flip, rotate, overlay a checker/stripes/dots pattern, paint a brush dot or fill with color, then download the PNG.
  Edits write back to the stored asset and are re-uploaded to the GPU; texture pixels are not part of undo history.
- Keyboard shortcuts can be rebound from **Tools → Keyboard & Gamepad Settings…**, including Play, Pause, Step,
  Stop, the three gizmo tools, Focus, and Toggle Second Camera. The active gamepad's four axes and 16 standard
  buttons are polled live outside Play mode, with an adjustable deadzone. Button edges publish `gamepad.*` keys to
  PlayMaker, and action fields accept `#gamepad.button.a`, `#gamepad.axes.0` and `#gamepad.active` tokens.

### Viewport
- WebGL forward renderer: Lambert + Blinn-Phong lighting, rim light, rough/smooth shading, transparency,
  contact shadows, GPU skinning and a sky gradient behind the canvas.
- Up to eight dynamic directional/point/spot lights, plus a Sun override, with point/spot cookie projection.
- Move / rotate / scale gizmos with axis picking, hover highlighting and Ctrl snapping.
- Orbit (RMB or Alt+LMB), pan (MMB) and wheel zoom; frame selection with `F`.
- Scene and Game tabs. Game uses the scene's active **Main Camera**.
- Picture-in-picture second camera: add the **Second Camera (PiP)** preset (or give any Camera component `pip: true`
  and `main: false`) and enable **Tools → Toggle Second Camera (PiP)** or the Game-view checkbox. The Game view
  renders that camera into a framebuffer and composites a bordered inset in the upper-right corner.
- Pressing Play automatically switches to Game view and renders from the Main Camera; Stop returns to the
  previous view. If no active Main Camera exists, the editor warns and keeps the scene camera.
- Grid, wireframe and snap toggles, live FPS/triangle/object stats.

### Audio and video
- Add an **Audio Source** or **Video Player** from the Component menu, or import a clip and press **Add** in
  **Project → Audio / Video**. Clip bytes are stored in `UE.AssetStore`; exported scenes embed only the referenced
  clips under `audioAssets` and `videoAssets`, then restore them on import.
- Audio accepts **WAV, MP3, OGG and M4A**. WAV duration/sample rate/channel metadata is read directly; every format
  is decoded by the browser's `AudioContext`. No files are transcoded and no server code is used.
- Audio Source has volume, pitch, loop, play-on-awake and optional **3D positional** playback with min/max distance
  and rolloff. The active Main Camera is the listener, and each spatial source follows its GameObject.
- Video accepts local **MP4/M4V, WebM and OGV** files (or an optional URL), played by a browser `HTMLVideoElement`.
  Support depends on the browser's installed codecs. Video can auto-play, loop, mute, control volume/rate, and route
  sound through the same 3D listener as audio.
- Put a **Mesh Renderer** on the same object as a Video Player: decoded frames become an unlit moving texture on
  that object's mesh, using texture unit 1 and the primitive UVs. Built-in primitives now include UVs.
- Play starts all play-on-awake/auto-play clips, Pause suspends audio and pauses video, Step prepares paused video
  frames, and Stop tears runtimes down and restores the pre-Play scene. Browsers may block auto-play until the user
  interacts with the page; press Play again or use the inspector's Play preview.

### Play mode
- Play, pause and single-step at a fixed 60 Hz. Audio Source and Video Player runtimes start, pause and stop with
  the transport.
- Scripts are plain JavaScript function bodies with `self`, `dt`, `time`, `scene` and `UE` in scope.
  `self.data` is persistent per-object state.
- PlayMaker FSMs run alongside scripts: states, actions, transitions, variables, timers, keyboard input and
  collision triggers.
- Fixed-step AABB physics with mass-weighted collision response, per-collider friction, gravity scale,
  restitution, drag, kinematic bodies, trigger colliders and freeze-position constraints. Each step runs three
  contact-relaxation passes so stacked boxes settle without sinking into each other. Rotation is deliberately
  outside the solver, so angular drag and rotation constraints are not simulated.
- Stop restores the scene exactly as it was before Play, like Unity.
- Script compile/runtime errors appear in the Console instead of breaking the frame loop.

### Multiplayer network
- Each object can carry a **Network Object** component with a stable network id, transform/mesh/tag/active sync
  toggles, send rate, interpolation and a runtime owner.
- Host or join a room from the **Network** pill or the Network menu. Transport is `BroadcastChannel` with a
  `localStorage`/`storage`-event fallback for tabs in the same browser and origin. There is no server, socket
  library, account system or database.
- Pressing Play starts replication: spawns, transform, mesh/material, tag, active state and ownership flow
  between tabs. A client that joins before Play asks the host for a fresh snapshot when Play starts.
- Four network object roles — **Player**, **Enemy/NPC**, **Pickup** and **Projectile** — bundle a tag, primitive,
  color, scale, trigger/gravity behavior and sync defaults. Use **Network → Role / …** on the selected object, or
  spawn a role with `UE.Network.spawn({ role: 'pickup' })`. Role defaults are starting points; the per-object sync
  toggles remain editable.
- Claim ownership to decide which peer drives an object's transform. Stop restores the pre-Play scene; Leave
  stops discovery and clears peer state.
- PlayMaker adds a **Network** category with spawn/claim/role/despawn actions, room host/join actions, event sending
  and state broadcasting.

### 3D models
- Supported formats: **OBJ** (with `.mtl` colors and `map_Kd` textures), **STL** (ASCII and binary), **GLB**
  (glTF 2.0 binary) and **glTF**. Textures are read from embedded GLB buffer views, data URIs, or sibling image
  files dropped/picked together with the model.
- glTF/GLB base-color materials use `baseColorTexture` (including `KHR_materials_pbrSpecularGlossiness`
  diffuse textures) and `baseColorFactor` as a tint, with `TEXCOORD_0` UVs. OBJ materials use `map_Kd` and `Kd`;
  OBJ UVs are V-flipped to match WebGL sampling. STL has no UVs, so it stays untextured.
- A model may contain **multiple primitives** when its glTF mesh or OBJ uses several materials. Each primitive
  keeps its own color and texture, so multi-material models render correctly.
- GLB import handles nested node hierarchies, `matrix` and translation/rotation/scale transforms, missing
  transforms, indexed and non-indexed primitives, triangle strips/fans, data-URI buffers and external `.bin`
  files. Mirrored nodes keep their front faces pointing outwards. Draco and meshopt compression are rejected
  with an explicit "re-export without compression" message.
- Each import is normalized to a 2-unit footprint, centered on X/Z and placed on `y = 0`, so it lands at a
  usable size immediately.
- Geometry is stored once in `UE.AssetStore` as base64 `Float32Array` positions/normals/UVs and `Uint16Array`
  or `Uint32Array` indices (automatically chosen from the highest index). Every instance shares it, and
  exported scene files embed only the meshes they reference. Textures stay as compressed PNG/JPEG/WebP bytes and
  are decoded to GPU textures by the renderer, so scenes survive a save/reload without the original image files.
- Selecting a model in the Mesh Renderer inspector switches the renderer between built-in primitives and
  imported assets. A bounds collider is created automatically. The material color tints a base-color texture, so
  textured imports start white and untextured imports keep their MTL/glTF color. Roughness and emissive still
  come from the inspector.
- The Mesh Renderer inspector also edits **UV tiling** and **UV offset** for imported base-color textures, with
  a one-click reset. PlayMaker's `Edit Texture` and `Scroll Texture` actions drive the same values at runtime.
- **Tools → Texture Editor…** edits the imported base-color image itself: tint, grayscale, flip, rotate, pattern,
  brush dot or fill, plus PNG download. The edited bytes are stored in the asset, uploaded to the GPU and embedded
  in exported scenes.
- Models appear in **Project → Models** with an **Add** button; re-importing the same name, size and texture
  reuses the existing asset.

### Animation & auto-rigging
- Select an imported model and press **Auto Rig & Animate** in the Mesh Renderer card (or
  **GameObject → Animation → Auto Rig & Animate**) to build a deterministic 20-bone humanoid skeleton from the
  model's bounds: Root, Hips, Spine, Chest, Neck, Head, shoulders/arms/hands and thighs/shins/feet on both sides.
- Skin weights are computed per vertex from the nearest bone segments, limited to four influences, normalized and
  cached per mesh + rig. The WebGL shader applies **linear-blend skinning** for up to 24 bones in one pass.
- A **library of 99,999 motions** is generated lazily from 25 categories of 4,000 clips each (3,999 in the last) —
  Idle, Walk, Run, Jump, Dance, Fight, Kick, Carry, Swim and more. Nothing is downloaded or stored: each clip's pose
  is derived deterministically from its index and time, so no clip data is materialized up front.
- The Animator card browses the library by category or free-text search, jumps to any index, and offers
  previous/next/random, play/pause/stop, speed, crossfade, loop and an edit-mode preview toggle.
- **Blending:** assign a second clip and set a weight with **Mix** (cross-blend the two poses) or **Additive**
  (layer the second clip as a pose delta). Crossfades interpolate from the outgoing clip when a new motion starts.
- Animation runs in Play mode and previews live in the editor. Scene files round-trip the rig, skeleton, motion,
  blend settings and playback state, and exported scenes keep working without the original model files.
- PlayMaker adds an **Animation** category: Play Motion, Crossfade Motion, Blend Motion, Random Motion, Set Motion
  Speed, Set Motion Playing, Stop Animation and Auto Rig Animator. Motion fields accept a name (`run`), a category
  word (`dance`), an exact index (`4000`) or an FSM variable (`$clip`).

### Lights
| Type | Behaviour |
| --- | --- |
| **Sun** | Overrides the global Sun & Lighting direction and color. The first active Sun wins. |
| **Directional** | Points along the object's local −Z axis; constant color at any distance. |
| **Point** | Position light with `Range` attenuation and an optional cookie in every direction. |
| **Spot** | Points along local −Z with `Outer angle` / `Inner angle`; cookies are projected through the cone. |

- `Color` and `Intensity` are common to all types. `Cast shadow` is reserved for future shadow maps. The
  scene keeps one global ambient term and sky gradient.
- Point and spot lights can project a **cookie**: procedural `checker`, `stripes`, `grid`, `dots` or `rings`
  patterns, or an uploaded image. The inspector previews the pattern and the upload button accepts any
  browser image.
- Cookies live in one 512×512 atlas (4×4 cells of 128×128) and are sampled by both the light shader and the
  spot projection. Only point and spot lights use them.
- Light icons are drawn in the viewport while editing so point/spot positions and directions are easy to
  find.

### PlayMaker
PlayMaker components are data-driven and serialize with the scene:

| Component | Purpose |
| --- | --- |
| **FSM** | A named state machine with a start state, variables, states, actions and transitions. |
| **Trigger** | Sends `enterEvent` / `exitEvent` when its collider pair with another object starts or stops. |
| **Timer** | Sends an event every `seconds` while playing; optional `repeat`. |
| **Input** | Sends an event when a keyboard key or polled gamepad button goes down or up. Click the viewport first so the page has focus. |
| **Event Proxy** | Sends one event on start, after a delay, or on a key press. |

- States run their actions in order. When a state finishes it receives `FINISHED`; transitions are matched by
  event name, and every transition can point at any state.
- Multiple FSMs can live on one object (handy for separating movement from light logic). Other components
  can target `self` by default, an object name/id, or `tag:Player` to find the first object with that tag.
- Every GameObject has a free-form **Tag** chosen from a closable Inspector popup. Tags serialize with the
  scene, hierarchy search accepts `tag:Name`, and `Set Tag`, `Activate Objects By Tag` and `Send Event To Tag`
  operate on every match. The popup lists tags with usage counts and can create new ones.
- Action parameters accept numbers and text. Prefix a text field with `$` to read an FSM variable, for
  example `$counter`; `Set Variable` writes numbers back.
- FSM actions can write variables on another FSM, send it an event, jump to a state, restart it, or
  enable/disable it. A disabled FSM skips updates but keeps its data.
- Timers expose a runtime `running` flag: `Start Timer` (with optional period and repeat), `Stop Timer` and
  `Reset Timer` control them without removing the component. Stopping Play restores every FSM and timer.
- `PHP Request` sends a GET/POST with `fetch`, stores the reply in `PM.web.replies`, writes it to a variable
  and sends a reply/error event. `PHP Reply Receive` waits for (or immediately consumes) a stored reply with
  an optional timeout. `Open Link` opens a URL in a tab. The PHP endpoint is external — this project ships no
  PHP runtime or server code — and must allow CORS for cross-origin browser requests.
- Action catalog: **84 actions in 15 categories** — Flow, FSM, Variables, Events, Transform, Material, Texture,
  Game Object, Rendering, Animation, Physics, Network, Timers, Web and Input. It includes event/parent/child
  messaging, object hierarchy and naming actions, camera/opacity rendering actions, collider tuning, extra
  network role/room actions, texture tiling, motion blending, keyboard/gamepad actions and **Change Scene**.
  `UE.PlayMaker.defineAction()` / `undefineAction()` add or remove custom actions at runtime. The manual generates
  the complete searchable parameter reference from the same catalog at runtime.
- The **Lights & PlayMaker** sample wires all of it together: a Sun override, a timer-pulsed point light with
  a rings cookie, a spot light with a grid cookie opened by an Event Proxy, and an orb FSM with
  Idle/Patrol/Return/Spin states and an `E`-key Boost.

### Persistence
- Autosave to `localStorage` 1.2 s after an edit, plus Ctrl+S for an immediate save.
- Export a scene to a readable `.json` file; import by file picker or drag-and-drop onto the viewport.
  Exported files include every referenced model, texture, audio clip and video clip, so they open on another machine
  without the source files.
- Five sample scenes: Starter, Physics Lab, Material Gallery, Lights & PlayMaker and Empty.
- A **preload scene menu** opens at startup after the default scene is ready. `Alt+O`, **File → Change Scene…**,
  the Project scene list or `UnirealEngine.changeScene(id)` switch to any sample, the saved scene or a blank scene.
  The PlayMaker **Change Scene** action calls the same function and stops Play before loading.

## Controls

| Action | Input |
| --- | --- |
| Orbit / pan / zoom | RMB or Alt+LMB / MMB / wheel |
| Move / rotate / scale | `W` / `E` / `R` |
| Frame selection / delete | `F` / `Del` |
| Snap while dragging | hold `Ctrl` |
| Play / pause / step / stop | `Ctrl+P` / `Ctrl+Shift+P` / `Ctrl+.` / `Esc` |
| Undo / redo | `Ctrl+Z` / `Ctrl+Y` |
| Duplicate / copy / paste | `Ctrl+D` / `Ctrl+C` / `Ctrl+V` |
| Save / new scene | `Ctrl+S` / `Ctrl+N` |
| Preload / change scene | `Alt+O` |
| Help & scripting reference | `F1` |
| Full manual (search with `Ctrl+K`) | `Ctrl+Shift+F1` |

Most editor shortcuts are defaults, not hard-coded. Open **Tools → Keyboard & Gamepad Settings…** to rebind them,
reset individual bindings, and set the gamepad deadzone. The panel also shows connected controllers, four axis
bars and all 16 polled buttons. Settings are stored in this browser under `unirealengine.prefs.v1`.

## Script API

Select an object, add a **Script** component, and edit the code in the inspector. The code is a function
body that runs once per fixed step during Play:

```js
// Spin around Y and float using per-object state.
self.transform.rotation.y += 90 * dt;
if (self.data.baseY === undefined) self.data.baseY = self.transform.position.y;
self.transform.position.y = self.data.baseY + Math.sin(time * 2) * 0.3;
```

Available names:

| Name | Meaning |
| --- | --- |
| `self` | the GameObject that owns the script (`transform`, `components`, `data`, …) |
| `dt` | fixed step length in seconds (1/60) |
| `time` | seconds since Play started |
| `scene` | the Scene (`scene.find('Name')`, `scene.objects`, `scene.add(…)`) |
| `UE` | engine namespace (`UE.V3`, `UE.Physics`, `UE.createPrimitive`, …) |
| `self.data` | persistent object state; cleared when Play stops |

Built-in examples in the script dropdown: `spin`, `bob`, `orbit`, `pulse`, `lookAtOrigin`.

Animation is scriptable too: `self.getComponent('Animator')` exposes `play`, `crossFadeTo`, `setBlend`, `stop`
and `boneMatrices`, and `UE.Animation.Library` resolves names or indexes and samples any of the 99,999 motions.

The editor and WebGL renderer are verified in the browser; open the page and check the Console for errors.

## Layout

```text
UnirealEngine/
  index.html            editor markup: menus, panels, viewport, console, help modal
  manual.html           searchable user manual and complete action reference
  css/styles.css        dark Unity-style theme, layout grid, responsive rules
  css/manual.css        manual theme, sidebar, action cards and search results
  js/math.js            Vec3, Quat, Mat4, AABB and Ray helpers (no DOM)
  js/engine.js          scene graph, components, asset store, serialization, physics, scripts
  js/models.js          OBJ / STL / GLB / glTF importers and mesh normalization
  js/animation.js       humanoid auto-rig, skin weights, procedural 99,999-motion library, Animator
  js/playmaker.js       PlayMaker FSMs, 84 actions, custom action API, scene changer, triggers, timers, input, network, web/PHP
  js/extras.js          object presets, FSM templates, network roles, texture tools and keyboard/gamepad input
  js/audio.js           WebAudio runtime: native decode, 3D panners, listener and media-element audio bridge
  js/video.js           HTMLVideoElement runtime: playback, video textures and spatial-audio routing
  js/network.js         browser-only multiplayer: rooms, transports, ownership, roles and state replication
  js/renderer.js        WebGL renderer: meshes, lights, cookies, grid, shadows, gizmos, skinning, video textures and PiP
  js/samples.js         example scripts and the five sample scenes
  js/editor.js          editor shell: preload/scene changer, panels, gizmos, Game view/PiP, Tools popups, network panel, persistence
  js/manual.js          manual navigation, action catalog and live search
  tests/index.html      browser test runner page
  tests/core.test.js    78 groups: math, scene, physics, scripts, importers, animation, PlayMaker, extras, media, network
```

## Limits (on purpose)

- Colliders are world-axis AABBs; rotated boxes are approximated. There is no rotational dynamics, angular
  drag or joint system. Trigger colliders report contacts to PlayMaker but do not push objects apart.
- Eight dynamic lights (directional/point/spot) plus the Sun override. Cookies share a 512×512 atlas with
  16 cells, but only the eight dynamic light slots are uploaded, so at most eight cookies render at once.
- Lights have no shadow maps; `castShadow` is reserved and the renderer draws soft contact shadows only.
- `Open Link` depends on the browser's popup policy. `PHP Request` needs network access and a CORS-enabled
  endpoint; the browser performs the request, so there is no PHP interpreter or backend in this project.
  Replies are plain text or JSON and arrive asynchronously.
- Models support base-color textures only: no normal maps, metallic/roughness maps, emissive maps, morph targets
  or alpha cutout, and imported skin/clip data is not read. Use **Auto Rig & Animate** for the built-in 20-bone
  humanoid rig; imported skeletons and animation clips are not converted. Tiling and offset are editable in the
  inspector and from PlayMaker, but imported `KHR_texture_transform` values are not read. OBJ materials contribute
  `Kd` and `map_Kd`; the inspector tint and UV controls apply to the imported texture. Draco and
  meshopt-compressed glTF/GLB files are not decoded; re-export them with compression disabled. The import limit is
  6,487,965 vertices (65,535 × 99) and 7,920,000 triangles (80,000 × 99) per mesh, exported as `UE.MODEL_LIMITS`.
- Meshes with more than 65,535 vertices use `Uint32Array` indices and are drawn with `gl.UNSIGNED_INT`. That
  needs the WebGL1 `OES_element_index_uint` extension (exposed by every current desktop browser); if it is
  missing, the mesh is rejected with a console error instead of drawing corrupt geometry. Wireframe edge lists
  are skipped for meshes above 600,000 indices (about 200,000 triangles) to keep import time and memory sane.
- There is no application-level file-size cap. Uploads are bounded only by browser memory; persistence uses
  `localStorage`, which typically tops out around 5–10 MB per origin, so very large imported scenes may exceed
  the autosave quota and live in memory only for that session.
- One world, no navmesh, imported animation clips, prefabs or asset database. The motion library is procedural:
  99,999 generated humanoid clips rather than mocap data, and auto-rigging is a heuristic fit rather than
  artist-authored skinning. Scenes are small enough for the editor's simple `O(n²)` collision pass.
- Media uses browser codecs only: WAV/MP3/OGG/M4A audio and MP4/WebM/OGV video play only when the current browser
  supports that format. There is no transcoding, subtitles, video seeking UI, per-material video UV selection or
  audio effect chain. Auto-play can be blocked until the user interacts with the page.
- Multiplayer is browser-local: `BroadcastChannel` (or the `localStorage` fallback) connects tabs in the same
  browser and origin, not different machines. There is no authoritative server, authentication, encryption,
  room persistence, lag compensation or rollback. Ownership is cooperative; any tab can claim an object. Network
  transform interpolation is linear and rotation is synchronized as Euler angles, with no angular velocity.
- Not a Unity replacement: it is a compact, readable editor/engine that demonstrates the same ideas in
  roughly 14,000 lines of dependency-free HTML, CSS and JavaScript.
