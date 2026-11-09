# Master AI Prompt: LEED v5 IEQc2 Single-File Web Simulator (`index.html`)

## Role & Mission
You are a principal web developer and 3D web application architect. Your objective is to generate a single, complete, production-ready `index.html` file that serves as an interactive simulator for **LEED v5 BD+C IEQ Credit EQc2 (Occupant Experience)**.

---

## Technical Specifications & CDN Stack
- **Architecture:** 100% self-contained single file (`index.html`). All CSS and JavaScript must be written inside `<style>` and `<script>` tags.
- **CDN Libraries Allowed:**
  - Tailwind CSS: `https://cdn.tailwindcss.com`
  - Three.js: `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`
  - OrbitControls: `https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js`
  - GLTFLoader: `https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js`
  - Lucide Icons (or FontAwesome CDN) for crisp UI icon badges.

---

## UI/UX Design System (Minimalist SaaS Style)
- **Palette:** Dark Slate / Off-White minimalist theme (`bg-slate-900`, `text-slate-100`, card backgrounds `bg-slate-800/80` with `backdrop-blur-md` and `border border-slate-700/50`).
- **Layout (Split Viewport):**
  - **Left Canvas Area (65% width):** Interactive 3D scene viewport with camera orbit, zoom, pan, and dynamic environment lighting.
  - **Right Control Sidebar (35% width, scrollable):** Header, preset control bar, real-time scorecard ticker, collapsible parameter accordions, and methodology cards.

---

## 3D Scene Engine Logic & GLTF Integration

1. **Procedural Fallback + GLTF Loader:**
   - Attempt to load `./room_model.glb` via `GLTFLoader`.
   - If `room_model.glb` is not found, render a sleek procedural 3D architectural room (floor plate, rear wall, window glazing frame, ceiling light fixtures, 2 desk setups, and indoor planter boxes).

2. **3D Mesh Tag Binding (`scene.traverse`):**
   The script must query object names inside the 3D model to bind visual properties directly to sidebar UI controls:
   - **`GLZ_Win`**: Vision glazing material. Adjust opacity (`0.2` to `0.8`) and tint based on the **Glazing VLT** slider.
   - **`LIGHT_Fixture`**: Ambient/Point lights. Adjust light intensity (`0.2` to `1.8`) and color temperature (warm 2700K to cool 5000K) based on **CRI & Dimming** controls.
   - **`PLANT_Group`**: Biophilic planters. Toggle visibility (`visible = true/false`) and scale based on **Indoor Plant Density** slider.
   - **`WALL_Acoust`**: Wall acoustic panels. Toggle texture pattern or material color depth based on **Wall STC Rating**.
   - **`DESK_Ergo`**: Furniture markers. Toggle sit-stand desk height state based on **Adaptable Layout** toggles.

3. **Viewport Floating Controls Overlay:**
   - [Reset Camera] button.
   - [Toggle Ceiling Visibility] button.
   - [Toggle Thermal Zone Overlay] button (displays subtle colored wireframe boundary boxes).

---

## Scorecard Engine & LEED Rules (Strict 7-Point Cap)

The app must dynamically calculate points across all 5 sub-categories and enforce a **hard cap at 7 points total**:

$$\text{Total Score} = \min\left(7, \; \text{Pts}_{\text{Bio}} + \text{Pts}_{\text{Adap}} + \text{Pts}_{\text{Therm}} + \text{Pts}_{\text{Sound}} + \text{Pts}_{\text{Light}}\right)$$

### Categories:
1. **Biophilic Design (1–4 pts):** Sliders for Direct Nature View %, Indoor Plant Coverage %, Natural Timber/Stone %.
2. **Adaptable Spaces (1 pt):** Toggles for Height-Adjustable Desks & Flexible Room Partitioning.
3. **Thermal Comfort (1 pt):** Operative Temperature Slider (18°C–28°C) & Fan Control Toggle.
4. **Sound & Acoustics (1–2 pts):** Wall STC Slider, Ceiling NRC Slider, HVAC Noise dBA Slider.
5. **Lighting Quality (1–6 pts):** Glazing VLT %, CRI Rating, Dynamic Dimming Control.

---

## Presets & Quick Actions Bar
Provide 3 top quick-action buttons that set all sliders simultaneously:
- **`[Reset Baseline]`**: Sets low/non-compliant values (0 / 7 Pts).
- **`[Recommended Strategy]`**: Sets balanced target thresholds (5 / 7 Pts).
- **`[Max 7 Points]`**: Sets high-performance values hitting the global 7-point cap (Capped 7 / 7 Pts).

---

## Code Quality Requirements
- Clean modular JavaScript structure (State Object, UI Event Listeners, 3D Renderer loop, Score Calculator).
- Zero broken tags or truncated scripts.
- Smooth animations for score progress bars and 3D camera transitions.