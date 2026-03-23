# Solar System 3D - GitHub Pages Deployment Plan

## Objective
Deploy the Solar System 3D application to GitHub Pages with full functionality matching the original nuwebspace.co.uk deployment.

---

## Current State Analysis

### What's Working (Local Development)
- ✅ All textures load correctly
- ✅ Sun with bloom effect renders
- ✅ All 8 planets + Pluto render with orbits
- ✅ Earth's day/night shader works
- ✅ Saturn and Uranus rings display
- ✅ Venus and Earth atmospheres visible
- ✅ Interactive GUI controls function
- ✅ Planet hover/selection works

### Deployment Status
- ✅ GLB files in `static/` — **already resolved** (see Change 1 below)
- ✅ `dist/` build already completed with all GLB files present
- ❌ `vite.config.js` base path still set to `'./'` — must change for GitHub Pages
- ❌ GitHub Actions workflow not yet created
- ❌ GitHub Pages not yet enabled in repository settings

---

## Root Cause

The application uses **two different asset loading mechanisms**:

| Mechanism | Asset Type | Build Behavior | GitHub Pages Status |
|-----------|-----------|----------------|---------------------|
| ES6 `import` | JPG/PNG textures | Bundled with hashed names | ✅ Works automatically |
| Runtime `GLTFLoader.load()` | GLB 3D models | Must be in `static/` folder | ✅ Fixed — files now in `static/` |

**Vite's `publicDir` configuration only copies files from `static/` to `dist/`.** The GLB files are now present in both `src/` (development) and `static/` (production build).

---

## Required Changes Summary

| Change | File(s) | Status | Purpose |
|--------|---------|--------|---------|
| 1. Copy GLB assets | `static/asteroids/`, `static/images/mars/` | ✅ **Already done** | 3D models available in production build |
| 2. Update vite.config.js | `vite.config.js` | ❌ **Still needed** | Set correct base path for GitHub Pages |
| 3. Add GitHub Actions workflow | `.github/workflows/deploy.yml` | ❌ **Still needed** | Automate build and deployment on push |
| 4. Enable GitHub Pages | Repository Settings | ❌ **Still needed** | Turn on Pages with GitHub Actions as source |

---

## Detailed Implementation Steps

### Change 1: Copy GLB Files to Static Folder — ✅ COMPLETE

The following files are present in `static/` and committed to the repository:
```
static/asteroids/asteroidPack.glb     ✅
static/images/mars/phobos.glb         ✅
static/images/mars/deimos.glb         ✅
```

No action needed — GitHub Actions checkout will include these files.

---

### Change 2: Update vite.config.js for GitHub Pages

**Current File:**
```javascript
export default {
    root: 'src/',
    publicDir: '../static/',
    base: './',  // ❌ Relative path - breaks on GitHub Pages subdirectories
    server: {
        host: true,
        open: !('SANDBOX_URL' in process.env || 'CODESANDBOX_HOST' in process.env)
    },
    build: {
        outDir: '../dist',
        emptyOutDir: true,
        sourcemap: true
    }
}
```

**Updated File:**
```javascript
export default {
    root: 'src/',
    publicDir: '../static/',
    base: '/Solar-System-3D/',  // ✅ Match your repository name
    server: {
        host: true,
        open: !('SANDBOX_URL' in process.env || 'CODESANDBOX_HOST' in process.env)
    },
    build: {
        outDir: '../dist',
        emptyOutDir: true,
        sourcemap: true
    }
}
```

**Important:** Replace `Solar-System-3D` with your actual repository name if different.

**Why:** GitHub Pages serves from `https://username.github.io/repo-name/`. The `base` path ensures all asset URLs are prefixed correctly.

---

### Change 3: Create GitHub Actions Workflow

**Create File:** `.github/workflows/deploy.yml`

**Content:**
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main, master]
  workflow_dispatch:  # Allow manual triggering

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      # GLB files are already committed to static/ — no copy step needed

      - name: Build project
        run: npm run build

      - name: Verify build artifacts
        run: |
          echo "=== Verifying GLB files in dist ==="
          ls -la dist/asteroids/
          ls -la dist/images/mars/

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Why:** This workflow:
1. Triggers on every push to main/master
2. Sets up Node.js environment
3. Builds the project with Vite (GLB files already in `static/` — no copy step needed)
4. Verifies all artifacts are present
5. Deploys to GitHub Pages

---

## Alternative: Manual Deployment (Without Actions)

If you prefer not to use GitHub Actions:

### Step 1: Make Changes Locally
```bash
# GLB files already in static/ — skip the copy step

# Update vite.config.js (change base to '/Solar-System-3D/')
# Edit vite.config.js manually

# Build
npm run build
```

### Step 2: Deploy dist/ folder to gh-pages branch
```bash
# From project root
cd dist

git init
git add .
git commit -m "Deploy to GitHub Pages"
git push -f git@github.com:Game-in-the-Brain/Solar-System-3D.git gh-pages
```

