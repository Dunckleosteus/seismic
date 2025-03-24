# Intro
The goal of this project is to provide a 2D synthetic seismic panel from velocity and density data on a carbonate outcrop. 
# Data 
An excel file split into 4 sheets: 
- X
- Y $\rightarrow$ z axis
- VP
- RHOB
# Code 
## Python import
The choice was made to run the code on google collab and store version on github to allow for collaboration and versioning of code. Keeping that in mind, the excel sheel was saved to 4 different csv files stored within the seismic zip file. 
To install the files directly in the current notebook from collab, we use the following bash commands : 
```shell
rm data.zip
!rm -r data
!wget https://github.com/Dunckleosteus/seismic/raw/18908e9e23b56529c5744c96fac2a5216cd298cd/data.zip
!unzip data.zip
```
The unzipped file will then be available @ path: `data/*.csv` 
