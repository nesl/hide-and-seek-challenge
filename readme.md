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
│   ├── hider_*.py                  # Hider solution module, containing the hider function.
│   ├── seeker_*.py                 # Seeker solution module, containing the seeker function.
│   ├── main.py                     # Script for testing solutions locally.
│   ├── make_hider.sh               # Helper bash script for zipping up a hider solution. Not needed if you run the coded locally.
│   ├── make_seeker.sh              # Helper bash script for zipping up a seeker solution. Not needed if you run the coded locally.
├── readme.md                       # This README.
└── requirements.txt                # The requirements file for local debugging.
```
### How to Run the Code

