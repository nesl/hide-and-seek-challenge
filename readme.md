# NESL Golden_Fleece Team's Solutons to the NeurIPS 2020 Hide-and-seek Privacy Challenge 

This repository contains NESL's solution in the NeurIPS 2020 hide-and-seek privacy challenge. Our team is named Golden_Fleece and we are the runner-up in the challenge's hider track. 

## 1. Background of the Competition
### 1.1. Privacy Issue in Medical Data and the Hide and Seek Challenge

Clinical time series, combined with ML, can enable powerful inferences about the health of individuals or larger populations of users.  However, healthcare datasets often contain sensitive information about its patients.  Thus, one approach to providing this clinical time series data is via synthetic generation, where the goal is to preserve the utility of the time series data with respect to certain inferences, yet ensure privacy.  The formulation of utility and privacy will be defined later.

In this Hide-and-Seek Privacy Challenge, several teams compete against each other in two tracks: hiders and seekers.  Teams that are joining as hiders must generate synthetic data for a subset of patients from the AmsterdamUMCdb dataset.  This synthetic data must be able to meet a minimal threshold on several utility metrics.  On the other hand, teams joining as seekers must try to break the privacy of the synthesized data from the hiders, by performing a membership inference attack (discussed later).

### 1.2. Utility and Privacy

In this competition, the goal is to generate synthetic clinical time series from the original time series.  In this document, we use privacy-preserving data and synthetic data interchangeably.

Data, such as clinical time series, can be transformed to ensure privacy for users participating in the original dataset.  However, the resulting privacy-preserving data is always subject to a utility-privacy tradeoff, which balances between the usefulness of privacy-preserving data (i.e. how useful it is at predicting heart disease compared to the original data), as well as the privacy characteristics (i.e. preventing re-identification of users).  In this competition, utility is measured using two metrics.  

The first metric is feature prediction.  A feature predictor NN is trained to predict a missing feature from the remaining present features in the original dataset. Another feature predictor is then trained to predict the same missing features using the privacy-preserving data.  Assuming there is some held-back testing data (i.e. original testing data), we can check the performance between these two feature predictors on the testing data.  The idea is that if the privacy-preserving data retains the utility of the original data, then both feature predictors should perform comparably. 

The second metric is 1-step-ahead prediction, where a NN is trained to predict the value of all features at the last time step in the clinical time series.  Like the feature prediction, two networks are trained - one for the original data, and another for the privacy-preserving data.  Both are evaluated on some testing set (of the original data), with the hopes that the NN trained on privacy-preserving data performs comparably to the NN trained on the original training data.

On the other hand, this competition formulates privacy as vulnerability to membership inference attacks.  In a broad sense, the goal of membership inference attacks is to infer whether or not a given data point was part of a given dataset.  In this competition, an seeker has data from multiple patients (i.e. call them A,B,C,D).  However, privacy-preserving data is generated from some subset of patients (i.e. B,C) - the seekers are assumed to have more data than than the hiders.  If seekers can determine which of the original patients (A,B,C,D) correspond to the privacy-preserving data (i.e. B,C), they have successfully performed the membership inference attack.  The goal of hiders is to generate synthetic data that prevents the success of such attacks.

### 1.3. Summary of goals
There are two roles (types of algorithms) in this challenge, namely the **hider** and the **seeker**

Hiders must generate synthetic data from a subset of randomly chosen patients in the AmsterdamUMCdb.  The synthetic data must meet a minimal threshold on both feature prediction, as well as one-step ahead prediction.  In addition, it must minimize the success of membership inference attacks.  

Seekers must perform membership inference attacks on the synthesized data from the hiders.  Seekers have access to N users' clinical time series data.  Of this data, hiders are given N/2 users' data in order to generate their synthetic data.  Seekers are not aware of which N/2 users were given to the hiders.  Thus, seekers must determine which of the N/2 users from the original data were used to generate the synthetic data.  In other words, this is a binary classification - either each user was 'originating member' (used to create the synthetic data) or a 'non member' (not used to create the synthetic data).


### 1.4. Additional Challenge Resources

Original paper proposal for this challenge: 
https://arxiv.org/pdf/2007.12087.pdf 

