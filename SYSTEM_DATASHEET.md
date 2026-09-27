# CT Simulation : System Datasheet

## 1. Purpose & Scope
This document is the single reference for the physical, geometric, and
acquisition parameters used across the CT Simulation Project (X-ray Source,
Tissue Interaction, Detector, Reconstruction, CT Configuration, Post
Processing, Radiation Dose Measurement). All modules must build against
these values unless a change is logged in the Change Log below.

## 2. Reference System Basis & Disclaimer
> This system is **not** a reproduction of any specific commercial CT
> scanner. Select parameters were informed by publicly available
> specifications from real scanners specifically a **Siemens SOMATOM
> Force** (dual-source) reference datasheet (Section 3) and, where
> available, GE Healthcare specifications (Section 4, pending). Other
> parameters, where manufacturer data is not public, were chosen
> independently by the project team for simulation purposes. Our simulated
> system is **single-source**, so any dual-source-specific reference value
> (e.g. detector row counts, temporal resolution, dual-energy modes) is
> used only as a design *reference range*, not copied directly this is
> called out explicitly wherever it applies.

---

## 3. Reference System 1 — Siemens SOMATOM Force (Dual-Source CT)
*(Source: Siemens SOMATOM Force Reference Datasheet, provided to the project team)*

### 3.1 System Geometry & Gantry Mechanics
| Parameter | Value | Notes |
|---|---|---|
| Gantry bore size | 78 cm | - |
| Minimal rotation time | 0.25 s | - |
| Max scan speed | 737 mm/s | Flash Spiral mode only |
| Source-to-Isocenter (R) | 53.5 cm | - |
| Source-to-Detector (D) | 97.6 cm | - |
| Temporal resolution | 66 ms | Dual-source specific N/A for our single-source sim |

### 3.2 X-Ray Generation (Dual Vectron™)
| Parameter | Value |
|---|---|
| Configuration | 2× tubes at ~95° separation |
| Total power | 240 kW (2× 120 kW) |
| Tube voltage | 70–150 kV (10 kV steps) |
| Tube current | 20–1300 mA per tube |
| Focal spot | 0.4×0.5 mm (Small) / 0.8×1.1 mm (Large) |

### 3.3 Detector Array (Stellar Infinity™)
| Parameter | Value |
|---|---|
| Type | 2× Curved Ultra-Fast Ceramic (UFC) arrays |
| Physical rows | 192 rows per detector (2× 96 physical) |
| Acquired slices | 384 per rotation (2× 192) |
| z-coverage at isocenter | 57.6 mm |
| In-plane channels | 3,120 total |

### 3.4 Helical Trajectory & Reconstruction
| Parameter | Value |
|---|---|
| Pitch factor range | 0.15–3.2 (Flash Spiral up to 3.4) |
| Primary sFOV | 50 cm (extended up to 78 cm) |
| Secondary sFOV | 35–50 cm (dual energy) |
| Max scan length | 200 cm |
| Dual energy | Simultaneous 80/140 kV or 90/150 kV |
| In-plane resolution | 0.24 mm (22–32 lp/cm @ 0% MTF) |
| Min. reconstructed slice thickness | 0.4 mm |
| Matrix sizes | 512×512 / 768×768 / 1024×1024 |
| Reconstruction algorithms | 3D Helical FDK / ADMIRE Iterative |
| Dynamic range | -8,192 HU to +57,343 HU (span = 65,535 = 2¹⁶−1) |

---

## 4. Reference System 2 — GE Healthcare
*(Source: GE HealthCare Revolution CT / Apex Reference Datasheet, provided to the project team)*

> **Note:** Unlike the Siemens Force, the GE Revolution CT is **single-source** —
> architecturally closer to our simulated system. Where the two references
> disagree, prefer this one for resolving single-source-specific design
> questions (detector layout, tube configuration).

### 4.1 System Geometry & Gantry Dynamics
| Parameter | Value | Notes |
|---|---|---|
| Gantry bore size | 80 cm | - |
| Minimal rotation time | 0.28 s / 0.23 s | Apex platform reaches 0.23 s |
| Max table speed | 437.5 mm/s | HyperDrive helical mode |
| Source-to-Isocenter (R) | ~53.5 cm | - |
| Source-to-Detector (D) | ~95.0 cm | - |
| Cone beam angle | ±9.1° | Wide-angle; requires 3D Katsevich correction |

### 4.2 X-Ray Generation (Quantix™ 160)
| Parameter | Value |
|---|---|
| Configuration | Single liquid-bearing high-power tube |
| Generator power | 120–130 kW continuous |
| Tube voltages | 70, 80, 100, 120, 140 kV (discrete steps) |
| Tube current | 10–1300 mA (5 mA increments) |
| Fast kV-switching | 80 ↔ 140 kV every 0.25 ms |

