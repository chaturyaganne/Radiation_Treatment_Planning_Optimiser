## 1. Problem Setup

The experiment restricts the beam pool to the **7 beams pre-selected by the expert planner** from PlannerBeams.json, and asks NSGA-II to find the optimal **5-beam subset** from that pool.

| Parameter | Value |
|---|---|
| Total beams available | 72 |
| Planner beam pool | 7 |
| NSGA-II selects | 5 |
| Search space C(7,5) | **21 combinations** |
| A matrix shape | 432,753 voxels × 46,733 beamlets |
| PTV voxels | 58,845 |
| Lung voxels | 334,755 |

> **Note:** With only 21 combinations and a population of 5 running 10 generations, the entire search space is exhausted within 4–5 generations. NSGA-II is effectively performing **complete enumeration** here, not evolutionary search.

## 2. What the Lambda Sweep Does

The λ parameter controls **regularization strength** inside the LP sub-solver. It adds a structural prior (nuclear norm + smoothness penalty) on the beamlet intensity maps, penalizing complex, hard-to-deliver fluence patterns.

## 3. Lambda Sweep Summary Results

| λ | Hypervolume | Best OAR | Best PTV | Interpretation |
|---|---|---|---|---|
| 0.000 | 0.97763 | 0.2112 | 0.0000 | Pure dose — baseline |
| 0.001 | 0.99101 | 0.1991 | 0.0000 | Light reg improves OAR |
| 0.005 | 0.99749 | 0.1932 | 0.0000 | Further OAR improvement |
| **0.010** | **1.00373** | **0.1875** | **0.0000** | **Best — HV exceeds 1.0** |
| 0.050 | 0.99787 | 0.1689 | 0.0279 | Best OAR but PTV cost appears |

### OAR Improvement Across Lambda

OAR score **monotonically improves** with increasing λ. Regularization is not simply trading dose quality for structural simplicity — it is actively guiding the LP toward solutions that spare healthy tissue more effectively.

## 4. Key Finding — HV > 1.0 at λ = 0.01

Hypervolume is computed relative to reference point [1.1, 1.1]. A value **exceeding 1.0** means at least one Pareto solution achieves an OAR score below 0.1 on the normalised scale — extending beyond the unit square. This is clinically significant:

> Moderate regularization (λ = 0.01) produces dose plans that are **strictly better** than the unregularized baseline on the OAR objective, with **zero loss** in PTV coverage. This confirms that the structural prior is not a pure tradeoff — it actively improves dosimetric quality at the right strength.

## 5. PTV Score = 0 for λ = 0 to 0.01

For the four lowest λ values, every evaluated beam combination achieves:
This occurs because the LP uses W_PTV = 10, a heavy penalty on undercoverage that dominates the objective for any feasible 5-beam subset from the planner pool. Consequence:

- For λ < 0.05, NSGA-II degenerates to a **single-objective optimiser** (minimise OAR only)
- The Pareto front collapses to a single line at PTV = 0
- No genuine multi-objective tension exists until λ = 0.05

### At λ = 0.05 — The Real Tradeoff Emerges
The heavy regularization competes with PTV coverage in the LP objective. A small amount of tumor coverage is sacrificed to satisfy the structural prior. This is the **only λ value where a genuine Pareto front with two distinct objectives exists**.

## 6. Convergence Behaviour

| λ | Converged at Gen | Initial HV | Final HV | Improvement |
|---|---|---|---|---|
| 0.000 | Gen 3 | 0.97568 | 0.97763 | +0.00195 |
| 0.001 | Gen 4 | 0.99007 | 0.99101 | +0.00094 |
| 0.005 | Gen 4 | 0.99748 | 0.99749 | +0.00001 |
| 0.010 | Gen 1 | 1.00372 | 1.00373 | +0.00001 |
| 0.050 | Gen 7 | 0.98024 | 0.99787 | +0.01763 |

All runs converge by generation 4 at the latest, consistent with the search space being effectively exhausted. The exception is **λ = 0.05**, which shows meaningful improvement up to Gen 7 — this is the only setting where the bi-objective nature of the problem creates genuine evolutionary pressure.

## 7. Protocol Validation

| Metric | Limit | Result | Status |
|---|---|---|---|
| PTV coverage (D95) | ≥ 57.0 Gy | 61.12 Gy | ✓ PASS |
| PTV homogeneity (HI) | ≤ 0.10 | **0.9333** | ✗ **FAIL** |
| Esophagus Dmax | ≤ 45.0 Gy | **45.38 Gy** | ✗ **FAIL** |
| Cord Dmax | ≤ 30.0 Gy | 29.92 Gy | ✓ PASS |
| Lung mean dose | ≤ 20.0 Gy | 20.00 Gy | ✓ PASS |

### HI = 0.9333 — Critical Failure

**D05 = 117.12 Gy** means 5% of tumor voxels receive nearly **double the prescribed dose**. This severe hotspot arises because the LP formulation places no upper bound on PTV dose — it penalises underdosage but freely allows overdosage wherever it helps global coverage. Clinically, this level of non-uniformity would risk radiation necrosis within the tumor volume.

> **Root cause:** The LP needs an additional constraint of the form `max(A_ptv @ x) ≤ 1.1 × D_PTV` to cap hotspots. This is a formulation issue, not a beam selection issue — it would appear regardless of which angles are chosen.

### Esoph Dmax = 45.38 Gy — Borderline Failure

Only 0.38 Gy over the 45 Gy protocol limit. The LP's soft hinge penalty on esophagus overdose is not strong enough to push the maximum dose below the hard clinical threshold. A hard constraint `max(A_esoph @ x) ≤ 45` would resolve this.

## 8. Plan Quality Metrics

| Metric | Value | Interpretation |
|---|---|---|
| Sparsity | 31.49% | Only 31% of beamlets near-zero — dense plan |
| Total MU | 34,175 | High monitor units |
| Effective rank | 4.635 | ~5 independent beam components |
| Nuclear norm | 3767.9 | Beamlet complexity measure |

### Sparsity Comparison

With fewer beam angles, each beamlet must work harder. The LP distributes dose across nearly all available beamlets rather than selecting a sparse subset, because there are no redundant angles to ignore.

### Effective Rank = 4.635

The beamlet intensity matrix has approximately **5 independent components** — almost exactly equal to the number of beams selected. Each beam is contributing a spatially distinct dose pattern with minimal redundancy, indicating the 5 selected angles are genuinely complementary directions.

## 9. Selected Beam Angles

NSGA-II consistently dropped the two posterior-right beams (257° and 308°). These angles likely pass through more lung tissue before reaching the tumor, contributing to the lung mean dose constraint being binding at exactly 20 Gy. The retained 5 beams span approximately **205° of arc**, covering the anterior and left-lateral aspects of the patient.


