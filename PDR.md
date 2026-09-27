# CT Simulation — Product Requirements Document (PRD)
**Project:** NITK-CTSim

## 1. Product Overview
NITK-CTSim is an educational, from-scratch simulation of the full
computed-tomography imaging chain — from X-ray generation through tissue
interaction, detection, reconstruction, post-processing, and radiation dose
estimation. It is not intended to reproduce any specific commercial
scanner; instead it models a physically plausible CT system, informed in
part by real reference datasheets (Siemens SOMATOM Force, GE HealthCare
Revolution CT), and built collaboratively by 7 independently-owned modules
that together form one pipeline.

## 2. Problem Statement
Understanding how a CT scanner turns X-ray photons into a diagnostic image
requires reasoning across several distinct physics and engineering
domains — source physics, tissue attenuation, detector electronics,
tomographic reconstruction math, image processing, and radiation
dosimetry — that are rarely built end-to-end by one team. This project
exists to build and integrate a working version of that full chain, so
each contributing team gains hands-on depth in one stage while the
combined pipeline demonstrates the complete process.

## 3. Goals & Objectives
- Produce a working, end-to-end simulated CT pipeline: X-ray Source to
  Tissue Interaction to Detector to Reconstruction to Post-Processing,
  with Radiation Dose Measurement as a parallel physics track.
- Ground key system parameters in real-world reference systems (partially
  informed by Siemens SOMATOM Force and GE HealthCare Revolution CT
  datasheets) rather than arbitrary values, while being explicit that this
  is not a reproduction of any single commercial scanner.
- Demonstrate physically correct, checkable behavior at every stage (e.g.
  beam hardening in the spectrum, monotonic attenuation with path length,
  energy/dose conservation) rather than optimizing for image realism alone.
- Validate the pipeline first on a dummy cylindrical phantom, then on the
  real voxel dataset once the professor provides it.
- Deliver the four requested artifacts: System Datasheet, TRD, this PRD,
  and a validation plan.

## 4. Stakeholders
| Stakeholder | Interest |
|---|---|
| Professor (evaluator) | Reviews all 4 deliverables; sets the real voxel model to be used eventually |
| Project Lead | Owns integration, documentation, and cross-team coordination |
| X-ray Source team | Owns photon generation and spectrum modeling |
| Tissue Interaction team | Owns phantom, ray tracing, and attenuation physics |
| Detector team | Owns detector physics and the data handoff to Reconstruction |
| Reconstruction team | Owns image reconstruction from projection data |
| Post Processing team | Owns denoising and image-quality improvement |
| Radiation Dose Measurement team | Owns Monte Carlo dose estimation |
| CT Configuration team | Owns the shared system geometry/protocol parameters used by every other team |

## 5. Scope

### In Scope
- A Python-based simulation pipeline (Radiation Dose Measurement uses
  Geant4/C++ as a separate, physics-appropriate track) covering source,
  tissue interaction, detection, reconstruction, and post-processing.
- Use of a dummy cylindrical voxel phantom as an interim test object.
- System parameters partially grounded in real reference scanner
  datasheets, documented transparently as such.
- Per-module validation (each team already defines its own checks — see
  TRD Section 4) plus a project-level validation plan (separate
  deliverable).
- The 4 documentation deliverables requested by the professor.

### Out of Scope
- Reproducing any specific commercial scanner's exact proprietary
  specifications.
- Clinical-grade dosimetric or diagnostic accuracy, or any real patient
  data.
- Real-time or production-grade performance; this is a from-scratch
  educational implementation (as the Reconstruction team's own README
  notes, performance on real clinical-sized datasets is not a current
  goal).
- A user-facing application or GUI — outputs are files (.npy, .npz, .csv,
  .json) consumed by the next stage or inspected directly.

