# CT Simulation — System Datasheet

## 1. Purpose & Scope
This document is the single reference for the physical, geometric, and
acquisition parameters used across the CT Simulation Project (X-ray Source,
Tissue Interaction, Detector, Reconstruction, CT Configuration, Post
Processing, Radiation Dose Measurement). All modules must build against
these values unless a change is logged in the Change Log below.

## 2. Reference System Basis & Disclaimer
> This system is **not** a reproduction of any specific commercial CT
> scanner. Select parameters (see notes below) were informed by publicly
> available specifications from GE Healthcare and Siemens scanners; other
> parameters, where manufacturer data is not public, were chosen
> independently by the project team for simulation purposes. Any
> resemblance to a specific commercial system's full spec sheet is
> coincidental — exact manufacturer specifications are not the goal here.

## 3. X-ray Source Specifications
| Parameter | Value | Unit | Source | Notes |
|---|---|---|---|---|
| Tube voltage (kVp) | 120.0 | kVp | CT Config repo default | |
| Tube current | 200.0 | mA | CT Config repo default | |
| Tube current range | 60 – 660 | mA | CT Config repo default | Used for TCM clamping |
| Exposure time | 1.0 | s | CT Config repo default | |
| Total mAs | 200.0 | mAs | CT Config repo default | |
| Focal spot size | TBD | mm | — | **Ask X-ray Source team** |
| Anode angle | TBD | degrees | — | **Ask X-ray Source team** |
| Target material | TBD (commonly W) | — | — | **Ask X-ray Source team** |
| Added filtration | TBD | mm Al/Cu eq. | — | **Ask X-ray Source team** |
| Spectrum model | TBD | mono/poly | — | **Ask X-ray Source team** |

## 4. Geometry & Acquisition Configuration
*(Source: CT Configuration team repo, `ct_configuration.py` defaults)*

| Parameter | Value | Unit |
|---|---|---|
| Scan geometry | Helical | — |
| Source-to-Isocenter Distance (SID) | 541.0 | mm |
| Source-to-Detector Distance (SDD) | 949.0 | mm |
| Bore diameter | 820.0 | mm |
| Display FOV | 500.0 | mm |
| Rotation time | 0.28 | s |
| Rotation angle | 360.0 | deg |
| Views per rotation | 984 | — |
| Pitch | 0.516 | — |
| Collimation | 40.0 | mm |
| Scan range | 350.0 | mm |
| TCM enabled | Yes | — |
| TCM modulation depth | 25 | % |

**Derived parameters** (computed with the same formulas as the config module):
| Parameter | Formula | Value |
|---|---|---|
| Geometric magnification | SDD / SID | 1.754 |
| Angular increment | 360° / 984 | 0.3659° |
| Table feed / rotation | Pitch × Collimation | 20.64 mm |
| Table speed | Table feed / rotation time | 73.71 mm/s |
| Number of rotations | ceil(scan range / table feed) | 17 |
| Total scan time | rotations × rotation time | 4.76 s |
| Total projection views | rotations × views/rotation | 16,728 |

## 5. Detector Specifications
| Parameter | Value | Notes |
|---|---|---|
| Detector type | TBD | scintillator vs photon-counting — **Ask Detector team** |
| Number of rows | TBD | — |
| Number of channels | TBD | — |
| Pixel pitch | TBD | mm — **Ask Detector team** |
| Quantum efficiency | TBD | — |
| ADC bit depth | TBD | — |

## 6. Tissue Interaction / Phantom Specifications
| Parameter | Value | Notes |
|---|---|---|
| Current phantom | Dummy cylinder voxel model | Placeholder — professor has not yet supplied the final voxel dataset |
| Cylinder dimensions | TBD | **Ask Tissue Interaction team** |
| Material / density assigned | TBD | **Ask Tissue Interaction team** |
| Attenuation coefficient source | TBD | e.g. NIST tables — **Ask Tissue Interaction team** |
| Interaction model | TBD | Beer-Lambert only / + Compton + photoelectric — **Ask Tissue Interaction team** |
| Scatter modeled? | TBD | Yes/No — **Ask Tissue Interaction team** |

## 7. Reconstruction Specifications
| Parameter | Value | Notes |
|---|---|---|
| Algorithm | TBD | FDK / FBP / iterative — **Ask Reconstruction team** |
| Reconstruction kernel | TBD | — |
| Output matrix size | TBD | — |
| Slice thickness | TBD | — |
| Helical rebinning/interpolation | TBD | — |

## 8. Post-Processing Specifications
| Parameter | Value | Notes |
|---|---|---|
| HU calibration approach | TBD | **Ask Post Processing team** |
| Filters applied | TBD | noise reduction / ring artifact correction |
| Output format | TBD | DICOM / NumPy / PNG |

## 9. Radiation Dose Measurement Specifications
| Parameter | Value | Notes |
|---|---|---|
| Dose metrics computed | TBD | CTDIvol / DLP / effective dose — **Ask Dose team** |
| Reference dose phantom | TBD | e.g. 16 cm / 32 cm CTDI phantom equivalent |
| Method | TBD | Analytical vs Monte Carlo |

## 10. Change Log
| Date | Change | Author |
|---|---|---|
| YYYY-MM-DD | Initial datasheet created from CT Configuration repo defaults | <your name> |
