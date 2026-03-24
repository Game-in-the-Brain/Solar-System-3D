# Solar System 3D — Product Requirements Document
### Backlog & Roadmap

---

## Scale: Two Modes, One Data Model

> **Presentation Scale** — the compressed, human-viewable representation used in the 3D viewport. Orbital distances and body sizes are scaled down so everything fits on screen. This is the artistic model used for exploration, editing, and navigation.
>
> **Real Distance Scale** — true astronomical measurements: AU, parsecs, km/s, days. Used in the Logistics Planner, the Procedural Star System Generator, and the Real Star Map. Nothing at Real Distance Scale is derived from or constrained by what fits on screen.
>
> Every body in the data model carries **both** a `presentation` orbit radius and a `real` semi-major axis / stellar distance. These are never mixed. The 3D viewport renders Presentation Scale; calculators and the star map operate on Real Distance Scale.

---

## Roadmap Overview

```
Phase 1 ──► Data Editor (planets editable in-browser)
Phase 2 ──► Extend Objects (dwarf planets, space stations)
Phase 3 ──► INRAS Editor (intrastellar elements + stars)
Phase 4 ──► 3D Model Integration (GiB World Generator / Blender)
Phase 5 ──► Procedural Ship & Habitat/Station Builder
             │
             └──► Logistics Planner (L1–L5)
                         (Distances → Delta-V → Travel Time
                          → Multi-Leg → Scheduled Routes
                          → intrastellar logistics patterns)
                         │
                         └──► Phase 6A: Procedural Star System Generator
                                        (mneme-world-generator-pwa integration
                                         — full systems from seed, Real Distance Scale)
                                        │
                                        └──► Phase 6B: Real Star Map
                                                       (Sol-centred, true parsec distances
                                                        — shift-select → distance/route totals
                                                        — save/load star systems
                                                        — click star → load its system)
```

---

## Phase 1 — In-Browser Data Editor

**Goal:** Give users a live panel to inspect and edit all planet/moon properties without touching source code. Changes persist via a downloaded JSON file that can replace the bundled data.

### FR-ED-01 — Data Panel UI
- A collapsible sidebar or modal accessible from the main view
- Selecting any planet or moon opens its data sheet
- Fields: name, radius (visual), orbit radius, orbital period, axial tilt, rotation speed, texture paths, bump map, ring present, atmosphere present

### FR-ED-02 — Edit & Preview
- All fields are editable inline
- Changes apply live to the 3D viewport so users can preview before saving
- "Reset to defaults" per object

### FR-ED-03 — Export as JSON
- "Save / Export" button downloads `solar-system-data.json`
- JSON schema documents every field with type and units
- On next load, the app checks for a locally stored override JSON (localStorage) and uses it instead of the bundled defaults

### FR-ED-04 — Import JSON
- "Import" button accepts a previously exported JSON file
- Validates schema before applying; shows field-level errors on mismatch

---

## Phase 2 — Extend: Dwarf Planets, Asteroids & Space Stations

**Goal:** Allow the addition of new objects beyond the original 9 planets — starting with named dwarf planets/large asteroids and artificial structures.

### FR-EX-01 — Dwarf Planet / Large Asteroid Support
Objects to add first:
| Object | Type | Belt |
|--------|------|------|
| Ceres | Dwarf Planet | Main Asteroid Belt |
| Eris | Dwarf Planet | Scattered Disc |
| Haumea | Dwarf Planet | Kuiper Belt |
| Makemake | Dwarf Planet | Kuiper Belt |
| Sedna | Dwarf Planet | Outer Solar System |

- Each uses the same `createPlanet()` pipeline as existing planets
- Orbit and size data sourced from real measurements, scaled by the same compression factor as existing planets
- Data Editor (Phase 1) supports adding these via the "Add Object" button

### FR-EX-02 — Space Station Support
- New object type: `createStation(name, orbitTarget, orbitRadius, orbitPeriod, model)`
- Stations orbit a parent body (planet, moon, or sun) rather than having their own orbit around the sun
- Initial stations: ISS (Earth orbit), Gateway (Lunar orbit) as reference examples
- Stations can use a simple placeholder geometry (box/cylinder) until a GLB model is assigned

