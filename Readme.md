#  Radiation Treatment Planning Optimizer

A unified optimization framework for Radiation Therapy Planning (RTP) combining  
**Convex Optimization, NSGA-II, and Inverse Optimization** with  
**Frobenius + Group Sparsity Regularization** for scalability and robustness.

---

##  Why This Project Matters

Radiation therapy must strike a precise balance:

-  Deliver high dose to tumors (PTV)  
-  Protect Organs At Risk (OARs)

This creates a large-scale, high-dimensional optimization problem with competing clinical objectives.

 This project builds a unified framework to compare optimization strategies under the same setting.

---

##  Core Idea

Beamlet Intensities  ──▶  Dose Influence Matrix  ──▶  Voxel Dose

Goal:
- ✔ Maximize tumor dose  
- ✔ Minimize organ damage  
- ✔ Keep plan simple & stable  

---

##  Methods Implemented

### 🔹 Convex Optimization (Baseline)
- Linear Programming formulation  
- Uses slack variables for constraint violations  
- Provides globally optimal solution  

✔ Reliable  
✔ Efficient  
✔ Strong baseline  

---

### 🔹 NSGA-II (Multi-Objective Optimization)
- Evolutionary algorithm  
- Produces a Pareto front of treatment plans  

Trade-off:
More Tumor Coverage  ←── trade-off ──→  More Organ Protection  

✔ Captures real clinical trade-offs  
✔ No need to scalarize objectives  

---

### 🔹 Inverse Optimization (Learning Preferences)
- Learns objective weights from reference plans  
- Formulated as bi-level optimization  

Reference Plan → Learn Weights → Generate Similar Plans  

✔ Personalized planning  
✔ Reduces manual tuning  
✔ Combines ML + optimization  

---

##  Regularization Strategy

### ✔ Frobenius + Group Sparsity (Used in this project)

Instead of spectral (nuclear norm) regularization, we use:

- Frobenius norm → controls overall magnitude (smoothness)  
- Group sparsity (ℓ₂,₁ norm) → enforces structured sparsity across beam groups  

### Why this choice?

 Nuclear norm → high memory + expensive SVD computations  
 Frobenius + Group Sparsity →
- Scales to large problems  
- Encourages structured beam selection  
- Much more computationally efficient  

---

##  System Overview

            +----------------------+
            |  Dose Matrix (A)     |
            +----------+-----------+
                       |
                       v
    +----------------------------------------+
    | Optimization Layer                     |
    |                                        |
    |  Convex   |  NSGA-II  |  Inverse       |
    |  Solver   |  Pareto   |  Learning      |
    +----------------------------------------+
                       |
                       v
            +----------------------+
            | Optimized Treatment  |
            | Plan (Beamlets)      |
            +----------------------+

---

##  Project Structure

```
Radiation_Treatment_Planning_Optimiser/
│
├── Convex_Optimisation/
│ ├── Convex_Optimisation.ipynb
│ ├── graphs/
│ ├── README.md
│ └── requirements.txt
│
├── NSGA-II/
│ ├── NSGA-II.ipynb
│ └── Graphs/
│
├── Inverse_Optimisation/
│ └── Inverse_Optimisation.ipynb
│
└── .gitignore

```

---

## ⚙️ Installation

```bash
git clone https://github.com/chaturyaganne/Radiation_Treatment_Planning_Optimiser.git
cd Radiation_Treatment_Planning_Optimiser
pip install -r Convex_Optimisation/requirements.txt

▶️ Run the Project

jupyter notebook Convex_Optimisation/Convex_Optimisation.ipynb
jupyter notebook NSGA-II/NSGA-II.ipynb
jupyter notebook Inverse_Optimisation/Inverse_Optimisation.ipynb
```

##  Outputs

Dose distribution plots

Pareto front visualizations

Beam intensity maps

Plan comparison graphs

Located in:

```
Convex_Optimisation/graphs/
NSGA-II/Graphs/
```

##  Dataset

LINK : https://huggingface.co/datasets/PortPy-Project/PortPy_Dataset

Uses the PortPy benchmark dataset, including:

Dose-influence matrices

Organ masks (HDF5)

Clinical beam configurations

## Authors

Chaturya Ganne

Dhathri Meda

Mohan Vamsi Varadaraju Priya

## Key Takeaways

Convex → optimal baseline

NSGA-II → trade-off exploration

Inverse → learned preferences

Frobenius + Group Sparsity → scalable + structured solutions


