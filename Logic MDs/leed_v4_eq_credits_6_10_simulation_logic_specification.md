# LEED v4 EQ Credits 6–10: Simulation Logic & Calculation Rules

This document outlines the simulation rules, input variables, decision trees, and mathematical evaluation algorithms for implementing LEED v4 Indoor Environmental Quality (EQ) Credits 6 through 10 in a web application calculator or simulation engine.

---

## Data Schema Overview

```json
{
  "project_metadata": {
    "rating_system": "BD+C_NC", // Options: BD+C_NC, Core_Shell, Schools, Retail, Data_Centers, Warehouses, Hospitality, Healthcare
    "total_regularly_occupied_area_sqft": 50000.0,
    "total_individual_occupant_spaces": 40,
    "total_shared_multioccupant_spaces": 10
  }
}
```

---

## 1. Credit 6: Thermal Comfort

### 1.1 Inputs
| Variable Name | Type | Unit / Enum | Description |
| :--- | :--- | :--- | :--- |
| `design_standard` | String | `"ASHRAE_55_2010" \| "ISO_7730_2005" \| "NONE"` | Standard used for HVAC envelope & comfort design |
| `indiv_space_controls_pct` | Float | Percentage ($0 - 100$) | % of individual spaces with thermal controls |
| `shared_space_controls_pct` | Float | Percentage ($0 - 100$) | % of shared multioccupant spaces with group controls |
| `patient_room_controls_pct` | Float | Percentage ($0 - 100$) | % of healthcare patient rooms with individual controls |

### 1.2 Evaluation Rules
```python
def evaluate_eq_credit_6(project, inputs):
    # Standard compliance check
    design_compliant = inputs.design_standard in ["ASHRAE_55_2010", "ISO_7730_2005"]
    
    # Control compliance check
    indiv_pass = inputs.indiv_space_controls_pct >= 50.0
    shared_pass = inputs.shared_space_controls_pct == 100.0
    
    if project.rating_system == "Healthcare":
        patient_pass = inputs.patient_room_controls_pct == 100.0
        controls_compliant = indiv_pass and shared_pass and patient_pass
    else:
        controls_compliant = indiv_pass and shared_pass

    # Scoring
    earned_points = 1 if (design_compliant and controls_compliant) else 0
    
    return {
        "credit": "EQc6_Thermal_Comfort",
        "earned_points": earned_points,
        "max_points": 1,
        "status": "PASS" if earned_points == 1 else "FAIL"
    }
```

---

## 2. Credit 7: Interior Lighting

### 2.1 Inputs
| Variable Name | Type | Unit / Enum | Description |
| :--- | :--- | :--- | :--- |
| `option1_indiv_control_pct` | Float | Percentage ($0 - 100$) | % of individual spaces with $\ge 3$ scene levels |
| `option1_shared_control_pct` | Float | Percentage ($0 - 100$) | % of shared spaces with multi-zone $\ge 3$ scene levels |
| `option1_presentation_control` | Boolean | True / False | Presentation lighting separately controlled |
| `option2_strategies_met` | Array | `["A", "B", "C", "D", "E", "F", "G", "H"]` | Array of satisfied lighting quality strategies |

### 2.2 Evaluation Rules
```python
def evaluate_eq_credit_7(project, inputs):
    points = 0
    
    # Option 1 Logic: Lighting Control (1 Point)
    opt1_pass = (
        inputs.option1_indiv_control_pct >= 90.0 and
        inputs.option1_shared_control_pct == 100.0 and
        inputs.option1_presentation_control == True
    )
    if opt1_pass:
        points += 1

    # Option 2 Logic: Lighting Quality (1 Point)
    # Requires meeting at least 4 out of 8 strategies
    unique_strategies = set(inputs.option2_strategies_met)
    opt2_pass = len(unique_strategies) >= 4
    if opt2_pass:
        points += 1

    return {
        "credit": "EQc7_Interior_Lighting",
        "earned_points": points,
        "max_points": 2,
        "option_1_status": "PASS" if opt1_pass else "FAIL",
        "option_2_status": "PASS" if opt2_pass else "FAIL"
    }
```

---

## 3. Credit 8: Daylight

### 3.1 Inputs
| Variable Name | Type | Unit / Enum | Description |
| :--- | :--- | :--- | :--- |
| `has_glare_control_devices` | Boolean | True / False | Mandatory glare-control in regularly occupied spaces |
| `selected_option` | Integer | `1`, `2`, or `3` | Selected compliance path |
| `sda_300_50_pct` | Float | Percentage ($0 - 100$) | Spatial Daylight Autonomy ($\text{sDA}_{300/50\%}$) |
| `ase_1000_250_pct` | Float | Percentage ($0 - 100$) | Annual Sunlight Exposure ($\text{ASE}_{1000,250}$) |
| `daylight_area_pct` | Float | Percentage ($0 - 100$) | Area meeting $300 - 3,000\text{ lux}$ (Options 2 & 3) |

