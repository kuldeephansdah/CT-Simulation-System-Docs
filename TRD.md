# CT Simulation — Technical Requirements Document (TRD)
**Project:** NITK-CTSim

## 1. Purpose & Scope
This document specifies the technical architecture, module interfaces, data
contracts, and current implementation status of the CT simulation pipeline.
Where the System Datasheet defines *what physical system* we're simulating,
this TRD defines *how the software is built* and *how the 7 modules talk to
each other*. Content below reflects the actual current state of each team's
repository as of this document's date — not an idealized target.

---

## 2. System Architecture / Pipeline Overview
CT Configuration (shared geometry/protocol parameters: SID, SDD, kVp,
pitch, rotation, TCM — see System Datasheet §5)
│ (feeds parameters to every stage below)
▼
X-ray Source (Task 01)
Generates photon_positions, photon_directions, photon_energies,
photon_weight per view.
│
▼
Tissue Interaction (Task 03)
Loads shared config + dummy cylinder phantom → deterministic ray
tracing → Beer-Lambert spectral attenuation → ALREADY energy-integrates
to a scalar detector signal per ray.
Output: view_<id>_detector.npy ⚠️ see Gap #1
│
▼
Detector (Task 04)
Expects raw photon fluence per energy bin (not pre-integrated) → adds
detector physics (efficiency, PSF blur, conversion gain, noise, dead
pixels) → outputs a clean handoff package.
Output: detector_handoff_package.npz + _metadata.json ⚠️ see Gap #1
│
▼
Reconstruction (Task 06)
Algebraic Reconstruction Technique (ART), iterative, cone-beam forward
model.
Output: ART_reconstructed_volume.npy, shape (Nz, Ny, Nx)
│
▼
Post Processing (also labeled "Task 06" — see Gap #2)
BM3D denoising (currently prototyped on synthetic 256×256 2D slices) →
3D U-Net planned.
Output: denoised volume + PSNR/SSIM/RMSE/MAE metrics

Radiation Dose Measurement — independent track
Geant4/C++ Monte Carlo, NOT yet fed by the shared CT Configuration or
the dummy cylinder phantom. ⚠️ see Gap #3
Own standalone 30×30×30 cm water phantom, 10×10×10 voxel grid.
Output: deposited energy, absorbed dose, photon-fate classification,
3D voxel dose map.

---

## 3. Technology Stack (as currently implemented)

| Module | Language | Key Libraries |
|---|---|---|
| CT Configuration | Python | (JSON export) |
| X-ray Source | Python | spectrum/photon sampling |
| Tissue Interaction | Python | NumPy, `xraylib`, Matplotlib, json, csv |
| Detector | Python | NumPy, SciPy, Matplotlib |
| Reconstruction | Python (Jupyter notebook) | NumPy |
| Post Processing | Python (Jupyter notebook) | OpenCV, BM3D, scikit-image (PSNR/SSIM) |
| Radiation Dose Measurement | **C++ (Geant4, CMake)** | Geant4 toolkit |

> ⚠️ **Note:** 6 of 7 modules are Python/NumPy. Radiation Dose Measurement is
> a separately compiled C++/Geant4 codebase — it cannot be imported directly
> into the Python pipeline. If end-to-end integration is required, define an
> explicit file-based hand-off (e.g. Geant4 exports dose results as CSV/JSON
> that Python reads).

---

## 4. Module-by-Module Technical Requirements

### 4.1 X-ray Source (Task 01)
| | |
|---|---|
| **Inputs** | `source_position`, `detector_position`, `cone_angle` (from geometry); `kVp`, `mA`, `exposure_time`, `focal_spot_size`, `N_photons` (protocol settings) |
| **Outputs** | `photon_positions (N,3)`, `photon_directions (N,3)`, `photon_energies (N,)`, `photon_weight` = mA×exposure_time/N |
| **Status** | Spectrum calc (filtered Kramers Bremsstrahlung), energy sampling, cone-direction sampling implemented |
| **Validation approach (team's own)** | Energy histogram vs analytical Kramers curve; mean-energy shift to confirm beam hardening; spatial/cone-angle bounds check |

### 4.2 Tissue Interaction (Task 03)
| | |
|---|---|
| **Inputs** | `ct_config_by_team2.json` (sections `1_source_team`, `3_detector_team`, `4_system_config_team`); hardcoded phantom: cylinder radius 25mm, height 50mm, voxel size 1mm, density 1.0, material ID 1 (Soft Tissue) |
| **Outputs** | `view_<id>_detector.npy` (2D detector signal, energy-integrated per ray), `view_<id>_rays.csv` (views 1–3 only) |
| **Ray count** | 64 × 888 = 56,832 rays/view (matches Detector team's config) |
| **Materials DB** | 7 materials mapped to NIST names via `xraylib` (Air, Soft Tissue, Cortical Bone, Lung, Adipose, Water, Skeletal Muscle) |
| **Status** | Full deterministic pipeline implemented; no Monte Carlo, no scatter modeled; air path ignored |
| **Validation approach (team's own)** | Geometry, detector-element-count, ray-shape, non-negative path-length, and transmission-monotonicity checks |

### 4.3 Detector (Task 04)
| | |
|---|---|
| **Inputs** | Photon fluence per energy bin from Tissue Interaction (expected — see Gap #1) |
| **Config** | 64 rows × 888 channels, 0.625mm pixel pitch, 555×40mm active area, energy-integrating type, GOS-mapped scintillator, 40mm collimation |
| **Physics modeled** | Quantum efficiency (Beer-Lambert × fill factor), MTF/DQE, conversion gain (50 e⁻/keV), electronic noise (500 e⁻), Gaussian PSF blur, gain nonuniformity, dead pixels |
| **Outputs** | `detector_handoff_package.npz` (`measured_signal_electrons`, `reference_signal_electrons`, `gain_map`, `dead_pixel_map`, `angles_deg`) + `_metadata.json` with normalization convention: `line_integral = -ln(measured/reference)` |
| **Status** | Full physics chain implemented; also includes a standalone FBP demo (`ct_reconstruction.py`) for self-testing only — **not** the official Reconstruction deliverable (see Gap #4) |
| **Known limitation (team's own note)** | Material attenuation tables are illustrative, not dosimetrically accurate; reconstruction demo assumes monochromatic-equivalent line integral (no beam-hardening correction) |

### 4.4 Reconstruction (Task 06)
| | |
|---|---|
| **Inputs** | Projection/sinogram `(total_views, detector_rows, detector_channels)` — documented example `(180, 16, 1024)` ⚠️ see Gap #3; gantry angles `(total_views,)`; CT geometry (SOD, SDD, pixel size, voxel size); ART iteration count + relaxation parameter |
| **Algorithm** | Algebraic Reconstruction Technique (ART) — iterative, cone-beam forward projection + trilinear interpolation |
| **Outputs** | 3D volume `(Nz, Ny, Nx)`, saved as `ART_reconstructed_volume.npy`; single-slice visualization |
| **Status** | Core math/geometry implemented and tested on synthetic phantom only; not yet run on real pipeline data |
| **Validation approach (team's own)** | Shape/NaN checks, synthetic-phantom round-trip comparison, projection-error monitoring across iterations |

### 4.5 Post Processing (labeled "Task 06" in repo — see Gap #2)
| | |
|---|---|
| **Inputs (current prototype)** | Synthetic 256×256 grayscale slice, normalized [0,1], with controlled Gaussian noise added |
| **Inputs (intended final)** | Reconstructed 3D chest CT volume from the Reconstruction module |
| **Algorithm** | BM3D denoising (baseline); 3D U-Net planned for later |
| **Outputs** | Denoised slice/volume + PSNR, SSIM, RMSE, MAE |
| **Status** | 2D BM3D prototype working on synthetic noise only; **not yet connected to real Reconstruction output** (see Gap #6) |

### 4.6 Radiation Dose Measurement
| | |
|---|---|
| **Inputs** | Simulated X-ray photons (own source/spectrum), photon count, standalone 30×30×30cm water phantom, 10×10×10 voxel grid (1,000 voxels) |
| **Physics** | Geant4 Monte Carlo photon transport; photon-fate classification (absorbed/scattered/transmitted/unclassified) |
| **Outputs** | Total energy deposited (MeV), absorbed dose (D = E_dep/m, in Gy), 3D voxel energy/dose distribution |
| **Sample result** | 10,000 photons → 555.643 MeV deposited, 27kg phantom, dose 3.297×10⁻¹² Gy |
| **Status** | Core Monte Carlo + voxel scoring implemented; **not yet using the shared CT geometry, dummy cylinder phantom, or a rotating source** (see Gap #3) |
| **Validation approach (team's own)** | Photon conservation (N_absorbed+scattered+transmitted+unclassified ≈ N_primary); energy-sum consistency; dose-formula check; voxel dimension/mass checks |

---

## 5. Interface Contracts / Shared Data Schema

The Tissue Interaction and Detector teams have already converged on a de
facto config convention worth formalizing project-wide:

```json
{
  "1_source_team": { "kvp": 120.0, "...": "..." },
  "3_detector_team": { "detector_rows": 64, "detector_channels": 888, "...": "..." },
  "4_system_config_team": { "sid_mm": 541.0, "sdd_mm": 949.0, "...": "..." }
}
```

**Recommendation:** adopt this (or a renamed, cleaned-up version) as the
single shared `system_config.json`, sourced directly from the CT
Configuration team's repo, so every module reads the same file instead of
each team maintaining its own copy. The Detector team's `.npz` +
`_metadata.json` handoff pattern is the cleanest existing example of a data
contract — recommend other module boundaries (Tissue Interaction→Detector,
Reconstruction→Post-Processing) adopt the same pattern.

---

## 6. ⚠️ Known Integration Gaps & Action Items

These are real mismatches found while reviewing the repos — resolve before
claiming an end-to-end pipeline works:

1. **Tissue Interaction ↔ Detector data mismatch.** Tissue Interaction
   already energy-integrates to a single scalar signal per ray (Beer-Lambert
   summed across the spectrum). The Detector module's `detect()` function
   expects **raw photon fluence per energy bin**, not a pre-integrated
   scalar. One of the two teams needs to change their output/input format —
   decide which side owns the energy-integration step.
2. **Duplicate "Task 06" naming.** Both the Reconstruction repo and the
   Post-Processing repo are labeled Task 06. Renumber so every task ID is
   unique — this will also matter for grading/tracking.
3. **Reconstruction's documented example shape doesn't match reality.**
   Reconstruction's README example is `(180, 16, 1024)` — 16 rows × 1024
   channels — but the actual Detector config is 64 rows × 888 channels.
   Confirm the ART code isn't hard-coded to the example shape.
4. **Two competing reconstruction implementations.** The Detector repo
   contains its own FBP reconstruction (`ct_reconstruction.py`) used only
   to self-test detector physics. Don't confuse it with the official
   ART-based Reconstruction deliverable in the Task 06 repo.
5. **Radiation Dose is fully isolated.** It runs on its own standalone
   water phantom and voxel grid — not the shared CT Configuration geometry,
   not the dummy cylinder phantom, and not a rotating source. Being a
   separate C++/Geant4 codebase, it also can't be called directly from the
   Python pipeline. Needs an explicit integration/hand-off plan.
6. **Post Processing is disconnected from real data.** Currently prototyped
   only on synthetic 256×256 slices with artificial noise — not yet fed by
   actual Reconstruction output. Confirm matrix-size compatibility once
   Reconstruction produces real volumes.
7. **X-ray Source's config source is unconfirmed.** Unlike Tissue
   Interaction and Detector, the X-ray Source README doesn't name a shared
   config file. Confirm whether it also reads `ct_config_by_team2.json`
   (section `1_source_team`) or maintains separate settings.

---

## 7. Non-Functional Requirements & Validation
Every module already documents its own validation checks (see §4 above and
each team's repo). Deliverable #4 (separate document) will define the
overall, pipeline-level validation strategy. At the TRD level, the
recommended next step is consolidating each module's individual checks into
one integration test suite so a full pipeline run can be regression-tested
end-to-end once the Gap #1–#7 items are resolved. No formal runtime/
performance requirements have been set yet (several modules are still
Jupyter-notebook prototypes) — worth deciding whether that's in scope.

---

## 8. Change Log
| Date | Change | Author |
|---|---|---|
| 27-09-2026 | Add Technical Requirements Document for CT Simulation | Kuldeep Hansdah, Subham Kumar Beura |
