# 3D CNN
## Introduction
In this folder, you will find python and shell scripts relating to reorganizing the .txt files into usable formats that can be readable for 3D Neural Network input, as well as shell scripts for the training of a 3D CNN model and getting confusion matrices.

## What Each File Does
### src
#### txt_to_numpy_array_3D.sh
This shell script uses two python scripts, `txt_to_npy_array_3D.py` and `npy_array_to_3Ddata.py`. `txt_to_npy_array_3D.py` is used to convert each of the five particle classes into readable numpy formats, while `npy_array_to_3Ddata.py` sums up all those particles into one big array, and creates two sets of data that can be used for general neural network use. Keep in mind that you can change the format of the data created with your own scripts to fit the type of data format you need for your respective model needs.

##### txt_to_npy_array_3D.py
Converts `.txt` files into readable numpy arrays. The shell script calls on a python function, `txt_to_npy_array_3D.py`, five times for each of the particle classes. If you have seen the code in 2DCNN, this python script basically combines the functions of `txt_to_npy_array.py` and `sum_pads_arrays.py`. The key difference between 2D and 3D data processing of the data is that the time data from the `.txt files` is not used. Since we want 3D data, we turn the time data into the third dimension. There are instances where there are multiple charges detected for one detector, which results in tuples being produced. The python script averages out the time data across one detector, and sums up the charge data. The data created is stored as five numpy arrays corresponding to each particle in the folder `3DsmallDataParticleChargeArrays`.

##### npy_array_to_3Ddata.py
The data created from `txt_to_npy_array_3D.py` is not particularly useful, since it is not really formatted in an optimized or useful format, so we change that by converting the original numpy arrays into two different types of data formats. 

The first format is a general multi-purpose format that can be used as inputs, or for further processing. It contains information on all the detectors, so there are parts of the array where there are no particle detections. The data is saved as 5 separate arrays for each of the particle classes. This data is stored in the folder `3DArrays`. 

The other format of the data contains only the locations of detected particles, and as such takes up less storage space since it only has detected particle events in the arrays. This set of data can be primarily used for plotting, or for models like the PointNet model. The data is saved as 5 separate arrays for each of the particle classes. The data is stored in the folder `3DplottingArrays`.

#### dataproccessingfor3DCNN.sh
Runs 2 python files, `npy_array_to_3Dcnndata.py`, and `train_test_split.py`. The python files read in the numpy arrays processed in `txt_to_numpy_array_3D.sh`, and turn them into 3D CNN data to be used for a 3D CNN model. Keep in mind that the data that is being loaded and processed in this script is from `txt_to_numpy_array_3D.sh`, so you have to run that shell script first before running this one. The script converts the numpy arrays into data that can be directed and loaded into a 3D CNN model. You may want to rerun this script if you want to change some of the train-test split properties.

##### npy_array_to_3Dcnndata.py
The data from the folder `3DArrays` is processed to a format readable by a 3D CNN model. The python script takes the five separate particle arrays in the original `3DArrays` folder and creates two arrays of data, one containing the input events, which contains data on the particle track, and also creates an array that has labels corresponding to an event’s respective particle. These two arrays will be created in the folder `3DinputCNNArrays`, and named `inputs.npz` and `labels.npy`. 

It is important to note that the inputs are a sparse array, which is a more optimized way to store data, as keeping the data as a numpy array would result in the data being around 70 GB. A sparse array is more condensed and stores the inputs more efficiently. Note the label array is kept as a numpy array, because it does not constrain too much data, so we keep it as an numpy array.

##### train_test_split.py
The python script `train_test_split.py` is ONLY to be used on performing a train-validation-test split on the 3D CNN data, and cannot be used to perform train-test splits on the other data formats in other folders. To create a train-test split for a data type that is not 3D CNN, you can adapt the code from this python script, but you will not be able to fully copy and paste it without making your own changes.

`train_test_split.py` performs a train-test split of 80-20 on the input and labels data. Before the split, the inputs and labels arrays are randomized in synchronization, and then a train test split is performed.

Normally, a validation split will be done by TensorFlow when fitting the data to a model, but because TensorFlow cannot perform train-validation splits on sparse arrays, we perform our own validation split in this script. A train-validation split of 80-20 is performed on the train data.

6 arrays will be produced from this python script,  `train_inputs.npz`, `validation_inputs.npz`, `test_inputs.npz`, `train_labels.npy`, `validation_labels.npy`, and `test_labels.npy`.

#### model_fit_3D.sh and model_fit_3D.py
The `model_fit_3D.sh` script only runs one python script, `model_fit_3D.py`, which loads in the train and validation inputs and labels that were generated from the `dataproccessingfor3DCNN.sh` script. `model_fit_3D.sh` then uses a 3D CNN architecture based on VGG16 to train a model that classifies the inputs according to their labels. The weights from each epoch will be saved, as well as a .png of the learning curve. 

An issue was the inability for the sparse data to have a validation split as mentioned earlier, so to generate the validation curve, each checkpoint is loaded and the validation loss is calculated for each epoch. Expect the script to run for many hours.

The checkpoints and learning curve are saved in a folder that can be found in the folder `3DmodelCheckpoints`, and the specific instance corresponds to the time the script was run.

You are able to load previous checkpoints into the model to continue training. There is a boolean you will have to change in line 290 if you wish to continue training the model.

#### confusion_matrices.sh and confusion_matrices.py
The `confusion_matrices.sh` script only runs one python script, `confusion_matrices.py`. After your model has trained and you looked at the loss curves, you can run this script to make confusion matrices to get an idea of how the model is fitting on the test data. 

In order to run the script, you will obviously first need to run `model_fit_3D.sh`. Then you will need to manually input the epoch you want to load checkpoints from and which folder those checkpoints are located in. Then the code will generate two confusion matrices, one normalized, and one not normalized.

### Workflow
To start on the project, one must have the data from `oldRawDataTxtFiles`. Check if `oldRawDataTxtFiles` is in the right directory.

Next you will want to run `txt_to_numpy_array_3D.sh` so the data can be converted from txt files to numpy arrays. If you are working with the 3D CNN model, you will have to continue to run more shell scripts, if not, then you have to run other separate scripts that fit the needs of processing data for your specific model.

For 3D CNN, you will need to run `dataproccessingfor3DCNN.sh`. Once that is done, all your data will be ready for model training. Run `model_fit_3D.sh`, but before you run the script, check the `main()` function, and adjust parameters accordingly. If you are loading checkpoints, make sure the boolean for `load_model` is `True`, and add what directory you will be loading the checkpoints from accordingly.

Once your model has finished training, you should look at the loss curves created, and make a judgment on whether the model needs further training or parameter tuning. You can also run `confusion_matrices.sh` to see how well your model is fitting on the train data.

### Important Notes
The notebooks in the directory don’t actually ever need to be used. You can use the notebooks if you want, but the content inside is mostly outdated and mainly experimental. The notebooks are primarily for debugging and to check how many parameters are in a model to estimate training time.

If you run into memory issues, you might want to change the batch size, or decrease the resolution of the data. Training times will be long as the data is three dimensional. For debugging, you will want to run the model on fewer epochs.
