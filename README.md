# Anomaly-Based Intrusion Detection System
### A Hybrid Machine Learning Approach Combining XGBoost and a Deep Autoencoder via Weighted Voting

**Author:** MursheenDurkin  
**Programme:** BSc (Hons) Computing (Computer Networks and Cybersecurity), final year dissertation, First Class  
**Academic Year:** 2025-2026

> This repository is the project write-up. The notebooks, prototype code and dissertation documents formed the assessed submission and are not published here; the sections below describe how the submitted project was organised and run.

---

## Overview

This project builds and evaluates a hybrid machine learning Intrusion Detection System (ML-IDS) designed to detect both known and previously unseen network attacks. It combines two different detection approaches:

- A **supervised XGBoost classifier**, trained on labelled attack and benign traffic to recognise known attack patterns.
- An **unsupervised deep autoencoder**, trained only on benign traffic to learn what normal looks like, so that anything it reconstructs poorly is treated as suspicious.

The two are combined through a **weighted voting ensemble**. The ensemble achieved higher recall than either component on its own, missing 34% fewer attacks than XGBoost alone.

The system was trained and evaluated on the **CIC-IDS2017** dataset and compared against a **Snort 2.9** signature-based baseline.

A **live traffic prototype** was also built. It ran on an Ubuntu VM receiving traffic mirrored from a Cisco Catalyst 2960 switch SPAN port, with a Raspberry Pi 4 generating test traffic, and classified real network flows in near real time.

---

## Key Results

Binary classification (attack vs benign) on the held-out test set of 499,609 flows:

| System | Accuracy | Precision | Recall | F1-Score | AUC | FPR |
|--------|----------|-----------|--------|----------|-----|-----|
| **Hybrid ML-IDS (Weighted Voting)** | 99.92% | 99.63% | **99.89%** | 99.76% | 1.0000 | 0.08% |
| XGBoost alone | 99.92% | 99.68% | 99.83% | 99.76% | 1.0000 | 0.06% |
| Autoencoder alone | 89.03% | 71.12% | 59.96% | 65.06% | 0.8644 | 5.00% |
| Snort 2.9 (simulated baseline) | 88.82% | 77.02% | 49.04% | 59.92% | N/A | 22.98% |

- **Missed attacks:** the hybrid missed 94 of 85,148 attack flows, compared with 143 for XGBoost alone, a 34% reduction.
- **False positive rate:** 0.08%, meaning 313 of 414,461 benign flows were wrongly flagged.
- **The trade-off:** the hybrid raises slightly more false alarms than XGBoost alone (0.08% vs 0.06%) in exchange for catching more attacks. For an IDS, a missed attack usually costs more than a false alarm.
- **Against Snort:** the hybrid's F1-score was 66.5% higher (99.76% vs 59.92%). Snort missed over half of all attacks.

