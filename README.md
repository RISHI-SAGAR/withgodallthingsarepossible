# 🧠 Neural AR — Synaptic Brain Network

A real-time AR neural network effect rendered in the browser using WebGL.
Face tracking via MediaPipe FaceMesh + Three.js rendering with custom GLSL shaders.

```
Live Demo: https://your-site.netlify.app
```

---

## 🚀 Deploy to Netlify (2 steps)

### Option A — Drag & Drop
1. Go to [app.netlify.com](https://app.netlify.com)
2. Drag the `neural-ar/` folder onto the Netlify dashboard
3. Done. Your live URL appears instantly.

### Option B — Git Deploy
```bash
# 1. Push to GitHub
git init
git add .
git commit -m "Neural AR"
git remote add origin https://github.com/YOUR_USER/neural-ar.git
git push -u origin main

# 2. In Netlify: "New site from Git" → select your repo → deploy
```

---

## 🏗 Architecture

```
index.html  (single-file, ~750 lines)
│
├── CSS          — UI, loading overlay, scanlines, corner HUD
│
├── MediaPipe FaceMesh (CDN)
│   └── 468 facial landmarks → head bounding box → world-space anchor
│
├── Three.js r160 (CDN via importmap)
│   ├── WebGLRenderer  (screen blend-mode → AR compositing)
│   ├── PerspectiveCamera (FOV 60°, z=1.5)
│   └── Scene
│       ├── NodeSystem    — Points geometry, custom glow shader
│       ├── ConnectionSystem — LineSegments, energy-pulse shader
│       ├── PulseSystem   — travelling spark particles
│       └── AmbientField  — background floating dust
│
├── Post-Processing (EffectComposer)
│   ├── RenderPass
│   ├── UnrealBloomPass  (threshold 0.04, str 1.55, radius 0.5)
│   ├── ShaderPass       (chromatic aberration + vignette + film grain)
│   └── OutputPass       (sRGB / tone-mapping output)
│
└── JavaScript Systems
    ├── Noise.fbm()       — 3D Perlin FBM (4 octaves)
    ├── AudioReactor      — Microphone FFT → intensity boost
    ├── Mode switcher     — Calm / Intense (bloom + pulse speed)
    ├── Color schemes     — Cyber / Bio / Fire (3 palettes)
    └── Density control   — Sparse / Normal / Dense connections
```

---

## 🎨 GLSL Shaders

### Node Shader (`nodeMat`)
- **Vertex:** Layered pulse animation — slow breathe `sin(t*1.2)` + fast twinkle `sin(t*5.5)`
- **Fragment:** 3-layer radial glow (core/inner/halo), tri-color cycle (blue→purple→pink)
- **Blending:** `THREE.AdditiveBlending` → naturally composites on dark background

### Connection Shader (`lineMat`)
- **Vertex:** Passes `aT` ([0,1] along segment) + per-line `aPhase`
- **Fragment:** Two energy pulses using `fract(vT - time * speed + phase)`, endpoint fade
- **Blending:** Additive — lines glow without darkening the scene

### Pulse Particle Shader (`pulseMat`)
- **Vertex:** Twinkle via `sin(time * 14.0 + phase)`, screen-space point sizing
- **Fragment:** Simple radial glow with dual-color cycle

### Post-FX Shader (custom `ShaderPass`)
- Chromatic aberration (±0.25% UV offset on R/B channels)
- Radial vignette (`smoothstep`)
- Film grain via noise hash function

---

## ⚙️ Performance Tips (Mobile)

| Setting | Mobile Budget | Desktop |
|---------|--------------|---------|
| Node count | 60–80 | 100–150 |
| Connection maxPerNode | 4 | 6 |
| Bloom radius | 0.3 | 0.5 |
| Pixel ratio cap | 1.5 | 2.0 |
| FaceMesh resolution | 480p | 720p |

**Quick mobile optimization** — edit `CFG` at the top of `index.html`:
```js
// Detect mobile
const isMobile = /Mobi|Android/i.test(navigator.userAgent);
const CFG = {
  nodes:  { count: isMobile ? 70 : 120, ... },
  bloom:  { strength: isMobile ? 1.2 : 1.55, ... },
};
// Also lower pixel ratio:
renderer.setPixelRatio(isMobile ? 1 : Math.min(devicePixelRatio, 2));
```

---

## 🎮 Controls

| Button | Action |
|--------|--------|
| `◈ CALM / ⚡ INTENSE` | Toggle effect intensity (bloom strength, pulse speed, noise amplitude) |
| `◎ AUDIO` | Enable microphone input — loud sounds boost pulse intensity |
| `◑ SCHEME` | Cycle color palettes: **CYBER** (blue-purple-pink) → **BIO** (green-teal) → **FIRE** (red-gold) |
| `⊕ DENSITY` | Cycle connection density: Sparse → Normal → Dense |

---

## 🔧 Extending the Effect

### Add VFX Graph-style ribbon trails
Replace `PulseSystem` Points with `TubeGeometry` segments, rebuild per-frame using
`THREE.CatmullRomCurve3` on the last N positions of each pulse.

### Head rotation tracking
MediaPipe provides per-landmark depth (`lm.z`). Compute the normal of the face plane
from landmarks 1, 168, 199 to get a 3D rotation quaternion and apply it to `headGroup`.

### Hand interaction
Add `@mediapipe/hands` alongside FaceMesh. On pinch gesture, trigger a shockwave:
scale the `headGroup` briefly and spike `bloomPass.strength`.

### VR/XR mode
Replace the camera feed with WebXR session and use `renderer.xr.enabled = true`.
The Three.js scene runs identically in XR space.

---

## 📦 Dependencies (all via CDN, no npm needed)

```
three@0.160.0              — 3D engine + post-processing
@mediapipe/face_mesh       — 468-point real-time face tracking  
@mediapipe/camera_utils    — Camera lifecycle management
Google Fonts               — Share Tech Mono + Orbitron
```

---

## 📄 License
MIT — free to use, modify, and deploy commercially.
