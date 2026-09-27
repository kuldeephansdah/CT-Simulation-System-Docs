# CT Simulation : Technical Requirements Document (TRD)
**Project:** NITK-CTSim

## 1. Purpose & Scope
This document specifies the technical architecture, module interfaces, data
contracts, and current implementation status of the CT simulation pipeline.
Where the System Datasheet defines *what physical system* we're simulating,
this TRD defines *how the software is built* and *how the 7 modules talk to
each other*. Content below reflects the actual current state of each team's
repository as of the date above not an idealized target.

---

## 2. System Architecture / Pipeline Overview

```
CT Configuration (shared geometry/protocol parameters: SID, SDD, kVp,
pitch, rotation, TCM — see System Datasheet Section 5)
        |  (feeds parameters to every stage below)
        v
X-ray Source 
  Generates photon_positions, photon_directions, photon_energies,
  photon_weight per view.
        |
        v
Tissue Interaction 
  Loads shared config + dummy cylinder phantom -> deterministic ray
  tracing -> Beer-Lambert spectral attenuation -> ALREADY energy-integrates
  to a scalar detector signal per ray.
  Output: view_<id>_detector.npy   
        |
        v
Detector 
  Expects raw photon fluence per energy bin (not pre-integrated) -> adds
  detector physics (efficiency, PSF blur, conversion gain, noise, dead
  pixels) -> outputs a clean handoff package.
  Output: detector_handoff_package.npz + _metadata.json   
        |
        v
Reconstruction
  Algebraic Reconstruction Technique (ART), iterative, cone-beam forward
  model.
  Output: ART_reconstructed_volume.npy, shape (Nz, Ny, Nx)
        |
        v
Post Processing 
  BM3D denoising (currently prototyped on synthetic 256x256 2D slices) ->
  3D U-Net planned.
  Output: denoised volume + PSNR/SSIM/RMSE/MAE metrics

Radiation Dose Measurement — independent track
  Geant4/C++ Monte Carlo, NOT yet fed by the shared CT Configuration or
  the dummy cylinder phantom.  
  Own standalone 30x30x30 cm water phantom, 10x10x10 voxel grid.
  Output: deposited energy, absorbed dose, photon-fate classification,
  3D voxel dose map.
```

---

## 3. Technology Stack (as currently implemented)

| Module | Language | Key Libraries |
|---|---|---|
| CT Configuration | Python | JSON export |
| X-ray Source | Python | spectrum/photon sampling |
| Tissue Interaction | Python | NumPy, xraylib, Matplotlib, json, csv |
| Detector | Python | NumPy, SciPy, Matplotlib |
| Reconstruction | Python (Jupyter notebook) | NumPy |
| Post Processing | Python (Jupyter notebook) | OpenCV, BM3D, scikit-image (PSNR/SSIM) |
| Radiation Dose Measurement | C++ (Geant4, CMake) | Geant4 toolkit |

> **Note:** 6 of 7 modules are Python/NumPy. Radiation Dose Measurement is a
> separately compiled C++/Geant4 codebase it cannot be imported directly
> into the Python pipeline. If end-to-end integration is required, define an
> explicit file-based hand-off (e.g. Geant4 exports dose results as CSV/JSON
> that Python reads).

---

## 4. Module-by-Module Technical Requirements