### FR-EX-03 — "Add Object" Flow in Data Editor
- User selects type: Planet / Dwarf Planet / Asteroid / Station
- Fills required fields in the panel
- Object appears immediately in the 3D view
- Saved to local JSON on export

---

## Phase 3 — INRAS Editor

**INRAS** = *Intrastellar Elements* — any non-planetary body of significance: stars, stellar remnants, rogue objects, artificial megastructures, and hypothetical bodies.

> This phase establishes the editing pipeline for star-level and exotic objects, building on the same data editor from Phase 1.

### FR-INRAS-01 — Star Data Sheet
Fields for any star object:
- Name, spectral class, luminosity (relative), radius (visual scale), colour temperature
- Position (x, y, z) for multi-star systems
- Rotation period
- Bloom intensity and corona size (post-processing parameters)

### FR-INRAS-02 — Star Orbit & Rotation Editor
- Binary/trinary star orbital parameters (semi-major axis, eccentricity, period)
- Each star's axial rotation speed
- Preview renders in real time in the 3D viewport

### FR-INRAS-03 — INRAS Object List
Additional object types beyond stars:
| Type | Examples |
|------|---------|
| Brown Dwarf | Nemesis hypothesis, rogue objects |
| Neutron Star / Pulsar | Reference / hypothetical |
| Black Hole | Sagittarius A* scale reference |
| Oort Cloud Shell | Visual boundary representation |
| Hypothetical Bodies | Planet Nine, Tyche |

### FR-INRAS-04 — 3D Model Assignment for INRAS
- Any INRAS object can have a `.glb` model assigned from the data editor
- Model replaces the default sphere geometry
- Supports models from the **Game in the Brain Blender World Generator**
- Drag-and-drop GLB into the panel → model loads live in the viewport

---

## Phase 4 — 3D Model Integration (Game in the Brain World Generator)

**Goal:** Connect the solar system viewer to Game in the Brain's Blender-based World Generator pipeline so procedurally generated planet and station models can be dropped directly into the scene.

### FR-MOD-01 — GLB Model Pipeline
- Any planet, moon, station, or INRAS object accepts a `.glb` file via the data editor
- Models respect the same scale compression as the rest of the scene
- Normal maps, roughness maps, and emissive maps read from the GLB material slots

### FR-MOD-02 — Ship Models
- New object type: `Ship` — moves along a defined trajectory (orbit or point-to-point path)
- Ships can have idle animation (thruster glow, slow rotation)
- Source: Game in the Brain Ship Generator exports

### FR-MOD-03 — Habitat / Station Models
- Habitat type is a Station subtype with interior dimensions stored in data (for Logistics Planner use)
- Supports docking port data (number of berths) relevant for scheduling in Phase 3B

---

## Phase 5 — Procedural Ship & Habitat/Station Builder

