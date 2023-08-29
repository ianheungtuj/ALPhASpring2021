# PointNet

## Introduction

In this folder you'll find python and shell scripts to process the .npy files obtained from .txt files, as well as shell scripts for the training the PointNet model and getting confusion matrices and learning curves. [Note: This repository does contain instructions on how to convert .txt files to .npy files. Please check 3DCNN to convert .txt raw data to .npy array. Using `txt_to_npy_array_3D.py` from 3DCNN should give you the 5 different .npy files we need to continue working here]

## Folder Structures

There are three folders `src` containg all the slurmscrips, `Scripts` containing all the python files, and `Notebook` containing a jupyter notebook. 

### Scripts 

#### Data_Processing_for_PointNet.py

It loads the `.npy` data for all the five particles. Then uses time to create a separate third dimensional point for the events. For each and every single event it only saves the points where a particle was detected. As a result, the array becomes much smaller since a lot of points are eliminated. 

PointNet model can only work there are equal number of points per event. So, this script later only take a sample of N points from each events and eliminates the events containing less than N points. Later it labels saves the data to `./Data/PointNetData1` as `points.npy` containing all the events and `labels.npy` containing the labels for the events respectively. 

#### Train_test_splitting.py

The `Train_test_splitting.py` takes the .npy files saved by `Data_Processing_for_PointNet.py` and shuffles and splits the data into 80-20 proportions and saves the data for training and testing purposes in `./Data/TrainTestSplitData`

#### Model_build_and_fit.py

This loads the train and test data and converts them to tensorflow dataset for training the model. Then it build and trains the model using that dataset. As it trains the model, it saves the checkpoints in `./CheckPoints` and it saves the learning curve in `Learning-Curve-PointNet-Model`. After its done training it validates using the test dataset and saves the confusion matrix in `confusion-matrix-directory`.

### src
This folder contains the slurm scripts for allocating resources and running the python files 


#### Model_build_and_fit.sh

This shell script is used to run `Model_build_and_fit.py`. This is to be noted that `DataProcessFull.sh` must be run before this script at least the very first time as this uses split train-test points and label `.npy` arrays to train the which are output from  `DataProcessFull.sh`. 

#### DataProcessFull.sh 
This shell script is used to run `Data_Processing_for_PointNet.py` and then `Train_test_splitting.py`. This should output split train test files to be used to train and build the model. 

#### Data_Processing_for_PointNet.sh (Optional)
This shell script is used to run `Data_Processing_for_PointNet.py` if you just want to run that python file for any testing purposes. 

#### Train_test_splitting (Optional)
This shell script is used to run `Train_test_splitting.py` if you just want to run that python file for any testing purposes.

### Notebook

The Jupyter Notebook contains code related to this project but it was only for testing purposes and has not updated and it is not recommended to use the notebook's code or information. 

## Workflow

Please make sure you have data from `oldRawDataTxtFiles` and then you'll have to convert it to `3DArrays`. Please check the documentation and code in `3DCNN` on how to do that. 

Next you'll have to run `DataProcessFull.sh`. Please make sure the `.npy` files containing `3DArrays` are in the directory `./Data/PointNetData1/`. If you want to keep them in a differnt directory then you can change the address of the direction in `line 26` of `Data_Processing_for_PointNet.py`. After it runs you should be able to see train and test files saved in `./Data/TrainTestSplitData/`. 

Then you can run `Model_build_and_fit.sh` for training the model. As the model trains you'll find checkpoints saved in `./CheckPoints`. Alongside it will also save the loss curve and confusion matrix in `./Learning-Curve-PointNet-Model/` and `./confusion-matrix-directory/` directories respectively as your model ends training. Based on the loss curve and confusion matrix you should be able to make judgements on whether the model needs further training or parameter tuning. 


