# Modular Thermal Battery Cooling System for Edge Computing

![Status](https://img.shields.io/badge/Status-Concept%20Stage-yellow)
![Domain](https://img.shields.io/badge/Domain-AI%20Chip%20Thermal%20Management-blue)
![Method](https://img.shields.io/badge/Method-Ice%20Storage%20%2B%20VIP%20Insulation-orange)
![Application](https://img.shields.io/badge/Application-H100%20%7C%20A100%20%7C%20Edge%20AI-purple)

A modular liquid cooling system for AI chips requiring direct liquid cooling — H100, A100, and equivalent high-TDP accelerators — built around a VIP-insulated ice thermal battery. Decouples cooling from continuous grid draw by charging the ice bank during off-peak or high-renewable periods and delivering 6 hours of autonomous cooling with no compressor running. Scales linearly by adding parallel units, identical to how split-unit ACs are deployed.

---

## The Problem This Solves

High-TDP AI accelerators (NVIDIA H100: 700 W, A100: 400 W) require direct liquid cooling — air cooling is insufficient at these power densities. Existing liquid cooling solutions for edge AI deployments have three persistent gaps:

| Gap | Problem |
|---|---|
| **Compressor always on** | Continuous grid draw; can't align with renewable availability |
| **Grid dependency** | Cooling fails or degrades when grid power is interrupted or fluctuates |
| **Fixed capacity** | No modular path — scaling compute means replacing the entire cooling system |

**This system decouples cooling production from cooling delivery.** The compressor makes ice during a 6-hour off-peak or solar window. The ice bank then delivers liquid cooling for the next 6 hours with zero compressor input — pump only, 11 W parasitic.

---

## Highlights

- **2 kW compressor → 5.8 kW sustained cooling** (COP 2.9)
- **~8 × H100-class GPUs** (700 W TDP) per base unit, or **~14 × A100-class** (400 W)
- **Ice thermal battery: 353 kg / 459-litre tank** — 6 hours autonomous cooling
- **VIP insulation: 14 W standby heat leak** — 0.2% of load per hour
- **Charging time = 6 hours** — charge overnight, cool all day
- **Modular:** 2 units = 16 H100s, 3 units = 24 H100s — add units like split ACs
- **Coolant: 5 °C supply, 5.9 L/min** — direct cold plate compatibility with NVIDIA OCP liquid cooling spec
- Pump parasitic during autonomous discharge: **11 W**

---

## System Architecture

```
OUTDOOR UNIT (refrigerant circuit)          INDOOR UNIT (ice bank + pump loop)
┌─────────────────────────────┐             ┌──────────────────────────────────┐
│  2 kW Compressor            │             │  VIP-insulated ice tank (459 L)  │
│  R600a circuit              │──evap coil──│  353 kg water/ice at 0 °C        │
│  Air-cooled condenser       │             │  Submerged evaporator coil       │
│  (16.5 m² finned, outdoor)  │             │  Brazed plate HX (ice → glycol)  │
└─────────────────────────────┘             │  Pump (11 W)                     │
                                            └──────────┬───────────────────────┘
                                                       │  5 °C glycol supply
                                                       ▼
                                            ┌──────────────────────┐
                                            │  AI Chip Cold Plates  │
                                            │  H100 / A100 nodes    │
                                            │  20 °C glycol return  │
                                            └──────────────────────┘
```

**Charging mode** (compressor on, 6 hr): refrigerant evaporator freezes water in tank.
**Autonomous mode** (compressor off, 6 hr): pump circulates glycol through ice → cold plates → chips.
**Simultaneous mode**: compressor runs while ice discharges — extends total window beyond 6 hours.

---

## Refrigerant Selection

Ice storage requires the refrigerant to run below 0 °C to freeze water. Evaporating temperature is set at **−5 °C** (5 K below 0 °C to drive freezing). The selection was made on COP and safety classification.

| Refrigerant | Class | COP at −5/45 °C | Notes |
|---|---|---|---|
| R134a | A1 (non-flammable) | ~2.4 | Safe indoors; lower COP |
| R290 (propane) | A3 (flammable) | ~2.8 | Good COP, outdoor unit only |
| **R600a (isobutane)** | **A3 (flammable)** | **2.9** | **Best COP; outdoor unit only — A3 acceptable** |
| R513A | A1 | ~2.5 | Low-GWP R134a alternative |

**R600a selected** — highest COP among available options (2.9 vs 2.4 for R134a). Contained entirely in the outdoor split unit, identical safety arrangement to R290-based residential heat pumps and window ACs.

---

## Thermodynamic Cycle (R600a)

### Operating Conditions

| Parameter | Value | Basis |
|---|---|---|
| Evaporating temperature | −5 °C | 5 K below ice bank (0 °C) — drives water freezing |
| Condensing temperature | 45 °C | Ambient 35 °C + 10 K approach ΔT |
| Compressor power | 2.0 kW | Design input |
| Isentropic efficiency | 72% | Typical hermetic reciprocating compressor |

### Cycle State Points

| State | Condition | Enthalpy |
|---|---|---|
| 1 — Compressor inlet | Saturated vapour, −5 °C | h₁ = 525 kJ/kg |
| 2 — Compressor outlet | Superheated vapour | h₂ = 611 kJ/kg |
| 3 — Condenser outlet | Saturated liquid, 45 °C | h₃ = 275 kJ/kg |
| 4 — Expansion valve outlet | After throttle | h₄ = 275 kJ/kg |

### Performance

```
q_evap = h₁ − h₄ = 525 − 275 = 250 kJ/kg
q_cond = h₂ − h₃ = 611 − 275 = 336 kJ/kg
w_comp = h₂ − h₁ = 611 − 525 =  86 kJ/kg

COP = q_evap / w_comp = 250 / 86 = 2.9

ṁ_refrigerant = 2,000 / 86,000 = 23.3 g/s

Q_evap (ice-making) = 23.3×10⁻³ × 250,000 = 5,810 W ≈ 5.8 kW
Q_cond (rejected)   = 23.3×10⁻³ × 336,000 = 7,830 W ≈ 7.8 kW

Check: 5.8 + 2.0 = 7.8 ✓
```

---

## AI Chip Cooling Capacity

### Per Base Unit

```
Coolant loop: 5 °C supply → 20 °C return  (15 °C ΔT across cold plates)
Sustained cooling per unit: 5.8 kW

H100-class (700 W TDP):  5,810 / 700  = 8.3 → ~8 GPUs per unit
A100-class (400 W TDP):  5,810 / 400  = 14.5 → ~14 GPUs per unit
```

### Modular Scaling

Units operate in parallel — each has its own compressor, ice tank, and pump loop, all feeding the same coolant distribution manifold. Each unit is independently charged and serviced.

| Units | Cooling Capacity | H100-class | A100-class |
|---|---|---|---|
| **1** | **5.8 kW** | **~8** | **~14** |
| 2 | 11.6 kW | ~16 | ~29 |
| 3 | 17.4 kW | ~24 | ~43 |
| 4 | 23.2 kW | ~33 | ~58 |
| N | N × 5.8 kW | N × 8 | N × 14 |

---

## Ice Thermal Battery

### Why Water/Ice

Water is the highest-energy-density PCM for below-ambient storage. Organic PCMs in the 15–20 °C range have lower latent heat and require a warmer coolant supply unsuitable for direct AI chip cold plates.

| PCM | Melting Point | Latent Heat | Notes |
|---|---|---|---|
| n-Hexadecane | 18 °C | 238 kJ/kg | Too warm — coolant supply would be ~15 °C, marginal for H100 |
| n-Pentadecane | 10 °C | 205 kJ/kg | Lower latent heat, marginal coolant supply |
| **Water / ice** | **0 °C** | **334 kJ/kg** | **40% higher energy density; 5 °C supply fully compatible with OCP cold plate spec** |

### Sizing

**Target: 5.8 kW × 6 hours = 34.8 kWh = 125,500 kJ**

```
Water/ice effective storage per kg:
  Latent heat (phase change at 0 °C):     334 kJ/kg
  Sensible (water cooling 5 °C → 0 °C):   21 kJ/kg
  Total effective:                         355 kJ/kg

Ice mass:               125,500 / 355 = 353 kg
Volume (liquid water):  353 / 1,000   = 353 litres
Volume (frozen ice):    353 / 917     = 385 litres  (+9% expansion — designed-in)

Tank volume (30% overhead — coil, expansion, structure): 459 litres
```

### Charging Time

```
Energy to freeze 353 kg:  353 × 334 = 117,900 kJ  (latent)
Compressor ice-making rate: Q_evap = 5,810 W

Charging time = 117,900,000 / 5,810 = 6.0 hours

→ 6 hr off-peak charging → 6 hr autonomous cooling at full load
→ Overnight charge window (00:00–06:00) covers full day shift (06:00–12:00)
→ Simultaneous compressor + ice discharge extends total window beyond 6 hr
```

---

## VIP Insulation Analysis

The ice bank operates at 0 °C in a server room at ~30 °C. Without adequate insulation, heat leak self-discharges the ice during standby and reduces effective autonomy. Insulation choice is critical.

### Tank Geometry

```
Tank inner dimensions: 600 × 550 × 970 mm
Total external surface area: 2.89 m²
ΔT (server room − ice): 30 °C
```

### Insulation Comparison

| Material | λ (W/m·K) | Thickness | R-value | Heat Leak | Standby Loss/hr |
|---|---|---|---|---|---|
| Polyurethane foam | 0.025 | 80 mm | 3.20 m²K/W | 27 W | 0.5% |
| Extruded polystyrene | 0.033 | 100 mm | 3.03 m²K/W | 29 W | 0.5% |
| Aerogel blanket | 0.015 | 30 mm | 2.00 m²K/W | 43 W | 0.7% |
| **VIP (vacuum insulated panel)** | **0.004** | **25 mm** | **6.25 m²K/W** | **14 W** | **0.2%** |

**VIP panels selected.**

```
λ_VIP  = 0.004 W/m·K  (effective, aged — new panels achieve 0.002 W/m·K)
Thickness: 25 mm
R-value: 0.025 / 0.004 = 6.25 m²K/W

Heat leak:  Q = A × ΔT / R = 2.89 × 30 / 6.25 = 13.9 W ≈ 14 W

Over 6-hour autonomous period:
  Energy lost to standby: 14 × 6 × 3600 = 302,400 J = 0.08 kWh
  Ice consumed by standby: 302,400 / 355,000 = 0.85 kg  (<0.3% of 353 kg) ✓
```

**Key insight:** A 25 mm VIP panel outperforms 156 mm of polyurethane foam at the same heat leak. Using PU foam at equivalent performance would add ~130 mm to every wall — increasing tank external footprint by ~500 mm in each dimension and making indoor installation impractical.

---

## Liquid Cooling Loop

### Design

```
Coolant: 30% ethylene glycol / water
  Freeze point: −14 °C  (safe in cold environments, no biological growth)
  Compatible with aluminium and copper cold plates

Supply / Return:  5 °C / 20 °C  (15 °C ΔT across chip cold plates)
Cp_glycol = 3,800 J/kg·K  |  ρ = 1,045 kg/m³

Mass flow:  ṁ = 5,810 / (3,800 × 15) = 102 g/s
Volume flow = 102 / 1,045 × 60,000 = 5.9 L/min
```

### Pump Power

```
Loop ΔP: 60 kPa  (distribution headers + cold plate manifolds, compact deployment)
Pump efficiency: 55%

P_pump = (ṁ/ρ × ΔP) / η = (102×10⁻³/1,045 × 60,000) / 0.55 = 10.7 W

Autonomous mode total draw: 10.7 W for the entire cooling loop ✓
```

---

## Heat Exchanger Sizing

### a) Ice Bank → Coolant — Brazed Plate HX

Glycol returns at 20 °C from chips, cooled to 5 °C by ice water at ~0.5 °C:

```
ΔT₁ = 20.0 − 0.5 = 19.5 °C  (warm glycol return vs ice water)
ΔT₂ =  5.0 − 0.5 =  4.5 °C  (cooled supply vs ice water)

LMTD = (19.5 − 4.5) / ln(19.5/4.5) = 10.2 °C

U = 2,000 W/m²K  (brazed plate HX, turbulent water-water)

A = 5,810 / (2,000 × 10.2) = 0.285 m² = 2,850 cm²
(achievable in a standard 20-plate BPHE unit — off-the-shelf component)
```

### b) Refrigerant Evaporator in Ice Tank — Charging Mode

R600a evaporating at −5 °C freezes water. Freezing is isothermal at 0 °C:

```
ΔT = 0 − (−5) = 5 °C  (isothermal — water freezes at 0 °C)

U = 400 W/m²K  (R600a boiling, submerged coil; decreases slightly as ice layer grows)

A = 5,810 / (400 × 5) = 2.9 m²

Coil length = A / (π × D) = 2.9 / (π × 0.032) ≈ 29 m of 32 mm OD copper tube
(wound and submerged in tank — standard ice-bank construction method)
```

### c) Outdoor Condenser — Air-Cooled

R600a condensing at 45 °C, ambient air from 30 °C → 38 °C:

```
ΔT₁ = 45 − 30 = 15 °C,  ΔT₂ = 45 − 38 = 7 °C

LMTD = (15 − 7) / ln(15/7) = 10.5 °C

U = 45 W/m²K  (finned air-cooled coil, forced fan convection)

A_cond = 7,830 / (45 × 10.5) = 16.5 m²  (finned surface area)
(standard outdoor split-unit coil — 700 × 600 mm face area, 3-row)
```

---

## Design Summary

| Parameter | Value |
|---|---|
| Refrigerant | R600a \| Evap −5 °C \| Cond 45 °C |
| Compressor input | 2.0 kW |
| Cooling output | **5.8 kW** |
| COP | **2.9** |
| H100-class capacity | **~8 GPUs per unit** (700 W TDP) |
| A100-class capacity | **~14 GPUs per unit** (400 W TDP) |
| PCM | Water/ice \| 0 °C \| 334 kJ/kg |
| Ice mass / tank volume | 353 kg / **459 litres** |
| Thermal storage | **34.8 kWh** |
| Charging time | **6.0 hr** → **6.0 hr autonomous** |
| VIP insulation | 25 mm \| λ = 0.004 W/m·K \| **14 W heat leak** |
| Coolant supply | **5 °C** \| 5.9 L/min \| **11 W pump** |
| Ice–coolant HX | 2,850 cm² brazed plate (off-the-shelf BPHE) |
| Evaporator coil | 2.9 m² — 29 m of 32 mm copper tube |
| Outdoor condenser | 16.5 m² finned |

---

## Target Applications

| Deployment | Fit |
|---|---|
| **AI inference server (8× H100 node)** | 1 unit, 5 °C coolant, direct OCP cold plate compatibility |
| **Edge AI at manufacturing / logistics** | Charge on overnight off-peak tariff; autonomous through day shift |
| **Telecom base station with AI** | 6-hr autonomy covers grid outages; no specialist HVAC |
| **Remote / off-grid AI** | Solar PV charges ice bank during daylight; zero-grid autonomous compute hours |
| **Micro data centre** | Stack units per rack; shared manifold; independent service per unit |

---

## Methodology

1. Cooling load analysis across AI accelerator TDP classes (75 W → 700 W)
2. Refrigerant selection at −5 °C / 45 °C with COP and safety comparison
3. R600a thermodynamic cycle: state points, COP, mass flow, capacity from 2 kW input
4. PCM selection: water/ice vs organic PCMs — energy density and coolant temperature
5. Ice bank sizing: latent + sensible storage calculation, charge/discharge time
6. VIP insulation vs PU foam vs aerogel: λ, R-value, heat leak, standby loss per hour
7. Liquid cooling loop: glycol selection, flow rate, pump power
8. Three heat exchanger calculations: ice–coolant BPHE, submerged evaporator coil, outdoor condenser

---

## Repository Structure

```
/analysis/  — thermodynamic cycle, ice sizing, VIP insulation, HX calculations
/concept/   — system architecture and form factor sketches
/renders/   — concept visualisations
README.md
```

---

## Tech Stack

| Tool | Use |
|---|---|
| Python (NumPy) | Cycle calculations, ice sizing, insulation comparison, HX sizing |
| Excel | Parametric: units × load × autonomy trade-off |
| SolidWorks | Tank geometry and split-unit form factor (next phase) |

---

## Status

Concept stage. Full thermal design calculated from first principles: refrigerant cycle, ice bank sizing, VIP insulation heat leak analysis, liquid cooling loop, and all three heat exchangers sized.

**Next phase:** SolidWorks CAD for ice tank geometry with integrated coil and VIP panel mounting, prototype ice-bank test cell to validate charging rate and glycol supply temperature stability during 6-hour discharge.
