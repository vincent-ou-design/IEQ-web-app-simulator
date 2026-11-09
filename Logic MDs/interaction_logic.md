# Interaction Matrix, Presets & Live Tweak Commands

This guide defines how the UI widgets interact with the 3D scene engine and provides prompt commands for tweaking the 3D graphics live.

---

## Preset Button State Matrix

| Parameter / Slider | Reset Baseline | Recommended Strategy | Max 7 Points |
| :--- | :--- | :--- | :--- |
| **Nature View %** | $30\%$ (0 pts) | $60\%$ (1 pt) | $80\%$ (2 pts) |
| **Plant Coverage %** | $0\%$ (0 pts) | $12\%$ (+1 pt) | $15\%$ (+1 pt) |
| **Natural Wood %** | $0\%$ | $35\%$ | $45\%$ |
| **Ergo Desks %** | $10\%$ (0 pts) | $65\%$ (1 pt) | $85\%$ (1 pt) |
| **Room Flex Toggle** | Off | On | On |
| **Operative Temp** | $26.5^\circ\text{C}$ (0 pts) | $22.5^\circ\text{C}$ (1 pt) | $21.5^\circ\text{C}$ (1 pt) |
| **Local Fan Control**| Off | On | On |
| **Wall STC Rating** | 35 (0 pts) | 48 (1 pt) | 54 (2 pts) |
| **Ceiling NRC** | 0.50 | 0.75 | 0.85 |
| **HVAC Noise dBA** | 48 dBA | 38 dBA | 33 dBA |
| **Glazing VLT %** | $25\%$ (0 pts) | $45\%$ (1 pt) | $60\%$ (1 pt) |
| **CRI Rating** | 75 | 92 (+1 pt) | 96 (+1 pt) |
| **Automated Dimming**| Off | On (+1 pt) | On (+3 pts sDA) |
| **UNCAPPED SUM** | **0 Pts** | **7 Pts** | **12 Pts** |
| **CAPPED SCORE** | **0 / 7 Pts** | **5 / 7 Pts** | **7 / 7 Pts (Capped)** |

---

## 3D Synchronous Action Table

When a user moves a slider, JavaScript fires two distinct actions in parallel:

```text
[User Moves Slider]
       │
       ├──► [Action A: Scorecard Engine] ──► Recalculate Points ──► Update UI Badges
       │
       └──► [Action B: Three.js Engine]  ──► Update Mesh Material/Scale/Visibility
```

| Slider Changed | Target 3D Object | Three.js Action |
| :--- | :--- | :--- |
| **Glazing VLT** | `GLZ_Win` | Modifies `material.opacity` and glass roughness. |
| **Plant Coverage** | `PLANT_Group` | Sets `visible = (value > 0)` and scales plant geometry. |
| **Wall STC** | `WALL_Acoust` | Swaps material color depth/bump map texture. |
| **CRI & Dimming** | `LIGHT_Fixture` | Changes `light.intensity` and `light.color.setHSL()`. |
| **Ergo Desks** | `DESK_Ergo` | Animates desk surface $Z$-height upward/downward. |

---

## AI Command Template: Modifying 3D Graphics Live

Use this exact prompt whenever you want Antigravity or Codex to tweak the visual appearance of the 3D room without breaking the LEED calculator logic:

```text
[LIVE 3D GRAPHICS TWEAK]
Please update index.html. Keep all LEED scorecard math, sub-category calculations, and slider event listeners EXACTLY as they are. 

Modify ONLY the Three.js viewport styling and rendering code:
- Target Element: [e.g., GLZ_Win material / Room Ambient Light / Floor Reflection / Furniture Colors]
- Change Required: [e.g., "Add soft directional shadow cast under desks", "Make window glass reflection smoother", "Change floor material to light oak timber texture"]

Ensure the 3D scene remains lightweight, fast, and fully responsive inside index.html.
```