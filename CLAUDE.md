# OndaEsfera

Simulação interativa da equação da onda em uma superfície esférica usando Three.js.

## Project Structure

- `ondaesfera.html` — Minimal simulation (Three.js r0.158, ~143 lines). Auto-rotating sphere, click to excite, vertex-color visualization. No external controls.
- `ondanaesfera-glm.html` — Full-featured simulation (Three.js r0.128 + OrbitControls). Custom ShaderMaterial, UI panel with sliders (velocity, damping, impact force, sound volume), reset button, color legend, and physically-modeled drum audio.
- `manual_test_report.md` — Manual QA report for both apps.

## Tech Stack

- **Language**: Vanilla JavaScript (no build step)
- **3D Engine**: Three.js (loaded via CDN)
- **Physics**: Finite-difference method for the wave equation on a sphere, discrete Laplacian over mesh neighbors
- **UI language**: Portuguese (pt-BR)

## Running Locally

```bash
python3 -m http.server 8000
# Open http://localhost:8000/ondaesfera.html
# Open http://localhost:8000/ondanaesfera-glm.html
```

## Key Implementation Details

- Both apps use `Float32Array` buffers (`u`, `uPrev`, `uNext`) with buffer-swap per frame.
- `ondaesfera.html` pre-computes a neighbor list from the index buffer for O(1) Laplacian lookups.
- `ondanaesfera-glm.html` pre-computes neighbors once in `createSphere()` via a distance threshold (`dist² < 0.1`). The CPU positions never change (displacement happens only in the vertex shader), so the per-frame Laplacian is O(n·k); only the one-time precompute is O(n²).
- `ondaesfera.html` maps duplicated SphereGeometry vertices (longitude seam / poles) to a canonical representative so waves propagate continuously across the seam.
- `ondanaesfera-glm.html` audio is physical modeling, not triggered samples: an `AudioWorklet` integrates the sphere's normal-mode ODEs (`q'' + 2γq' + ω_l²q = 0`, with inharmonic eigenfrequencies `ω_l = ω₁·√(l(l+1)/2)`) at audio rate. The output sample is the sum of modal displacements — the membrane itself ringing. The worklet is loaded from an inline Blob URL to keep the app single-file; it must be started from a user gesture (first click). Velocity→pitch (ω₁), damping→decay (γ), impact force→strike energy.
- Interaction: raycaster → Gaussian impulse centered on the click point.
- No tests or linting configured. Validate changes by opening in a browser.

## Style Conventions

- Inline `<script>` and `<style>` blocks (single-file HTML apps).
- Comments and UI text in Portuguese.
- Keep each app self-contained (no shared JS/CSS files).
