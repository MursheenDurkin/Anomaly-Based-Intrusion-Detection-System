# Anomaly-Based Intrusion Detection System
### A Hybrid Machine Learning Approach Combining XGBoost and Deep Autoencoder via Weighted Voting

**Author:** MursheenDurkin  
**Programme:** BSc (Hons) Cyber Security and Networking, Final Year Dissertation  
**Academic Year:** 2025-2026

> This repository is the project write-up. The notebooks, prototype code and dissertation documents formed the assessed submission and are not published here.

---

## Overview

This project built, evaluated and deployed a hybrid machine-learning intrusion detection system (ML-IDS) capable of detecting both known and previously unseen network attacks. It combines two fundamentally different detection approaches in a single system:

- A **supervised XGBoost classifier**, trained on labelled attack and benign traffic to recognise known attack patterns with high precision
- An **unsupervised deep autoencoder**, trained only on benign traffic to learn what a healthy network looks like and flag anything unusual as a potential threat

The two components are blended through a **weighted voting ensemble**, producing a combined detection rate that exceeds either component operating alone.

The system was trained and evaluated on the **CIC-IDS2017 dataset** (2,498,043 network flow records, 15 traffic categories) and benchmarked against **Snort 2.9**, a widely deployed signature-based IDS.

A **live traffic prototype** was then deployed on physical hardware: a Raspberry Pi 4 receiving mirrored traffic from a Cisco Catalyst 2960 switch via a SPAN port, demonstrating that the trained models classify real network traffic in near real time.

---

## Key Results

| System | Accuracy | Recall | F1-Score | AUC |
|--------|----------|--------|----------|-----|
| Hybrid ML-IDS (Weighted Voting) | 99.92% | 99.89% | 99.90% | 1.0000 |
| XGBoost alone | 99.90% | 99.83% | n/a | 0.9999 |
| Snort 2.9 (baseline) | 88.82% | 49.04% | 59.92% | n/a |

- The hybrid ensemble reduced missed attacks by **34%** compared with XGBoost alone
- The ML-IDS outperformed Snort by **66.5 percentage points in F1-score**
- False positive rate of **0.08%**; only 94 of 85,148 attack flows were missed

---

## How It Works

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

## What Was Built

**Analysis and modelling pipeline** (five Jupyter notebooks, each saving its outputs for the next stage):

1. Data exploration: dataset profiling and class distribution
2. Preprocessing: cleaning, feature scaling, train/test split
3. Snort 2.9 baseline: evidence-based evaluation of the signature-based comparator
4. Stacking ensemble: RandomForest + XGBoost exploration
5. Autoencoder and weighted voting: the core hybrid ML-IDS implementation

**Live traffic prototype** (Python):

- A flow extractor performing real-time packet capture and CICFlowMeter-compatible feature extraction with Scapy
- A live IDS engine that scores each flow through the ensemble and exposes alerts over a Flask REST API (`/api/alerts`, `/api/stats`, `/api/health`)

**Project documentation**:

- A phase-by-phase work journal covering the project from inception to completion, including challenges, decisions and outcomes
- A prior-knowledge document covering the technical stack, literature, results and future research directions, written as a reference for students building on the work

---

## Dataset

The project used the **CIC-IDS2017** dataset created by the Canadian Institute for Cybersecurity at the University of New Brunswick.

- **Source:** https://www.unb.ca/cic/datasets/ids-2017.html
- **Size:** approximately 2.8 million flow records, 80+ features
- **Attack types:** DoS, DDoS, PortScan, Brute Force, Web Attacks, Botnet, Infiltration, Heartbleed

---

## Environment

**Software:** Python, XGBoost, TensorFlow, scikit-learn, pandas, NumPy, Scapy, Flask, Matplotlib, Seaborn, Jupyter

**Hardware:**
- Ubuntu 22.04 Server VM (Oracle VirtualBox), 4 vCPU cores, 8 GB RAM, for training and evaluation
- Cisco Catalyst 2960 managed switch with SPAN port mirroring
- Raspberry Pi 4 as the live-traffic sensor

---

## Future Work

- **Red team / purple team pipeline**: automated adversarial attack generation with a model-retraining feedback loop
- **Concept drift monitoring**: ADWIN or Page-Hinkley detection to trigger retraining as traffic patterns evolve
- **Federated learning**: privacy-preserving collaborative model training across multiple network sensors
- **SHAP explainability**: per-alert feature contribution scores to support analyst trust
- **Robotic cybersecurity**: applying the hybrid IDS framework to robotic systems and autonomous platforms

---

## Acknowledgements

Thanks to my project supervisor and second marker for their guidance and feedback throughout.  
Dataset: **Canadian Institute for Cybersecurity**, University of New Brunswick

---

*This work was submitted as a final-year dissertation and is shared for academic reference. If you build on it, please cite appropriately.*
