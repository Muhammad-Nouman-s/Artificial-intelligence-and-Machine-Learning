# Artificial Intelligence and Machine Learning

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Muhammad-Nouman-s/Artificial-intelligence-and-Machine-Learning/blob/main/Exploring%20K-Nearest%20Neighbour%20Hyperparameters%20for%20Enhances%20IoT%20Intrusion%20Detection.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Machine-learning experiments for **IoT security**. The current project studies how the
hyperparameters of the **k-nearest neighbours (KNN)** classifier affect a multi-class
**intrusion detection system (IDS)** trained on the **TON_IoT** network dataset.

---

## Project: KNN Hyperparameters for IoT Intrusion Detection

**Notebook:** [`Exploring K-Nearest Neighbour Hyperparameters for Enhances IoT Intrusion Detection.ipynb`](Exploring%20K-Nearest%20Neighbour%20Hyperparameters%20for%20Enhances%20IoT%20Intrusion%20Detection.ipynb)

### Objective

Quantify the trade-off between detection quality and computational cost when changing the
number of neighbours `k`, the weighting scheme, the neighbour-search algorithm, the leaf size
and the Minkowski distance order `p`.

### Dataset

| Item | Details |
|---|---|
| Source | [TON_IoT datasets, UNSW Canberra](https://research.unsw.edu.au/projects/toniot-datasets) — network traffic subset (`ToN_IoT_Test_Network.csv`) |
| Raw size | 461,043 records × 45 features |
| Classes (10) | `normal`, `backdoor`, `ddos`, `dos`, `injection`, `mitm`, `password`, `ransomware`, `scanning`, `xss` |
| Customisation | The `normal` class is randomly down-sampled to 30,000 records (other classes unchanged) → 191,043 records |

> The dataset is **not** included in this repository. Download it from the official TON_IoT
> page and place `ToN_IoT_Test_Network.csv` in the working directory (or your Google Drive).

### Pipeline

1. **Pre-processing** – drop identifier columns (`ts`, `src_ip`, `src_port`, `dst_ip`, `dst_port`),
   label-encode the target and categorical features.
2. **Feature analysis** – decision-tree and random-forest feature importance
   (`SelectFromModel`), correlation heat-map.
3. **Class balancing** – ADASYN over-sampling followed by random under-sampling
   (29,368 samples per class).
4. **Dimensionality reduction** – PCA to 20 components.
5. **Split** – 70 % train / 30 % test (`random_state=42`).
6. **Modelling** – four KNN configurations, evaluated with accuracy, macro precision,
   macro recall, macro/micro F1, Matthews correlation coefficient (MCC) and prediction time.

### Results (test set)

| Model | k | Weights | Algorithm | Leaf size | p | Accuracy | Macro F1 | MCC | Test prediction time |
|---|---:|---|---|---:|---:|---:|---:|---:|---:|
| **KNN-1** | 3 | distance | auto | 30 | 2 (Euclidean) | **0.9028** | **0.9030** | **0.8922** | 108 s |
| KNN-2 | 5 | uniform | kd_tree | 50 | 1 (Manhattan) | 0.8973 | 0.8977 | 0.8861 | **25 s** |
| KNN-3 | 9 | distance | ball_tree | 40 | 3 | 0.8874 | 0.8874 | 0.8752 | 200 s |
| KNN-4 | 11 | uniform | brute | 20 | 4 | 0.8381 | 0.8373 | 0.8206 | 8,733 s |

### Key findings

- **Small `k` with distance weighting (KNN-1) gives the best detection quality** (≈ 90.3 % accuracy, MCC 0.89).
- **KNN-2 (Manhattan distance, kd-tree) is the best speed/accuracy trade-off:** only 0.5 pp
  lower accuracy than KNN-1 but ≈ 4× faster at inference — attractive for resource-constrained IoT gateways.
- Increasing `k` and the Minkowski order `p` steadily reduces accuracy; the brute-force search
  with `p = 4` is ~80× slower than the kd-tree configuration, showing that the search
  algorithm and distance metric dominate inference cost.

### Limitations and future work

- Resampling (ADASYN / under-sampling) and PCA are applied **before** the train/test split, so
  synthetic samples can leak into the test set and the reported scores may be optimistic.
  A stricter protocol would fit resampling and PCA on the training fold only
  (e.g. an `imblearn.pipeline.Pipeline` inside stratified k-fold cross-validation).
- Categorical features are label-encoded; one-hot or target encoding may suit distance-based models better.
- Features are not standardised before PCA/KNN; adding `StandardScaler` is expected to change the ranking.
- Planned: systematic grid / Bayesian search over `k` and `p`, per-class confusion matrices,
  and comparison with tree ensembles and neural models.

---

## Getting started

```bash
git clone https://github.com/Muhammad-Nouman-s/Artificial-intelligence-and-Machine-Learning.git
cd Artificial-intelligence-and-Machine-Learning
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook
```

The notebook was developed on **Google Colab** (it mounts Google Drive in the first cells).
To run locally, skip the Drive/`%cd` cells and point `pd.read_csv(...)` at your local copy of the dataset.

> ⚠️ KNN-4 (`algorithm='brute'`, `p=4`) takes several hours to predict on the full split;
> skip it or reduce the test size for a quick run.

## Repository structure

```
.
├── Exploring K-Nearest Neighbour Hyperparameters for Enhances IoT Intrusion Detection.ipynb
├── requirements.txt
├── LICENSE
└── README.md
```

## Citation

If you use this work, please cite the TON_IoT dataset:

> N. Moustafa, “A new distributed architecture for evaluating AI-based security systems at the
> edge: Network TON_IoT datasets,” *Sustainable Cities and Society*, vol. 72, 102994, 2021.

## Author

**Muhammad Nouman** — PhD Researcher, Telematics Lab, Politecnico di Bari
[Website](https://muhammad-nouman-s.github.io/) ·
[Google Scholar](https://scholar.google.com/citations?user=ebfBz34AAAAJ) ·
[ORCID](https://orcid.org/0000-0003-4233-5506) ·
[LinkedIn](https://www.linkedin.com/in/muhammadnoumanml)

## License

Code is released under the [MIT License](LICENSE). The TON_IoT dataset is subject to its own licence terms.
