# NESL Golden_Fleece Team's Soluton to the NeurIPS 2020 Hide-and-seek Privacy Challenge 

This repository contains NESL's solution in the NeurIPS 2020 hide-and-seek privacy challenge. Our team is named Golden_Fleece and we are the runner-up in the challenge's hider track. 

## 1 Background of the Competition
### 1.1 Privacy Issue in Medical Data and the Hide and Seek Challenge




Conpetition website: 
https://www.vanderschaar-lab.com/privacy-challenge/
Competition Publications:

Scoreboard: https://docs.google.com/spreadsheets/d/1L_WSLckNBAdVb7HGIUVuqh2JjX_D0KlX/edit#gid=423478332
Challenge codebase:
https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/



### 1.2 The Goal of the Hiders



### 1.3 The Goal of the Seekers



### 1.4 Data Utility: Feature Prediction and One-step-ahead Prediction


## 2 How to Use These Code

The original codebase is provided by the organizers in the [Challenge Codebase](https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/). We generally kept the structure and added our solution code. For details of the original competition code, please refer to organizers/README.md.

### 2.1 Code Structure
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
### 2.2 How to Run the Code
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

## 3 Baselines

### 3.1 Hider: Addnoise

### 3.2 Hider: TimeGAN

### 3.3 Seeker: KNN

### 3.4 Seeker: Binary Classifier




## 4 Our Solutions

### 4.1 Hider: Adversarial Noise Generation
This method is inspired by [Fawkes](https://sandlab.cs.uchicago.edu/fawkes/#paper) by Shan S. et. al. The general idea is to add noise to the data, where the noise is trained in a adversarial way. A trained feature extraction neural network will generate a feature embedding for each data entry. A small perturbation in the raw data entry may cause its embedding to change drastically because of the inconsistency of neual network functions. The added noise is trained to "drag" the feature embedding of the current data entry towards that of a data entry from a different user so that their identity can be confused. 

The method can be visualized as follows:
![1.png](https://i.loli.net/2021/02/03/8U2y1ZzvDxCLOhE.png)

The alogrithm can be described in three steps:
Step1: Train a CNN-based feature extractor using a siamese way. The feature extractor works to recognize the user and maximize the embedding distances between different users

Step2: Define a CNN with exactly the same structure, except for that in the input layer we add a noise layer of the same dimension. Then we copy the trained CNN weights and fix them. The only learnable thing is the noise.

Step3: For each user (data entry $x_1$), select another user ($x_2$) whose embedding is very different. Then we train the noise to move the current embedding($x_1+noise$) towards embedding($x_2$). Then retrun ($x_1+noise_{opt}$) as the generated data. Repeat for all the users.

A sketch pseudo-code:
![Screenshot from 2021-02-01 19-32-55.png](https://i.loli.net/2021/02/03/Vz2Iu5q1PSmONW9.png)


### 4.2 Hider: Genetic Noise Adder
We apply the genetic algorithm to find synthetic data that meets the utility threshold and has maximal noise in this hider implementation.

For each user, we create several candidates for their synthetic data.  We then add noise to these candidates, and combine the best ones that meet the utility (feature and one-step-ahead) threshold.  This process is looped until either we reach some max number of iterations, or all of the candidates fail to meet the utility threshold (in which case the previous candidates that passed the utility threshold are used).

Persudo Code:
![Screenshot from 2021-02-02 15-02-44.png](https://i.loli.net/2021/02/03/2Ig9wil3FETA8kh.png)


### 4.3 Hider: Binning and Swapping
In this hider algorithm, we introduce noise to the data by grouping the datapoints in each feature dimension based on their cumulative distribution, and randomly switch the datapoints that falls into the same bin.

The following figure illustrates our solutions, we take one feature dimension as an example:

![2.png](https://i.loli.net/2021/02/03/hy1OPtTHsxRGAK7.png)

For each feature dimension (in this example, feature #32), we aggregate the datapoints from all the users. Then divide the data into a number of bins based on cumulative distribution percentage. For each data point (here by data point I mean one numerical number, I will refer to each user as “data entry”), it is randomly replaced with another value within the same bin. For example, the first value of $x_1$ is 0.23, which falls into bin 2 in the empirical cumulative distribution function plot. We randomly pick another value in the same pliot, 0.215, to replace 0.23.

Through this method, we swap the data values of different users and introduce some "quantization noise" to confuse the seekers.

Pesudo Code:
![Screenshot from 2021-02-02 15-00-12.png](https://i.loli.net/2021/02/03/dEp6jGbS1HkrJnD.png)


### 4.4 Seeker: TimeKNN
This is a modified version of the baseline KNN seeker (See Section 3.3 for details). Our insight is that, the sampling length and interval is discrepant across different users. We found that doing KNN classification on the time dimension only is actually better than doing it over all the feature dimensions.

Pesudo Code:
![Screenshot from 2021-02-02 14-51-29.png](https://i.loli.net/2021/02/03/JGE7phCi3vNW9kH.png)

## A Link to Other Team's Solutions
For reference, submitted codes from all the teams are available on the organizers' GitHub: https://github.com/vanderschaarlab/hide-and-seek-submissions. We put the links here for your reference.



