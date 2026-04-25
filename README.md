# Behavioral Anomaly Detection in IoT Networks Using Artificial Intelligence
**Authors: 
-Karla Fišić
-Mila Matijević
-Matej Kvesić
Institution: Faculty of Mechanical Engineering, Computing and Electrical Engineering (FSRE), University of Mostar, Bosnia and Herzegovina**

This repository contains the source code for the paper. The implementation uses an autoencoder trained exclusively on normal IoT network traffic to detect anomalous (attack) flows based on reconstruction error. A comparative evaluation with Isolation Forest and One-Class SVM is also included.

---

## Requirements

- Google Account with Google Drive access
- Google Colab
- Python 3.x (provided by Colab)
- Libraries: `tensorflow`, `scikit-learn`, `pandas`, `numpy`, `matplotlib`

All libraries are pre-installed in Google Colab. No additional installation is needed.

---

## Dataset

The experiments use the **MQTT-IoT-IDS2020** dataset:

> H. Hindy, C. Tachtatzis, R. Atkinson, E. Bayne and X. Bellekens, "MQTT-IoT-IDS2020: MQTT Internet of Things Intrusion Detection Dataset", IEEE DataPort, 2020. https://ieee-dataport.org/open-access/mqtt-iot-ids2020-mqtt-internet-things-intrusion-detection-dataset

### Setup

1. Create a folder named `SIS` in the root of your Google Drive.
2. Place the following CSV files inside the `SIS` folder:

| File | Description | Samples |
|------|-------------|---------|
| `biflow_normal.csv` | Normal (benign) network traffic | 86,008 |
| `biflow_mqtt_bruteforce.csv` | MQTT Brute Force attack traffic | 16,696 |
| `biflow_sparta.csv` | SPARTA SSH Brute Force attack traffic | 91,318 |
| `biflow_scan_A.csv` | Aggressive Scan attack traffic | 25,693 |
| `biflow_scan_sU.csv` | UDP Scan attack traffic | 39,664 |

---

## Part 1 — Autoencoder Detection on MQTT Brute Force

This section reproduces the autoencoder anomaly detection results on MQTT Brute Force traffic.

**Expected results:**

| Metric | Value |
|--------|-------|
| Total Attack Samples | 16,696 |
| Detected Anomalies | 14,648 |
| Missed Attacks | 2,048 |
| Detection Rate | 87.73% |

### Steps

Run the following blocks in order in the Colab notebook:

1. **Block 1** — Mount Google Drive
2. **Block 2** — Set data path (`/content/drive/MyDrive/SIS`)
3. **Block 3** — Load `biflow_normal.csv`
4. **Block 4** — Inspect dataset (`.info()` and `.describe()`)
5. **Block 5** — Drop non-numeric and label columns (`ip_src`, `ip_dst`, `is_attack`), resulting in 29 numeric features
6. **Block 6** — Normalize features to [0, 1] using `MinMaxScaler` fitted on normal traffic
7. **Block 7** — Verify scaling (min = 0.0, max = 1.0)
8. **Block 8** — Define and compile the Autoencoder (architecture: Input(29) → 32 → 16 → 32 → Output(29), Adam lr=0.001, MSE loss)
9. **Block 9** — Train the Autoencoder (20 epochs, batch size 256, 10% validation split, normal traffic only)
10. **Block 10** — Plot training and validation loss curves
11. **Block 11** — Compute reconstruction error on normal traffic; set anomaly threshold at the **95th percentile**
12. **Block 12** — Plot reconstruction error histogram with threshold line
13. **Block 13** — Load `biflow_mqtt_bruteforce.csv`, scale using the fitted scaler, run through the trained Autoencoder, and evaluate detection

---

## Part 2 — Comparison of Unsupervised Methods on MQTT Brute Force

This section reproduces the comparative evaluation of the Autoencoder against Isolation Forest and One-Class SVM on the same MQTT Brute Force dataset.

**Expected results:**

| Method | Detection Rate |
|--------|---------------|
| Autoencoder | 87.73% |
| Isolation Forest | 95.71% |
| One-Class SVM | 89.35% |

### Steps

Continue from the state after completing Part 1 (the fitted scaler, trained Autoencoder, and loaded normal data must already be in memory), then run:

14. **Block 22** — Train `IsolationForest` on normal scaled data (100 estimators, contamination=0.05, random_state=42)
15. **Block 23** — Train `OneClassSVM` on normal scaled data (RBF kernel, nu=0.05, gamma='scale')
16. **Block 24** — Load and scale `biflow_mqtt_bruteforce.csv` attack data
17. **Block 26** — Run Isolation Forest prediction → expected detection rate: **95.71%**
18. **Block 27** — Run One-Class SVM prediction → expected detection rate: **89.35%**

The Autoencoder result (87.73%) is taken directly from the output of Block 13 in Part 1.

---

## Notes on Reproducibility

- The **anomaly threshold** is set at the **95th percentile** of reconstruction error computed on normal traffic (Block 11).
- The **Autoencoder** uses random weight initialization — minor variation across runs is expected. Results should remain close to those reported.
- **Isolation Forest** is seeded with `random_state=42` for full reproducibility.
- All models are trained **exclusively on normal traffic**, following the unsupervised anomaly detection paradigm.
- The `MinMaxScaler` is fitted only on normal training data and applied (without refitting) to attack data during evaluation.
