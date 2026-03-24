# Solar System 3D — Procedural Star System Visualiser

> **Based on the original [3D Solar System](https://github.com/karol-fryc/solar-system-threejs) by Karol Fryc.** This project has diverged significantly from that work and is now developed by [Game in the Brain](https://github.com/Game-in-the-Brain) toward a full Procedural 3D Star System Generator and world-builder toolkit.

## The Problem

> *I want to play a sci-fi setting — and be allowed to learn about the science at my own pace.*

Most science fiction games either hand-wave the physics entirely, or front-load so much technical knowledge that the table never gets past character creation. Neither serves the people at the table who are curious but not yet literate — and neither serves the game master who wants the setting to feel real without becoming a lecture.

Game in the Brain's starting point is a simple observation: **the hard science fiction of 50 to 100 years ago is the common knowledge of today.** Rocketry, nuclear energy, computers, satellite communications, the structure of DNA — these were the speculative frontier once. Writers who used them accurately were writing hard sci-fi. Now they are school curriculum.

The science that feels like hard sci-fi *today* — orbital mechanics, delta-V budgets, Hill sphere dynamics, planetary formation by stellar mass, the logistics of a civilisation spread across multiple star systems with no faster-than-light travel — is simply the common knowledge of 50 to 100 years from now. The people who will take it for granted have not been born yet.

What a gaming group does when it sits down with a procedurally generated star system and asks *"how long does it actually take to get from here to there, and what does that mean for these people?"* is begin that process early. Not because they set out to study astrophysics, but because the question was interesting and the game made it possible to ask it.

**The goal is not to teach. The goal is to make the science explorable at the table's own pace** — so that the lessons arrive through play, through the consequences of decisions, through the moment when a player realises that the outer colony has been functionally independent for three generations not because of politics but because of light-lag.

## What this is

Solar System 3D is growing into a tool for world builders who want to do more than imagine a setting — they want to *understand* it.

Using the **[Mneme World Generator](https://github.com/Game-in-the-Brain/mneme-world-generator-pwa)** as its generation engine — a system grounded in Stellar Mass, Planetary Formation rules, Hill Sphere Radius positioning, and Waterfall Main World Creation — this tool will allow you to generate complete, physically plausible star systems from first principles and then explore them in a live 3D viewport.

But generation is only the beginning. The long-term goal is a **Realistic Travel Planner**: Delta-V budgets, Hohmann transfer windows, multi-leg itineraries, and scheduled trade routes — so that the distances and travel times between worlds are not just numbers in a rulebook, but things you can see, measure, and feel in three dimensions.

Distance and time are the invisible architects of every civilisation. A colony 0.3 AU from its sun lives in a different world than one at 2 AU — different day length, different seasons, different communication lag with home. A civilisation spread across three stars with no faster-than-light travel is not one civilisation at all, but several, slowly diverging. This tool is designed to make those realities visible.

For TTRPG game masters and players running hard-science campaigns, the aim is a shared library of detailed, internally consistent star systems — built by world builders, published for the community, playable and explorable by anyone. And to demonstrate something counterintuitive: that a handful of nearby stars, rendered at true scale, contains room for **trillions of people across thousands of worlds** — that the universe does not need to be infinite to be endless.

![Solar_System](images/solar_system.png)

![Earth](images/earthnew.png)

![Mercury](images/mercury.png)

![Mars](images/mars.png)

## Features

### Standard Setup
- **Scene, Camera, Renderer**: Basic setup for rendering 3D scenes using THREE.js.
- **Controls**: Interactive controls for navigating the 3D space.
- **Texture Loaders**: Efficient loading of textures for planets, moons, and other objects.

### Postprocessing Effects
- **BloomPass**: Adds a glowing effect to the Sun.
- **OutlinePass**: Highlights planets with a white outline when hovered over.
- **EffectComposer**: Manages and combines all postprocessing effects for rendering.

### Star Background
- A realistic starry sky that provides a beautiful backdrop for the solar system.

### Interactive Controls
- **dat.GUI**: Allows users to adjust parameters such as orbit speed and the intensity of the Sun's glow.

### Lighting
- **AmbientLight**: Provides soft lighting throughout the scene.
- **PointLight**: Positioned at the center of the Sun to cast realistic shadows.

### Detailed Planet Creation
- **Attributes**: Size, position, tilt, texture, bump material, rings, and atmospheres.
- **Moons**: Includes moons with realistic textures and orbits.
- **Special Materials**: Earth’s ShaderMaterial for day/night transitions and moving clouds.
- **Non-Spherical Moons**: Phobos and Deimos are modeled from 3D objects for realism.

### Realistic Orbits and Rotations
- Planets and moons orbit the Sun and rotate on their axes with scaled distances and speeds.
- Scaled sizes for better visual representation: Mercury, Venus, Earth, Mars, and Pluto are at actual scale, while larger planets are scaled down for balance.

### Shadows
- Realistic shadow casting from the PointLight at the Sun’s center.

### Asteroid Belts
- **Procedurally Generated**: 1000 asteroids for the belt between Mars and Jupiter, 3000 for the Kuiper belt.
- **Performance Optimization**: Simplified textures to ensure high performance.

### Select Feature
- **Hover Effect**: White outline around planets when hovered.
- **Zoom In**: Camera zooms in and displays planet details on click.
- **Zoom Out**: Returns to default view on closing the pop-up.

## Resources
3D objects and textures were sourced from the following free repositories:
- [NASA 3D Resources](https://nasa3d.arc.nasa.gov/images)
- [Solar System Scope Textures](https://www.solarsystemscope.com/textures/)
- [Planet Pixel Emporium](https://planetpixelemporium.com/index.php)
- [TurboSquid](https://www.turbosquid.com/)

## Live Demo

**[https://game-in-the-brain.github.io/Solar-System-3D/](https://game-in-the-brain.github.io/Solar-System-3D/)**

---

## Fork & Deploy to GitHub Pages

You can host your own copy for free in about 5 minutes.

### 1. Fork the repository
Click **Fork** at the top of this page. GitHub will create `https://github.com/YOUR_USERNAME/Solar-System-3D`.

### 2. Update the base path
Open `vite.config.js` and change the `base` value to match your repository name:
```javascript
base: '/Solar-System-3D/',   // keep as-is if your fork keeps the same name
```
If you renamed the repository, replace `Solar-System-3D` with your repo name exactly.

### 3. Enable GitHub Pages
In your forked repository go to **Settings → Pages** and set:
- **Source:** GitHub Actions

That's it. The included workflow (`.github/workflows/deploy.yml`) will automatically build and deploy whenever you push to `main`.

### 4. Your live URL
```
https://YOUR_USERNAME.github.io/Solar-System-3D/
```

> The first deployment runs automatically after you enable Pages. Subsequent pushes deploy within ~2 minutes.

---

## Local Development

1. Clone the repository:
    ```sh
    git clone https://github.com/your-username/Solar-System-3D.git
    ```
2. Navigate to the project directory:
    ```sh
    cd Solar-System-3D
    ```
3. Install dependencies:
    ```sh
    npm install
    ```
4. Start the development server:
    ```sh
    npm run dev
    ```
5. Open your browser and navigate to the URL shown in the terminal (typically `http://localhost:5173`).

## Roadmap

> Full backlog and feature specifications: **[PRD.md](./PRD.md)**

### A note on scale

This project operates at two distinct scales:

- **Presentation Scale** — the compressed, human-viewable model used in the 3D viewport. Orbital distances and body sizes are scaled down so everything fits on screen. This is the artistic representation you navigate and explore.
- **Real Distance Scale** — true astronomical measurements: AU, parsecs, km/s, days. Used exclusively in the Logistics Planner and the Real Star Map. Nothing in Real Distance Scale is derived from or limited by what fits on screen.

These two scales coexist in the data model. Every body carries both a `presentation` orbit radius and a `real` semi-major axis. They are never mixed.

| Phase | Title | What it adds |
|-------|-------|-------------|
| 1 | **Data Editor** | Edit any planet/moon property in-browser; export/import as JSON |
| 2 | **Extend Objects** | Dwarf planets (Ceres, Eris…), space stations, "Add Object" flow |
| 3 | **INRAS Editor** | Stars, exotic bodies, intrastellar elements — data sheet, orbits, 3D model assignment |
| 4 | **3D Model Integration** | Plug in GLB assets from the Game in the Brain Blender World Generator; ship and habitat models |
| 5 | **Procedural Ship & Habitat/Station Builder** | Modular ship and habitat/station construction using habitat modules sourced from [MNEME Space Combat](https://www.drivethrurpg.com/en/product/434090/Mneme-Space-Combat-full) |
| L1 | **Logistics: Distances** | Real AU distances, point-to-point calculator, light-travel time |
| L2 | **Logistics: Delta-V** | Hohmann transfers, propulsion profiles, fuel-mass budget |
| L3 | **Logistics: Travel Time** | Transfer arcs on the 3D view, journey duration in days/months/years |
| L4 | **Logistics: Multi-Leg** | Itinerary builder, waypoints, layovers, best-window finder |
| L5 | **Logistics: Scheduled Routes** | Repeating routes, ship registry, passenger journey planner, trade network visualiser |
| 6A | **Procedural Star System Generator** | Full star systems generated procedurally using the [Mneme World Generator](https://github.com/Game-in-the-Brain/mneme-world-generator-pwa) ruleset — stars, planets, moons, INRAS bodies, and orbital data all generated from seed. Systems display at Presentation Scale in the 3D viewport; all underlying data is at Real Distance Scale. |
| 6B | **Real Star Map** | Sol-centred 3D map of nearby stars at true parsec distances. Shift-select any two stars to read the distance between them; shift-select multiple stars to total the parsecs along a route. Save generated star systems to named slots. Click any star to load its full procedurally generated system into the 3D viewport. The complete pipeline: real stellar neighbourhood → procedurally generated plausible worlds → detailed by world builders. |

---

## The Vision

Start with a real star. Look it up on the Real Star Map — Alpha Centauri, Tau Ceti, 61 Cygni — and generate its system from the Mneme World Generator rules. Load it into the 3D viewport. Place your worlds. Plan a trade route from the inner habitable zone to the ice miners at the system's edge. Calculate how long the mail takes. Understand why the outer colonies think differently.

Then share it. Another world builder loads your system, extends it, adds a station, plans a different route. A game master runs a campaign in it. Players make decisions shaped by real orbital mechanics — not because they studied astrophysics, but because the tool showed them.

A civilisation of a trillion people living across five stars in a 4-parsec bubble of space. Every world procedurally generated, every travel time calculated from real delta-V, every trade route a line on a map that means something. That is a setting worth exploring.

## License

This project is licensed under the [MIT License](./LICENSE).

Feel free to contribute, suggest improvements, or use this project as a foundation for your own THREE.js experiments. Happy exploring!
