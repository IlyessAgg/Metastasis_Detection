# Metastasis_Detection
> Challenge offered by Owkin : Detecting breast cancer metastases.

![owkin](imgs/header.jpg)

_You can find the original description of the challenge [here](https://challengedata.ens.fr/challenges/18). Below is a summary of it._
## Challenge
[Owkin](https://owkin.com/) is a French-American startup that deploys AI and Federated Learning for medical research. Their mission is to improve patient treatment, drug discovery and augment medical research by developing machine-learning models on real-world clinical data.

The goal of the challenge they offer is to develop new algorithms to detect **lymph node metastases** in histological images of patients diagnosed with breast cancer. 

### Clinical Context

Metastasis is the **spread of cancer cells** from the place where they first formed (here breast) to another part of the body. For patients diagnosed with breast cancer, even after finding the primary tumor, an examination of the regional lymph nodes in the axilla is always performed to determine if the breast cancer ***metastasized*** (i.e. if it spread through the lymphatic system).

Metastasis lymph nodes involvement is a criterion for grading breast cancer, and is also a very important prognostic factor for breast cancer survival. Prognosis is poorer when cancer has spread to lymph nodes.

### Purpose

Assessing the presence of metastases in histology images is a demanding, time-consuming task for pathologists, which requires extensive study under the microscope. Highly trained pathologists must review vast whole-slide images of very high digital resolution (100,000 * 100,000 pixels). Identifying small metastases is a tedious exercise and some might be missed.

An automated solution to detect metastases has high clinical relevance as it could help to reduce the workload of pathologists and improve the reliable detection of metastases.

### Goal

The challenge proposed by Owkin is a **weakly-supervised binary classification problem** : predict whether a patient has any metastases in its lymph node or not, given its slide.

### Data

To each patient corresponds one slide (*from the Camelyon dataset*). However, due to the extremely large dimensions of the whole-slide images, the data had to be preprocessed : for each slide, smaller images (tiles) of size 896x896 (then resized at 224x224) were extracted for a total of **1,000 tiles per patient** (or less if there is not enough tissue). 

Each patient is thus represented by **1,000 images** (or less). **ResNet** features were also extracted for each of these images, providing another input alternative.

### Distribution

The training data is composed of **279** patients, among which **112** have metastases.

Furthermore, for some specific cases a pathologist could have **locally annotated the slide**, which provides a tile-level labeling and therefore allows a strongly supervised approach. Among the training patients, **11** are fully annotated (with a total of **10,024** annotated tiles).

### Benchmark

The metric for the challenge is the **Area Under the Curve** (AUC) .

The baseline model is a logistic regression on the mean of the ResNet features. This model reaches an AUC of **0.7** on the public test set.

## Approach

My approach was the following :

- Use the annotated patients to create a model (***tile_predictor***) that would predict if a **tile** has any metastases. 

  *(Input : Tile. — Output : Probability of metastases in tile.)*

- Make predictions on every tile of each non-annotated patient and create a model (***patient_predictor***) that would predict if a **patient** has any metastases. 

  *(Input : Predictions of all the tiles. — Output : Probability of metastases in patient.)*

At first, I tried my approach directly on the images but I wasted a lot of time trying to make it work on Google Colab and eventually ran into some complications, so I used the ResNet features instead. 

The model that performed the best was a bagging of logistic regression classifiers for both models (*tile_predictor* and *patient_predictor*). It achieved **0.8206** AUC on the public test set.

#### Improvements

Several improvements can be made :

- For the ***tile_predictor***, fix my attempt at a CNN so that we can apply our approach directly on images.
- For the ***patient_predictor*** :
  - Using a simple neural network on the probabilities.
  - Improving feature engineering (adding new features, making use of the x and y positions to find tile neighbors etc...)