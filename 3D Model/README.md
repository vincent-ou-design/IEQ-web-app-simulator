# 3D Model Folder & SketchUp Export Guide

Place your exported `.glb` file directly in this folder:
- **`Web App Simulator/3D Model/room_model.glb`**

---

### Component Naming Conventions

#### 1. Biophilic Plant Tiers (Opacity / Visibility Staged Fading)
Name your plant components using these exact prefix patterns:
- **`PLANT_10`** (e.g. `PLANT_10_A`, `PLANT_10_B`, `PLANT_10_Wall`, etc.):
  - Activates when Plant Coverage is between **1% and 10%**.
  - Opacity smoothly fades in from 0.0 to 1.0 as the slider approaches 10%.
- **`PLANT_20`** (e.g. `PLANT_20_A`, `PLANT_20_B`, `PLANT_20_Corner`, etc.):
  - Activates when Plant Coverage is between **11% and 20%**.
  - Opacity smoothly fades in from 0.0 to 1.0 as the slider approaches 20%.
- **`PLANT_30`** (e.g. `PLANT_30_A`, `PLANT_30_B`, `PLANT_30_Hanging`, etc.):
  - Activates when Plant Coverage is between **21% and 30%** (Max).
  - Opacity smoothly fades in from 0.0 to 1.0 as the slider approaches 30%.

*(Note: In SketchUp, avoiding special characters like `%` in component names is recommended—e.g. `PLANT_10_A` instead of `PLANT_10% A`—though the loader script will match both `PLANT_10` and `PLANT_10%`).*

---

#### 2. Windows & Glazing
- **`GLZ_Win`** (or `GLZ_Win_Front`, `GLZ_Win_Side`):
  - Controls glass transparency, transmission, tint, and daylighting penetration.

#### 3. Light Fixtures
- **`LIGHT_Fixture`** (or `LIGHT_Fixture_1`, `LIGHT_Fixture_2`):
  - Ceiling luminaires that adjust intensity and Kelvin temperature (cool to warm).

#### 4. Acoustic Walls / Ceiling Panels
- **`WALL_Acoust`**:
  - Acoustic wall finishes/panels and acoustic boundary surfaces.

#### 5. Ergonomic Furniture
- **`DESK_Ergo`**:
  - Sit-stand desks with dynamic height adjustments.

---

### SketchUp Export Checklist
1. Go to **File -> Export -> 3D Model**.
2. Select **`GLTF / GLB File (*.glb)`**.
3. Under Export Options:
   - ✅ **Export Component Hierarchy**
   - ✅ **Preserve Component Names**
4. Save as **`room_model.glb`** inside this `3D Model/` directory.