**Goal:** Modular construction of ships and space habitats/stations using module-based geometry, compatible with the GLB model pipeline from Phase 4. Source rules and module definitions from [MNEME Space Combat](https://www.drivethrurpg.com/en/product/434090/Mneme-Space-Combat-full).

### FR-SHB-01 — Modular Station Builder
- Core + ring + spoke + docking arm geometry assembled from discrete module meshes
- Parameters: crew capacity, function (mining / transit / research / military), module count
- Generates a THREE.js mesh; optionally exports as GLB

### FR-SHB-02 — Ship Builder
- Ship type: moves along a defined trajectory (orbit or point-to-point path)
- Ships carry idle animation: thruster glow (emissive material pulse), slow yaw rotation
- Source models and module configs from GiB Ship Generator exports

### FR-SHB-03 — Habitat Interior Data
- Each habitat carries interior dimension data and docking port count
- These fields are referenced by the Logistics Planner (Phase L-series) for scheduling and capacity calculations

---

## Phase L — Logistics Planner

> **Scale note:** The 3D viewport remains at Presentation Scale for navigation. All Logistics Planner calculations operate exclusively on **Real Distance Scale** — AU, km/s, days. The two scales are never mixed.

### Milestone L1 — Distance Calculator

**FR-LOG-01 — Real Coordinate System**
- Each body in the data JSON carries its real semi-major axis in AU alongside the visual compressed orbit radius
- At any simulated date, the calculator computes true 3D positions via Keplerian orbital elements

**FR-LOG-02 — Point-to-Point Distance**
- User selects Origin and Destination from a dropdown or by clicking objects in the 3D view
- App displays: current distance (AU and km), minimum distance (closest approach), maximum distance (opposition)

**FR-LOG-03 — Distance Panel UI**
- Floating panel separate from the 3D view
- Live-updates as the simulation time advances
- Shows light-travel time as a reference

---

### Milestone L2 — Delta-V Budget Calculator

**FR-LOG-04 — Delta-V Reference Database**
- Curated table of common transfers: LEO→Moon, Earth→Mars (Hohmann), Earth→Jupiter, etc.
- Values sourced from standard astrodynamics references
- Users can add custom manoeuvres

**FR-LOG-05 — Hohmann Transfer Calculator**
- Input: origin body, destination body, departure window
- Output: total delta-V (m/s), transfer time (days), arrival date
- Assumes circular coplanar orbits as baseline; flags high-inclination penalty

**FR-LOG-06 — Propulsion Profiles**
- User selects propulsion type: Chemical / Ion / Nuclear Thermal / Solar Sail
- Each profile has Isp and thrust-to-weight; calculator adjusts burn time and fuel mass accordingly

---

### Milestone L3 — Travel Time & Journey Visualiser

**FR-LOG-07 — Travel Arc Display**
- When a transfer is calculated, the transfer arc is drawn on the 3D viewport (dashed curve)
- Arc colour-codes by delta-V cost (green = cheap, red = expensive)
- Departure and arrival positions shown as ghost planet markers

**FR-LOG-08 — Time Display**
- Journey duration in days, months, and years
- Arrival date shown in calendar format
- Communication delay at departure, mid-point, and arrival

---

### Milestone L4 — Multi-Leg Journeys

**FR-LOG-09 — Itinerary Builder**
- User adds waypoints: Earth → Mars → Ceres → Jupiter L4
- Each leg calculated independently; total delta-V and time summed
- "Best window" finder suggests optimal departure date for minimum total delta-V

**FR-LOG-10 — Layover & Resupply**
- Each waypoint can have a layover duration (days at station/planet)
- Resupply adds local delta-V cost (launch from surface vs. orbit)

---

### Milestone L5 — Scheduled Routes & Intrastellar Logistics Patterns

**FR-LOG-11 — Repeating Route Schedules**
- A route can be set as recurring: every N days, every synodic period, or on a custom cadence
- Scheduler finds the next N departure windows automatically

**FR-LOG-12 — Fleet / Ship Registry**
- Named ships, each with a propulsion profile and cargo capacity
- Ships are assigned to routes; schedule shows which ship departs when

**FR-LOG-13 — Passenger Journey Planner**
- User inputs: I want to travel from A to B, departing after [date]
- Planner finds the next available ship on a matching route, with connection options
- Output: full itinerary with dates, ships, layovers, and total travel time

**FR-LOG-14 — Logistics Pattern Visualiser**
- Active routes drawn as persistent arcs on the 3D viewport
- Arc thickness = cargo/passenger volume
- Togglable filters: by ship, by cargo type, by destination cluster
- Emergent trade network becomes visible as routes accumulate

---

---

## Phase 6A — Procedural Star System Generator

**Goal:** Generate complete, plausible star systems procedurally from a seed, using the world-generation mechanics defined in the [Mneme World Generator PWA](https://github.com/Game-in-the-Brain/mneme-world-generator-pwa). A generated system is immediately loadable into the Solar System 3D viewport at Presentation Scale, while all underlying data is stored at Real Distance Scale.

> This phase follows the Logistics Planner because it depends on the Real Distance Scale data infrastructure established in L1 (real coordinates, orbital elements) and the INRAS hierarchy from Phase 3. The procedural output must be a first-class citizen of the same data model — not a separate pipeline.

### FR-PSG-01 — Mneme World Generator Integration
- Consume the star system generation ruleset from `mneme-world-generator-pwa` directly
- A star system is generated from: seed value, stellar class input (or randomised), number of body slots
- Output is a fully populated INRAS tree: Level-1 star(s) → Level-2 planets/dwarf planets → Level-3 moons/stations → Level-4+ as needed
- All orbital data generated at Real Distance Scale (AU, km) with a corresponding Presentation Scale mapping applied automatically

### FR-PSG-02 — Procedural Planet Surfaces
- Noise-based icosphere displacement for terrestrial planet geometry (no GLB required)
- Parameters derived from the Mneme World Generator output: terrain type, water coverage, atmosphere density, volcanic activity
- Technique: layered simplex/Perlin noise applied as per-vertex displacement on a high-subdivision `THREE.IcosahedronGeometry`
- Parameters are also directly editable via the INRAS Editor (Phase 3) after generation

### FR-PSG-03 — Fractal Mapping
- Multi-octave fractal Brownian motion (fBm) noise stack extends `FR-PSG-02`
- Each octave doubles frequency, halves amplitude — produces self-similar terrain at all zoom levels
- Biome assignment via a `f(altitude, moisture)` lookup table → biome colour/material
- LOD-aware: base geometry at IcosahedronGeometry detail 2; subdivide and re-displace on camera zoom

### FR-PSG-04 — Icosahedral Map Export
- Render the procedurally generated planet from a fixed camera set covering all 20 icosahedron faces
- Composite into a flat icosahedral net image (PNG export)
- Optional separate heightmap export (grayscale displacement values)
- Compatible with external world-building tools: Wonderdraft, Inkarnate, GIMP, and `mneme-world-generator-pwa`

### FR-PSG-05 — System Seed & Reproducibility
- Every generated system is fully determined by its seed value
- Seed is stored in the system's data JSON — reloading the same seed always produces the same system
- "Randomise" button generates a new random seed; "Lock" freezes the current result before editing

### FR-PSG-06 — Save Generated System
- A generated system can be saved to a named slot in `localStorage` (or exported as JSON)
- Saved systems appear in a system library panel
- Any saved system can be loaded into the 3D viewport; all INRAS Editor tools remain available on loaded systems

---

## Phase 6B — Real Star Map

**Goal:** A Sol-centred 3D map of the real stellar neighbourhood rendered at true parsec distances. From this map, world builders select actual nearby stars, load their procedurally generated systems, and build out detailed settings grounded in real astrophysics.

> This is the capstone of the entire pipeline. The Real Star Map is the entry point for world builders: real stars → procedurally generated plausible systems (Phase 6A) → detailed by hand using Phases 1–4 tools.

### FR-RSM-01 — Stellar Catalogue Data
- Ingest a curated subset of the HYG stellar database (or equivalent open catalogue) — all stars within a configurable parsec radius of Sol
- Each star entry carries: proper name, Bayer/Flamsteed designation, spectral class, real 3D coordinates (parsecs from Sol), apparent magnitude, distance (ly and pc)
- Includes non-stellar objects in the catalogue where known: brown dwarfs, rogue planets, neutron stars, white dwarfs within range
- Sol is fixed at the origin (0, 0, 0) in all Real Distance Scale coordinates

### FR-RSM-02 — 3D Star Map Viewport
- A dedicated **Star Map** mode separate from the Solar System viewport
- Stars rendered as points/billboards sized by apparent magnitude, coloured by spectral class (OBAFGKM colour ramp)
- Sol highlighted at centre; navigable with OrbitControls
- Toggle between flat 2D top-down view and full 3D parallax view
- Distance rings / parsec grid overlay (toggleable)

### FR-RSM-03 — Shift-Select: Distance Measurement
- **Shift-click two stars** → displays the straight-line distance between them in parsecs and light-years
- Distance label floats at the midpoint of the line drawn between the two stars
- The connecting line persists until cleared or a new selection is made

### FR-RSM-04 — Shift-Select: Multi-Point Route Total
- **Shift-click three or more stars** → builds a route path in selection order
- The cumulative parsec total for the entire route is displayed in a panel
- Each leg's individual distance is shown in a leg-by-leg breakdown
- Route can be saved as a named travel corridor

### FR-RSM-05 — Save Star System to Slot
- Any star in the map can have a generated system (Phase 6A) attached to it
- **Generate System** → runs Phase 6A generation seeded from the star's spectral class and known parameters
- Generated system is saved to a named slot and appears as a marker on the star map
- Systems can be exported as JSON and shared between users

### FR-RSM-06 — Load Star System into Viewport
- **Click any star with a saved system** → loads that system into the Solar System 3D viewport at Presentation Scale
- The viewport header shows the star's name and real distance from Sol
- All Phase 1–4 editor tools are available on the loaded system
- "Return to Star Map" button takes the user back to the Real Star Map without losing the loaded system state

### FR-RSM-07 — World Builder Entry Point
- The complete user flow: Real Star Map → select a real nearby star → generate a plausible system → load into the 3D viewport → detail with the INRAS Editor, Data Editor, and Procedural Surface tools
- Multiple users can independently generate and save systems for the same star; seeds and JSON export allow community sharing
- The aggregate result: a shared, real-star-anchored, procedurally generated interstellar setting — thousands of plausible worlds built from real astrophysical data

---

## Milestones Summary

| Milestone | Title | Phase | Status |
|-----------|-------|-------|--------|
| ED | Data Editor | 1 | Backlog |
| EX | Extend Objects | 2 | Backlog |
| INRAS | INRAS Editor | 3 | Backlog |
| MOD | 3D Model Integration | 4 | Backlog |
| SHB | Ship & Habitat Builder | 5 | Backlog |
| L1 | Distance Calculator | L | Backlog |
| L2 | Delta-V Budget | L | Backlog |
| L3 | Travel Time & Visualiser | L | Backlog |
| L4 | Multi-Leg Journeys | L | Backlog |
| L5 | Scheduled Routes & Logistics Patterns | L | Backlog |
| PSG | Procedural Star System Generator | 6A | Backlog |
| RSM | Real Star Map | 6B | Backlog |

---

## Glossary

| Term | Definition |
|------|-----------|
| **INRAS** | Intrastellar Elements — non-planetary significant bodies (stars, exotic objects, megastructures) |
| **Presentation Scale** | The compressed visual model used in the 3D viewport — orbital distances and body sizes are scaled down so everything fits on screen. Used for navigation and exploration only. |
| **Real Distance Scale** | True astronomical measurements: AU, parsecs, km/s, days. Used in the Logistics Planner, the Procedural Star System Generator, and the Real Star Map. Never mixed with Presentation Scale. |
| **Delta-V** | The change in velocity required to perform a space manoeuvre; the fundamental currency of spaceflight |
| **Hohmann Transfer** | The minimum-energy elliptical orbit connecting two circular orbits |
| **Synodic period** | The time between two identical alignments of a planet and Earth as seen from the Sun |
| **GiB World Generator** | Game in the Brain's Blender-based procedural asset pipeline |
| **Mneme World Generator PWA** | Game in the Brain's browser-based star system generator — source of world-generation mechanics and the canonical object hierarchy for Phase 6A |
| **LOD** | Level of Detail — using simpler geometry for distant objects to save performance |
| **fBm** | Fractal Brownian Motion — multi-octave noise summation used to generate natural-looking terrain at multiple scales |
| **Icosahedral projection** | A method of unfolding a sphere onto a 2D map using the 20 faces of an icosahedron; minimises distortion vs. Mercator |
| **HYG database** | A merged star catalogue (Hipparcos, Yale Bright Star, Gliese) used as the source for the Real Star Map stellar data |
| **Barycenter** | The centre of mass of a multi-body system; the true orbital origin in binary/trinary star configurations |
| **Parsec** | Unit of stellar distance ≈ 3.26 light-years; the unit used throughout the Real Star Map |
