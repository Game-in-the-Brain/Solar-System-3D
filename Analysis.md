# Solar System 3D - Technical Analysis

## Table of Contents
1. [High-Level Overview](#high-level-overview)
2. [Key Libraries and Technologies](#key-libraries-and-technologies)
3. [Architecture and Components](#architecture-and-components)
4. [How Each Component Works](#how-each-component-works)
5. [Editing and Placing Worlds](#editing-and-placing-worlds)
6. [Deployment Issues](#deployment-issues)
7. [File Structure](#file-structure)

---

## High-Level Overview

**Solar System 3D** is an interactive, browser-based 3D visualization of our solar system built with **Three.js** and **Vite**. The application renders the Sun, eight planets (plus Pluto), their moons, asteroid belts, and provides interactive features like planet selection, camera zoom, and real-time orbital animation.

### Key Features
- **Real-time 3D rendering** of celestial bodies with realistic textures
- **Interactive controls** for navigation and parameter adjustment
- **Post-processing effects** including bloom (for the Sun) and outline highlighting
- **Planet selection** with detailed information display
- **Procedurally generated asteroid belts** (Main Belt and Kuiper Belt)
- **Realistic lighting** with shadows cast by the Sun
- **Animated orbits and rotations** with adjustable speed controls

---

## Key Libraries and Technologies

### Core Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| **three** | ^0.160.0 | Core 3D rendering engine - handles scenes, cameras, geometries, materials, lighting |
| **dat.gui** | ^0.7.9 | Interactive GUI for real-time parameter adjustment (orbit speed, sun intensity) |
| **postprocessing** | ^7.0.0-alpha-1 | Listed in `package.json` but **not imported or used** — all post-processing is handled via `three/addons/postprocessing/*` (Three.js built-in modules) |
| **vite** | ^4.5.0 | Modern build tool and development server with hot module replacement |

### Three.js Modules Used

| Module | Purpose |
|--------|---------|
| `OrbitControls` | Mouse-based camera navigation (rotate, zoom, pan) |
| `EffectComposer` | Manages post-processing render pipeline |
| `UnrealBloomPass` | Creates glowing bloom effect (used for the Sun) |
| `RenderPass` | Standard scene rendering pass |
| `OutlinePass` | Adds white outline on hover/selection |
| `GLTFLoader` | Loads 3D models (for Phobos, Deimos moons, asteroids) |

### Web Technologies
- **HTML5** - Page structure and UI containers
- **CSS3** - Styling for planet info panels and GUI positioning
- **ES6+ JavaScript** - Modern JavaScript with modules, arrow functions, const/let
- **WebGL** - Low-level graphics API (via Three.js)

---

## Architecture and Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Solar System 3D                          │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Core 3D    │  │ Post-Process │  │   Interactive│      │
│  │    Engine    │  │   Pipeline   │  │   Controls   │      │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤      │
│  │ - Scene      │  │ - BloomPass  │  │ - OrbitControls│    │
│  │ - Camera     │  │ - OutlinePass│  │ - dat.GUI    │      │
│  │ - Renderer   │  │ - EffectComposer│ - Raycaster   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                      Celestial Objects                        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌────────┐ │
│  │   Sun   │ │ Planets │ │  Moons  │ │  Rings  │ │Asteroids│ │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    Data & Configuration                       │
│         (Planet data, textures, orbital parameters)          │
└─────────────────────────────────────────────────────────────┘
```

### Component Breakdown

#### 1. **Core 3D Setup**
- **Scene** (`THREE.Scene`) - Container for all 3D objects
- **Camera** (`THREE.PerspectiveCamera`) - 45° FOV perspective camera
- **Renderer** (`THREE.WebGL1Renderer`) - Renders the scene with ACESFilmic tone mapping
- **Controls** (`OrbitControls`) - User navigation with damping

#### 2. **Lighting System**
- **AmbientLight** - Soft fill light (0x222222, intensity 6)
- **PointLight** - Positioned at Sun's center, casts shadows with 1024x1024 shadow map

#### 3. **Post-Processing Pipeline**
1. **RenderPass** - Renders the base scene
2. **OutlinePass** - Adds white outline on hover (edgeStrength: 3, edgeGlow: 1)
3. **UnrealBloomPass** - Creates glow effect (threshold: 1, radius: 0.9)

#### 4. **Celestial Body Factory**
- `createPlanet()` function - Generates planets with configurable:
  - Size and position
  - Textures and bump maps
  - Axial tilt
  - Rings (Saturn, Uranus)
  - Atmospheres (Venus, Earth)
  - Moons

#### 5. **Interaction System**
- **Raycaster** - Detects mouse interactions with planets
- **GUI (dat.GUI)** - Adjustable parameters:
  - `accelerationOrbit` (0-10): Orbital speed multiplier — default `1`
  - `acceleration` (0-10): Rotation speed multiplier — default `1`
  - `sunIntensity` (1-10): Sun's emissive glow intensity — default `1.9`

---

## How Each Component Works

### 1. Scene Initialization

```javascript
// Scene setup creates the 3D world container
const scene = new THREE.Scene();

// Camera with 45° field of view
var camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(-175, 115, 5); // Initial position looking at the system

// WebGL renderer with tone mapping for realistic lighting
const renderer = new THREE.WebGL1Renderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.toneMapping = THREE.ACESFilmicToneMapping;
```

**What it does:**
- Creates the 3D coordinate system where all objects exist
- Sets up the viewing frustum (what the user can see)
- Configures the rendering pipeline with filmic tone mapping for cinematic look

### 2. The Sun

```javascript
const sunSize = 697/40; // Scaled down 40x from actual Earth comparison
const sunGeom = new THREE.SphereGeometry(sunSize, 32, 20);
sunMat = new THREE.MeshStandardMaterial({
  emissive: 0xFFF88F,        // Self-illuminating color
  emissiveMap: loadTexture.load(sunTexture),
  emissiveIntensity: settings.sunIntensity
});
```

**What it does:**
- Creates a glowing sphere at the center (0,0,0)
- Uses emissive material so it appears to emit light
- Bloom effect makes it appear to radiate light
- PointLight at same position casts shadows on other objects

### 3. Planet Creation System

The `createPlanet()` function is the **factory pattern** implementation:

```javascript
function createPlanet(planetName, size, position, tilt, texture, bump, ring, atmosphere, moons)
```

**Internal Structure:**
```
planet3d (rotates for orbit)
└── planetSystem (contains all planet elements)
    ├── planet (the mesh itself)
    │   └── Atmosphere (optional, child of planet)
    ├── orbit (visual orbit path line)
    ├── Ring (optional, for Saturn/Uranus)
    └── moons (array of moon meshes)
```

**What it does:**
1. Creates planet mesh with sphere geometry
2. Applies textures (diffuse + optional bump map for surface detail)
3. Sets axial tilt via `rotation.z`
4. Adds orbit path visualization (white ellipse)
5. Optionally adds:
   - **Rings**: Using `THREE.RingGeometry` with transparency
   - **Atmosphere**: Slightly larger sphere with transparent material
   - **Moons**: Separate meshes that orbit the planet

### 4. Known Texture Issues

- **`venusBump`** imports `venusmap.jpg` (the same file as the diffuse map) — `import venusBump from '/images/venusmap.jpg'`. A dedicated `venusbump.jpg` exists in `src/images/` but is not imported.
- **Pluto** is created without a bump map despite `plutobump2k.jpg` being present in `src/images/`. The `createPlanet` call passes no bump argument: `createPlanet('Pluto', 1, 350, 57, plutoTexture)`.
- **Unused imports in `src/images/`**: `mercury.jpg`, `moon.jpg`, `earth_normalmap.jpg`, `earth_specularmap.jpg`, `jup1vuu2.jpg` are present but not imported in `script.js`.

### 5. Earth's Day/Night Shader

Earth uses a **custom ShaderMaterial** instead of standard material:

```javascript
const earthMaterial = new THREE.ShaderMaterial({
  uniforms: {
    dayTexture: { type: "t", value: loadTexture.load(earthTexture) },
    nightTexture: { type: "t", value: loadTexture.load(earthNightTexture) },
    sunPosition: { type: "v3", value: sun.position }
  },
  vertexShader: "...",
  fragmentShader: "..."
});
```

**How it works:**
- **Vertex Shader**: Calculates surface normal and direction to sun for each vertex
- **Fragment Shader**: 
  - Samples both day and night textures
  - Calculates dot product between normal and sun direction
  - Mixes between night and day based on sunlight exposure
  - Creates realistic terminator (day/night line)

### 6. Moon Systems

#### Earth's Moon (Simple Sphere)
```javascript
const earthMoon = [{
  size: 1.6,
  texture: earthMoonTexture,
  bump: earthMoonBump,
  orbitSpeed: 0.001,
  orbitRadius: 10
}]
```

#### Mars' Moons (3D Models)
```javascript
const marsMoons = [{
  modelPath: '/images/mars/phobos.glb',
  scale: 0.1,
  orbitRadius: 5,
  orbitSpeed: 0.002
}];
// Loaded via GLTFLoader for realistic irregular shapes
```

**Animation:**
```javascript
// Each frame, moons are repositioned using circular motion
const moonX = planet.position.x + moon.orbitRadius * Math.cos(time * moon.orbitSpeed);
const moonZ = planet.position.z + moon.orbitRadius * Math.sin(time * moon.orbitSpeed);
```

### 7. Asteroid Belt Generation

```javascript
function loadAsteroids(path, numberOfAsteroids, minOrbitRadius, maxOrbitRadius) {
  // Loads a pack of 12 asteroid models
  // Clones each asteroid ~83 times (1000/12)
  // Randomly distributes in orbital radius
  // Applies random rotation and slight scale variation
}
```

**Two belts created:**
- **Main Asteroid Belt**: 1000 asteroids between Mars (130) and Jupiter (160)
- **Kuiper Belt**: 3000 asteroids beyond Neptune (352-370)

### 8. Interaction System

#### Hover Detection
```javascript
raycaster.setFromCamera(mouse, camera);
const intersects = raycaster.intersectObjects(raycastTargets);
if (intersects.length > 0) {
  outlinePass.selectedObjects = [intersectedPlanet];
}
```

#### Click to Select
```javascript
function onDocumentMouseDown(event) {
  // Raycast to find clicked planet
  // Stop orbital movement
  // Calculate camera target position
  // Start zoom animation
  // Show info panel
}
```

**Camera Zoom Animation:**
```javascript
if (isMovingTowardsPlanet) {
  camera.position.lerp(targetCameraPosition, 0.03); // Smooth interpolation
}
```

### 9. Animation Loop

```javascript
function animate() {
  // 1. Rotate each planet on its axis
  sun.rotateY(0.001 * settings.acceleration);
  mercury.planet.rotateY(0.001 * settings.acceleration);
  
  // 2. Orbit planets around sun
  mercury.planet3d.rotateY(0.004 * settings.accelerationOrbit);
  
  // 3. Animate moons
  // 4. Rotate asteroids
  // 5. Check for hover outlines
  // 6. Handle camera zoom
  
  controls.update();
  composer.render(); // Use post-processing pipeline
}
```

---

## Editing and Placing Worlds

### Adding a New Planet

To add a new planet, follow these steps:

#### Step 1: Add Texture Imports
```javascript
// At the top of script.js, add:
import newPlanetTexture from '/images/newplanetmap.jpg';
import newPlanetBump from '/images/newplanetbump.jpg';
```

#### Step 2: Create the Planet
```javascript
// After existing planet creations (around line 495-525)
const newPlanet = new createPlanet(
  'NewPlanet',      // Name
  10,               // Size (radius)
  400,              // Distance from sun (position)
  20,               // Axial tilt (degrees)
  newPlanetTexture, // Diffuse texture
  newPlanetBump,    // Bump map (optional, pass null if none)
  null,             // Ring config (or pass ring object)
  null,             // Atmosphere texture (or pass texture)
  null              // Moons array (or pass moons)
);
```

#### Step 3: Add Planet Data
```javascript
// In planetData object (around line 528-610)
const planetData = {
  // ... existing planets ...
  'NewPlanet': {
    radius: '10,000 km',
    tilt: '20°',
    rotation: '24 hours',
    orbit: '365 days',
    distance: '400 million km',
    moons: '0',
    info: 'Description of your new planet.'
  }
};
```

#### Step 4: Add to Raycast Targets
```javascript
// Around line 614-617, add to raycastTargets array:
const raycastTargets = [
  mercury.planet, venus.planet, // ... existing planets ...
  newPlanet.planet  // Add your new planet
];
```

#### Step 5: Add Animation
```javascript
// In animate() function (around line 661-684)
newPlanet.planet.rotateY(0.005 * settings.acceleration);
newPlanet.planet3d.rotateY(0.0005 * settings.accelerationOrbit);
```

#### Step 6: Configure Shadows (Optional)
```javascript
// After other shadow configurations (around line 630-656)
newPlanet.planet.castShadow = true;
newPlanet.planet.receiveShadow = true;
```

### Adding a Moon to an Existing Planet

```javascript
// Define moon configuration
const newMoons = [{
  size: 2.0,                    // Moon radius
  texture: moonTexture,         // Texture path
  bump: moonBumpTexture,        // Optional bump map
  orbitRadius: 15,              // Distance from planet
  orbitSpeed: 0.001             // Orbital velocity
}];

// Pass to createPlanet
createPlanet('PlanetName', size, position, tilt, texture, bump, null, null, newMoons);
```

### Adding a Ring System

```javascript
// Create ring configuration object
const ringConfig = {
  innerRadius: 15,      // Inner radius of ring
  outerRadius: 25,      // Outer radius of ring
  texture: ringTexture  // Ring texture (with transparency)
};

// Pass to createPlanet (5th parameter)
createPlanet('PlanetName', size, position, tilt, texture, bump, ringConfig);
```

### Adding an Atmosphere

```javascript
// Pass atmosphere texture to createPlanet (6th parameter)
createPlanet('PlanetName', size, position, tilt, texture, bump, null, atmosphereTexture);

// Atmosphere is created as a slightly larger sphere (size + 0.1)
// with transparency (opacity: 0.4) and depth handling
```

### Modifying Orbit/Rotation Speeds

```javascript
// In the animate() function, modify the rotation values:
mercury.planet3d.rotateY(0.004 * settings.accelerationOrbit);  // Orbit speed
mercury.planet.rotateY(0.001 * settings.acceleration);         // Rotation speed

// Values are in radians per frame
// Higher = faster movement
```

### Adjusting the Scale

The project uses **scaled proportions** for visual balance:
- **Sun**: 697/40 = ~17.4 units (40x smaller than real Earth comparison)
- **Mercury, Venus, Earth, Mars, Pluto**: Actual relative scale
- **Jupiter, Saturn, Uranus, Neptune**: Scaled down 4x (69/4, 58/4, 25/4, 24/4)

To adjust scales uniformly, modify the size parameters in `createPlanet()` calls.

### Adding Custom Asteroid Belts

```javascript
// Call loadAsteroids with:
// - path to GLB file
// - number of asteroids
// - min orbit radius
// - max orbit radius
loadAsteroids('/asteroids/asteroidPack.glb', 500, 200, 250);
```

---

## Deployment Issues

### Asset Loading: Two Methods

The project uses two different methods to load assets, which determines where each file must live:

| Method | Asset Type | Location | Build Handling |
|--------|-----------|----------|----------------|
| ES6 `import` | JPG/PNG textures | `src/images/` | ✅ Bundled by Vite with hashed filenames |
| `GLTFLoader.load()` | GLB models | Runtime path | Must be in `publicDir` (`static/`) to be copied to `dist/` |

In `vite.config.js`:
```javascript
publicDir: '../static/'  // Files here are copied as-is to dist/
```

### ✅ GLB Files — Already Resolved

The GLB files required for runtime loading are already present in `static/`:

```
static/
├── asteroids/
│   └── asteroidPack.glb     ✅ present
└── images/mars/
    ├── phobos.glb            ✅ present
    └── deimos.glb            ✅ present
```

The originals also remain in `src/` for development consistency:
- `src/asteroids/asteroidPack.glb`
- `src/images/mars/phobos.glb` (loaded at line 449)
- `src/images/mars/deimos.glb` (loaded at line 457)

A completed build in `dist/` confirms all three GLB files are present alongside the bundled JS/CSS/texture assets.

### GitHub Pages Deployment

The project is deployable to GitHub Pages. The only configuration change needed is updating `vite.config.js` with the correct base path:

```javascript
export default {
    root: 'src/',
    publicDir: '../static/',
    base: '/Solar-System-3D/',  // Change from './' to match your repo name
    // ... rest of config
}
```

Then build and deploy the `dist/` folder:

```bash
npm run build

# Manual deploy to gh-pages branch
git subtree push --prefix dist origin gh-pages
```

Or via GitHub Actions — the `Prepare static assets` step is not needed since GLB files are already in `static/`.

#### Deployment Checklist

- [ ] Update `base` in `vite.config.js` to `/Solar-System-3D/` (or your repo name)
- [ ] Run `npm run build`
- [ ] Verify `dist/asteroids/asteroidPack.glb` exists
- [ ] Verify `dist/images/mars/phobos.glb` exists
- [ ] Verify `dist/images/mars/deimos.glb` exists
- [ ] Test locally with `npx serve dist/`
- [ ] Deploy the `dist/` folder contents

### Original Deployment (nuwebspace.co.uk)

The live demo at `https://w21030911.nuwebspace.co.uk/graphics/assessment/` is hosted on a **university student web server**. The URL pattern (`w21030911` = student ID, `graphics/assessment` = coursework folder) indicates this was a **university graphics/3D programming assignment**.

---

## File Structure

```
Solar-System-3D/
├── src/
│   ├── index.html              # Main HTML entry point
│   ├── script.js               # Main application logic (~800 lines)
│   ├── style.css               # UI styling for planet info panel
│   ├── asteroids/
│   │   └── asteroidPack.glb    # 3D asteroid models (12 variations) — keep here for dev; copy exists in static/
│   └── images/                 # All texture assets
│       ├── sun.jpg
│       ├── mercurymap.jpg, mercurybump.jpg
│       ├── venusmap.jpg, venusbump.jpg (unused — venusmap.jpg used as bump), venus_atmosphere.jpg
│       ├── earth_daymap.jpg, earth_nightmap.jpg, earth_atmosphere.jpg
│       ├── earth_normalmap.jpg, earth_specularmap.jpg  # Present but not imported
│       ├── marsmap.jpg, marsbump.jpg
│       ├── mars/
│       │   ├── phobos.glb      # Mars moon 3D model — keep here for dev; copy exists in static/
│       │   └── deimos.glb      # Mars moon 3D model — keep here for dev; copy exists in static/
│       ├── jupiter.jpg, jup1vuu2.jpg  # jup1vuu2.jpg present but not imported
│       ├── jupiterIo.jpg, jupiterEuropa.jpg, jupiterGanymede.jpg, jupiterCallisto.jpg
│       ├── saturnmap.jpg, saturn_ring.png
│       ├── uranus.jpg, uranus_ring.png
│       ├── neptune.jpg
│       ├── plutomap.jpg, plutobump2k.jpg  # plutobump2k.jpg present but not used — Pluto has no bump map
│       ├── moonmap.jpg, moon.jpg, moonbump.jpg  # moon.jpg present but not imported
│       ├── mercury.jpg  # Present but not imported (mercurymap.jpg is used instead)
│       └── 1.jpg, 2.jpg, 3.jpg, 4.jpg  # Skybox textures
├── static/                     # Static assets (served as-is to dist/)
│   ├── .gitkeep
│   ├── door.jpg                # Unused file
│   ├── asteroids/
│   │   └── asteroidPack.glb    # ✅ Correctly placed for deployment
│   └── images/mars/
│       ├── phobos.glb          # ✅ Correctly placed for deployment
│       └── deimos.glb          # ✅ Correctly placed for deployment
├── package.json                # Dependencies and scripts
├── vite.config.js              # Vite configuration
├── readme.md                   # Project documentation
└── LICENSE                     # MIT License
```

### Post-Build Structure (dist/)

```
dist/
├── index.html                  # Entry point
├── assets/                     # Bundled JS, CSS, and images
│   ├── index-[hash].js         # Main JavaScript bundle
│   ├── index-[hash].css        # Styles
│   └── *.jpg, *.png            # Hashed texture files
├── asteroids/                  # Copied from static/asteroids/
│   └── asteroidPack.glb        # ✅ Present (GLBs correctly in static/)
└── images/mars/                # Copied from static/images/mars/
    ├── phobos.glb              # ✅ Present
    └── deimos.glb              # ✅ Present
```

---

## Quick Reference: Planet Parameters

| Planet | Size | Position | Tilt | Ring | Atmosphere | Moons |
|--------|------|----------|------|------|------------|-------|
| Mercury | 2.4 | 40 | 0° | No | No | 0 |
| Venus | 6.1 | 65 | 3° | No | Yes | 0 |
| Earth | 6.4 | 90 | 23° | No | Yes | 1 |
| Mars | 3.4 | 115 | 25° | No | No | 2 (3D models) |
| Jupiter | 17.25 | 200 | 3° | No | No | 4 |
| Saturn | 14.5 | 270 | 26° | Yes | No | 0 |
| Uranus | 6.25 | 320 | 82° | Yes | No | 0 |
| Neptune | 6.0 | 340 | 28° | No | No | 0 |
| Pluto | 1.0 | 350 | 57° | No | No | 0 |

---

*Analysis generated for Solar System 3D - Three.js Interactive Visualization*
