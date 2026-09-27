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
> specifications from real scanners — specifically a **Siemens SOMATOM
> Force** (dual-source) reference datasheet (Section 3) and, where
> available, GE Healthcare specifications (Section 4, pending). Other
> parameters, where manufacturer data is not public, were chosen
> independently by the project team for simulation purposes. Our simulated
> system is **single-source**, so any dual-source-specific reference value
> (e.g. detector row counts, temporal resolution, dual-energy modes) is
> used only as a design *reference range*, not copied directly — this is
> called out explicitly wherever it applies.

---

## 3. Reference System 1 — Siemens SOMATOM Force (Dual-Source CT)
*(Source: Siemens SOMATOM Force Reference Datasheet, provided to the project team)*

### 3.1 System Geometry & Gantry Mechanics
| Parameter | Value | Notes |
|---|---|---|
| Gantry bore size | 78 cm | |
| Minimal rotation time | 0.25 s | |
| Max scan speed | 737 mm/s | Flash Spiral mode only |
| Source-to-Isocenter (R) | 53.5 cm | |
| Source-to-Detector (D) | 97.6 cm | |
| Temporal resolution | 66 ms | Dual-source specific — N/A for our single-source sim |

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
**Status: TBD.** No GE datasheet has been provided yet. If a value below is
attributed to "GE Healthcare," it came from a team member's own research —
upload the source datasheet here (same format as Section 3) so it can be
properly cited.

---

## 5. Our Simulated System — Actual Configuration
*(Source: CT Configuration team repo, `ct_configuration.py` defaults)*

| Parameter | Our Value | Siemens Force Reference | Comparison |
|---|---|---|---|
| Scan geometry | Helical | Helical | Match |
| SID | 541.0 mm | 535 mm | +1.1% |
| SDD | 949.0 mm | 976 mm | −2.8% |
| Geometric magnification | 1.754 | 1.824 | −3.8% |
| Bore diameter | 820.0 mm | 780 mm | +5.1% |
| Display FOV | 500.0 mm | 500 mm (primary sFOV) | Exact match |
| Rotation time | 0.28 s | 0.25 s (minimal) | Close, slightly slower |
| Rotation angle | 360° | 360° | Match |
| Views per rotation | 984 | — | — |
| Tube voltage (kVp) | 120.0 | 70–150 (range) | Within range |
| Tube current | 200.0 mA | 20–1300 mA/tube (range) | Within range |
| Tube current range | 60–660 mA | 20–1300 mA/tube | Narrower (single-source) |
| Exposure time | 1.0 s | — | — |
| Pitch | 0.516 | 0.15–3.2 (range) | Within range |
| Collimation | 40.0 mm | — (57.6mm z-cov, dual) | Not directly comparable |
| Scan range | 350.0 mm | up to 2000 mm (max) | Within range |
| TCM enabled | Yes | — | — |
| TCM modulation depth | 25% | — | — |

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
| kVp (operating) | 120.0 | Within Siemens Force's 70–150 kV range |
| Focal spot size | TBD | Reference range: 0.4×0.5mm (S) / 0.8×1.1mm (L) — **ask X-ray Source team to pick and justify** |
| Anode angle | TBD | **Ask X-ray Source team** |
| Target material | TBD (commonly tungsten) | **Ask X-ray Source team** |
| Added filtration | TBD | mm Al/Cu equivalent — **ask X-ray Source team** |
| Spectrum model | TBD | Mono vs polychromatic — **ask X-ray Source team** |

## 7. Detector Specifications
| Parameter | Value | Notes |
|---|---|---|
| Detector type | Scintillator (EID) | Scintillator vs photon-counting — **ask Detector team** |
| Number of physical rows | 64 | Siemens ref (single array): 192 rows — use as an upper-bound reference, not a target |
| Pixel pitch | 0.625mm | **ask Detector team** |
| z-coverage at isocenter | TBD | Siemens single-array equivalent ≈ 28.8mm (57.6/2) — reference only |
| In-plane channels | 888 single array | Siemens ref: 3,120 total (dual-array) |
| Target in-plane resolution | TBD | Siemens ref: 0.24 mm (22–32 lp/cm @ 0% MTF) |

## 8. Tissue Interaction / Phantom Specifications
| Parameter | Value | Notes |
|---|---|---|
| Current phantom | Dummy cylinder voxel model | generated using the cylinder voxel model; final voxel dataset not yet supplied |
| Cylinder dimensions (radius) | 25mm | - |
| Cylinder dimensions (height) | 50mm | - |
| Cylinder dimensions (voxel size) | 1mm | - |
| Material / density assigned | Soft tissue -- taken from xraylib library | dummy phantom uses material ID 1 with density 1.0 g/cm³ |
| Attenuation coefficient source | xraylib / NIST data | total energy-dependent mass attenuation coefficient obtained using CS_Total_CP() |
| Interaction model | Beer–Lambert law | Beer–Lambert law with energy-dependent total attenuation : \(I(E)=I_0(E)e^{-\mu(E)L}\), with \(\mu(E)=(\mu/\rho)\rho\) |
| Scatter modeled | No  | explicit Compton/Rayleigh scattering is not separately simulated; the current model uses total attenuation |

## 9. Reconstruction Specifications
| Parameter | Value | Notes |
|---|---|---|
| Algorithm | TBD | Siemens ref uses 3D Helical FDK / ADMIRE Iterative — **FDK (Feldkamp-Davis-Kress) is a natural fit** since we already use helical cone-beam geometry — confirm with Reconstruction team |
| Reconstruction kernel | TBD | **ask Reconstruction team** |
| Output matrix size | TBD | Siemens ref options: 512×512 / 768×768 / 1024×1024 |
| Slice thickness | TBD | Siemens ref minimum: 0.4 mm |
| Helical rebinning/interpolation | TBD | **ask Reconstruction team** |

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
| Dose metrics computed | TBD | CTDIvol / DLP / effective dose — **ask Dose team** |
| Reference dose phantom | TBD | e.g. 16 cm / 32 cm CTDI phantom equivalent |
| Method | TBD | Analytical vs Monte Carlo |

## 12. Change Log
| Date | Change | Author |
|---|---|---|
| YYYY-MM-DD | Initial datasheet created from CT Configuration repo defaults | <your name> |
| YYYY-MM-DD | Added Siemens SOMATOM Force reference datasheet (Section 3) and comparison table (Section 5) | <your name> |
