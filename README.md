# Anomaly-Based Intrusion Detection System
### A Hybrid Machine Learning Approach Combining XGBoost and Deep Autoencoder via Weighted Voting

**Author:** MursheenDurkin  
**Programme:** BSc (Hons) Cyber Security and Networking, Final Year Dissertation  
**Academic Year:** 2025-2026

---

## Overview

This project builds, evaluates, and deploys a hybrid machine learning-based Intrusion Detection System (ML-IDS) capable of detecting both known and previously unseen network attacks. It combines two fundamentally different detection approaches into a single system:

- A **supervised XGBoost classifier**: trained on labelled attack and benign traffic to recognise known attack patterns with high precision
- An **unsupervised deep autoencoder**: trained only on normal (benign) traffic to learn what a healthy network looks like, flagging anything unusual as a potential threat

These two components are blended through a **weighted voting ensemble**, producing a system whose combined detection rate exceeds either component operating alone.

The system was trained and evaluated on the **CIC-IDS2017 dataset** (2,498,043 network flow records, 15 traffic categories) and benchmarked against **Snort 2.9**, a widely deployed signature-based IDS.

A **live traffic prototype** was also developed and deployed on physical hardware, a Raspberry Pi 4 connected to a Cisco Catalyst 2960 switch via SPAN port, demonstrating that the trained models can classify real network traffic in near real-time.

---

## Key Results

| System | Accuracy | Recall | F1-Score | AUC |
|--------|----------|--------|----------|-----|
| Hybrid ML-IDS (Weighted Voting) | 99.92% | 99.89% | 99.90% | 1.0000 |
| XGBoost alone | 99.90% | 99.83% |: | 0.9999 |
| Snort 2.9 (baseline) | 88.82% | 49.04% | 59.92% |: |

- The hybrid ensemble reduced missed attacks by **34%** compared to XGBoost alone
- The ML-IDS outperformed Snort by **+66.5% in F1-score**
- False positive rate: **0.08%**: only 94 of 85,148 attack flows missed

---

## Repository Structure

```
├── notebooks/
│   ├── 01_data_exploration.ipynb        # Initial dataset exploration and class distribution
│   ├── 02_preprocessing.ipynb           # Cleaning, feature scaling, train/test split
│   ├── 03_snort_baseline.ipynb          # Snort 2.9 evidence-based evaluation
│   ├── 04_stacking_ensemble.ipynb       # RandomForest + XGBoost stacking exploration
│   └── 05_autoencoder_weighted_voting.ipynb  # Core hybrid ML-IDS implementation
├── prototype/
│   ├── flow_extractor.py                # Real-time packet capture and flow feature extraction
│   └── live_ids.py                      # Live IDS engine with REST API
├── docs/
│   ├── Work_Journal_IDS_Project.docx    # Full project work journal (all phases)
│   └── Prior_Knowledge_Document.docx   # Project knowledge base for future researchers
├── data/
│   └── README.md                        # Dataset download instructions (CIC-IDS2017)
└── README.md
```

---

## System Architecture

The hybrid ML-IDS processes network traffic through the following pipeline:

```
Raw Network Traffic
        ↓
Flow Feature Extraction (CICFlowMeter-compatible, 70 features)
        ↓
    ┌─────────────────────────────────────┐
    │         Weighted Voting Ensemble     │
    │                                      │
    │  XGBoost (weight: 0.7)              │
    │  → Attack probability score         │
    │                                      │
    │  Deep Autoencoder (weight: 0.3)     │
    │  → Reconstruction error score       │
    └─────────────────────────────────────┘
        ↓
  Combined Score > 0.3093 threshold?
        ↓              ↓
    ATTACK          BENIGN
        ↓
  REST API Alert (/api/alerts)
```

---

## Dataset

This project uses the **CIC-IDS2017** dataset created by the Canadian Institute for Cybersecurity at the University of New Brunswick.

- **Download:** https://www.unb.ca/cic/datasets/ids-2017.html
- **Size:** ~2.8 million flow records, 80+ features
- **Attack types:** DoS, DDoS, PortScan, Brute Force, Web Attacks, Botnet, Infiltration, Heartbleed

Place the downloaded CSV files in the `data/` directory before running the notebooks.

---

## Requirements

```bash
pip install xgboost tensorflow scikit-learn pandas numpy scapy flask matplotlib seaborn jupyter
```

**Hardware used for development:**
- Ubuntu 22.04 Server VM (Oracle VirtualBox)
- 4 vCPU cores, 8 GB RAM
- Cisco Catalyst 2960 managed switch (for live prototype)
- Raspberry Pi 4 (traffic source for live testing)

---

## Running the Notebooks

Run the notebooks in order:

```bash
jupyter notebook
```

Start with `01_data_exploration.ipynb` and work through to `05_autoencoder_weighted_voting.ipynb`. Each notebook saves its outputs (preprocessed data, trained models) to disk so subsequent notebooks can load them without retraining.

---

## Live Traffic Prototype

The prototype requires root privileges and a network interface receiving mirrored traffic from a SPAN port:

```bash
sudo python3 prototype/live_ids.py --interface eth0
```

Once running, alerts are accessible via the REST API:

```bash
# Recent alerts
curl http://localhost:5000/api/alerts

# Summary statistics
curl http://localhost:5000/api/stats

# System health
curl http://localhost:5000/api/health
```

---

## Project Documentation

Full project documentation is available in the `docs/` folder:

- **Work Journal**: a detailed phase-by-phase record of the project from inception through to completion, including challenges encountered, decisions made, and outcomes achieved
- **Prior Knowledge Document**: a comprehensive knowledge base covering the technical stack, literature, results, and future research directions, intended as a reference for future students building on this work

---

## Future Work

The following directions have been identified for future development:

- **Red Team / Purple Team pipeline**: automated adversarial attack generation and model retraining feedback loop
- **Concept drift monitoring**: ADWIN or Page-Hinkley detection to trigger retraining as traffic patterns evolve
- **Federated learning**: privacy-preserving collaborative model training across multiple network sensors
- **SHAP explainability**: per-alert feature contribution scores for analyst trust
- **Robotic cybersecurity**: applying the hybrid IDS framework to robotic systems and autonomous platforms

---

## Acknowledgements

Thanks to my project supervisor and second marker for their guidance and feedback throughout.  
Dataset: **Canadian Institute for Cybersecurity**, University of New Brunswick  

---

## Licence

This project was submitted as a final year dissertation. The code and documentation are made available for academic reference. If you build on this work, please cite appropriately.

---

*For questions or collaboration enquiries: Barneyhk9@protonmail.com*  
*GitHub: [github.com/MursheenDurkin](https://github.com/MursheenDurkin)*
