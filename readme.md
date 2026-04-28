# Radiation Treatment Planning via Spectral Regularization

## Overview

This project focuses on **Intensity-Modulated Radiation Therapy (IMRT)** planning using advanced optimization techniques. The goal is to compute optimal beamlet intensities that:

- Deliver sufficient radiation dose to the tumor (PTV)
- Minimize exposure to surrounding healthy tissues (OARs)
- Ensure the treatment plan is **clinically deliverable**

We introduce a **computationally efficient alternative to nuclear norm minimization** using:

- Group sparsity (ℓ₂,₁ norm)
- Frobenius norm regularization

This approach avoids expensive **Singular Value Decomposition (SVD)** while still promoting **low-rank structure** in beamlet intensity maps.

---

## Key Idea

Traditional IMRT optimization uses **nuclear norm minimization**:

||W||_* = Σ σᵢ(W)

However, this is computationally expensive for large datasets due to repeated SVD computations.

### Proposed Approach

We replace nuclear norm with a **dual-penalty proxy**:

λ₁ ||W||₂,₁ + λ₂ ||W||_F²

Where:
- ||W||₂,₁ = sum of ℓ₂ norms of columns of W
- ||W||_F² = sum of squared entries of W

### Interpretation:

- **Group sparsity (ℓ₂,₁ norm)** → removes unnecessary beamlets (structured sparsity)
- **Frobenius norm** → smooths intensity distribution and stabilizes solution

Together, they induce **low-rank-like behavior without SVD**

---

## Problem Formulation

We solve the optimization problem:

minimize:
F₁(w) + μ F₂(w) + λ₁ ||W||₂,₁ + λ₂ ||W||_F²

subject to:
w ≥ 0

Where:
- F₁(w): Tumor underdose penalty  
- F₂(w): OAR overdose penalty  
- W: reshaped beamlet intensity matrix  

---

## Optimization Frameworks

### 1. Convex Optimization (SOCP)
- Implemented using **CVXPY**
- Guarantees **global optimality**
- Fast and scalable

### 2. NSGA-II (Multi-objective Evolutionary Optimization)
- Explores trade-offs between competing objectives
- Produces a **Pareto front of solutions**
- Computationally expensive but clinically insightful

### 3. Inverse Optimization
- Recovers objective weights from a reference clinical plan
- Uses **KKT conditions + QCQP formulation**
- Captures implicit expert decision-making

---

## Dataset

- **PortPy Lung_Patient_3**
- ~200,000 voxels  
- 4,420 beamlets  
- Sparse dose influence matrix  

---

## Results

| Metric | Baseline | Regularized |
|--------|----------|-------------|
| Homogeneity Index (HI) | 0.831 | 0.102 |
| Nuclear Norm | 6547.9 | ~1000 |
| Monitor Units (MU) | ~40,000 | ~20,000 |
| Solve Time | ~7 min | <2 min |

---

## Key Observations

- Homogeneity Index improved by ~88%
- Solve time reduced significantly
- Beam intensity maps became smoother and more structured
- Plans became significantly more clinically deliverable

---

## Features

- Convex IMRT optimization using SOCP
- Spectral regularization without SVD
- Group sparsity implementation (ℓ₂,₁ norm)
- DVH (Dose Volume Histogram) analysis
- Inverse optimization pipeline
- Lambda (λ) sweep experiments for trade-off analysis

---

## Clinical Significance

This approach reduces treatment complexity while preserving dose accuracy, enabling:

- Faster treatment planning cycles
- Reduced MLC motion complexity
- Improved patient throughput
- More stable and interpretable fluence maps

---

## Summary

This work demonstrates that **structured convex regularization can effectively replace nuclear norm minimization**, achieving:

- High-quality dose distributions
- Significant computational savings
- Clinically meaningful deliverability improvements