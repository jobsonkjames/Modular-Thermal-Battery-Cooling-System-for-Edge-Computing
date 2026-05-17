# Modular-Thermal-Battery-Cooling-System-for-Edge-Computing

![Status](https://img.shields.io/badge/Status-Early_Concept-yellow)
![Domain](https://img.shields.io/badge/Domain-AI_Chip_Cooling-blue)
![Method](https://img.shields.io/badge/Method-Thermal_Battery-orange)
![Form_Factor](https://img.shields.io/badge/Form_Factor-Modular_Split_Unit-green)
 
An early-stage concept for a **modular liquid cooling system** designed for AI chip and edge compute thermal management. Decouples cooling capacity from grid availability via a **thermal battery** that buffers compressor operation, enabling autonomous cooling under variable grid conditions and renewable-aligned charging.
 
## Overview
 
Edge compute infrastructure — AI inference nodes, telecom base stations, industrial compute units, micro data centres — generates concentrated heat that conventional cooling handles poorly. Existing options have three main gaps:
 
- **Installation complexity** — precision AC units need specialist setup
- **Grid dependency** — cooling fails if grid power fluctuates
- **Fixed capacity** — over-invest upfront or run hot as compute scales
 
The concept addresses all three with one architecture: charge a thermal reservoir during off-peak or high-renewable periods; deliver cooling from storage during peak compute load without the compressor running.
 
## Highlights
 
- **2 kW compressor** sized for typical 2–4 kW edge compute node
- Thermal battery designed for **~6 hours of autonomous cooling buffer**
- Split-unit form factor — installs like a window AC, no specialist setup
- **Modular parallel-unit operation** for server room scale
- Renewable-aligned charging: store cooling during high-PV / off-peak periods
- Preliminary thermal calculations and sizing complete
 
## Concept Visualisation
 
![Concept render](docs/concept.png)
*Modular window-unit form factor with scalable parallel deployment*
 
## Methodology
 
1. Cooling load analysis for typical edge compute deployment scenarios
2. Thermal battery sizing for target autonomous operation window (~6 hr)
3. Refrigerant circuit and thermal store integration concept
4. Modular split-unit form factor definition with parallel-unit interface
5. Use-case mapping across edge deployment categories
 
## Target Applications
 
- **AI inference nodes** at manufacturing plants, logistics hubs, remote facilities
- **Telecom base stations** where grid reliability is variable
- **Industrial edge compute** in environments where HVAC integration is impractical
- **Micro data centres** in markets with intermittent grid access
- Any deployment benefiting from renewable-aligned thermal charging
 
## Repository Structure
 
- `/analysis/` — cooling load and thermal battery sizing calculations
- `/concept/` — system architecture diagrams and form factor sketches
- `/renders/` — AI-generated concept visualisations
- `README.md`
 
## Tech Stack
 
- **Analysis:** First-principles thermal calculations
- **Visualisation:** AI-generated concept renders
 
## Status
 
Early concept stage. Preliminary thermal calculations established storage volume and operating parameters. Detailed design, CAD modelling, and prototype work form the next phase of the project.
 
 