### Step 3: Enable GitHub Pages
1. Go to repository Settings → Pages
2. Source: Deploy from a branch
3. Branch: gh-pages / (root)

---

## File Structure After Changes

### Repository Structure
```
Solar-System-3D/
├── .github/
│   └── workflows/
│       └── deploy.yml          # NEW - GitHub Actions workflow
├── src/
│   ├── index.html
│   ├── script.js
│   ├── style.css
│   ├── asteroids/
│   │   └── asteroidPack.glb    # Keep for dev
│   └── images/
│       └── mars/
│           ├── phobos.glb      # Keep for dev
│           └── deimos.glb      # Keep for dev
├── static/                     # ✅ Already contains GLB files
│   ├── asteroids/
│   │   └── asteroidPack.glb    # ✅ Already present
│   ├── images/
│   │   └── mars/
│   │       ├── phobos.glb      # ✅ Already present
│   │       └── deimos.glb      # ✅ Already present
│   └── door.jpg
├── vite.config.js              # ❌ NEEDS CHANGE — update base path
├── package.json
└── readme.md
```

### Build Output Structure (dist/)
```
dist/
├── index.html
├── assets/
│   ├── index-[hash].js
│   ├── index-[hash].css
│   └── *.jpg (hashed texture files)
├── asteroids/
│   └── asteroidPack.glb        # ✅ Present via static/
├── images/
│   └── mars/
│       ├── phobos.glb          # ✅ Present via static/
│       └── deimos.glb          # ✅ Present via static/
└── door.jpg
```

---

## Verification Checklist

Before considering deployment complete, verify:

### Local Build Verification
- [ ] Run `npm run build` completes without errors
- [ ] `dist/asteroids/asteroidPack.glb` exists (141KB)
- [ ] `dist/images/mars/phobos.glb` exists (3.7MB)
- [ ] `dist/images/mars/deimos.glb` exists (1.6MB)
- [ ] `dist/index.html` references assets with `/Solar-System-3D/` prefix
- [ ] `npx serve dist/` works locally

### GitHub Repository Verification
- [ ] All files committed to main branch
- [ ] `.github/workflows/deploy.yml` exists
- [ ] `static/asteroids/asteroidPack.glb` tracked in git
- [ ] `static/images/mars/*.glb` tracked in git
- [ ] `vite.config.js` has correct `base` path

### GitHub Pages Verification
- [ ] Settings → Pages shows "Your site is published at..."
- [ ] URL loads without 404 errors
- [ ] Console shows no 404 errors for GLB files
- [ ] All 4,000 asteroids render (Main Belt + Kuiper Belt)
- [ ] Mars shows both Phobos and Deimos moons
- [ ] Can click planets to zoom and view info
- [ ] GUI controls work (orbit speed, sun intensity)

### Feature Parity with nuwebspace.co.uk
- [ ] Sun renders with bloom effect
- [ ] All 9 planets visible and orbiting
- [ ] Earth shows day/night transition
- [ ] Saturn and Uranus rings visible
- [ ] Asteroid belts visible
- [ ] Mars moons (Phobos, Deimos) orbit correctly
- [ ] Planet hover outlines work
- [ ] Click to zoom works
- [ ] Camera controls (rotate, zoom, pan) work

---

## Expected GitHub Pages URL

After successful deployment:
```
https://game-in-the-brain.github.io/Solar-System-3D/
```

Repository: `https://github.com/Game-in-the-Brain/Solar-System-3D`

---

## Troubleshooting

### Issue: 404 errors for GLB files
**Cause:** Files not in `static/` folder
**Fix:** Run the copy commands in Change 1

### Issue: Blank page, console shows MIME type errors
**Cause:** `base` path incorrect in vite.config.js
**Fix:** Ensure `base: '/Solar-System-3D/'` matches your repo name exactly

### Issue: GitHub Actions fails at "Install dependencies"
**Cause:** package-lock.json missing or out of sync
**Fix:** Run `npm install` locally and commit `package-lock.json`

### Issue: Workflow runs but site doesn't update
**Cause:** GitHub Pages source not configured
**Fix:** Settings → Pages → Source: GitHub Actions

### Issue: Asteroids/Mars moons visible but wrong position
**Cause:** GLB files corrupted or wrong path
**Fix:** Verify files are binary identical: `diff src/asteroids/asteroidPack.glb static/asteroids/asteroidPack.glb`

---

## Summary

| Task | Effort | Status |
|------|--------|--------|
| Copy GLB files to static/ | — | ✅ Already done |
| Update vite.config.js base path | 2 min | ❌ Still needed |
| Create GitHub Actions workflow | 10 min | ❌ Still needed |
| Enable GitHub Pages in settings | 2 min | ❌ Still needed |
| Test deployment | 5 min | After above |

**Remaining setup time: ~15 minutes**

Once configured, every push to main will automatically deploy to GitHub Pages.