### 4.3 Detector Array (Gemstone Clarity)
| Parameter | Value |
|---|---|
| Material | Ultra-fast Gemstone™ scintillator (0.03 µs) |
| z-coverage at isocenter | 160 mm (16 cm) |
| Physical grid | 256 rows × 896 channels/row |
| Total channels | Over 229,376 |
| Min. slice thickness | 0.625 mm |

### 4.4 Helical Trajectory & Reconstruction
| Parameter | Value |
|---|---|
| Volumetric mode | 16 cm axial single-rotation organ coverage |
| Pitch factor range | 0.15–1.531 |
| Max scan length | 2000 mm |
| Spectral mode | Fast single-tube pulse-by-pulse kV switching |
| Temporal window | 29 ms (SnapShot Freeze) |
| In-plane resolution | 0.23 mm (18.2 lp/cm, z-direction) |
| Matrix sizes | 512×512 / 1024×1024 |
| Reconstruction algorithms | TrueFidelity™ DL / 3D Katsevich FDK |
| Reconstruction speed | Up to 80 images/sec |
| Noise mitigation | Volara™ Modular DAS |

---

## 5. Our Simulated System Configuration
*(Source: CT Configuration team repo, `ct_configuration.py` defaults)*

| Parameter | Our Value | Siemens Force Reference | GE Healthcare Reference |
|---|---|---|---|
| Scan geometry | Helical | Helical | Helical / Volumetric Cone-Beam |
| SID | 541.0 mm | 535 mm | ~535 mm |
| SDD | 949.0 mm | 976 mm | ~950 mm |
| Geometric magnification | 1.754 | 1.824 | 1.776 |
| Bore diameter | 820.0 mm | 780 mm | 800 mm |
| Display FOV | 500.0 mm | 500 mm (primary sFOV) | Not specified (16 cm z-coverage given instead) |
| Rotation time | 0.28 s | 0.25 s (minimal) | 0.28 s / 0.23 s (Apex) |
| Rotation angle | 360° | 360° | 360° |
| Views per rotation | 984 | - | - |
| Tube voltage (kVp) | 120.0 | 70–150 (range) | 70/80/100/120/140 (discrete steps) |
| Tube current | 200.0 mA | 20–1300 mA/tube (range) | 10–1300 mA (5 mA increments) |
| Tube current range | 60–660 mA | 20–1300 mA/tube | 10–1300 mA |
| Exposure time | 1.0 s | - | - |
| Pitch | 0.516 | 0.15–3.2 (range) | 0.15–1.531 (range) |
| Collimation | 40.0 mm | - (57.6mm z-cov, dual) | - (160mm z-cov at isocenter) |
| Scan range | 350.0 mm | up to 2000 mm (max) | up to 2000 mm (max) |
| TCM enabled | Yes | - | - |
| TCM modulation depth | 25% | - | - |

**Key observations:**
- Our rotation time (0.28 s) matches GE Revolution's baseline exactly.
- Our kVp (120) lands exactly on one of GE's discrete voltage steps.
- Our SDD (949 mm) is nearly identical to GE's (~950 mm), and close to Siemens' (976 mm).
- Our SID sits almost exactly between both references (535 mm Siemens/GE vs our 541 mm).

**Derived parameters** (computed with the same formulas as the config module):
| Parameter | Formula | Value |
|---|---|---|
| Angular increment | 360° / 984 | 0.3659° |
| Table feed / rotation | Pitch × Collimation | 20.64 mm |
| Table speed | Table feed / rotation time | 73.71 mm/s |
| Number of rotations | ceil(scan range / table feed) | 17 |
| Total scan time | rotations × rotation time | 4.76 s |
| Total projection views | rotations × views/rotation | 16,728 |

---

## 6. X-ray Source Specifications
| Parameter | Value | Notes |
|---|---|---|
| kVp (operating) | 50-120 | Maximum accelerating potential applied between the cathode and anode. It determines the maximum X-ray photon energy; at 120 kVp, the spectrum extends up to approximately 120 keV |
| Focal spot size | 0.6 mm radius | Defines the spatial extent of the X-ray emission region at the anode target |
| Target material | Tungsten (W) | High atomic number (Z = 74); commonly used for X-ray tube targets because of its high X-ray production efficiency and high melting point |
| Added filtration | 4 mm Aluminum (Al) | Removes a significant portion of low-energy photons and hardens the X-ray beam |

