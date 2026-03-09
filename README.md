# Rouse Validation ï¿½ SURPASS-Alpha Monte Carlo Dynamics
## Deliverable Package

Based on: Kuriata, Gront & Sikorski (2016) CMST 22(4), 179-185

### Simulation Parameters
- Chain lengths: N = 25, 50, 100, 250, 500
- Volume fraction: phi = 0.035 (constant for all N)
- Athermal excluded volume (contactEnergy = 0, RepulsiveEnergy = 1e6)
- MC moves: Segmented hinge + N-tail + C-tail (local) + pivot (global)
- Bond spacing: l0 = 1.5 * sigma = 5.7 A (sigma = 3.8 A)
- Temperature: 300 K
- Random walk initialization for all chain lengths
- Backend: CPU only (CellListSegmentedEnergyComputer with CaContactKernel)
- GPU was not used; all simulations ran on CPU. GPU acceleration (ILGPU) is
  available in the codebase but was not benchmarked for this validation campaign.
  For large N (especially N=500), GPU would significantly reduce wall-clock time.

### Chain Counts and Equilibration
| N   | Chains | Eq Sweeps | Prod Sweeps | Box (A) | phi   |
|-----|--------|-----------|-------------|---------|-------|
| 25  | 500    | 2,000     | 5,000       | 269.6   | 0.035 |
| 50  | 500    | 5,000     | 5,000       | 339.7   | 0.035 |
| 100 | 500    | 5,000     | 5,000       | 428.0   | 0.035 |
| 250 | 200    | 10,000    | 5,000       | 428.0   | 0.035 |
| 500 | 110    | 25,000    | 10,000      | 441.8   | 0.035 |

### Results Summary

#### Static Properties (Table 1 from proposal)
| N   | <R2> (A^2)  | <Rg2> (A^2) | R2/Rg2 |
|-----|-------------|-------------|--------|
|  25 |     1433.06 |      229.27 | 6.2505 |
|  50 |     3284.41 |      526.02 | 6.2438 |
| 100 |     7261.44 |     1170.20 | 6.2053 |
| 250 |    18796.46 |     2994.87 | 6.2762 |
| 500 |    34780.32 |     5422.45 | 6.4141 |

- Static scaling exponent: 2nu = 1.0675 (R2), 1.0598 (Rg2)
- Expected: 2nu = 1.1756 (Clisby 2010), tolerance: 1.18 +/- 5%
- R2/Rg2 ratio: all values ~6.21-6.41, consistent with 3D SAW (6.254)

#### Dynamic Properties
- Diffusion coefficient scaling: D ~ N^(-2.338) (expected: -1.0)
- Relaxation time scaling: tau_R ~ N^(2.593) (expected: ~2.2)

#### Scaling Law Summary (Table 1 from proposal)
| Property            | Expected Exponent | Measured      | Status |
|---------------------|-------------------|---------------|--------|
| R2 ~ N^(2nu)        | 2nu = 1.18        | 1.067         | CHECK  |
| Rg2 ~ N^(2nu)       | 2nu = 1.18        | 1.060         | CHECK  |
| D ~ N^(-1)          | -1.0              | -2.338        | CHECK  |
| tau_R ~ N^(2.2)     | 2.2               | 2.593         | CHECK  |

### Directory Structure
```
rouse_validation_deliverable/
  README.md                          <- This file
  01_static_properties/              <- R2, Rg2, ratio vs N (scaling plots)
  02_dynamic_properties/             <- g_CM, g1, D vs N, tau_R vs N
  03_per_chain_length/               <- Individual N folders with per-run plots
    N25/  N50/  N100/  N250/  N500/
      R2_vs_MC_sweep.png
      Rg2_vs_MC_sweep.png
      autocorrelation_end_to_end_vector.png
      gcm_center_of_mass_msd.png
      g1_middle_segment_msd.png
  04_equilibration_evidence/         <- Combined equilibration monitoring
  05_data/                           <- Raw TSV data and JSON summary
    tavg_validation_summary.json
    N25/  N50/  N100/  N250/  N500/
  06_executable/                     <- Release build (.exe + all DLLs)
```

### Autocorrelation Analysis (End-to-End Vector)

| N   | Decorrelation time (tau_R) | Status                                         |
|-----|----------------------------|------------------------------------------------|
| 25  | ~4 sweeps                  | Fully decorrelated, many independent samples   |
| 50  | ~15 sweeps                 | Fully decorrelated                             |
| 100 | ~66 sweeps                 | Fully decorrelated                             |
| 250 | ~1000+ sweeps              | Decorrelates during production                 |
| 500 | ~9162 sweeps               | Barely crosses 1/e at end of 10k production    |

The autocorrelation function g_R(t) measures how quickly the end-to-end vector
loses memory of its initial orientation. A chain is considered decorrelated when
g_R(t) drops below 1/e. For N=25 through N=100, chains decorrelate rapidly and
production runs contain many independent samples. For N=250, decorrelation occurs
within the production window but yields fewer independent samples.

**N=500 is the critical case:** g_R(t) only reaches 1/e near sweep 9162, meaning
chains barely decorrelate once during the entire 10,000-sweep production run.
This confirms that N=500 at phi=0.035 needs significantly longer runs
(50k-100k production sweeps) or GPU acceleration for proper equilibrium sampling.
This under-sampling directly explains the elevated R2/Rg2 ratio (6.41 vs
expected ~6.25) and the depressed scaling exponents in the 5-point fits.

### Notes
- N=500 at phi=0.035 shows incomplete equilibration (R2/Rg2=6.41 vs expected ~6.25).
  Longer equilibration (50k-100k sweeps) or GPU acceleration needed.
- All runs used seed=42. Reproducibility confirmed for N=100 with seed=137.
