# Solar System 3D — Features & Problems Gap Analysis

> **World Mechanism Reference:** [mneme-world-generator-pwa](https://github.com/Game-in-the-Brain/mneme-world-generator-pwa)
> This repository defines the world-generation mechanics, object hierarchy, and data schema that Solar System 3D should align with for cross-tool consistency.

---

## Definitions: The INRAS Object Hierarchy

**INRAS** — *Intrastellar Elements* — is the collective term for all significant non-planetary objects within a stellar system: stars, exotic bodies, megastructures, rogue objects, and hypothetical bodies. The term is deliberately broad to accommodate both real astrophysics and speculative/fictional world-building.

The INRAS hierarchy is a parent-child tree of unlimited depth. Each level inherits orbital context from its parent:

| Level | Name | Description | Examples |
|-------|------|-------------|---------|
| **Level 1** | **INRAS / Root Bodies** | The primary stellar and sub-stellar objects anchoring a system. These are the top-level parents — they orbit the system barycenter or are fixed reference points. | Stars, brown dwarfs, black holes, neutron stars, Oort Cloud shells, hypothetical mega-bodies |
| **Level 2** | **Primary Children** | Objects that orbit a Level-1 body directly. The classic "planet" — but in the INRAS model, this includes any first-order satellite of a stellar or quasi-stellar object. | Planets, dwarf planets, captured rogue bodies, close-orbit stations |
| **Level 3** | **Secondary Children** | Objects that orbit a Level-2 body. Classic "moons," but also includes artificial structures in planetary orbit. | Natural moons, space stations in planetary orbit, orbital habitats |
| **Level 4+** | **Deep Children** | Each successive level is a child of the level above. The hierarchy has no hard cap — a moonlet orbiting a moon orbiting a planet orbiting a star is a Level-4 object. | Moonlets, subsatellites, docking platforms attached to habitats |

**Key Rules:**
- A Level-1 INRAS body is always the root of at least one orbital tree.
- Any body may be both a child (has a parent) and a parent (has children).
- Orbital parameters are always expressed relative to the parent body, not the system origin.
- The `mneme-world-generator-pwa` data schema is the canonical reference for field names and units across all Game in the Brain tools.

---

## Current State Summary

The current `script.js` (~800 lines) is a **hardcoded scene file**: all planets, their orbital radii, speeds, sizes, textures, and info panels are written as direct function calls. There is no runtime data layer, no editor UI, no persistence layer, and no mechanism to add or modify objects without editing source code.

**What exists today:**
- `createPlanet()` factory function — builds a planet mesh hierarchy (planet3d → planetSystem → planet mesh)
- `dat.GUI` — exposes only three global controls: `accelerationOrbit`, `acceleration`, `sunIntensity`
- `planetData` object — static key/value info strings shown in the info panel on click
- `raycastTargets` array — manually maintained list of clickable mesh objects
- `animate()` loop — hardcoded `rotateY()` calls per planet for both axis rotation and orbital motion

**What is missing:** everything described below.

---

## Gap 1 — INRAS Level-1 Editor: Edit Root Body Properties

**Priority: Highest — all other gaps depend on this foundation.**

This is the ability to select any Level-1 INRAS body in the 3D view and edit its live properties through a panel UI, with changes reflected in the viewport in real time and an explicit Apply/Refresh mechanism.

### Problem Statement

Currently, changing any orbital or physical parameter of the Sun or any top-level body requires editing `script.js` directly, rebuilding, and reloading. There is no separation between the data model and the rendering layer. The `createPlanet()` function consumes parameters at construction time — there is no update path.

### Required Features

#### GAP-1.1 — Data Model Separation
- All planet/body parameters must be extracted from `script.js` into a structured JSON data object (or module)
- Each body entry must carry all fields needed for both rendering and the editor panel
- This data object is the single source of truth; `createPlanet()` and the animator both read from it
- Schema should align with the `mneme-world-generator-pwa` field definitions

**Fields required per Level-1 body:**
```
name                string
type                "star" | "brown_dwarf" | "black_hole" | "neutron_star" | "oort_shell" | "hypothetical"
spectral_class      string (e.g. "G2V")
radius_visual       number   — compressed scene units
luminosity          number   — relative (1.0 = Sol)
colour_temperature  number   — Kelvin
bloom_intensity     number   — UnrealBloomPass strength
corona_size         number   — bloom radius
position            { x, y, z }  — for multi-star systems; barycenter-relative
rotation_period     number   — seconds per full rotation (scene time)
axial_tilt          number   — degrees
texture_map         string   — path
emissive_map        string   — path (for self-illuminating bodies)
emissive_colour     hex string
notes               string   — free-text editorial notes
```

#### GAP-1.2 — Orbital Parameter Controls
The following orbital parameters must be editable per body in the editor panel and update the live scene:

| Parameter | Description | Current State |
|-----------|-------------|---------------|
| **Orbital Velocity** | `planet3d.rotateY()` multiplier in `animate()` — how fast the body completes one orbit | Hardcoded per planet in `animate()` |
| **Axial Rotation Speed** | `planet.rotateY()` multiplier — how fast the body spins on its own axis | Hardcoded per planet in `animate()` |
| **Orbital Plane / Inclination** | The tilt of the orbital plane relative to the ecliptic | Not modelled — all orbits are flat on Y=0 |
| **Orbital Axis / Argument of Periapsis** | Orientation of the orbit ellipse within its plane | Not modelled |
| **Axial Tilt** | `rotation.z` on the planet mesh at creation | Set once at `createPlanet()` — no update path |
| **Orbit Radius** | Distance from parent body, sets the `planet3d` position | Set once at `createPlanet()` — no update path |
| **Orbit Eccentricity** | Shape of the orbit ellipse (0 = circle, >0 = ellipse) | Not modelled — all orbits are circular |

**Gap:** The `animate()` loop applies rotation increments directly as magic numbers per planet. To make these editable, each body's animation parameters must live in the data model, and the `animate()` loop must read from that model per frame.

#### GAP-1.3 — Editor Panel UI
- Collapsible sidebar or floating panel
- Opens when a Level-1 body is selected (click or panel list)
- Grouped sections: Identity, Physical, Orbital Motion, Visual/Shader
- All fields editable inline (number inputs, sliders, colour pickers, text fields)
- Changes visible live in the 3D viewport as values are modified

#### GAP-1.4 — Apply / Refresh Mechanism
- An explicit **Apply** button commits the current panel state to the data model
- **Refresh** rebuilds affected scene objects from the updated data model (needed when structural properties like orbit radius or texture path change)
- Live preview (during edit, before Apply) uses direct object mutations; Apply+Refresh triggers a clean rebuild
- **Reset to Default** per body restores bundled values
- Changes persist to `localStorage` as a JSON override; a **Clear Saved Overrides** option restores all defaults

**Why Apply/Refresh is needed separately from live preview:**
Some changes (orbit radius, adding/removing rings, swapping texture paths, changing mesh geometry) cannot be applied incrementally — the scene object must be destroyed and recreated. Live preview handles cheap mutations (rotation speed, tilt angle, colour); Apply+Refresh handles structural rebuilds.

#### GAP-1.5 — Orbital Plane / Axis Implementation Gap
This is the deepest technical gap in this cluster. Currently all orbital planes are flat (Y-axis rotation on `planet3d`). To support inclined orbits:

- Each body's `planet3d` wrapper needs a parent `orbitPlane` group that carries the inclination rotation
- `orbitPlane.rotation.x = inclination` sets the orbital tilt
- `planet3d.rotateY()` in `animate()` then orbits within that tilted plane
- The orbit path visual (the white ellipse) must be regenerated to match the inclined plane
- Axial tilt (`rotation.z` on the planet mesh) remains independent of orbital inclination

**Affected objects:** Every planet in the scene requires this structural change for the feature to work at all.

---

## Gap 2 — INRAS Children Editor: Add & Edit Level-2, Level-3, and Deeper Objects

**Priority: High — depends on Gap 1 data model being in place.**

This is the ability to add children to any body in the hierarchy and edit their properties using the same editor panel pattern established in Gap 1.

### Problem Statement

The current system has no concept of a parent-child hierarchy beyond the implicit Sun → Planet relationship baked into `createPlanet()`. Moons are a hardcoded array parameter at planet-creation time. There is no way to add a moon to an existing planet at runtime, edit moon properties, or create deeper nesting (moonlets, orbital stations as Level-3+ objects).

### Required Features

#### GAP-2.1 — Hierarchy Data Model
- The data JSON must support recursive children: each body entry can have a `children[]` array
- Each child is a full body entry with its own fields, including its own `children[]`
- The renderer traverses the tree recursively to build the scene hierarchy
- Example structure:
```json
{
  "id": "sol",
  "level": 1,
  "type": "star",
  "children": [
    {
      "id": "earth",
      "level": 2,
      "type": "planet",
      "orbital_radius": 90,
      "children": [
        {
          "id": "luna",
          "level": 3,
          "type": "moon",
          "orbital_radius": 10
        }
      ]
    }
  ]
}
```

#### GAP-2.2 — Add Child Flow
- In the editor panel for any body, an **Add Child** button opens a "New Object" form
- User selects child type (Planet / Moon / Dwarf Planet / Station / Asteroid / Custom)
- Required fields for that type are pre-populated with sensible defaults
- On confirm, the child is added to the parent's `children[]` in the data model, and the scene is refreshed
- The new object appears in the 3D view orbiting its parent immediately

#### GAP-2.3 — Child Orbital Parameters (Same as GAP-1.2, scoped to children)
All orbital parameters from GAP-1.2 apply equally to Level-2, Level-3, and deeper objects. The editor panel is the same component — it just receives a different body's data as input.

**Additional child-specific orbital fields:**
- `parent_id` — reference to parent body
- `orbital_radius` — distance from parent (not from system origin)
- `orbital_period` — time for one full orbit of parent
- `orbital_inclination` — tilt relative to parent's equatorial plane

#### GAP-2.4 — Edit Existing Children
- Clicking any Level-2+ body in the 3D scene opens its editor panel (same as Level-1)
- The panel header shows the full ancestry path: `Sol → Earth → Luna`
- "Up" button navigates to the parent body's panel
- All fields editable; Apply/Refresh works the same way

#### GAP-2.5 — Remove Child
- A **Remove** button in the child's editor panel deletes the body from its parent's `children[]` and removes it from the scene
- Confirmation dialog for safety
- Removing a parent also removes all its descendants

---

## Gap 3 — Information Text Editor: Edit & Format Info Panel Content

**Priority: Medium-High — usability gap independent of data model complexity.**

### Problem Statement

The `planetData` object in `script.js` contains static strings displayed in the info panel when a body is clicked. These strings are hardcoded, cannot be edited at runtime, and support no formatting (no headings, no lists, no links). The panel layout is fixed HTML.

### Required Features

#### GAP-3.1 — Editable Info Fields
- Each body in the data model carries an `info` object with structured fields (not a single flat string):
```json
"info": {
  "headline": "Earth",
  "subtitle": "Third planet from the Sun",
  "body": "Markdown or rich text...",
  "stats": [
    { "label": "Radius", "value": "6,371 km" },
    { "label": "Day", "value": "23h 56m" },
    { "label": "Orbital Period", "value": "365.25 days" }
  ],
  "links": [
    { "label": "NASA Fact Sheet", "url": "https://..." }
  ]
}
```
- The editor panel includes a text area for `body` content and a stats table editor
- Stats rows can be added, edited, reordered, and removed

#### GAP-3.2 — Markdown / Rich Text Support
- The `body` field supports a subset of Markdown: headings, bold/italic, bullet lists, numbered lists, blockquotes
- A rendered preview updates as the user types
- The info panel in the 3D view renders the Markdown to HTML

#### GAP-3.3 — Info Panel Layout Options
- Panel can be toggled between two layouts: **Compact** (stat pills only) and **Expanded** (full body text + stats)
- Layout preference persists per-body in the data model

#### GAP-3.4 — Custom Stat Categories
- Stats are categorised: Physical, Orbital, Atmospheric, Human Interest
- Categories can be toggled visible/hidden in the info panel
- User can add custom categories

---

## Gap 4 — Add Objects: New Bodies Beyond the Existing Set

**Priority: Medium — depends on Gaps 1 and 2 being established.**

This gap covers the ability to add entirely new objects to the scene that are not children of existing INRAS bodies — i.e., new Level-1 root bodies (additional stars, exotic objects) or new Level-2 planets in a multi-star system.

> **Note:** Adding children to existing bodies is Gap 2. Gap 4 is specifically about adding new root-level objects or importing entirely new system configurations.

### Problem Statement

Currently, the scene contains exactly the objects hardcoded in `script.js`. There is no "Add Object" flow, no object type registry, and no import path for external body definitions.

### Required Features

#### GAP-4.1 — "Add Object" Button & Type Registry
- A top-level **+ Add Object** button in the UI (distinct from the "Add Child" flow in Gap 2)
- Opens a type selector: Star / Brown Dwarf / Black Hole / Neutron Star / Rogue Planet / Oort Shell / Hypothetical / Custom
- Each type has a default data template (sensible placeholder values)
- New object appears at the selected orbital radius around the system barycenter

#### GAP-4.2 — Multi-Star System Support
- Multiple Level-1 bodies are supported simultaneously
- Binary/trinary star orbital parameters: semi-major axis, eccentricity, period relative to barycenter
- Each star independently manages its `children[]` tree
- The system barycenter becomes the true origin point; all Level-1 bodies orbit it

#### GAP-4.3 — JSON Import / Export
- **Export:** Downloads the full current scene data as `solar-system-data.json`
- **Import:** Accepts a previously exported JSON file; validates schema before applying
- On load, the app checks `localStorage` for a saved override and uses it instead of bundled defaults
- Schema documented inline in the export file
- Compatible with the `mneme-world-generator-pwa` data format for cross-tool portability

#### GAP-4.4 — Object Templates Library
- A small built-in library of named templates: Sol-Like Star, Red Dwarf, Gas Giant, Terrestrial, Ice Moon, Space Station
- User selects a template as a starting point; fields are pre-filled
- Custom templates can be saved to the library from any body's editor panel

---

## Gap 5 — 3D Models: Planet Generator, Icosahedral Maps, and Fractal Mapping

**Priority: Medium — can begin in parallel with Gaps 1–3, but requires a stable data model to bind model assignments.**

This gap covers the integration of procedurally generated and imported 3D models as replacements for the default sphere geometry. It draws on techniques from the Game in the Brain Blender World Generator pipeline and known open-source procedural generation methods.

### Problem Statement

All bodies currently use `THREE.SphereGeometry` with flat textures. There is no model assignment system, no procedural surface generation, and no pipeline for importing externally generated meshes. The `FR-MOD-01` and `FR-PWG-01` entries in the PRD define the goal but not the implementation approach.

### Required Features

#### GAP-5.1 — GLB Model Assignment
- Any body in the data model can have a `model_path` field pointing to a `.glb` file
- When present, the body's sphere geometry is replaced by the loaded GLB mesh at scene build time
- The GLB respects the same scale compression factor as sphere geometry
- Material slots in the GLB (normal, roughness, emissive) are read and applied
- Drag-and-drop GLB onto the editor panel to assign a model live

**Current partial implementation:** Phobos and Deimos (Mars moons) already use GLTFLoader — this is the pattern to generalise.

#### GAP-5.2 — In-Browser Procedural Planet Generator

> **Technique reference:** Based on the approach studied by Justin and Nicco during the GiB World Generator work.

The core technique is noise-based sphere displacement:
- A high-subdivision `THREE.IcosahedronGeometry` (detail level 5–7) is used as the base mesh
- Per-vertex displacement is driven by layered **simplex or Perlin noise** (multiple octaves for fractal detail)
- Displacement magnitude is controlled by terrain parameters: `roughness`, `amplitude`, `sea_level`
- Vertex colours or a generated texture map encode biome/altitude bands
- The result is a fully procedural, non-repeating planet mesh generated at runtime with no external assets

**Generator parameters (editor-accessible):**
```
seed            number   — deterministic noise seed
terrain_type    "terrestrial" | "gas_giant" | "ice" | "rocky" | "volcanic" | "ocean"
roughness       0.0–1.0  — noise octave contribution scaling
amplitude       0.0–1.0  — max displacement height relative to radius
sea_level       0.0–1.0  — fraction of surface below "water"
atmosphere      boolean
atmosphere_density  0.0–1.0
cloud_coverage  0.0–1.0
colour_palette  preset or custom gradient
```

**Gap:** This capability does not exist at all in the current codebase. It requires:
1. A noise library (e.g. `simplex-noise` npm package or a custom GLSL shader implementation)
2. A `PlanetGenerator` class that builds the geometry from parameters
3. Editor panel integration (sliders for all parameters, live preview)
4. An optional **Regenerate** button that applies a new random seed while keeping all other parameters

#### GAP-5.3 — Icosahedral Map Export

The procedurally generated planet mesh is based on an icosphere, which means it maps cleanly to an **icosahedral projection** — a technique for unfolding a sphere onto a flat map with minimal distortion. This is useful for world-building: the exported map can be used in other tools (Wonderdraft, Inkarnate, GIMP, the `mneme-world-generator-pwa`) as a base map.

**Implementation approach:**
- Render the planet mesh from a fixed set of camera angles corresponding to the 20 icosahedron faces
- Composite the renders into a flat icosahedral net image
- Export as PNG or SVG
- Optionally export a heightmap (grayscale displacement values) separately from the colour/albedo map

**Gaps:**
- No render-to-texture pipeline exists in the current code
- No UV unwrapping to icosahedral projection exists
- Requires an off-screen `WebGLRenderTarget` pass dedicated to the export

#### GAP-5.4 — Fractal Mapping Pipeline

Fractal mapping extends the procedural generator by applying successive levels of fractal noise at increasing detail scales. The key properties:

- **Self-similarity:** Zooming into any region reveals detail at the same statistical character as the global surface — coastlines stay coastlines, mountain ridges stay mountain ridges
- **Biome propagation:** High-level biome assignments (ocean, desert, forest, ice) at low frequency; fine-grained surface features at high frequency
- **LOD-aware generation:** At far zoom, only low-frequency noise is computed; detail octaves are added as the camera zooms in (same principle as `FR-PWG-03` Low-LOD Mode in the PRD)

**Implementation approach:**
- Extend `PlanetGenerator` with a multi-octave fractal Brownian motion (fBm) stack
- Each octave doubles the frequency and halves the amplitude (Hurst exponent ~0.5 for natural terrain)
- Biome assignment via a 2D lookup table: `f(altitude, moisture)` → biome colour/material
- Progressive LOD: base geometry at IcosahedronGeometry detail 2; subdivide and re-displace on zoom

**Gaps:**
- fBm stack implementation does not exist
- Biome lookup table does not exist
- LOD subdivision system does not exist
- The `FR-PWG-03` entry in the PRD names the goal but the implementation path is unspecified

#### GAP-5.5 — Ship and Station Models

Per `FR-MOD-02` and `FR-MOD-03` in the PRD:
- Ship type: a body that moves along a trajectory rather than a fixed orbit
- Station/Habitat type: a body in orbit with interior dimension data (for later Logistics Planner integration)
- Both accept GLB models from the GiB Ship Generator export pipeline
- Ships support idle animation (thruster glow via emissive material pulsing, slow yaw rotation)

**Gap:** No `Ship` or `Station` type exists; no trajectory animation system exists; no interior data fields exist in the data model.

---

## Implementation Order & Dependencies

```
Gap 1 — INRAS Level-1 Editor
  └── Data model separation (GAP-1.1) ← BLOCKER for everything below
      ├── Orbital parameter controls (GAP-1.2)
      ├── Editor panel UI (GAP-1.3)
      └── Apply/Refresh mechanism (GAP-1.4)
          └── Orbital plane/axis (GAP-1.5) ← requires structural scene change

Gap 2 — Children Editor
  └── Requires: GAP-1.1 (data model), GAP-1.3 (editor UI pattern)
      ├── Hierarchy data model (GAP-2.1)
      ├── Add Child flow (GAP-2.2)
      └── Edit/Remove child (GAP-2.3 → 2.5)

Gap 3 — Info Text Editor
  └── Requires: GAP-1.1 (data model has info fields), GAP-1.3 (editor panel exists)
      ├── Structured info fields (GAP-3.1)
      ├── Markdown support (GAP-3.2)
      └── Layout options (GAP-3.3 → 3.4)

Gap 4 — Add Objects
  └── Requires: GAP-1.1, GAP-2.1, GAP-4.3 (JSON schema stable before import/export)
      ├── Add Object button & type registry (GAP-4.1)
      ├── Multi-star system support (GAP-4.2)
      └── JSON import/export (GAP-4.3)

Gap 5 — Models
  └── GAP-5.1 (GLB assignment) requires: GAP-1.1 data model field
  └── GAP-5.2 (Procedural generator) can begin in parallel with Gaps 1–3
  └── GAP-5.3 (Icosahedral export) requires: GAP-5.2 (mesh exists to export)
  └── GAP-5.4 (Fractal mapping) requires: GAP-5.2 (extends the generator)
  └── GAP-5.5 (Ships/Stations) requires: GAP-2.1 (children model), GAP-5.1 (GLB pipeline)
```

---

## Known Technical Debt (Current Codebase)

These issues exist today and will need to be resolved during Gap 1 implementation:

| Issue | Location | Description |
|-------|----------|-------------|
| `venusBump` imports wrong file | `script.js` top imports | `venusBump` imports `venusmap.jpg` (diffuse) instead of `venusbump.jpg` |
| Pluto missing bump map | `createPlanet('Pluto', ...)` call | `plutobump2k.jpg` exists in `src/images/` but is not passed to `createPlanet` |
| All orbits are flat (Y=0) | `animate()` loop | `rotateY()` on `planet3d` means all orbital planes are identical — inclination is not modelled |
| Orbital velocity hardcoded | `animate()` | Each planet's orbital speed is a magic number, not data-driven |
| `postprocessing` package unused | `package.json` | Listed as dependency but all post-processing uses `three/addons` — should be removed |
| `door.jpg` in `static/` | `static/door.jpg` | Unknown/unused file |
| Unused textures in `src/images/` | Various | `mercury.jpg`, `moon.jpg`, `earth_normalmap.jpg`, `earth_specularmap.jpg`, `jup1vuu2.jpg` imported or present but not used |

---

## Glossary (Extended)

| Term | Definition |
|------|-----------|
| **INRAS** | Intrastellar Elements — the top-level non-planetary significant bodies of a stellar system (Level-1 in the hierarchy) |
| **Level-1** | Root INRAS bodies: stars, exotic objects, system anchors |
| **Level-2** | Primary children of Level-1 bodies: planets, dwarf planets, primary orbital structures |
| **Level-3** | Secondary children: moons, orbital stations, rings treated as discrete objects |
| **Level-4+** | Any body nested deeper than Level-3; the hierarchy has no maximum depth |
| **mneme-world-generator-pwa** | The Game in the Brain world-generation PWA whose data schema and object model defines the canonical field names and hierarchy for all GiB tools |
| **Apply/Refresh** | The two-stage commit mechanism: Apply writes changes to the data model; Refresh rebuilds affected scene objects |
| **fBm** | Fractal Brownian Motion — multi-octave noise summation used to generate natural-looking terrain |
| **Icosahedral projection** | A method of unfolding a sphere onto a 2D map using the 20 faces of an icosahedron; minimises distortion compared to Mercator |
| **LOD** | Level of Detail — using simpler geometry for distant objects and adding detail on zoom |
| **GiB World Generator** | Game in the Brain's Blender-based procedural asset pipeline for planets, ships, and stations |
| **Compressed scale** | The visual model shrinks orbital distances and body sizes to fit on screen — not 1:1 with real measurements |
| **Barycenter** | The centre of mass of a multi-body system; the point all bodies orbit in a multi-star configuration |