### 4.1 X-ray Source 
| | |
|---|---|
| Inputs | source_position, detector_position, cone_angle (from geometry); kVp, mA, exposure_time, focal_spot_size, N_photons (protocol settings) |
| Outputs | photon_positions (N,3), photon_directions (N,3), photon_energies (N,), photon_weight = mA x exposure_time / N |
| Status | Spectrum calc (filtered Kramers Bremsstrahlung), energy sampling, cone-direction sampling implemented |
| Validation approach (team's own) | Energy histogram vs analytical Kramers curve; mean-energy shift to confirm beam hardening; spatial/cone-angle bounds check |

### 4.2 Tissue Interaction 
| | |
|---|---|
| Inputs | ct_config_by_team2.json (sections 1_source_team, 3_detector_team, 4_system_config_team); hardcoded phantom: cylinder radius 25mm, height 50mm, voxel size 1mm, density 1.0, material ID 1 (Soft Tissue) |
| Outputs | view_<id>_detector.npy (2D detector signal, energy-integrated per ray), view_<id>_rays.csv (views 1-3 only) |
| Ray count | 64 x 888 = 56,832 rays/view (matches Detector team's config) |
| Materials DB | 7 materials mapped to NIST names via xraylib (Air, Soft Tissue, Cortical Bone, Lung, Adipose, Water, Skeletal Muscle) |
| Status | Full deterministic pipeline implemented; no Monte Carlo, no scatter modeled; air path ignored |
| Validation approach (team's own) | Geometry, detector-element-count, ray-shape, non-negative path-length, and transmission-monotonicity checks |

### 4.3 Detector
| | |
|---|---|
| Inputs | Photon fluence per energy bin from Tissue Interaction |
| Config | 64 rows x 888 channels, 0.625mm pixel pitch, 555x40mm active area, energy-integrating type, GOS-mapped scintillator, 40mm collimation |
| Physics modeled | Quantum efficiency (Beer-Lambert x fill factor), MTF/DQE, conversion gain (50 e-/keV), electronic noise (500 e-), Gaussian PSF blur, gain nonuniformity, dead pixels |
| Outputs | detector_handoff_package.npz (measured_signal_electrons, reference_signal_electrons, gain_map, dead_pixel_map, angles_deg) + _metadata.json with normalization convention: line_integral = -ln(measured/reference) |
| Status | Full physics chain implemented; also includes a standalone FBP demo (ct_reconstruction.py) for self-testing only not the official Reconstruction deliverable |
| Known limitation (team's own note) | Material attenuation tables are illustrative, not dosimetrically accurate; reconstruction demo assumes monochromatic-equivalent line integral (no beam-hardening correction) |

### 4.4 Reconstruction
| | |
|---|---|
| Inputs | Projection/sinogram (total_views, detector_rows, detector_channels) documented example (180, 16, 1024); gantry angles (total_views,); CT geometry (SOD, SDD, pixel size, voxel size); ART iteration count + relaxation parameter |
| Algorithm | Algebraic Reconstruction Technique (ART) iterative, cone-beam forward projection + trilinear interpolation |
| Outputs | 3D volume (Nz, Ny, Nx), saved as ART_reconstructed_volume.npy; single-slice visualization |
| Status | Core math/geometry implemented and tested on synthetic phantom only; not yet run on real pipeline data |
| Validation approach (team's own) | Shape/NaN checks, synthetic-phantom round-trip comparison, projection-error monitoring across iterations |

### 4.5 Post Processing
| | |
|---|---|
| Inputs (current prototype) | Synthetic 256x256 grayscale slice, normalized [0,1], with controlled Gaussian noise added |
| Inputs (intended final) | Reconstructed 3D chest CT volume from the Reconstruction module |
| Algorithm | BM3D denoising (baseline); 3D U-Net planned for later |
| Outputs | Denoised slice/volume + PSNR, SSIM, RMSE, MAE |
| Status | 2D BM3D prototype working on synthetic noise only; not yet connected to real Reconstruction output |

### 4.6 Radiation Dose Measurement
| | |
|---|---|
| Inputs | Simulated X-ray photons (own source/spectrum), photon count, standalone 30x30x30cm water phantom, 10x10x10 voxel grid (1,000 voxels) |
| Physics | Geant4 Monte Carlo photon transport; photon-fate classification (absorbed/scattered/transmitted/unclassified) |
| Outputs | Total energy deposited (MeV), absorbed dose (D = E_dep/m, in Gy), 3D voxel energy/dose distribution |
| Sample result | 10,000 photons -> 555.643 MeV deposited, 27kg phantom, dose 3.297e-12 Gy |
| Status | Core Monte Carlo + voxel scoring implemented; not yet using the shared CT geometry, dummy cylinder phantom, or a rotating source |
| Validation approach (team's own) | Photon conservation (N_absorbed+scattered+transmitted+unclassified ~= N_primary); energy-sum consistency; dose-formula check; voxel dimension/mass checks |

---

## 6. Change Log
| Date | Change | Author |
|---|---|---|
| 27-09-2026 | Initial TRD created from all 7 team repos' current documentation | Kuldeep Hansdah, Subham Kumar Beura |
