#  Radiation Treatment Planning (RTP) Optimizer

A unified optimization framework for Radiation Therapy Planning (RTP) combining  
**Convex Optimization (SOCP), NSGA-II, and Inverse Optimization** with  
**Frobenius + Group Sparsity Regularization** for scalability, stability, and clinical robustness.

> Built on the [PortPy](https://github.com/PortPy-Project/PortPy) benchmark framework and validated against TG-101 clinical protocols.

---

##  Why This Project Matters

Radiation therapy must strike a precise balance:

-  **Deliver high dose** to the tumor (PTV — Planning Target Volume)
-  **Protect Organs At Risk** (OARs: lungs, heart, spinal cord)

This creates a large-scale, high-dimensional optimization problem with **competing clinical objectives**.

> **Example:** Maximizing lung tumor dose risks heart damage. Over-protecting the heart risks tumor survival. This framework automates finding that clinical "sweet spot."

**Additional challenges this project addresses:**

-  **Clinical Efficiency** — Manual beam weight tuning takes hours. Our Inverse Optimization module learns from past successful plans to automate this.
-  **Plan Stability** — Patients move slightly during treatment (e.g., breathing). Frobenius Regularization ensures small positional changes don't cause catastrophic dose delivery failures.

---

##  Core Pipeline

```
Patient CT + Structures (PortPy)
          │
          ▼
  ┌─────────────────────────────────┐
  │       Preprocessing Layer       │
  │  • Define PTV / OAR voxel sets  │
  │  • Reshape w → W (B × K)        │
  │  • Build Dose Influence Matrix  │
  └────────────┬────────────────────┘
               │
       ┌───────┼───────────────┐
       ▼       ▼               ▼
  ┌─────────┐ ┌──────────┐ ┌──────────────┐
  │ Convex  │ │ NSGA-II  │ │   Inverse    │
  │  SOCP   │ │  (EMO)   │ │Optimization  │
  └────┬────┘ └────┬─────┘ └──────┬───────┘
       │           │              │
  Optimal w*  Pareto Set {w}  Learned w*
  (Single     (Trade-off      (Clinically
   Best Plan)  Plans)          Aligned Plan)
       │           │              │
       └───────────┼──────────────┘
                   ▼
     ┌─────────────────────────────────┐
     │    Post-processing & Evaluation  │
     │  Dose = D·w  │  DVH Curves      │
     │  HI, D95     │  OAR metrics     │
     │  Nuclear Norm / Sparsity        │
     └───────────────┬─────────────────┘
                     ▼
         ┌───────────────────────┐
         │   Visualization Layer  │
         │  • Dose maps           │
         │  • Beamlet intensity   │
         │  • Pareto fronts       │
         │  • Plan comparisons    │
         └───────────────────────┘
```

---

##  Methods Implemented

### 🔹 Convex Optimization — *The Gold Standard Baseline*

- **Formulation:** Second-Order Cone Program (SOCP) via CVXPY
- **Objective:** Minimizes $F_{PTV} + F_{OAR} + \lambda_1 \|\mathbf{W}\|_{2,1} + \lambda_2 \|\mathbf{W}\|_F$
- **Mechanism:** Uses slack variables for soft constraint violations
- **Output:** Single globally optimal beamlet weight vector $w^*$

 Reliable &nbsp;&nbsp;  Efficient &nbsp;&nbsp;  Strong baseline

---

### 🔹 NSGA-II — *Multi-Objective Pareto Exploration*

- **Formulation:** Evolutionary Multi-Objective Optimization (EMO)
- **Steps:** Population initialization → Dose evaluation → Non-dominated sorting → Crossover & Mutation
- **Output:** Pareto Set $\{w\}$ representing the full tumor coverage vs. organ protection trade-off

```
More Tumor Coverage  ◄────── trade-off ──────►  More Organ Protection
```

 Captures real clinical trade-offs &nbsp;&nbsp;  No objective scalarization needed

The algorithm identifies the **"elbow point"** — where further tumor coverage leads to exponential increases in organ toxicity — enabling evidence-based clinical decision-making.

---

### 🔹 Inverse Optimization — *Learning Institutional Style*

- **Formulation:** Bi-level optimization
- **Mechanism:** Given a reference plan $x^*$, computes subgradients via KKT conditions, then solves a QCQP to recover objective weights
- **Output:** Learned weights $w^*$ that reproduce clinically aligned plans

```
Reference Plan (x*) → KKT Subgradients → QCQP → Learned Weights → New Plans
```

 Personalized to institutional planning style &nbsp;&nbsp;  Reduces manual tuning &nbsp;&nbsp;  Bridges ML + optimization

---

##  Regularization Strategy

### Why Not Nuclear Norm?

| Approach | Memory | Compute | Beam Structure |
|---|---|---|---|
|  Spectral / Nuclear Norm | High | Expensive SVD | Unstructured |
|  **Frobenius + Group Sparsity** | Low | Efficient | Structured |

### Our Approach: Frobenius + Group Sparsity ($\ell_{2,1}$)

- **Frobenius norm** $\|\mathbf{W}\|_F$ — controls overall magnitude, encourages smooth beam intensity maps
- **Group Sparsity** $\|\mathbf{W}\|_{2,1}$ — enforces structured sparsity: if a beam is "on," neighboring beamlets are coherently activated

**Clinical benefit:** Fewer "checkerboard" patterns in beam intensity maps → reduced mechanical wear on the Linac's Multi-Leaf Collimators (MLC) → more deliverable plans.

**Efficiency gain:** ~60% reduction in computation time vs. nuclear norm regularization.

---

##  Key Results

Based on simulations using the PortPy benchmark dataset:

| Metric | Result |
|---|---|
| PTV Coverage (D95) |  95% of tumor volume reaches prescription dose |
| OAR Constraints |  Mean dose to Lungs & Heart below TG-101 thresholds |
| Computation Time |  ~60% faster than nuclear norm baseline |
| MLC Deliverability |  Reduced checkerboard artifacts in beam maps |
| Pareto Front |  Elbow point identified for clinical trade-off decisions |

## Cost Effectiveness

1. Convex Optimization (The "Budget" Speedster)
    Cost (Time/Compute): Lowest. It solves in seconds or minutes using highly optimized math (like Interior Point methods).
    Effectiveness: High, but only if you already know the correct weights ($w_1, w_2$).
    Verdict: This is the most cost-effective for execution. If you have a standard protocol and just need a plan now, this is your best bet.
  
3. NSGA-II (The "Premium" Explorer)
   Cost (Time/Compute): Highest. It has to run the optimization loop hundreds or thousands of times (for every individual in every generation). It requires significant CPU/GPU power and time.
   Effectiveness: Maximum Information. It doesn't give you one plan; it gives you the entire Pareto Front.
   Verdict: This is not cost-effective for daily routine, but it is highly effective for research or "difficult cases" where doctors don't know what the best trade-off is yet.

5. Inverse Optimization (The "Long-term" Investment)
   Cost (Time/Compute): Medium. There is a high "upfront cost" to train the model on historical plans, but once the weights are learned, it's very fast.
   Effectiveness: Very High. It eliminates the "human cost" of a physicist sitting at a computer for 3 hours manually tuning weights.
   Verdict: This is the most cost-effective for a Hospital System. It standardizes quality and saves the most expensive resource: the clinician's time.

---

##  Related Work & Literature

| Reference | Contribution |
|---|---|
| Bortfeld et al. | Classical Inverse Planning — single-objective formulation |
| Babier et al. | $L_1$ regularization shown to produce noisy beam maps → motivation for Group Sparsity |
| PortPy Framework | Industry-standard benchmarking dataset and dose computation library |

---

##  Project Structure

```
Radiation_Treatment_Planning_Optimiser/
│
├── Convex_Optimisation/
│   ├── Convex_Optimisation.ipynb   # LP/SOCP model & DVH analysis
│   ├── graphs/                     # Output dose distribution plots
│   ├── README.md
│   └── requirements.txt
│
├── NSGA-II/
│   ├── NSGA-II.ipynb               # Evolutionary algorithm & Pareto front
│   └── Graphs/                     # Pareto front visualizations
│
├── Inverse_Optimisation/
│   ├── Inverse_Optimisation.ipynb  # Weight-learning via bi-level optimization
│   └── Graphs/
│
└── .gitignore                      # Excludes large HDF5 dose matrices
```

---

##  Setup & Usage

### 1. Clone & Install

```bash
git clone https://github.com/chaturyaganne/Radiation_Treatment_Planning_Optimiser.git
cd Radiation_Treatment_Planning_Optimiser
pip install -r Convex_Optimisation/requirements.txt
```

### 2. Download Dataset

Obtain the PortPy benchmark dataset (includes dose-influence matrices, organ masks in HDF5, and clinical beam configurations):

🔗 [PortPy Dataset on HuggingFace](https://huggingface.co/datasets/PortPy-Project/PortPy_Dataset)

### 3. Run Notebooks

```bash
jupyter notebook Convex_Optimisation/Convex_Optimisation.ipynb
jupyter notebook NSGA-II/NSGA-II.ipynb
jupyter notebook Inverse_Optimisation/Inverse_Optimisation.ipynb
```

---

##  Outputs

All results are saved to the respective `graphs/` or `Graphs/` directories:

- Dose-Volume Histogram (DVH) curves
- Pareto front visualizations
- Beam intensity maps
- Plan comparison plots

---

##  Key Takeaways

| Method | Best For |
|---|---|
| **Convex SOCP** | Single optimal plan; reliable baseline |
| **NSGA-II** | Exploring the full trade-off space; clinical decision support |
| **Inverse Optimization** | Replicating institutional planning style; reducing manual effort |
| **Frobenius + Group Sparsity** | Scalable, structured, deliverable beam plans |

---

##  Contributors

- **Chaturya Ganne**
- **Dhathri Meda**
- **Mohan Vamsi Varadaraju Priya**
