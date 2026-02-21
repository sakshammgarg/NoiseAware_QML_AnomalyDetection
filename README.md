# Noise-Aware Variational Quantum Machine Learning for Tabular Anomaly Detection

This project implements a robust, research-grade **network anomaly detection pipeline** evaluated on both the **UNSW-NB15** and **NSL-KDD** datasets. By leveraging a **noise-aware hybrid quantum–classical variational model**, the system accurately detects malicious network traffic while maintaining robustness under noisy and realistic conditions.

## Project Overview

Network intrusion detection is a core challenge in modern cybersecurity due to evolving attack patterns and high-dimensional traffic data. This project addresses the limitations of traditional datasets and purely classical models by using the **UNSW-NB15 dataset** as the primary benchmark (with **NSL-KDD** for cross-dataset validation), both of which reflect contemporary network behaviors and attack types.

The system employs a **hybrid quantum machine learning (QML)** approach, where classical preprocessing is combined with **variational quantum circuits (VQCs)**. Noise awareness is explicitly incorporated into the model via a **trainable noise injection parameter**, making the pipeline suitable for realistic deployment scenarios and quantum research experimentation.

## Key Features

* **Hybrid Quantum–Classical Architecture**: Combines classical feature preprocessing with variational quantum circuits using PennyLane's `TorchLayer` for native batched quantum execution.
* **Noise-Aware Modeling**: A trainable input noise scale (`input_noise_scale`) is learned end-to-end alongside circuit weights, with an explicit noise penalty regularization term.
* **Publication-Ready Pipeline**: End-to-end workflow including preprocessing, multi-seed training, ablation studies, statistical significance testing, and comprehensive visualization.
* **Binary Anomaly Detection**: Classifies network flows as normal or malicious (binary label) on both datasets.
* **Classical Baseline Benchmarking**: Evaluated against XGBoost, Gradient Boosting, Random Forest, MLP, Isolation Forest, and Logistic Regression under identical training conditions.

## Technical Architecture

### 1) Feature Processing

Raw features from UNSW-NB15 (and NSL-KDD) are standardized using `StandardScaler`, then projected to **4 principal components** via PCA—one per qubit—for input to the quantum circuit. Categorical features in NSL-KDD (`protocol_type`, `service`, `flag`) are label-encoded before scaling. A 5-method **augmentation pipeline** (consistent noise, mixup, feature masking, PCA perturbation, adversarial-style) is available during training.

### 2) Variational Quantum Model

The hybrid model (`NoiseAwareQML`) consists of:

* **Data Re-Uploading Encoding**: Classical PCA features are re-encoded via `AngleEmbedding` at every layer—essential for expressibility.
* **Variational Quantum Circuit (VQC)**:
  * 4 qubits × 4 layers = **48 trainable circuit parameters**
  * Per-qubit RX, RY, RZ rotations followed by circular CNOT entanglement
  * All 4 qubits measured via PauliZ expectation values
* **Trainable Noise Injection**: A learnable `input_noise_scale` parameter injects Gaussian noise during training for robustness.
* **Classical Readout**: A single linear layer maps 4 qubit expectations to a binary logit.
* **Backend**: `lightning.qubit` (PennyLane) with adjoint differentiation; falls back to `default.qubit`.

### 3) Classical Optimizer

* **Optimizer**: Adam with learning rate 0.005
* **Loss Function**: Binary cross-entropy combined with an AUC ranking loss and a noise penalty regularization term (λ = 0.1)
* **Training Loop**: Classical–quantum feedback via hybrid optimization through PennyLane's `TorchLayer`

## Training Strategy

* **Data Split**: Stratified 15,000-sample training subset of UNSW-NB15 (from 175K), evaluated on the full 82K test set. NSL-KDD uses an 8,000-sample training subset.
* **Multi-Seed Validation**: 3 random seeds (42, 123, 256) on UNSW-NB15 for statistical reliability.
* **Training Configuration**: 10 epochs, batch size 1024, on GPU (CUDA) when available.
* **Ablation Studies**: Three variants tested—No Regularization (λ=0), Fixed Noise (σ=0.05, non-trainable), and No Noise Injection—over 3 epochs on a 5,000-sample subset.
* **Evaluation Metrics**:
  * ROC-AUC and PR-AUC
  * Accuracy, Precision, Recall, F1-score
  * Expected Calibration Error (ECE)
  * Noise robustness AUC at σ = 0.0, 0.05, 0.10, 0.15, 0.20, 0.25

## Results & Visualization

The pipeline generates the following artifacts for analysis and reporting:

* **Figure 1 (ROC Curves)**: Multi-model ROC comparison (Noise-Aware QML, No-Reg QML, XGBoost, Random Forest, MLP).
* **Figure 2 (Circuit Architecture)**: Schematic of the 4-qubit × 4-layer VQC with data re-uploading.
* **Figure 11 (Confidence Intervals)**: Multi-seed ROC-AUC, Accuracy, and F1 with 95% confidence intervals.
* **XAI Figures**: Per-layer, per-qubit, and per-gate gradient norm importance; input feature importance via permutation.
* **Noise Robustness Curves**: AUC degradation across six noise levels for all models.
* **Bloch Sphere Visualization**: Qubit state representations via PauliX, PauliY, PauliZ expectations.
* **Decision Boundary Plots**: 2D projections of learned boundaries in PCA space.
* **Calibration Curves**: Predicted probability reliability across models.

## Datasets

**Primary — UNSW-NB15**  
**Source**: [Kaggle – UNSW-NB15 Network Intrusion Detection Dataset](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)  
**Content**: Modern network traffic generated using the IXIA PerfectStorm tool, including normal traffic and multiple attack categories such as Fuzzers, Analysis, Backdoors, DoS, Exploits, Generic, Reconnaissance, Shellcode, and Worms. Used as the primary training and evaluation dataset.

**Cross-Validation — NSL-KDD**  
**Source**: [Kaggle – NSL-KDD](https://www.kaggle.com/datasets/hassan06/nslkdd)  
**Content**: Classic network intrusion dataset (`KDDTrain+.txt` / `KDDTest+.txt`) with 41 features and binary normal/attack labels. Used exclusively for cross-dataset generalization evaluation.

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

1. **Quantum Encoding Optimization**
   Explore amplitude encoding and more expressive data re-uploading strategies for higher-dimensional PCA projections.

2. **Advanced Noise Modeling**
   Incorporate hardware-calibrated noise models (depolarizing, thermal relaxation) for realistic quantum device simulation via PennyLane's noise module.

3. **Real-Time Intrusion Detection**
   Adapt the pipeline for streaming network traffic and low-latency inference using quantized or distilled classical surrogates.

---

This project demonstrates how **noise-aware hybrid quantum models** can be applied to realistic network intrusion detection tasks across multiple datasets, providing a strong foundation for future research in quantum-enhanced cybersecurity and anomaly detection.
