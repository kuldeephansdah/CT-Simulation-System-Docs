# NITK-CTSim — CT Simulation Project

A from-scratch, educational simulation of the full computed-tomography
imaging chain: X-ray generation, tissue interaction, detection,
reconstruction, post-processing, and radiation dose estimation — built
across 7 independently-owned modules and integrated into one pipeline.

> **Note:** This is not a reproduction of any specific commercial CT
> scanner. Some system parameters are informed by real reference
> datasheets (Siemens SOMATOM Force, GE HealthCare Revolution CT); others
> are chosen independently where manufacturer data isn't public. See
> `SYSTEM_DATASHEET.md` for the full breakdown.

## What This Repo Is
This repository holds the **project-level documentation** for
NITK-CTSim — the shared reference material every team's module is built
against. It does not contain the simulation code itself; each stage of
the pipeline lives in its own team repository (linked below).

## Pipeline at a Glance

    CT Configuration  --(shared geometry/protocol params)--> every stage below
    X-ray Source -> Tissue Interaction -> Detector -> Reconstruction -> Post Processing
    Radiation Dose Measurement (parallel Monte Carlo track)

Full architecture, data contracts, and current integration status are in
`TRD.md`.

## Documentation in This Repo
| File | Contents |
|---|---|
| `SYSTEM_DATASHEET.md` | Reference system specs (Siemens/GE datasheets) vs. our actual simulated configuration |
| `TRD.md` | Technical architecture, module interfaces, data contracts, known integration gaps |
| `PRD.md` | Product goals, scope, success criteria, risks, milestones |

## Team Repositories
| Module | Repository |
|---|---|
| CT Configuration | https://github.com/VedantK2709/CT-Task05-Config |
| X-ray Source | https://github.com/dhruvaaa/CT-Task01-Xray-Source |
| Tissue Interaction | https://github.com/sridharbalsamy/CT-Task03-TissueInteraction |
| Detector | https://github.com/sakshamchhapane21-web/CT-Task4-Detector |
| Reconstruction | https://github.com/Tarunsundu111/CT_Task06_Reconstruction |
| Post Processing | https://github.com/VSGopiChanduNeelam/CT_TASK_06_POST_PROCESSING |
| Radiation Dose Measurement | https://github.com/UtsavSharma-28/CT-Task07-Radiation_Dose_Measurement |

## Current Status
- Pipeline currently runs against a **dummy cylindrical voxel phantom**
  (built by the Tissue Interaction team), pending the real voxel dataset
  from the professor.
- Each module works and validates in isolation; several cross-module
  interface gaps need to be
  resolved before a full end-to-end run.

## License
Academic project — no license specified yet.
