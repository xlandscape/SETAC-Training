# xLandscape

**Slide deck:** [SETAC xLandscape Training — (2) xLandscape Intro](../static/SETAC_xLandscapeTraining%20-%20%282%29%20xLandscape%20Intro%20v0.1.pptx)

---

## Why a Modular Landscape Modelling Framework?

The demand for landscape-scale modelling in pesticide risk assessment is growing, driven by:

- **More realistic, higher-tier risk assessment** — EFSA increasingly calls for spatially explicit reference scenarios in tiered schemes.
- **Improved process integration** — linking hydrology, fate, exposure, individual effects, and population dynamics in a single modelling chain.
- **Holistic view to pesticide risk** — covering multiple stressors, recovery, biodiversity, and indirect effects of weed control.
- **Integration of monitoring and modelling** — to continuously improve knowledge and validate predictions.
- **Digital Agriculture and Digital Twins** — enabling *what-if* analyses for integrated pest control and environmental risk management (e.g., Destination Earth).

---

## Conceptual Drivers for xLandscape

The xLandscape framework was designed around three key concepts:

| Concept | Description |
|---------|-------------|
| **Tiered approach** | Flexibility and adaptivity — *"get only as complex as necessary"* |
| **Specific Protection Goals (SPGs)** | Models operate in explicit spatial, temporal, and structural dimensions aligned to SPGs |
| **Systems Approaches & Digital Twins** | Foundation for future real-world digital twins and DestinE-compatible scenarios |

---

## The Vision

> *Imagine you have a question at hand — research, risk assessment, risk management, landscape design, or population protection — and you have a system where you can link geodata APIs, add process modules, run simulations, and analyse results (in natural language), all at scale, starting from your laptop.*

xLandscape aims to make this possible: an open, AI-supportable platform where researchers can **build on what others have already done** rather than starting from scratch.

Wherever your expertise lies — toxicology, hydrology, ecology, agronomy — the modular architecture lets you **focus on your core competence** while benefiting from a consistent, validated framework.

---

## Modularity as a Principle

Modularity is central to xLandscape because it simultaneously enables:

- **Focus** — developers and users work within their domain expertise.
- **Stepwise validation** — each component can be tested independently.
- **Collaboration** — teams can contribute without needing to understand the entire system.
- **Flexibility** — model complexity is adapted to the problem, not forced by the platform.
- **Harmonisation** — consistent interfaces and semantics across all models.
- **Transparency** — similar solutions to similar problems; modules can become authorised APIs.

The framework integrates diverse domains: hydrology, land use, habitats, pesticide use, exposure, environmental fate, individual effects, and population effects.

---

## Model Composition: Components

The xLandscape microkernel provides the foundational services:

**Interfaces · Control · Monte Carlo · Dimensions · Scales · Semantics · Data Store**

Individual **components** wrap existing models (TOXSWA, PRZM5, GUTS, StreamCom, …) and are integrated at four levels via standardised interfaces. Models written in Python, R, C++, Fortran, Java, or Smalltalk can be incorporated.

### xAquaticRisk as an example composition

| Layer | Components |
|-------|-----------|
| Agriculture / Land management | Crop cultivation, PPP use, land use/cover |
| Exposure / Fate | xDrift, xRunoff, xDrainage, Toxswa, Steps1234, CmfContinuous |
| Effects | GUTS (CVASI), StreamCom, MASTEP |
| Analysis | LP50, population exposure, reporting |
| Environment | Weather, soil, elevation, hydrographic network |

### The xLandscape model ecosystem

| Model | Focus |
|-------|-------|
| xAquaticRisk (invertebrates) | Aquatic invertebrate population risk |
| xAquaticRisk (water plants) | Aquatic macrophytes and algae |
| xPollinator | Honeybee and wild bee population risk |
| xNTTP-EU / xNTP-US | Non-target terrestrial plants |
| xOffFieldSoil | Off-field soil exposure |
| xWeedIndirectEffects | Indirect effects of weed control |
| xResidues | Crop residue exposure |

All models are available at [https://github.com/xlandscape](https://github.com/xlandscape).

---

## Propagating Real-World Variability

xLandscape handles input variability (agriculture, environment, biology) and propagates it to output variability (PECs, effects, population sizes) using a **hybrid explicit–Monte Carlo** approach:

- **Discretisation** in space and time captures landscape heterogeneity.
- **Monte Carlo simulation** samples from probability density functions for uncertain parameters.
- **Multidimensional HDF5 data arrays** store and exchange all intermediate and final outputs (hydrology, exposure, effects) in a semantically consistent model space, accessible from R, Python, Jupyter, Matlab, KNIME, and more.

---

## User Levels and Roles

xLandscape supports different levels of engagement — from model users applying pre-built scenarios to developers building new components and contributing to the framework. The training focuses on building familiarity across these roles.

---

## Take-Home Messages

- **Landscape modelling = improved realism and integration** — it places risk assessment in the real world.
- **Modularisation manages complexity** and enables effective collaboration across disciplines.
- **xLandscape = an open-source modular landscape modelling approach** — components, models, and scenarios can be built for a wide range of problems.
- The goal is not to establish **fixed, authoritative** landscape models, but to enable **flexible, adaptive** research and higher-tier models of the highest possible harmonisation, continuity, and transparency.

---

## Our Values

**Openness & Collaboration** — open-source principles foster innovation, participation, and real-world relevance.

**FAIR Principles** — Findable, Accessible, Interoperable, and Reusable. Modular frameworks and transparent sharing let researchers build on existing work.

**Invitation to Participate** — xLandscape is a *mindset*. Everyone is welcome to contribute ideas, share expertise, and help shape the future of landscape modelling.

---

## What's Next

- Growing AI support for model composition and analysis
- Community building — connecting research and development teams
- xLandscape Version 4: microkernel and component-based clean architecture, improved component autonomy, Python packaging, digital-twin readiness
- Stepwise introduction of formal semantics up to established ontologies
- Continued development, testing, documentation, and publication of components