## 7. Detector Specifications
| Parameter | Value | Notes |
|---|---|---|
| Detector type | Energy Integrating | Label recorded in metadata; tells downstream code/teams this detector sums total energy per pixel rather than counting individual photons |
| Number of physical rows | 64 | Number of detector rows along the z-axis (patient long axis); sets how many CT slices are acquired per rotation |
| Pixel pitch | 0.625mm | - |
| z-coverage at isocenter | 40 | Beam collimation width at the detector; recorded in metadata as the z-coverage the detector is exposed to |
| In-plane channels | 888 single array | Number of detector elements across the fan angle (in-plane); sets the width of each sinogram row and the in-plane sampling density |
| Target in-plane resolution | 0.365 | Requires the same SDD/SAD magnification factor to convert detector-plane pixel pitch (0.625mm) into an isocenter-referenced resolution (lp/cm at 0% MTF) |

## 8. Tissue Interaction / Phantom Specifications
| Parameter | Value | Notes |
|---|---|---|
| Current phantom | Dummy cylinder voxel model | generated using the cylinder voxel model; final voxel dataset not yet supplied |
| Cylinder dimensions (radius) | 25mm | - |
| Cylinder dimensions (height) | 50mm | - |
| Cylinder dimensions (voxel size) | 1mm | - |
| Material / density assigned | Soft tissue -- taken from xraylib library | dummy phantom uses material ID 1 with density 1.0 g/cm³ |
| Attenuation coefficient source | xraylib / NIST data | total energy-dependent mass attenuation coefficient obtained using CS_Total_CP() |
| Interaction model | Beer–Lambert law | Beer–Lambert law with energy-dependent total attenuation : $I(E) = I_0(E)e^{-\mu(E)L}$, with $\mu(E) = (\mu/\rho)\rho$ |
| Scatter modeled | No  | explicit Compton/Rayleigh scattering is not separately simulated; the current model uses total attenuation |

## 9. Reconstruction Specifications
| Parameter | Value | Notes |
|---|---|---|
| Algorithm | 3D Helical FDK / ADMIRE Iterative | FDK (Feldkamp-Davis-Kress) is standard for helical cone-beam geometry; iterative ADMIRE can be applied for noise reduction |
| Reconstruction kernel | Br40 (Standard Hann/Ramp filter) | Standard medium-smooth kernel for soft tissue; use sharp kernels (e.g., Hr60) for bone or high spatial resolution |
| Output matrix size | 512 × 512 | Standard clinical matrix resolution; 768 × 768 or 1024 × 1024 are available for high-resolution target imaging |
| Slice thickness | 0.6 mm (or 0.4 mm) | 0.6 mm is commonly configured for standard multi-slice helical body scans; 0.4 mm represents the system minimum |
| Helical rebinning/interpolation | 180° LI (Linear Interpolation) + Fan-to-Parallel Rebinning | Optimizes temporal resolution and slice sensitivity profile (SSP) while compensating for helical cone-beam geometry |

## 10. Post-Processing Specifications
| Parameter | Value | Notes |
|---|---|---|
| HU calibration approach | Water/air-based calibration | Air - 1000 HU, Water - 0 HU |
| Filters applied | BMD + 3D U-Net | Evaluated as separate denoising methods |
| Output format | Numpy+DICOM | NumPy for ML, DICOM for CT output |
| Dynamic range to support | -8192 to +57343 HU | 16-bit range/offset to be confirmed |

## 11. Radiation Dose Measurement Specifications
| Parameter | Value | Notes |
|---|---|---|
| Dose metrics computed | CTDI100 (centre): 0.69811 nGy, CTDI100 (peripheral mean): 0.90622 nGy, CTDIw: 0.83685 nGy, CTDIvol: 0.83685 nGy, DLP: 12.5527 nGy·cm | Calculated for 100,000 primary photons; pitch = 1 and scan length = 15 cm |
| Reference dose phantom | Solid PMMA cylinder: 160 mm diameter × 150 mm length, Mean absorbed dose: 0.09054 nGy | G4_PLEXIGLASS; density 1.19 g/cm3; mass 3.58896 kg |
| Method | Monte Carlo simulation using Geant4 11.4.2 | Livermore electromagnetic physics with a rotating 120 kVp polychromatic X-ray source |
| Effective Dose | Not calculated / Not reported | The simulation uses a homogeneous PMMA phantom and does not provide organ/tissue-specific doses |

## 12. Change Log
| Date | Change | Author |
|---|---|---|
| 27-09-2026 | Added detailed specifications for GE Healthcare Revolution CT, including system geometry, X-ray generation, detector array, and helical trajectory. Updated our simulated system configuration with comparisons to GE and Siemens references | Kuldeep Hansdah, Subham Kumar Beura |