### 3.2 Evaluation Rules
```python
def evaluate_eq_credit_8(project, inputs):
    # Mandatory prerequisite condition
    if not inputs.has_glare_control_devices:
        return {"credit": "EQc8_Daylight", "earned_points": 0, "status": "FAIL_GLARE_PREREQUISITE"}

    is_healthcare = project.rating_system == "Healthcare"
    max_pts = 2 if is_healthcare else 3
    points = 0

    if inputs.selected_option == 1: # Simulation: sDA + ASE
        # ASE must be <= 10%
        if inputs.ase_1000_250_pct <= 10.0:
            if inputs.sda_300_50_pct >= 75.0:
                points = 2 if is_healthcare else 3
            elif inputs.sda_300_50_pct >= 55.0:
                points = 1 if is_healthcare else 2

    elif inputs.selected_option in [2, 3]: # Illuminance Simulation or Measurement
        if inputs.daylight_area_pct >= 90.0:
            points = 2 if is_healthcare else 3 if inputs.selected_option == 3 else 2
        elif inputs.daylight_area_pct >= 75.0:
            points = 1

    return {
        "credit": "EQc8_Daylight",
        "earned_points": min(points, max_pts),
        "max_points": max_pts,
        "status": "PASS" if points > 0 else "FAIL"
    }
```

---

## 4. Credit 9: Quality Views

### 4.1 Inputs
| Variable Name | Type | Unit / Enum | Description |
| :--- | :--- | :--- | :--- |
| `direct_line_of_sight_pct` | Float | Percentage ($0 - 100$) | % of regularly occupied area with direct line of sight |
| `qualifying_view_area_pct` | Float | Percentage ($0 - 100$) | % of area satisfying $\ge 2$ of 4 view criteria |
| `atria_view_contribution_pct` | Float | Percentage ($0 - 100$) | Atria view contribution (capped at $30\%$ of total) |

### 4.2 Evaluation Rules
```python
def evaluate_eq_credit_9(project, inputs):
    is_healthcare = project.rating_system == "Healthcare"
    max_pts = 2 if is_healthcare else 1
    
    # Cap atrium contribution at 30% of total compliant area
    effective_atria = min(inputs.atria_view_contribution_pct, 30.0)
    adjusted_view_area = inputs.qualifying_view_area_pct + effective_atria

    # Direct line of sight check
    sight_compliant = inputs.direct_line_of_sight_pct >= 75.0
    view_criteria_compliant = adjusted_view_area >= 75.0

    if sight_compliant and view_criteria_compliant:
        earned_points = max_pts
    else:
        earned_points = 0

    return {
        "credit": "EQc9_Quality_Views",
        "earned_points": earned_points,
        "max_points": max_pts,
        "status": "PASS" if earned_points > 0 else "FAIL"
    }
```

---

## 5. Credit 10: Acoustic Performance

### 5.1 Inputs
| Variable Name | Type | Unit / Enum | Description |
| :--- | :--- | :--- | :--- |
| `hvac_background_noise_pass` | Boolean | True / False | HVAC noise meets ASHRAE Ch. 48 / AHRI 885 limits |
| `sound_transmission_pass` | Boolean | True / False | Partition assemblies meet $\text{STC}_c$ design targets |
| `reverberation_time_pass` | Boolean | True / False | $T_{60}$ values meet space-by-space thresholds |
| `sound_reinforcement_masking_pass` | Boolean | True / False | STI $\ge 0.60$ for $>50$ seats; masking $\le 48\text{ dBA}$ |

### 5.2 Evaluation Rules
```python
def evaluate_eq_credit_10(project, inputs):
    is_healthcare = project.rating_system == "Healthcare"
    max_pts = 2 if is_healthcare else 1
    
    # All 4 core criteria must be met for non-healthcare projects
    core_components = [
        inputs.hvac_background_noise_pass,
        inputs.sound_transmission_pass,
        inputs.reverberation_time_pass,
        inputs.sound_reinforcement_masking_pass
    ]
    
    passed_count = sum(1 for c in core_components if c)
    
    if is_healthcare:
        # Healthcare awards 1 pt for Option 1, 1 pt for Option 2
        earned_points = 2 if passed_count == 4 else (1 if passed_count >= 2 else 0)
    else:
        earned_points = 1 if passed_count == 4 else 0

    return {
        "credit": "EQc10_Acoustic_Performance",
        "earned_points": earned_points,
        "max_points": max_pts,
        "status": "PASS" if earned_points > 0 else "FAIL"
    }
```