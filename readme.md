# NESL Golden_Fleece Team's Soluton to the NeurIPS 2020 Hide-and-seek Privacy Challenge 

This repository contains NESL's solution in the NeurIPS 2020 hide-and-seek privacy challenge. Our team is named Golden_Fleece and we are the runner-up in the challenge's hider track. 

## Background of the Competition
### Privacy Issue in Medical Data and the Hide and Seek Challenge




Conpetition website: 
https://www.vanderschaar-lab.com/privacy-challenge/
Competition Publications:

Scoreboard: https://docs.google.com/spreadsheets/d/1L_WSLckNBAdVb7HGIUVuqh2JjX_D0KlX/edit#gid=423478332
Challenge codebase:
https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/



### The Goal of the Hiders



### The Goal of the Seekers



### Data Utility: Feature Prediction and One-step-ahead Prediction


## How to Use These Code

The original codebase is provided by the organizers in the [Challenge Codebase](https://bitbucket.org/mvdschaar/mlforhealthlabpub/src/master/app/hide-and-seek/). We generally kept the structure and added our solution code. For details of the original competition code, please refer to organizers/README.md.

### Code Structure
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
### How to Run the Code
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

## Baselines

### Hider: Addnoise

### Hider: TimeGAN

### Seeker: KNN

### Seeker: Binary Classifier




## Our Solutions

### Hider: Adversarial Noise Generation
This method is inspired by [Fawkes](https://sandlab.cs.uchicago.edu/fawkes/#paper) by Shan S. et. al. The general idea is to add noise to the data, where the noise is trained in a adversarial way. A trained feature extraction neural network will generate a feature embedding for each data entry. A small perturbation in the raw data entry may cause its embedding to change drastically because of the inconsistency of neual network functions. The added noise is trained to "drag" the feature embedding of the current data entry towards that of a data entry from a different user so that their identity can be confused. 

The method can be visualized as follows:

The alogrithm can be described in three steps:
Step1: Train a CNN-based feature extractor using a siamese way. The feature extractor works to recognize the user and maximize the embedding distances between different users

Step2: Define a CNN with exactly the same structure, except for that in the input layer we add a noise layer of the same dimension. Then we copy the trained CNN weights and fix them. The only learnable thing is the noise.

Step3: For each user (data entry $x_1$), select another user ($x_2$) whose embedding is very different. Then we train the noise to move the current embedding($x_1+noise$) towards embedding($x_2$). Then retrun ($x_1+noise_{opt}$) as the generated data. Repeat for all the users.

A sketch pseudo-code:



### Hider: Genetic Noise Adder

### Hider: Binning and Swapping

### Seeker: TimeKNN


## A Link to Other Team's Solutions
For reference, submitted codes from all the teams are available on the organizers' GitHub: https://github.com/vanderschaarlab/hide-and-seek-submissions. We put the links here for your reference.



