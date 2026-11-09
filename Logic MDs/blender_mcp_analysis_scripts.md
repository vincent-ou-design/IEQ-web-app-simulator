# LEED v5 IEQ: Blender Python MCP Analysis Scripts & SketchUp Naming Dictionary

This document contains object naming conventions, SketchUp-to-Blender export instructions, and ready-to-execute Blender Python (`bpy`) scripts for Claude to analyze 3D models via the Model Context Protocol (MCP).

---

## 1. SketchUp Modeling & Component Definition Naming Dictionary

When modeling in SketchUp, set the **Component Definition Name** using these exact script prefixes. Duplicated copies inherit `.001`, `.002` suffixes in Blender, which the scripts automatically catch:

| Element | Required Prefix | Example Definition Name | Purpose in Analysis Script |
| :--- | :--- | :--- | :--- |
| **Occupant Desks / Seats** | `OCC_` | `OCC_Desk_Standard` | Sightline raycast origins (eye level 1.1m). |
| **Vision Glazing / Windows** | `GLZ_` | `GLZ_Win_VLT45_H2.8m` | Target for Quality Views (3 x H) and Daylight Proximity (20ft). |
| **Regularly Occupied Floor** | `AREA_OCC_` | `AREA_OCC_OpenOffice_Therm-Perimeter` | Square footage denominator + embedded thermal/acoustic zone tags. |
| **Unoccupied Floor** | `AREA_UNOCC_` | `AREA_UNOCC_Corridor` | Excluded service/circulation areas. |
| **Columns & Core** | `OBST_` | `OBST_Col_300mm` | Visual and solar ray obstructions (>1ft). |
| **Walls & Partitions** | `WALL_` | `WALL_Int_Partition` | Physical enclosure & acoustic barrier partitions. |
| **Ceilings & Baffles** | `CEIL_` | `CEIL_AcousticTile_NRC70` | Audited for room acoustic absorption ratings. |
| **Doors & Access** | `DR_` | `DR_Terrace_Outdoor` | Audited for Outdoor Connection proximity (<= 200ft). |

---

## 2. Exporting SketchUp to Blender

* **Option A (Direct `.skp`):** In Blender, use `File > Import > SketchUp (.skp)`.
* **Option B (Collada `.dae`):** In SketchUp, go to `File > Export > 3D Model > COLLADA (.dae)`. Check *"Preserve Component Hierarchies"*. In Blender, use `File > Import > Collada (.dae)`.

---

## 3. Python Analysis Scripts (`bpy`)

### A. Quality Views Script (Option 1 Path 2)
```python
import bpy, mathutils

def analyze_quality_views():
    scene = bpy.context.scene
    depsgraph = bpy.context.evaluated_depsgraph_get()

    occupants = [obj for obj in scene.objects if obj.name.startswith("OCC_")]
    glazings = [obj for obj in scene.objects if obj.name.startswith("GLZ_")]

    total = len(occupants)
    compliant = 0

    for occ in occupants:
        occ_loc = occ.matrix_world.translation
        has_view = False

        for glz in glazings:
            bbox = [glz.matrix_world @ mathutils.Vector(c) for c in glz.bound_box]
            max_z = max(v.z for v in bbox)
            max_dist = max_z * 3.0  # 3x Head Height Rule
            
            glz_center = sum(bbox, mathutils.Vector()) / 8.0
            dist = (glz_center - occ_loc).length

            if dist <= max_dist:
                direction = (glz_center - occ_loc).normalized()
                hit, loc, norm, idx, hit_obj, matrix = scene.ray_cast(depsgraph, occ_loc, direction, distance=dist)

                if not hit or hit_obj.name.startswith("GLZ_") or not (hit_obj.name.startswith("OBST_") or hit_obj.name.startswith("WALL_")):
                    has_view = True
                    break

        if has_view: compliant += 1

    pct = (compliant / total) * 100 if total > 0 else 0
    points = 3 if pct >= 90.0 else (2 if pct >= 75.0 else 0)
    return {"total_occupants": total, "compliant": compliant, "percentage": pct, "points": points}

print(analyze_quality_views())
```

### B. Daylight Proximity Script (Option 5 Path 3)
```python
import bpy, mathutils

def analyze_daylight_proximity():
    scene = bpy.context.scene
    floors = [obj for obj in scene.objects if obj.name.startswith("AREA_OCC_")]
    glazings = [obj for obj in scene.objects if obj.name.startswith("GLZ_")]

    total_area, compliant_area = 0.0, 0.0

    for flr in floors:
        matrix = flr.matrix_world
        for face in flr.data.polygons:
            area = face.area
            total_area += area
            center = matrix @ face.center
            
            min_dist = min([mathutils.Vector((center.x - g.matrix_world.translation.x, center.y - g.matrix_world.translation.y)).length for g in glazings], default=float('inf'))
            if min_dist <= 6.0:  # 20 feet (6.0m) rule
                compliant_area += area

    pct = (compliant_area / total_area) * 100 if total_area > 0 else 0
    points = 1 if pct >= 30.0 else 0
    return {"total_area_sqm": total_area, "compliant_area_sqm": compliant_area, "percentage": pct, "points": points}

print(analyze_daylight_proximity())
```