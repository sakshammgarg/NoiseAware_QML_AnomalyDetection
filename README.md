# Noise-Aware QML: Adaptive Noise-Regularized Variational Quantum Circuits for Parameter-Efficient Tabular Anomaly Detection

This project implements **NoiseAware-QML**, a parameter-efficient **hybrid quantum–classical variational model** evaluated on the **UNSW-NB15** and **NSL-KDD** datasets. By embedding a learnable noise parameter directly into the quantum ansatz, the system achieves superior noise robustness, calibration, and cross-dataset generalization at a 54-parameter footprint—a 715× compression over Random Forest.

## Project Overview

Anomaly detection on high-dimensional tabular data—spanning fraud analysis, industrial defect detection, medical outlier identification, and security auditing—demands models that are simultaneously compact, robust to noise, and interpretable. Traditional ensemble methods fail this requirement under strict parameter budgets: Random Forest requires 38,640 parameters and XGBoost 4,892 on UNSW-NB15, making them unsuitable for edge or federated deployment.

This project addresses these limitations via **NoiseAware-QML**, a 4-qubit, 4-layer **Variational Quantum Circuit (VQC)** with only **54 trainable parameters**. A differentiable noise scale σ is embedded inside the ansatz and updated end-to-end via the reparameterization trick, enabling the circuit to adaptively learn noise robustness during training. The result is a multi-seed mean **ROC-AUC of 0.8366 ± 0.0083** (95% CI: [0.8252, 0.8446]) on UNSW-NB15—outperforming all equal-budget classical baselines—with an AUC-per-1k-parameters of **15.49**, which is 645× higher than Random Forest.

## Key Features

* **Hybrid Quantum–Classical Architecture**: Combines classical PCA preprocessing with a 4-qubit, 4-layer VQC using PennyLane's `default.qubit` exact state-vector backend.
* **Adaptive Noise Injection**: A trainable scalar σ injects Gaussian noise into all rotation angles at every forward pass via the reparameterization trick, providing structural robustness—1.84% AUC loss at σ=0.10 versus 4.12% for Random Forest.
* **Parameter Efficiency**: 54 total trainable parameters (48 circuit weights + 1 σ + 5 readout), achieving 715× compression over Random Forest and an ≈216-byte model footprint.
* **Binary Anomaly Detection**: Classifies tabular instances as normal or anomalous on UNSW-NB15 (primary) and NSL-KDD (cross-dataset generalization).
* **Classical Baseline Benchmarking**: Evaluated against XGBoost, Gradient Boosting, Random Forest, MLP (705 params), Logistic Regression (15 params), Isolation Forest, **MLP-54** (parameter-matched classical baseline), and **Dropout-MLP-54** (equal-budget noise-robustness comparator).

## Technical Architecture

### 1) Feature Processing

Raw features from UNSW-NB15 (and NSL-KDD) are standardized using `StandardScaler`, then projected to **4 principal components** via PCA—one per qubit—for input to the quantum circuit. Categorical features in NSL-KDD (`protocol_type`, `service`, `flag`) are label-encoded before scaling. PCA components are encoded into rotation angles via a tanh transform.

### 2) Variational Quantum Model

The hybrid model (`NoiseAwareQML`) consists of:

* **Angle Encoding via tanh**: Classical PCA features are encoded into rotation angles at every layer for expressibility.
* **Variational Quantum Circuit (VQC)**:
  * 4 qubits × 4 layers = **48 trainable circuit parameters**
  * Per-qubit RX, RY, RZ rotations followed by a circular CNOT ring generating all-pairs entanglement
  * Single-qubit Pauli-Z expectation value ⟨Ẑ₀⟩ mapped to a probability via sigmoid
* **Trainable Noise Injection**: A learnable scalar σ perturbs all rotation angles during training (˜θ = θ + σ·ε, ε ~ N(0,1)), updated via the reparameterization trick gradient.
* **Classical Readout**: A 5-parameter fully-connected layer maps the circuit output to a binary logit. Total parameters: 54.
* **Backend**: `default.qubit` (PennyLane) with exact state-vector simulation.

### 3) Classical Optimizer

* **Optimizer**: Adam with learning rate 0.01
* **Loss Function**: Binary cross-entropy combined with a pairwise AUC ranking loss and a quadratic noise penalty regularization term (λσ², λ = 0.01)
* **Training Loop**: Classical–quantum feedback via hybrid optimization through PennyLane's gradient-based differentiation

## Training Strategy

* **Data Split**: Stratified 15,000-sample training subset of UNSW-NB15 (from 175K), evaluated on the full 82K test set. NSL-KDD uses an 8,000-sample training subset.
* **Multi-Seed Validation**: 3 random seeds {42, 123, 256} with 2000-iteration bootstrap confidence intervals for statistical reliability.
* **Training Configuration**: 10 epochs, batch size 64, on GPU (CUDA) when available.
* **Ablation Studies**: Four variants tested—Adaptive Full (trainable σ, λ=0.01), No Regularization (trainable σ, no penalty), Fixed Noise (σ=0.05, non-trainable), and No Noise Injection—confirming adaptive noise regularization improves over all variants by ≈0.089 AUC.
* **Evaluation Metrics**:
  * ROC-AUC and F1-score (multi-seed mean ± std)
  * Accuracy
  * Expected Calibration Error (ECE)
  * Noise robustness AUC at σ = 0.0, 0.05, 0.10, 0.15, 0.20, 0.25
  * AUC per 1,000 parameters

