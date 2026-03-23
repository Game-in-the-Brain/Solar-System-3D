# 3D Solar System in THREE.js

Welcome to the **3D Solar System** project, a dynamic and interactive simulation of our solar system created using THREE.js and the Vite framework. This project showcases various advanced features and effects to provide an immersive experience of the celestial bodies in our solar system. The project is fully created by Karol Fryc.

Overview available at: https://w21030911.nuwebspace.co.uk/graphics/assessment/

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

## Conclusion
This project is a comprehensive representation of our solar system, bringing together realistic modeling, advanced visual effects, and interactive features. Explore the planets, their moons, and the vast asteroid belts, all from the comfort of your screen.

## License

This project is licensed under the [MIT License](./LICENSE).

Feel free to contribute, suggest improvements, or use this project as a foundation for your own THREE.js experiments. Happy exploring!
