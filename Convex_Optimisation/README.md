#  Convex Optimization for IMRT with Spectral Regularization  
### Lung_Patient_3 • PortPy Dataset

---

## Overview

This project builds a **clinically-inspired IMRT treatment planning system** using **convex optimization**, and introduces a **memory-efficient spectral regularization strategy** that:

-  Reduces **hotspots** (D05: 110 Gy → ~66 Gy)  
-  Cuts **nuclear norm by >80%**  
-  Lowers **total MU ~50%**  
-  Produces **simpler, sparse, structured beam patterns**  

 All without using computationally expensive **nuclear norm (SDP)** methods.

---

##  Problem Setting

In **Intensity-Modulated Radiation Therapy (IMRT)**, we aim to compute beamlet intensities that:

- Deliver **prescribed dose (60 Gy)** to tumor (PTV)  
- Minimize exposure to **Organs At Risk (OARs)**  
- Produce **deliverable, smooth, and sparse fluence maps**

Mathematically:

d = A x

- `x` → beamlet intensities  
- `A` → dose influence matrix  
- `d` → voxel dose  

---

##  Core Challenge

Standard approaches for structured fluence use:

> **Nuclear Norm Minimization (Low-Rank Constraint)**

But in IMRT:
-  Requires **semidefinite programming (SDP)**  
-  Leads to **RAM explosion** (not scalable)

---

##  Key Idea: Spectral Regularization (Efficient Surrogate)

We replace nuclear norm with:

R(X) = λ ||X||_F + λ Σ ||X_i,:||_2

Where `X` is the reshaped beam matrix.

### Why this works:

| Component | Effect |
|----------|--------|
| Frobenius norm | Shrinks singular values (global smoothing) |
| Group sparsity | Removes entire beam directions |
| Combined | Mimics **low-rank structure** efficiently |

✅ No SDP  
✅ Scales to real IMRT sizes  
✅ Retains spectral behavior  

---

##  Objectives

The optimization balances:

### Clinical Goals
- ✅ PTV coverage: **D95 ≥ 60 Gy**  
- ✅ OAR protection (cord, esophagus, lung)  
- ✅ Lung mean dose ≤ 20 Gy  

### Structural Goals
-  Reduce nuclear norm  
-  Reduce effective rank  
-  Lower total monitor units (MU)  
-  Increase sparsity & interpretability  

---

##  Project Structure
```
Convex_Optimisation/
│
├── Convex_Optimisation.ipynb # Main pipeline
├── graphs/ # DVH, fluence maps, comparisons
└── README.md
```

---

##  Pipeline Overview

### 1. Data Loading
Using PortPy:
- CT scans  
- Structure masks  
- Beam influence matrices  

Dataset:
> **Lung_Patient_3**

---

### 2. Preprocessing

- Extract voxel masks for:
  - PTV
  - Esophagus
  - Cord
  - Lung
- Build structure-specific matrices:
  - `A_ptv`, `A_esoph`, `A_cord`, `A_lung`

---

### 3. Optimization Model

Built using CVXPY

#### Decision Variable
x ∈ R^4420,  x ≥ 0

#### Objective
- F1 → PTV underdose  
- F2 → OAR overdose  
- Regularization → spectral surrogate  

#### Constraint
mean(A_lung x) ≤ 20

---

### 4. Beam Selection

- Uses **7 planner-selected beams**
- Reduces search space → more realistic planning

---

### 5. Metrics Computation

#### Tumor Metrics
- D95, D05  
- Homogeneity Index (HI)  
- Mean dose  

#### OAR Metrics
- Cord max dose  
- Esophagus max dose  
- Lung mean + V20  

#### Plan Complexity
- Nuclear norm  
- Effective rank  
- Sparsity (%)  
- Total MU  

---

##  Results & Insights

###  1. Dose Volume Histogram (DVH)
- Maintains **tumor coverage**
- Strong reduction in **hotspots**
- Improved **dose uniformity**

---

###  2. Beamlet Intensity Maps
- Much **smoother fluence**
- Clear **beam pruning**
- Higher **sparsity**

---

###  3. Lambda Sweep

| Metric | Trend |
|------|------|
| F1 (PTV underdose) | Slight ↑ |
| F2 (OAR overdose) | Slight ↑ |
| Nuclear norm | 🔻 Massive drop |

 Trade-off is **controlled and clinically acceptable**