Overview of the challenge: 
https://www.vanderschaar-lab.com/privacy-challenge/

Scoreboard: https://docs.google.com/spreadsheets/d/1L_WSLckNBAdVb7HGIUVuqh2JjX_D0KlX/edit#gid=423478332

Challenge codebase:
https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/



## 2. How to Use These Code

The original codebase is provided by the organizers in the [Challenge Codebase](https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/). We generally kept the structure and added our solution code. For details of the original competition code, please refer to organizers/README.md.

### 2.1. Code Structure
```
.
├── data/                           # Location to put the data file(s) in.
├── dockerfiles/                    # Dockerfile examples and a helper bash script for building docker containers. Not needed if you run the coded locally.
├── examples/                       # Solutions code (including baseline solutions).
│   │── hider/                      # Example hiders.
│	│   ├── Each folder contains the supporting code and documents of one solution
│   └── seeker/                     # Example seekers.
│	│   ├── Each folder contains the supporting code and documents of one solution
├── orgainzers/                     # Documentations from the challenge organizers.
├── utils/                          # Utilities code folder, can be used by code imported in solutions.
├── hider_*.py                  # Hider solution module, containing the hider function.
├── seeker_*.py                 # Seeker solution module, containing the seeker function.
├── main.py                     # Script for testing solutions locally.
├── make_hider.sh               # Helper bash script for zipping up a hider solution. Not needed if you run the coded locally.
├── make_seeker.sh              # Helper bash script for zipping up a seeker solution. Not needed if you run the coded locally.
├── readme.md                       # This README.
└── requirements.txt                # The requirements file for local debugging.
```
### 2.2. How to Run the Code
(1) Envionment: Python>=3.6, Python package requirements are specified in requirements.txt. A virtual python envionment is preferred.

(2) Rename the files: In the root directory, among hider_*.py  and seeker_*.py, select out the hider method and the seeker method you want to use. Make a **copy** of the corresponding file in the root directory, and **rename** the copy as hider.<nolink>py and seeker.<nolink>py seperately. The two modules will then be called by main<nolink>.py.

(3) Run main.<nolink>py
```
python main.py
```
For full details on the arguments of this script (there are a few knobs we can tune for training and evaluation), run:
```
python main.py --help
```

## 3. Baselines

### 3.1. Hider: Addnoise
Adds Gaussian noise to every value in the dataset with zero mean and a standard deviation as a hyperparameter. The added noise are all i.i.d, and the standard deviation needed to be specified as input.

### 3.2. Hider: TimeGAN
As an extremely high level overview, this method uses Generative Adversarial Networks (GANs) to create synthetic data.  The general idea is to train a Time-series Generative Adversarial Network on the orignal data, and generate a synthetic data of the same size.

