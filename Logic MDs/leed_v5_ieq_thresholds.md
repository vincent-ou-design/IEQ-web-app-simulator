# LEED v5 IEQ Credit EQc2 Metric & Point Calculation Rules

This document outlines the mathematical rules, thresholds, and scoring logic embedded into the simulator.

---

## Global Scoring Rule
- **Credit Maximum Cap:** $\mathbf{7.0\text{ Points}}$
- Sum of individual sub-categories may equal up to $14\text{ points}$, but the final score displayed on the scorecard must be capped at $7$.

---

## Sub-Category Breakdown & Logic

### 1. Biophilic Design (Max 4 Points)
$$\text{Pts}_{\text{Bio}} = \text{Pts}_{\text{View}} + \text{Pts}_{\text{Elements}} + \text{Pts}_{\text{Feature}}$$

* **Direct View to Nature ($\text{Pts}_{\text{View}}$):**
  * $\text{View} < 50\% \implies 0\text{ pts}$
  * $50\% \le \text{View} < 75\% \implies 1\text{ pt}$
  * $\text{View} \ge 75\% \implies 2\text{ pts}$
* **Indoor Natural Elements ($\text{Pts}_{\text{Elements}}$):**
  * $\text{Plant Coverage} \ge 10\% \text{ area} \text{ AND } \text{Natural Timber/Stone} \ge 30\% \text{ surface} \implies +1\text{ pt}$
* **Water/Courtyard Connection ($\text{Pts}_{\text{Feature}}$):**
  * Active outdoor courtyard / water feature connection toggle $\implies +1\text{ pt}$

---

### 2. Adaptable Spaces (Max 1 Point)
$$\text{Pts}_{\text{Adap}} = \begin{cases} 1 & \text{if Ergonomic Desks } \ge 50\% \text{ AND Multi-Zone Toggle = True} \\ 0 & \text{otherwise} \end{cases}$$

---

### 3. Thermal Comfort (Max 1 Point)
Complying with ASHRAE Standard 55-2023 operative comfort zones:

$$\text{Pts}_{\text{Therm}} = \begin{cases} 1 & \text{if } 20.0^\circ\text{C} \le T_{\text{op}} \le 24.0^\circ\text{C} \text{ AND Local Fan Control = True} \\ 0 & \text{otherwise} \end{cases}$$

---

### 4. Sound & Acoustics (Max 2 Points)

* **Tier 1 ($1\text{ pt}$):** $\text{STC} \ge 45 \text{ AND } \text{HVAC Noise} \le 40\text{ dBA}$
* **Tier 2 ($2\text{ pts}$):** $\text{STC} \ge 50 \text{ AND } \text{NRC} \ge 0.75 \text{ AND } \text{HVAC Noise} \le 35\text{ dBA}$

---

### 5. Lighting Quality (Max 6 Points)

* **Daylighting VLT:** $\text{VLT} \ge 40\% \implies +1\text{ pt}$
* **High CRI & Task Control:** $\text{CRI} \ge 90 \text{ AND Task Control = True} \implies +1\text{ pt}$
* **Automated Color Tuning:** $\text{Continuous Dimming = True} \implies +1\text{ pt}$
* **Spatial Daylight Autonomy (sDA):**
  * $\text{sDA}_{300/50\%} \ge 55\% \implies +1\text{ pt}$
  * $\text{sDA}_{300/50\%} \ge 75\% \implies +3\text{ pts}$

---

## Certification Status Badges

| Capped Score Range | Status Badge Label | UI Badge Color |
| :--- | :--- | :--- |
| **0.0 – 2.9 Points** | Non-Compliant / Baseline | Gray / Red (`bg-red-900/50 text-red-300`) |
| **3.0 – 4.9 Points** | LEED Silver Target | Amber (`bg-amber-900/50 text-amber-300`) |
| **5.0 – 6.9 Points** | LEED Gold / Certified Strategy | Emerald (`bg-emerald-900/50 text-emerald-300`) |
| **7.0 Points (Max)** | Platinum Exemplary Performance | Electric Cyan (`bg-cyan-900/50 text-cyan-300`) |