---

###  4. Plan Comparison

| Metric | Improvement |
|-------|------------|
| Hotspots (D05) | 🔻 Huge |
| Homogeneity (HI) | 🔻 Dramatic |
| Nuclear norm | 🔻 >80% |
| MU | 🔻 ~50% |
| Sparsity | 🔺 Significant |

---

##  Key Takeaways

- Nuclear norm optimization is **impractical** for IMRT at scale  
- Spectral structure **can be approximated efficiently**  
- Combined regularization gives:
  - ✔ Low-rank behavior  
  - ✔ Sparse beams  
  - ✔ Clinically valid plans  

This is a strong example of:
**Bridging optimization theory with real-world clinical constraints**

---
##  Evaluation Metrics & Analysis

The quality of each treatment plan is evaluated using **dosimetric**, **clinical**, and **structural** metrics computed from the optimized beamlet intensities.

---

### Dose Computation

All metrics are derived from the voxel dose:

d = A x

- `x` → optimized beamlet intensities  
- `A` → dose influence matrix  
- `d` → dose delivered to each voxel  

---

##  Tumor Coverage Metrics (PTV)

### **D95 (Coverage)**
- Dose received by 95 percent of tumor volume  
- Computed as the 5th percentile of tumor voxel doses  
- Ensures adequate treatment  

✅ Target: ≥ 60 Gy  

---

### **D05 (Hotspot Indicator)**
- Dose received by top 5 percent of tumor voxels  
- Computed as the 95th percentile  

 High values indicate hotspots  

---

### **Homogeneity Index (HI)**

HI = (D05 - D95) / D95

- Measures uniformity of dose inside tumor  
- Lower is better  

| Value | Interpretation |
|------|--------------|
| ~0   | Ideal |
| >0.1 | Poor |

---

##  Organ at Risk (OAR) Metrics

### **Maximum Dose**
- Highest dose in organ  
- Protects sensitive structures  

---

### **Mean Dose**
- Average dose across organ  

---

### **V20 (Lung)**
- Fraction of lung receiving more than 20 Gy  
- Indicates radiation spread  

---

## Optimization Objective Metrics

### **F1 — Tumor Underdose**
- Penalizes tumor voxels receiving less than prescription  

---

### **F2 — OAR Overdose**
- Penalizes doses exceeding organ limits  

---

##  Plan Complexity Metrics

These are computed from reshaped beam matrix.

---

### **Nuclear Norm**
- Measures structural complexity  
- Lower values indicate simpler, more deliverable plans  

---

### **Effective Rank**
- Number of dominant intensity patterns  
- Lower → more structured plan  

---

### **Sparsity**
- Percentage of near-zero beamlets  
- Higher → fewer active beamlets  

---

### **Total Monitor Units (MU)**
- Total radiation delivered  
- Lower → faster and safer delivery  

---

## Visualization Analysis

### **Dose Volume Histogram (DVH)**

- X-axis: Dose (Gy)  
- Y-axis: Fraction of volume receiving at least that dose  

**Interpretation:**
- Steep curve → uniform dose  
- Right shift → higher dose  
- Left shift → lower dose  

---

### **Beamlet Intensity Maps**

- X-axis: Beamlet index  
- Y-axis: Beam index  
- Color: Intensity value  

**Interpretation:**
- Smooth patterns → good deliverability  
- Sparse patterns → fewer active beamlets  
- Noisy patterns → difficult to deliver  

---

### **Lambda Sweep**

- X-axis: Regularization strength  
- Y-axis: Metric values  

**Observed Trends:**
- Increasing lambda reduces complexity  
- Slight increase in dose penalties  
- Demonstrates tradeoff between accuracy and deliverability  

---

## Key Insight

The optimization balances:

- Tumor coverage  
- Organ protection  
- Plan simplicity  

by solving for beamlet intensities that produce clinically valid and physically deliverable radiation plans.

##  Tech Stack

- Python  
- CVXPY  
- NumPy  
- Matplotlib  
- PortPy  

---

##  How to Run

```bash
pip install portpy cvxpy numpy matplotlib
jupyter notebook Convex_Optimisation.ipynb