The Time-GAN baseline is explained in full in [Yoon et al. [2019]](https://www.damtp.cam.ac.uk/user/dkj25/pdf/yoon2019time.pdf). This work seeks to generate realistic time series data to capture both temporal dynamics of the time series, as well as the ability to generate new sequences of synthetic data.  Formally, they seek to model two distributions - a global joint distribution of features which represents how 'realistic' the time series is, as well as a local conditional distribution of features over time (meant to capture the temporal dynamics between datapoints in a sequence).


### 3.3 Seeker: KNN
To explain the baseline algorithms, we have to introduce some notations. 

Since seekers are given both the original data (2N users) and the synthetic data (N users), this approach computes the distance between each user's time series in the original data to each user in the synthetic data.  The closest N users in the original data are selected and predicted as 'originating members' (used to create the synthetic data).
### 3.4 Seeker: Binary Classifier

Similar to the Nearest Neighbor method, this approach trains a binary classifier using both the original data and synthetic data.  All patient data in the original dataset are labelled as 1, and all patient data in the synthetic data are marked as 0.  Finally, to determine the 'originating members', they perform inference on the original dataset again (contains 2N users), and pick the top N lowest predictions as the 'originating members' (used to create the synthetic data).


## 4 Our Solutions

### 4.1 Hider: Adversarial Noise Generation
This method is inspired by [Fawkes by Shan S. et. al. [2020]](https://sandlab.cs.uchicago.edu/fawkes/#paper) . The general idea is to add noise to the data, where the noise is trained in a adversarial way. A trained feature extraction neural network will generate a feature embedding for each data entry. A small perturbation in the raw data entry may cause its embedding to change drastically because of the inconsistency of neual network functions. The added noise is trained to "drag" the feature embedding of the current data entry towards that of a data entry from a different user so that their identity can be confused. 

The method can be visualized as follows:
![1.png](https://i.loli.net/2021/02/03/8U2y1ZzvDxCLOhE.png)

The alogrithm can be described in three steps:
Step1: Train a CNN-based feature extractor using a siamese way. The feature extractor works to recognize the user and maximize the embedding distances between different users

Step2: Define a CNN with exactly the same structure, except for that in the input layer we add a noise layer of the same dimension. Then we copy the trained CNN weights and fix them. The only learnable thing is the noise.

Step3: For each user (data entry x1), select another user (x3) whose embedding is very different. Then we train the noise to move the current embedding f(x1+noise) towards embedding f(x3). Then retrun (x1+noise_opt) as the generated data. Repeat for all the users.

A sketch pseudo-code:
![Screenshot from 2021-02-01 19-32-55.png](https://i.loli.net/2021/02/03/Vz2Iu5q1PSmONW9.png)


### 4.2 Hider: Genetic Noise Adder
We apply the genetic algorithm to find synthetic data that meets the utility threshold and has maximal noise in this hider implementation.


Details:
We first train several models for checking the utility of our synthesized data (used to measure if we can pass the utility threshold).  Then, for each user, we create synthetic data that passes our utility check via the previously trained models, and has maximum noise added.  More specifically, we run the genetic algorithm for a particular user to determined the best synthesized data for this user (i.e. passes the utility thresholds and has a large amount of noise added.)
The genetic algorithm requires several steps:

1. Create the initial population of synthesize data

2. Check if this initial population passes the fitness test (passes utility threshold)

3. Then loop:

	- Crossover candidates (this creates 'children' candidates)

	- Mutate all candidates

	- Check fitness again

4. Until we reach some maximum number of generations (or all candidates fail the fitness test.)

Persudo Code:
![Screenshot from 2021-02-02 15-02-44.png](https://i.loli.net/2021/02/03/2Ig9wil3FETA8kh.png)


### 4.3 Hider: Binning and Swapping
In this hider algorithm, we introduce noise to the data by grouping the datapoints in each feature dimension based on their cumulative distribution, and randomly switch the datapoints that falls into the same bin.

The following figure illustrates our solutions, we take one feature dimension as an example:

![2.png](https://i.loli.net/2021/02/03/hy1OPtTHsxRGAK7.png)

For each feature dimension (in this example, feature #32), we aggregate the datapoints from all the users. Then divide the data into a number of bins based on cumulative distribution percentage. For each data point (here by data point I mean one numerical number, I will refer to each user as “data entry”), it is randomly replaced with another value within the same bin. For example, the first value of $x_1$ is 0.23, which falls into bin 2 in the empirical cumulative distribution function plot. We randomly pick another value in the same pliot, 0.215, to replace 0.23.

Through this method, we swap the data values of different users and introduce some "quantization noise" to confuse the seekers.

Pesudo Code:
![Screenshot from 2021-02-02 15-00-12.png](https://i.loli.net/2021/02/03/Va3bTNrYRqxHgvS.png)


### 4.4. Seeker: TimeKNN
This is a modified version of the baseline KNN seeker (See Section 3.3 for details). Our insight is that, the sampling length and interval is discrepant across different users. We found that doing KNN classification on the time dimension only is actually better than doing it over all the feature dimensions.

Pesudo Code:
![Screenshot from 2021-02-02 14-51-29.png](https://i.loli.net/2021/02/03/JGE7phCi3vNW9kH.png)

## 5. A Link to Other Team's Solutions
For reference, submitted codes from all the teams are available on the organizers' GitHub: https://github.com/vanderschaarlab/hide-and-seek-submissions. We put the links here for your reference.

