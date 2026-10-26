# Concept Drift Detection and Adaptation in ML-Based Network Intrusion Detection

Implementation and evaluation of the **HDDAF** (Hybrid Drift Detection and Adaptation Framework) on the **CIC-IDS2017** dataset.

**Selected paper:** A. Lara-Gutiérrez, C. Fernández-Gago, E. Onieva, "A framework for drift detection and adaptation in AI-driven anomaly and threat detection systems," *International Journal of Information Security*, vol. 24, art. 199, 2025. doi:10.1007/s10207-025-01118-9

---

## Repository Contents

| File | Description |
|------|-------------|
| `DriftGuard.zip` | Source code (HDDAF implementation, experiments and plotting scripts) |
| `Graphs.zip` | All result graphs (Figures 1-6) |
| `Research Paper.pdf` | The selected research paper |
| Project report (PDF) | Full report: methodology, literature table, results and limitations |
| `DataSet Link.txt` | Link to download the CIC-IDS2017 dataset |
| `MachineLearningCSV.zip` | Dataset CSV files (available in the **Releases** section of this repo) |

> Large files are stored with Git LFS or in Releases. Download them using the GitHub download button.

---

## About the Project

HDDAF keeps a network intrusion detector accurate when traffic changes over time (concept drift). It combines:

1. **Preprocessing:** Information Gain selects the top 32 features and SMOTE balances rare attack classes.
2. **Hybrid model:** a neural network feeds an ensemble of Random Forest and LightGBM, with a Logistic Regression meta-classifier.
3. **Drift detection:** a Hoeffding-bound detector tests each incoming batch.
4. **Mixed-drift handling:** mild drift triggers fine-tuning, severe drift triggers full retraining.
5. **Poisoning defence:** KMeans clustering and RONI (Reject On Negative Impact) remove suspicious or label-poisoned samples.
6. **GAN drift simulator:** proposed in the paper with no reported results, so it is **not evaluated** here.

---

## Experimental Setup

- Dataset: CIC-IDS2017, replayed as a stream of **120 chunks of 2,000 flows**.
- **6 true drift points** (changes of day or attack type), using the real chronological order of the data rather than swapping batches at preset points.
- Positive class: attack. Precision, recall and F1 are computed on it.
- Compared schemes: Static, Periodic, ADWIN, DDM, PSI, KS, HDDAF and HDDAF-noDefence (poisoning defence off).
- Two scenarios: **clean labels** and **poisoned labels** during retraining.
- Fixed random seeds are used.

---

## Key Results (clean labels)

| Scheme | F1 | Drifts found | Mean delay (chunks) | False alarms | Retrainings |
|--------|-----|--------------|---------------------|--------------|-------------|
| Static | ~0 | - | - | 0 | 0 |
| Periodic | 0.592 | - | - | 0 | 12 |
| ADWIN | 0.841 | 3/6 | 1.67 | 3 | 6 |
| DDM | 0.825 | 3/6 | 1.67 | 0 | 3 |
| PSI | 0.943 | 4/6 | 0.25 | 17 | 25 |
| KS | 0.943 | 5/6 | 0.8 | 28 | 36 |
| **HDDAF** | **0.938** | **5/6** | **0.8** | 20 | 27 |
| HDDAF-noDefence | 0.930 | 5/6 | 1.0 | 22 | 28 |

HDDAF overall: accuracy 96.96%, F1 0.938, recall 90.02%, precision 0.980, about 1638 ms per chunk.

**Label poisoning:** F1 drops only 0.011 for HDDAF (0.938 to 0.927), versus 0.042 for HDDAF-noDefence. The KMeans + RONI defence keeps HDDAF the most robust scheme.

---

## Main Findings

- Static and Periodic models collapse after later drifts because they have never seen the new attacks.
- HDDAF performs comparably to PSI and KS on this stream, not clearly better.
- HDDAF has the highest recall (0.900) and the best coverage-versus-delay balance together with KS.
- The poisoning defence clearly improves robustness to label poisoning.
- Main cost: about 3 false alarms per true drift, and about 7 times the computation per chunk of the Static baseline.

---

## Limitations

- This is a re-implementation, not the authors' original code.
- One dataset (original CIC-IDS2017, which has known labeling errors) and six drift points.
- Poisoning is random and results vary between runs; averaging over several seeds is future work.
- The false-positive rate was not logged.
- Only simple label poisoning is tested, with no adaptive attacks. The GAN simulator is not evaluated.
- Timings vary on a laptop, so only the order of magnitude is reliable.

---

## How to Run

1. Download and unzip `DriftGuard.zip`.
2. Download the dataset using the link in `DataSet Link.txt` (or use `MachineLearningCSV.zip` from Releases) and place the CSV files where the code expects them.
3. Install the required Python packages (scikit-learn, LightGBM, imbalanced-learn, TensorFlow or PyTorch, matplotlib, etc.).
4. Run the experiments, then generate the plots:
   ```
   python src/run_experiments.py
   python src/plots.py
   ```

---

## Reference

A. Lara-Gutiérrez, C. Fernández-Gago, E. Onieva, "A framework for drift detection and adaptation in AI-driven anomaly and threat detection systems," Int. J. Inf. Secur., vol. 24, art. 199, 2025.
