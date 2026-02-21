The datasets used in this project are sourced from Kaggle.

Main Dataset: UNSW-NB15 Network Intrusion Detection Dataset

Link: [https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)

The UNSW-NB15 dataset contains modern network traffic data generated using the IXIA PerfectStorm tool in a controlled cyber range environment, comprising a mix of normal network activity and multiple categories of malicious attacks, including Fuzzers, Analysis, Backdoors, DoS, Exploits, Generic, Reconnaissance, Shellcode, and Worms. Each network flow record is represented by 49 features spanning packet-level, flow-level, content-based, and time-based attributes, along with class labels indicating whether the traffic is normal or malicious. The dataset is designed to support the training and evaluation of intrusion detection and anomaly detection models, addressing the limitations of older datasets by more accurately reflecting contemporary attack behaviors and realistic network conditions.

Cross-Validation Dataset: NSL-KDD

Link: [https://www.kaggle.com/datasets/hassan06/nslkdd](https://www.kaggle.com/datasets/hassan06/nslkdd)

The NSL-KDD dataset is a refined version of the KDD Cup 1999 dataset, widely used as a benchmark for network intrusion detection research. It contains labeled network connection records across normal traffic and four major attack categories — DoS, Probe, R2L, and U2R — represented by 41 features covering basic connection attributes, content-based statistics, and traffic-based features. In this project, all attack categories are mapped to a single binary malicious label for consistency with the primary dataset. NSL-KDD is used exclusively for cross-dataset generalization evaluation to assess model robustness beyond the training distribution.
