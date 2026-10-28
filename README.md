# Artificial Intelligence and Machine Learning

Machine learning experiments on IoT security. The current notebook looks at how the
hyperparameters of a k-nearest neighbours (KNN) classifier affect a multi-class intrusion
detection system trained on the TON_IoT network dataset.

Notebook: [Exploring K-Nearest Neighbour Hyperparameters for Enhanced IoT Intrusion Detection](Exploring%20K-Nearest%20Neighbour%20Hyperparameters%20for%20Enhanced%20IoT%20Intrusion%20Detection.ipynb)
([open in Colab](https://colab.research.google.com/github/Muhammad-Nouman-s/Artificial-intelligence-and-Machine-Learning/blob/main/Exploring%20K-Nearest%20Neighbour%20Hyperparameters%20for%20Enhanced%20IoT%20Intrusion%20Detection.ipynb))

## Dataset

I used the network traffic file of the [TON_IoT datasets](https://research.unsw.edu.au/projects/toniot-datasets)
from UNSW Canberra (`ToN_IoT_Test_Network.csv`, 461,043 records, 45 features, 10 classes:
normal, backdoor, ddos, dos, injection, mitm, password, ransomware, scanning, xss).

The data is heavily imbalanced towards the normal class. Because of time and compute limits,
I kept 30,000 randomly chosen normal records and left the other classes as they are, which gives
191,043 records in total.

The dataset is not included here. Download it from the link above and put the CSV in a `data/`
folder, or change `DATA_PATH` in the notebook.

## Method

1. Drop identifier columns (`ts`, `src_ip`, `src_port`, `dst_ip`, `dst_port`) and label-encode
   the target and the categorical features.
2. Look at feature importance (decision tree and random forest) and the correlation matrix.
3. Balance the classes with ADASYN, then random under-sampling (29,368 samples per class).
4. Reduce to 20 components with PCA.
5. Split 70/30 into train and test (`random_state=42`).
6. Train four KNN models and compare accuracy, macro precision/recall, macro and micro F1,
   MCC and prediction time.

## Results

Test set results:

| Model | k | Weights | Algorithm | Leaf size | p | Accuracy | Macro F1 | MCC | Prediction time (test) |
|---|---:|---|---|---:|---:|---:|---:|---:|---:|
| KNN 1 | 3 | distance | auto | 30 | 2 | 0.9028 | 0.9030 | 0.8922 | 108 s |
| KNN 2 | 5 | uniform | kd_tree | 50 | 1 | 0.8973 | 0.8977 | 0.8861 | 25 s |
| KNN 3 | 9 | distance | ball_tree | 40 | 3 | 0.8874 | 0.8874 | 0.8752 | 200 s |
| KNN 4 | 11 | uniform | brute | 20 | 4 | 0.8381 | 0.8373 | 0.8206 | 8,733 s |

KNN 1 (k = 3, distance weighting, Euclidean) gave the best scores. KNN 2 (k = 5, Manhattan,
kd-tree) was only about half a point lower in accuracy but around four times faster at
prediction, so it is the better choice when inference time matters, for example on an IoT
gateway. Larger k and higher p made results worse, and the brute-force search with p = 4 was
by far the slowest (about 340 times slower than KNN 2).

## Limitations

- Resampling and PCA are done before the train/test split, so some synthetic samples can end up
  in the test set and the scores are probably a bit optimistic. Fitting these steps on the
  training data only (inside cross-validation) would give a fairer estimate.
- Features are label-encoded and not scaled before PCA and KNN. One-hot encoding and
  standardisation may change the results.
- KNN 4 takes a few hours to predict on the full test set. Skip it for a quick run.

## Running the notebook

```bash
pip install -r requirements.txt
jupyter notebook
```

The notebook was first written in Google Colab. It also runs locally once `DATA_PATH` points to
the CSV file.

## Reference

N. Moustafa, "A new distributed architecture for evaluating AI-based security systems at the
edge: Network TON_IoT datasets," Sustainable Cities and Society, vol. 72, 102994, 2021.

## Author

Muhammad Nouman, PhD researcher at the Telematics Lab, Politecnico di Bari.
[Website](https://muhammad-nouman-s.github.io/)

Code is released under the [MIT License](LICENSE).
