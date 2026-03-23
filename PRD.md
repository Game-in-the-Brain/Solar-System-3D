# Solar System 3D — Product Requirements Document
### Backlog & Roadmap

---

## Important Disclaimer: Scale & Fidelity

> **This simulation is not 1:1.** Distances, sizes, and orbital periods are compressed to fit a human-viewable screen. The visual model is an artistic representation designed for exploration and education — not a physically accurate simulator.
>
> **The Logistics Planner (Phase 3 onward) will use real-world measurements** — actual AU distances, real delta-V figures, and accurate travel time calculations. The 3D viewport will remain compressed for navigation; the calculator will operate on true physics.

---

## Roadmap Overview

```
Phase 1 ──► Data Editor (planets editable in-browser)
Phase 2 ──► Extend Objects (dwarf planets, space stations)
Phase 3 ──► INRAS Editor (intrastellar elements + stars)
Phase 4 ──► 3D Model Integration (GiB World Generator / Blender)
             │
             ├──► Path A: Procedural World Generator
             │           (stations + worlds generated with code)
             │
             └──► Path B: Logistics Planner
                         (Delta-V → distances → travel time
                          → multi-leg → scheduled routes
                          → intrastellar logistics patterns)
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

## Phase 5A — Procedural World Generator

*Parallel path — runs alongside Phase 5B*

**Goal:** Generate planet surfaces, station interiors, and asteroid structures procedurally using code, as a lightweight complement to the Blender pipeline.

### FR-PWG-01 — Procedural Planet Surfaces
- Noise-based heightmap generator for terrestrial planets
- Parameters: seed, terrain roughness, water coverage, atmosphere density
- Output: live THREE.js geometry (no GLB required)

### FR-PWG-02 — Procedural Station Generation
- Modular station builder: core + ring + spoke + docking arms
- Parameters: crew capacity, function (mining / transit / research / military)
- Generates a THREE.js mesh; optionally exports as GLB for the Blender pipeline

### FR-PWG-03 — Low-LOD Mode
- Distant objects use procedurally generated low-detail spheres
- Swap to full model/texture on zoom — same LOD pattern as modern space games

---

## Phase 5B — Logistics Planner

*Parallel path — runs alongside Phase 5A*

> **Scale note:** The 3D viewport remains compressed for human-viewable navigation. All Logistics Planner calculations operate on **real astronomical distances and physics** — AU, km/s, days.

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

## Milestones Summary

| Milestone | Title | Phase | Status |
|-----------|-------|-------|--------|
| ED | Data Editor | 1 | Backlog |
| EX | Extend Objects | 2 | Backlog |
| INRAS | INRAS Editor | 3 | Backlog |
| MOD | 3D Model Integration | 4 | Backlog |
| PWG | Procedural World Generator | 5A | Backlog |
| L1 | Distance Calculator | 5B | Backlog |
| L2 | Delta-V Budget | 5B | Backlog |
| L3 | Travel Time & Visualiser | 5B | Backlog |
| L4 | Multi-Leg Journeys | 5B | Backlog |
| L5 | Scheduled Routes & Logistics Patterns | 5B | Backlog |

---

## Glossary

| Term | Definition |
|------|-----------|
| **INRAS** | Intrastellar Elements — non-planetary significant bodies (stars, exotic objects, megastructures) |
| **Compressed scale** | The visual model shrinks orbital distances and planet sizes so everything fits on screen |
| **Real measures** | AU, km, m/s, days — used exclusively in the Logistics Planner calculations |
| **Delta-V** | The change in velocity required to perform a space manoeuvre; the fundamental currency of spaceflight |
| **Hohmann Transfer** | The minimum-energy elliptical orbit connecting two circular orbits |
| **Synodic period** | The time between two identical alignments of a planet and Earth as seen from the Sun |
| **GiB World Generator** | Game in the Brain's Blender-based procedural asset pipeline |
| **LOD** | Level of Detail — using simpler geometry for distant objects to save performance |
