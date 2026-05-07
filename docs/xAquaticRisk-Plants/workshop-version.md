# xaquatic-funne (Workshop Version)

**xaquatic-funne** is a modified version of xAquaticRisk specifically developed for the SETAC Training Workshop. This version includes enhanced features and configurations tailored for workshop participants.


## Overview

xaquatic-funne is based on [xAquaticRisk](original.md) but includes:
- Modifications specific to workshop training scenarios
- Pre-configured settings for aquatic plant assessments

This version is distributed via a secure download link shared with registered workshop participants only.

## What You Will Learn

- How to configure **xaquatic-funne** for aquatic plant risk assessment
- Modelling **pesticide exposure** in surface waters relevant to macrophytes 
- Applying **effect models** for aquatic plant communities


## Key Model Components

| Component | Description |
|-----------|-------------|
| **PPP Use (Application)** | |
| PPM (xLandscape default) | Plant protection product application simulation on fields |
| **Exposure Pathways** | |
| *Spray Drift* | |
| XSprayDrift | Landscape-scale spray-drift deposition (XDrift R package) |
| DadDrift | Alternative drift deposition model |
| Samples | use of pre-defined values, e.g. FOCUS-Rautmann |
| *Run-off* | |
| RunOffPrzm | Run-off simulation with PRZM (Pesticide Root Zone Model) |
| *Drainage* | |
| FocusMacro | Drainage water and solute flux from agricultural fields |
| **Fate (Environmental Fate)** | |
| StepsRiverNetwork | Environmental fate modeling in river networks (STEPS) |
| **Effect (Effect Assessment)** | |
| CvasiLemLandscape | Effect model for aquatic plants (Lemna / duckweed) |
| **Observer (Analysis)** | |
| CatchmentObserver | Landscape-scale monitoring and output at target reaches

## License

"License Restrictions — Workshop Participants Only" - **xaquatic-funne** is provided exclusively for workshop participants under a restrictive license. By downloading, you agree to:
    
    - ✅ Use it for educational/research purposes within the workshop context
    - ✅ Run and modify it on your local machine  
    - ❌ **NOT share or distribute** the code to others
    - ❌ **NOT upload to Git or any version control system**
    - ❌ **NOT make it publicly available**
    
    **[Read the full license terms](LICENSE-xaquatic-funne.md)** before downloading.

## Download

!!! note "Access Required"
    The download link will be provided to registered workshop participants via email.
    
    **Do not share the download link** — it is for your personal use only.

## Resources

- [License Terms](LICENSE-xaquatic-funne.md)
- [Original xAquaticRisk](original.md) (open-source version)