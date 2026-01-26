# Noise-Aware Hybrid Quantum Anomaly Detection

This project implements a robust, research-grade **network anomaly detection pipeline** on the **UNSW-NB15** dataset. By leveraging a **noise-aware hybrid quantum–classical variational model**, the system aims to accurately detect malicious network traffic while maintaining robustness under noisy and realistic conditions.

## Project Overview

Network intrusion detection is a core challenge in modern cybersecurity due to evolving attack patterns and high-dimensional traffic data. This project addresses the limitations of traditional datasets and purely classical models by using the **UNSW-NB15 dataset**, which reflects contemporary network behaviors and attack types.

The system employs a **hybrid quantum machine learning (QML)** approach, where classical preprocessing is combined with **variational quantum circuits (VQCs)**. Noise awareness is explicitly incorporated into the model to improve stability and generalization, making the pipeline suitable for realistic deployment scenarios and quantum research experimentation.

## Key Features

* **Hybrid Quantum–Classical Architecture**: Combines classical feature preprocessing with variational quantum circuits.
* **Noise-Aware Modeling**: Explicitly accounts for input uncertainty and quantum noise during training.
* **Publication-Ready Pipeline**: End-to-end workflow including preprocessing, training, evaluation, and visualization.
* **Multi-Class Attack Detection**: Supports multiple attack categories alongside normal traffic.
* **Realistic Dataset**: Evaluated on UNSW-NB15, designed to overcome limitations of legacy intrusion datasets.

## Technical Architecture

### 1) Feature Processing
Each network flow is represented using **49 features**, including:
* Packet-level attributes  
* Flow-based statistics  
* Content-based features  
* Time-based features  

Categorical features are encoded, and numerical features are normalized before being passed to the model.

### 2) Variational Quantum Model
The hybrid model consists of:
* **Quantum Feature Encoding**: Classical features mapped to quantum states.
* **Variational Quantum Circuit (VQC)**:
  * Parameterized rotation gates
  * Entangling layers
* **Noise Modeling**: Simulated noise channels to improve robustness.

### 3) Classical Optimizer
* **Optimizer**: Adam / gradient-based optimizers  
* **Loss Function**: Cross-entropy for binary or multi-class classification  
* **Training Loop**: Classical–quantum feedback via hybrid optimization

## Training Strategy

* **Data Split**: Train / validation / test sets
* **Class Handling**: Supports both binary (normal vs attack) and multi-class attack classification
* **Evaluation Metrics**:
  * Accuracy
  * Precision / Recall / F1-score
  * Confusion Matrix

## Results & Visualization

The pipeline generates the following artifacts for analysis and reporting:

* **Figure 1 (Training Curves)**: Loss and accuracy trends across epochs.
* **Figure 2 (Confusion Matrix)**: Performance across normal traffic and attack categories.
* **Figure 3 (Feature Impact / Sensitivity Analysis)**: Effect of feature subsets and noise levels on detection accuracy.

## Dataset

**Source**: [Kaggle – UNSW-NB15 Network Intrusion Detection Dataset](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)

**Content**: Modern network traffic generated using the IXIA PerfectStorm tool, including normal traffic and multiple attack categories such as Fuzzers, Analysis, Backdoors, DoS, Exploits, Generic, Reconnaissance, Shellcode, and Worms.

**Objective**: Classify network flows as normal or malicious and support robust intrusion and anomaly detection research.

## Dependencies

```bash
qiskit
pennylane
torch
scikit-learn
pandas
numpy
matplotlib
seaborn
````

## Installation & Usage

1. **Clone the Repository**

```bash
git clone https://github.com/sakshammgarg/NoiseAware_QML_AnomalyDetection.git
cd NoiseAware_QML_AnomalyDetection
```

2. **Download the Dataset**

* Visit the Kaggle dataset page:[https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)
* Download and extract the dataset files.
* Upload the extracted CSV files to your Google Colab environment.
* Organize the files into the following directory structure:
  ```
  /content
    /unsw-nb15
      UNSW_NB15_training-set.csv
      UNSW_NB15_testing-set.csv
  ```

3. **Run the Notebook / Scripts**

* Execute the notebook end-to-end.
* Results and evaluation figures are generated automatically.

## Future Improvements

1. **Quantum Encoding Optimization**
   Explore amplitude and angle encoding strategies for high-dimensional data.

2. **Advanced Noise Modeling**
   Incorporate hardware-inspired noise models for realistic quantum simulation.

3. **Real-Time Intrusion Detection**
   Adapt the pipeline for streaming network traffic and low-latency inference.

---

This project demonstrates how **noise-aware hybrid quantum models** can be applied to realistic network intrusion detection tasks, providing a strong foundation for future research in quantum-enhanced cybersecurity and anomaly detection.