## 6. Key Deliverables / Features by Module
| Module | Feature delivered |
|---|---|
| CT Configuration | Shared geometry/protocol parameter set (SID, SDD, kVp, pitch, rotation, TCM, etc.) consumed by every other module |
| X-ray Source | Polychromatic photon generation with realistic spectrum shape and beam-hardening behavior |
| Tissue Interaction | Ray tracing through a voxelized phantom with energy-dependent Beer-Lambert attenuation |
| Detector | Realistic energy-integrating detector response: efficiency, blur, gain, and noise on top of the raw signal |
| Reconstruction | 3D volume reconstruction from projection data via ART (Algebraic Reconstruction Technique) |
| Post Processing | Image denoising (BM3D now, 3D U-Net planned) with measurable quality improvement |
| Radiation Dose Measurement | Monte Carlo-based absorbed dose estimate with 3D voxel-level dose distribution |

## 7. Success Metrics / Acceptance Criteria
Drawn from each team's own current validation approach (see TRD Section 4
for full detail):
- **X-ray Source:** sampled energy spectrum matches the analytical Kramers
  curve shape; mean energy increases after filtration (beam hardening
  confirmed).
- **Tissue Interaction:** transmitted signal decreases monotonically with
  increasing tissue path length; no negative path lengths; detector image
  shape matches configured rows x channels.
- **Detector:** DQE and MTF behave as physically expected functions of
  spatial frequency; noise decomposes correctly into quantum + electronic
  components.
- **Reconstruction:** reconstructed volume has no NaN/infinite values and
  visually matches a known synthetic test phantom; projection error
  decreases across ART iterations.
- **Post Processing:** denoised output shows measurable PSNR/SSIM
  improvement over the noisy input, without erasing anatomical structure.
- **Radiation Dose Measurement:** photon-fate counts (absorbed + scattered
  + transmitted + unclassified) approximately equal the number of primary
  photons; voxel-summed energy matches the independently reported total.
- **Pipeline-level (new):** data can flow from one module's real output
  into the next module's real input without manual reformatting — this is
  currently blocked by the gaps listed in TRD Section 6 and is the key
  project-level acceptance bar before end-to-end integration is
  considered done.

## 8. Assumptions & Dependencies
- The professor has not yet supplied the final voxel model; all
  development and testing currently uses a dummy cylindrical phantom
  built by the Tissue Interaction team. Switching to the real dataset is a
  hard external dependency with no controlled timeline.
- System parameters are a deliberate mix of values informed by real
  scanners (Siemens SOMATOM Force, GE HealthCare Revolution CT) and values
  chosen independently by the team where manufacturer data isn't public —
  this is documented in full in SYSTEM_DATASHEET.md and is not to be
  represented as an exact commercial reproduction.
- The Radiation Dose Measurement module depends on Geant4/C++ tooling
  separate from the rest of the Python pipeline; any teammate working on
  that module needs that toolchain installed.

## 9. Risks & Mitigations
| Risk | Impact | Mitigation |
|---|---|---|
| Interface mismatches between modules | Pipeline cannot run end-to-end even if every module works in isolation | Resolve each gap with the relevant team pair before attempting a full integration run; track in TRD Change Log |
| Real voxel model arrives late or with an unfamiliar format | Final validation phase gets compressed against the deadline | Keep the dummy-cylinder pipeline fully working now so only the phantom-loading step needs to change later |
| Mixed tech stack (Python pipeline + C++/Geant4 dose module) | Harder to integrate and to onboard new contributors | Keep the Geant4 module's inputs/outputs file-based (CSV/JSON) rather than attempting in-process integration | Confusion in tracking, grading, and cross-references | Renumber task IDs project-wide |

## 10. Milestones & Timeline

| Phase | Description | Target Date |
|---|---|---|
| Phase 1 | Deliverables complete: System Datasheet, TRD, PRD, Validation Plan | |
| Phase 2 | Dummy-cylinder pipeline runs end-to-end, module to module | |
| Phase 3 | Real voxel model integrated once provided by professor | |
| Phase 4 | Final validation run, results write-up, and presentation | |

## 11. Change Log
| Date | Change | Author |
|---|---|---|
| 27-09-2026 | Initial PRD created, cross-referenced against SYSTEM_DATASHEET.md and TRD.md | Kuldeep Hansdah, Subham Kumar Beura |
