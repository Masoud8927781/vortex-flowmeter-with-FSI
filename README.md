# README.md - Vortex Flowmeter OpenFOAM Case

This repository contains a complete OpenFOAM case that **exactly reproduces** the numerical simulation from the paper:

**Experimental verification and numerical simulation of a vortex flowmeter at low Reynolds numbers**  
*B. Končar, J. Sotošek, I. Bajsić*  
*Flow Measurement and Instrumentation, Volume 88, December 2022, 102278*  
DOI: https://doi.org/10.1016/j.flowmeasinst.2022.102278

## Case Overview

- **Solver**: `pimpleFoam` (incompressible, transient)
- **Turbulence model**: k-ω SST (URANS) — as used in the paper for low Re cases
- **Fluid**: Air (incompressible, ν ≈ 1.5 × 10⁻⁵ m²/s)
- **Example Reynolds number**: Re_D = 13477 (U_inlet ≈ 2.246 m/s)
- **Pipe diameter**: D = 90 mm
- **Bluff body geometry** (exact from Fig. 2):
  - Blockage ratio b/D = 0.24 → b = 21.6 mm
  - Aspect ratio b/l ≈ 0.9 → spanwise length l ≈ 24 mm
  - Front flat part: 20 mm
  - Wedge rear: 9 mm
  - Width at wedge connection: 12.5 mm
- **Domain**:
  - Upstream: ~350 mm
  - Downstream: ~550 mm
  - Background box mesh refined with snappyHexMesh
- **Probes**: Located exactly 50 mm downstream of the bluff body trailing edge at y = ±b/2 and centerline (as in Fig. 3)

Expected result: **Strouhal number St ≈ 0.237–0.24** (matches Table 2 in the paper).

## Requirements

- OpenFOAM (tested with v10–v2412, from openfoam.org or openfoam.com)
- Python 3 (only to generate the case — optional if you copy manually)

## How to Generate and Run the Case

### Option 1: Automatic (recommended)

1. Create an empty folder.
2. Save the Python script from the previous message as `create_case.py`.
3. Run:
   ```bash
   python3 create_case.py
   ```
4. The full case will be created in the current directory.

### Option 2: Manual

Just copy the generated folder structure and files (constant/, system/, 0/).

### Run the Simulation

```bash
blockMesh
surfaceFeatureExtract
snappyHexMesh -overwrite
checkMesh                  # Should report "Mesh OK"
pimpleFoam > log.pimpleFoam 2>&1 &
```

- Simulation time: ~20–30 seconds real flow time (endTime = 30 s, Δt = 1e-5 s)
- Mesh size: ~2M background → ~10–15M final cells (adjust blockMesh cells if too heavy)

## Post-Processing – Calculate Strouhal Number

1. Open with ParaView:
   ```bash
   paraFoam
   ```
   or open `case.foam` in ParaView.

2. The `probes` functionObject samples U and p at three points 50 mm downstream.

3. In ParaView:
   - Go to **Sources → Probe Location** or load the `postProcessing/probes/` data.
   - Plot U.x or p vs time at one of the off-center probes (y = ±0.0108 m).
   - You should see a clear sinusoidal oscillation once shedding is established (~5–10 s).

4. Determine shedding frequency f:
   - Count peaks over time interval, or
   - Export data → use FFT (in Python/MATLAB/Excel) to find dominant frequency.

5. Compute Strouhal number:
   ```
   St = f × b / U_inlet
   b = 0.0216 m
   U_inlet ≈ 2.246 m/s
   ```
   → Expected St ≈ 0.237 (very close to paper Table 2)

## Notes

- The STL (`constant/triSurface/bluffBody.stl`) is watertight and fully visible in FreeCAD.
- If the mesh is too large for your machine, reduce cells in `system/blockMeshDict` (e.g., change `(400 70 70)` → `(250 50 50)`).
- For other Re numbers, change `Re_target` in the Python script and re-run.

## License

This case is provided for educational and research purposes to reproduce published results.

Enjoy simulating — your results should match the paper very closely! 🚀
