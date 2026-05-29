# Quantum-Classical Hybrid Pipeline for Protein-Protein Interaction Prediction

A proof-of-concept hybrid pipeline combining classical machine learning with Grover's quantum search algorithm for protein-protein interaction (PPI) prediction. Built using real UniProt data, validated on both Qiskit AerSimulator and real IBM Quantum hardware (ibm_kingston, 156 qubits).

---

## Results at a Glance

| Experiment | Search Space | Steps | Outcome |
|---|---|---|---|
| Classical linear search | 1,000 pairs | 1,000 | Correct |
| Grover's — AerSimulator | 1,000 pairs | 24 | Correct (~100%) |
| Grover's — IBM Quantum | 32 pairs | 4 | Correct (17.2% vs 3.1% noise floor) |
| Theoretical — full dataset | 70,876 pairs | 209 vs 70,876 | **339x speedup** |

Best classical model: **Random Forest + SMOTE** — 91% accuracy, F1: 0.22, Recall: 0.36

---

## Pipeline Overview

```
UniProt Swiss-Prot API
        ↓
Feature Engineering (70,876 pairwise samples)
        ↓
SVM + Random Forest classifiers (with SMOTE)
        ↓
RF scores all pairs → identifies target
        ↓
Grover's algorithm (Qiskit) searches for best pair
        ↓
Classical O(N) vs Quantum O(√N) comparison
```

---

## Dataset

- **Source**: UniProt Swiss-Prot (reviewed human proteins, organism_id:9606)
- **Proteins**: 500 fetched, 377 with interaction annotations
- **Pairs**: 70,876 (C(377,2))
- **Features**: Length1, Length2, length\_diff, n\_int\_sum
- **Labels**: Shared interaction partners (1) or not (0)
- **Class balance**: 2,593 positive / 68,283 negative (27:1 imbalance)

---

## Notebooks

| Notebook | Description |
|---|---|
| `01_classical_pipeline.ipynb` | Data collection, feature engineering, SVM + RF + SMOTE training |
| `02_quantum_grover.ipynb` | Grover's circuit, AerSimulator run, IBM Quantum job submission and results |

---

## Quantum Circuit Details

**Oracle**: Phase flip on target state using X gates + H-MCX-H (phase kickback)  
**Diffusion**: Inversion about mean using H-X-MCX-X-H  
**Iterations**: k = floor(π/4 × √N)

IBM Quantum specs:
- Backend: ibm\_kingston (156 qubits)
- Transpiled gates: 2,302 (from 154 logical)
- Circuit depth: 1,423
- Success: 176/1024 shots on target vs ~32/1024 noise floor

---

## Setup

```bash
git clone https://github.com/DivyaS09122006/Protein_Protein_Interaction_Quantum-Classical-hybrid.git
cd Protein_Protein_Interaction_Quantum-Classical-hybrid
pip install -r requirements.txt
```

Run `01_classical_pipeline.ipynb` first — it fetches data and trains the model.  
Run `02_quantum_grover.ipynb` second — it loads the trained model and runs Grover's.

For IBM Quantum execution, you need a free account at [quantum.ibm.com](https://quantum.ibm.com) and an API token.

---

## Requirements

```
qiskit
qiskit-aer
qiskit-ibm-runtime
scikit-learn
imbalanced-learn
pandas
numpy
requests
joblib
```

---

## Key Findings

1. **SMOTE significantly improved minority class recall** (RF: 0.09 → 0.36) at the cost of some overall accuracy
2. **Grover's algorithm works correctly** on both simulator and real hardware
3. **NISQ hardware introduces noise** but quantum signal survives for shallow circuits (depth ~1,400)
4. **Deep circuits fail on real hardware** — 10-qubit transpilation produced depth 93,575, far beyond coherence limits
5. **339x theoretical speedup** is achievable once error-corrected quantum hardware matures

---

## Future Work

- Replace UniProt interaction annotations with STRING database (experimental labels + confidence scores)
- Add richer features: amino acid composition, physicochemical indices (BioPython)
- Frame as regression problem — predict STRING confidence scores instead of binary labels
- Replace classical SVM kernel with quantum kernel (PennyLane)
- Apply error mitigation (zero-noise extrapolation) for deeper circuit execution

---

## Reference

Rehman, A., et al. *Computational Analysis: Unveiling the Quantum Algorithms for Protein Analysis and Predictions.* IEEE Access, 2023.

---

*S Divya — IIIT Dharwad*