Results above 99% are typical on CIC-IDS2017 and may not carry over to real networks. See [Limitations](#limitations).

---

## How It Works

```
Network flow (70 features)
        ↓
StandardScaler
        ↓
    ┌──────────────────────────────────────┐
    │       Weighted Voting Ensemble       │
    │                                      │
    │  XGBoost (weight 0.7)                │
    │  → attack probability                │
    │                                      │
    │  Deep Autoencoder (weight 0.3)       │
    │  → reconstruction error,             │
    │    normalised to 0-1                 │
    └──────────────────────────────────────┘
        ↓
  Combined score > 0.3093 ?
        ↓              ↓
     ATTACK         BENIGN
        ↓
  REST API alert (/api/alerts)
```

- **XGBoost:** binary classifier (benign vs attack) with 200 estimators.
- **Autoencoder:** TensorFlow/Keras, layers 70 → 64 → 32 → 16 → 32 → 64 → 70. Trained only on 1,657,841 benign samples (90/10 train/validation split) using the Adam optimiser and MSE loss, with early stopping.
- **Ensemble:** score = 0.7 × XGBoost attack probability + 0.3 × normalised autoencoder error. The 0.3093 threshold was chosen using Youden's J statistic on the ROC curve.
- **Also tested:** a stacking ensemble (RandomForest and XGBoost base models with a LogisticRegression meta-learner) for comparison. It reached 99.91% accuracy but was not used in the final system.

The most important feature for XGBoost was **Bwd Packet Length Std** (the variation in packet sizes travelling back from the server), followed by Average Packet Size and Idle Mean.

---

## Dataset

This project uses the **CIC-IDS2017** dataset from the Canadian Institute for Cybersecurity, University of New Brunswick.

- **Download:** https://www.unb.ca/cic/datasets/ids-2017.html (CSV flow records)
- **Raw size:** over 2.8 million flow records across eight CSV files, 78 features each
- **After cleaning:** 2,498,043 records, 70 features, 15 classes (benign plus 14 attack types)
- **Split:** 80/20 stratified train/test (1,998,434 / 499,609)
- **Attack types:** DoS (Hulk, GoldenEye, Slowloris, Slowhttptest), DDoS, PortScan, FTP and SSH brute force, web attacks (brute force, XSS, SQL injection), Bot, Infiltration and Heartbleed

**Preprocessing:** removed rows with missing and infinite values and duplicates, encoded labels, scaled features with StandardScaler, and downcast to 32-bit floats to fit within 8 GB of RAM.

Place the downloaded CSV files in the `data/` directory before running the notebooks.

---

## Repository Structure

```
├── notebooks/
│   ├── 01_data_exploration.ipynb             # Dataset exploration and class distribution
│   ├── 02_preprocessing.ipynb                # Cleaning, scaling, train/test split
│   ├── 03_snort_baseline.ipynb               # Snort 2.9 evidence-based evaluation
│   ├── 04_stacking_ensemble.ipynb            # RandomForest + XGBoost stacking comparison
│   └── 05_autoencoder_weighted_voting.ipynb  # Core hybrid ML-IDS
├── prototype/
│   ├── flow_extractor.py                     # Packet capture and flow feature extraction
│   └── live_ids.py                           # Live IDS engine with REST API
├── docs/
│   ├── Work_Journal_IDS_Project.docx         # Phase-by-phase project journal
│   └── Prior_Knowledge_Document.docx         # Knowledge base for future work
├── data/
│   └── README.md                             # Dataset download instructions
└── README.md
```

---

## Requirements

```bash
pip install xgboost tensorflow scikit-learn pandas numpy scapy flask matplotlib seaborn jupyter
```

Versions used: scikit-learn 1.5, XGBoost 2.1, TensorFlow/Keras 2.18.

**Development environment:**
- Ubuntu 22.04 Server VM (Oracle VirtualBox), 4 vCPU cores, 8 GB RAM, CPU only
- Cisco Catalyst 2960 managed switch (live prototype)
- Raspberry Pi 4 (traffic source for live testing)

Training times on this VM: XGBoost 14.3 seconds, autoencoder 2.3 minutes, stacking ensemble 12 minutes.

---

## Running the Notebooks

```bash
jupyter notebook
```

Run them in order, from `01_data_exploration.ipynb` to `05_autoencoder_weighted_voting.ipynb`. Each notebook saves its outputs (preprocessed data, trained models) so the next one can load them without retraining.

---

## Live Traffic Prototype

The prototype captures packets with scapy, groups them into flows by 5-tuple (source and destination IP, source and destination port, protocol), extracts the same 70 features, and classifies each flow with the trained ensemble. Capture and classification run in separate threads.

It needs root privileges and a network interface receiving mirrored traffic from a SPAN port:

```bash
sudo python3 prototype/live_ids.py --interface eth0
```

Alerts are available through a Flask REST API, ready for SIEM integration:

```bash
curl http://localhost:5000/api/alerts   # Recent alerts
curl http://localhost:5000/api/stats    # Summary statistics
curl http://localhost:5000/api/health   # System health
```

---

## Limitations

- **Dataset age:** CIC-IDS2017 was recorded in 2017 in a controlled testbed, so it does not reflect every modern attack or real network conditions. Very high accuracy is common on this dataset.
- **Simulated Snort baseline:** the dataset is provided as CSV flow records rather than packet captures, so traffic could not be replayed through Snort. Instead, Snort's 3,948 active community rules were analysed and an evidence-based detection rate was assigned to each attack type.
- **No significance testing:** improvements are reported as absolute differences without statistical significance tests.
- **Hardware:** all training ran on a CPU-only VM with 8 GB RAM.
- **Live testing scope:** the prototype was tested on a single switch with one Raspberry Pi 4 as the traffic source.

---

## Future Work

- **Red Team and Purple Team pipeline:** automatically generate adversarial attack variants and feed them back into training.
- **Concept drift monitoring:** detect when traffic patterns change (for example with ADWIN or Page-Hinkley) and trigger retraining.
- **Federated learning:** train across multiple network sensors without pooling raw traffic data.
- **Explainability:** per-alert feature contribution scores (for example SHAP) to help analysts trust and act on alerts.

---

## Acknowledgements

Thanks to my project supervisor and second marker for their guidance and feedback.  
Dataset: Canadian Institute for Cybersecurity, University of New Brunswick.

---

## Licence

Submitted as a final year dissertation. The code and documentation are available for academic reference. If you build on this work, please cite it.

---

*GitHub: [github.com/MursheenDurkin](https://github.com/MursheenDurkin)*
