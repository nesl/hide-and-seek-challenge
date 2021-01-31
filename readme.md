# NESL Golden_Fleece team's solution to the NeurIPS 2020 hide-and-seek privacy challenge

Clinical time series, combined with ML, can enable powerful inferences about the health of individuals or larger populations of users.  However, healthcare datasets often contain sensitive information about its patients.  Thus, one approach to providing this clinical time series data is via synthetic generation, where the goal is to preserve the utility of the time series data with respect to certain inferences, yet ensure privacy.  The formulation of utility and privacy will be defined later.


### Challenge Overview

In this Hide-and-Seek Privacy Challenge, several teams compete against each other in two tracks: hiders and seekers.  Teams that are joining as hiders must generate synthetic data for a subset of patients from the AmsterdamUMCdb dataset.  This synthetic data must be able to meet a minimal threshold on several utility metrics.  On the other hand, teams joining as seekers must try to break the privacy of the synthesized data from the hiders, by performing a membership inference attack (discussed later).

### Utility and Privacy

In this competition, the goal is to generate synthetic clinical time series from the original time series.  In this document, we use privacy-preserving data and synthetic data interchangeably.

Data, such as clinical time series, can be transformed to ensure privacy for users participating in the original dataset.  However, the resulting privacy-preserving data is always subject to a utility-privacy tradeoff, which balances between the usefulness of privacy-preserving data (i.e. how useful it is at predicting heart disease compared to the original data), as well as the privacy characteristics (i.e. preventing re-identification of users).  In this competition, utility is measured using two metrics.  

The first metric is feature prediction.  A feature predictor NN is trained to predict a missing feature from the remaining present features in the original dataset. Another feature predictor is then trained to predict the same missing features using the privacy-preserving data.  Assuming there is some held-back testing data (i.e. original testing data), we can check the performance between these two feature predictors on the testing data.  The idea is that if the privacy-preserving data retains the utility of the original data, then both feature predictors should perform comparably. 

The second metric is 1-step-ahead prediction, where a NN is trained to predict the value of all features at the last time step in the clinical time series.  Like the feature prediction, two networks are trained - one for the original data, and another for the privacy-preserving data.  Both are evaluated on some testing set (of the original data), with the hopes that the NN trained on privacy-preserving data performs comparably to the NN trained on the original training data.

On the other hand, this competition formulates privacy as vulnerability to membership inference attacks.  In a broad sense, the goal of membership inference attacks is to infer whether or not a given data point was part of a given dataset.  In this competition, an seeker has data from multiple patients (i.e. call them A,B,C,D).  However, privacy-preserving data is generated from some subset of patients (i.e. B,C) - the seekers are assumed to have more data than than the hiders.  If seekers can determine which of the original patients (A,B,C,D) correspond to the privacy-preserving data (i.e. B,C), they have successfully performed the membership inference attack.  The goal of hiders is to generate synthetic data that prevents the success of such attacks.

### Summary of goals

Hiders must generate synthetic data from a subset of randomly chosen patients in the AmsterdamUMCdb.  The synthetic data must meet a minimal threshold on both feature prediction, as well as one-step ahead prediction.  In addition, it must minimize the success of membership inference attacks.  

Seekers must perform membership inference attacks on the synthesized data from the hiders.  Seekers have access to N users' clinical time series data.  Of this data, hiders are given N/2 users' data in order to generate their synthetic data.  Seekers are not aware of which N/2 users were given to the hiders.  Thus, seekers must determine which of the N/2 users from the original data were used to generate the synthetic data.  In other words, this is a binary classification - either each user was 'originating member' (used to create the synthetic data) or a 'non member' (not used to create the synthetic data).

### Additional Challenge Resources

Original paper proposal for this challenge: https://arxiv.org/pdf/2007.12087.pdf 

Overview of the challenge: https://www.vanderschaar-lab.com/privacy-challenge/

Repo for the competition evaluation code: https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/

### Baselines used in the competition

The codebase includes baselines for both hiders and seekers.

#### Baseline Hiders

##### Add Noise

Adds Gaussian noise to every value in the dataset with zero mean and a standard deviation as a hyperparameter.

##### TimeGAN

As an extremely high level overview, this method uses Generative Adversarial Networks (GANs) to create synthetic data.  This work seeks to generate realistic time series data to capture both temporal dynamics of the time series, as well as the ability to generate new sequences of synthetic data.  Formally, they seek to model two distributions - a global joint distribution of features which represents how 'realistic' the time series is, as well as a local conditional distribution of features over time (meant to capture the temporal dynamics between datapoints in a sequence).

Reference: Yoon, Jinsung, Daniel Jarrett, and Mihaela van der Schaar. "Time-series generative adversarial networks." (2019).


#### Baseline Seekers

##### Nearest Neighbor

Since seekers are given both the original data (2N users) and the synthetic data (N users), this approach computes the distance between each user's time series in the original data to each user in the synthetic data.  The closest N users in the original data are selected and predicted as 'originating members' (used to create the synthetic data).

##### Binary Predictor

Similar to the Nearest Neighbor method, this approach trains a binary classifier using both the original data and synthetic data.  All patient data in the original dataset are labelled as 1, and all patient data in the synthetic data are marked as 0.  Finally, to determine the 'originating members', they perform inference on the original dataset again (contains 2N users), and pick the top N lowest predictions as the 'originating members' (used to create the synthetic data).


### Our Solutions

##### Genetic Approach

Overview: apply the genetic algorithm to find synthetic data that meets the utility threshold and has maximal noise

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