## Results & Visualization

The pipeline generates the following artifacts for analysis and reporting:

* **Figure 1 (ROC Curves)**: Multi-model ROC comparison (NoiseAware-QML, No-Reg QML, XGBoost, Random Forest, MLP).
* **Figure 2 (Circuit Architecture)**: Schematic of the 4-qubit × 4-layer VQC with circular CNOT ring and adaptive noise injection.
* **Figure 10 (Multi-Seed CIs)**: ROC-AUC = 0.8366 ± 0.0094, F1 = 0.7657 ± 0.0069, Accuracy = 0.7562 ± 0.0076 across 3 seeds.
* **Figure 11 (Bootstrap CIs)**: 2000-iteration bootstrap 95% CIs for all key metrics, including ∆AUC@0.10 CI of [2.22%, 3.02%].
* **XAI Figures**: Per-layer, per-qubit, and per-gate gradient norm importance heatmaps revealing that RY gates dominate and PCA-1 is actively harmful (∆AUC = −0.0086).
* **Noise Robustness Curves**: AUC degradation across six noise levels for all models (QML: 1.84% vs RF: 4.12% at σ=0.10).
* **Bloch Sphere Visualization**: Qubit state representations showing partial class separation in qubit 0's lower hemisphere.
* **Calibration Curves**: ECE comparison (QML: 0.0726, RF: 0.1127, MLP: 0.1151) without post-hoc calibration correction.
* **Deployment Simulation**: NISQ hardware emulation under Profile A (IBM Falcon-like) and Profile B (aggressive), confirming <2% AUC degradation.

## Datasets

**Primary — UNSW-NB15**  
**Source**: [Kaggle – UNSW-NB15](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)  
**Content**: Modern network traffic generated using the IXIA PerfectStorm tool (175,341 samples, 42 features), including normal traffic and multiple attack categories such as Fuzzers, Analysis, Backdoors, DoS, Exploits, Generic, Reconnaissance, Shellcode, and Worms. Used as the primary training and evaluation dataset (15K train / 82K test).

**Cross-Validation — NSL-KDD**  
**Source**: [Kaggle – NSL-KDD](https://www.kaggle.com/datasets/hassan06/nslkdd)  
**Content**: Classic network intrusion dataset (`KDDTrain+.txt` / `KDDTest+.txt`) with 41 features and binary normal/attack labels. Used exclusively for cross-dataset generalization evaluation—NoiseAware-QML achieves 5.36% AUC drop versus 8.47% for Random Forest at σ=0.10.

## Dependencies

```bash
pennylane
pennylane-lightning-gpu
torch
scikit-learn
xgboost
pandas
numpy
matplotlib
seaborn
scipy
tqdm
```

## Installation & Usage

1. **Clone the Repository**
```bash
git clone https://github.com/sakshammgarg/NoiseAware_QML_AnomalyDetection.git
cd NoiseAware_QML_AnomalyDetection
```

2. **Download the Datasets**
* **UNSW-NB15**: Visit [https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15) and add to your Kaggle environment.
* **NSL-KDD**: Visit [https://www.kaggle.com/datasets/hassan06/nslkdd](https://www.kaggle.com/datasets/hassan06/nslkdd) and add to your Kaggle environment.
* The notebook expects datasets at the following paths:
  ```
  /kaggle/input/unsw-nb15/
    UNSW_NB15_training-set.csv
    UNSW_NB15_testing-set.csv
  /kaggle/input/nslkdd/
    KDDTrain+.txt
    KDDTest+.txt
  ```

3. **Run the Notebook / Scripts**
* Execute the notebook cells end-to-end on Kaggle (GPU recommended).
* A session checkpoint is saved after the cross-dataset validation step (`checkpoint_after_cell11.pkl`) for recovery in case of interruption.
* All result figures and saved models are written to `/kaggle/working/`.

## Future Improvements

1. **Hardware Validation on IBM Quantum**
   Validate the learned σ⋆ directly on IBM Quantum hardware to confirm that simulation-derived NISQ robustness transfers to physical devices.

2. **XAI-Guided Circuit Pruning**
   Remove the near-zero RZ gates at layer 4 identified by gradient norm heatmaps, reducing the model to ≈50 parameters without measurable AUC cost.

3. **Multi-Class Anomaly Categorization**
   Extend binary detection to multi-class attack categorization by leveraging all four Pauli-Z qubit measurements for a richer output representation.

4. **Federated Quantum Learning**
   Adapt the 216-byte model footprint for privacy-preserving federated deployment across distributed edge nodes, exploiting the inherent compression advantage.

5. **Approximate Simulation Backends**
   Explore tensor-network and approximate simulation backends to enable training on the full 175K UNSW-NB15 dataset rather than the 15K stratified subset currently required by exact state-vector simulation.

---

This project demonstrates that **adaptive noise-aware hybrid quantum models** can achieve superior parameter efficiency, calibration, and noise robustness over equal-budget classical baselines on tabular anomaly detection tasks, establishing a rigorous foundation for future research in quantum-enhanced anomaly detection and near-term NISQ deployment.
