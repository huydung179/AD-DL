# `train slice` - Train classification CNN using 2D slices

This option allows training a network on 2D slices. For more information on slices please refer to [tensor extraction](../Preprocessing/Extract.md).
There is no network type choice for `slice` as the only network type is the single-CNN.

One architecture is implemented in `clinicadl` for the `slice` mode: `resnet18`.
If this architecture is chosen, the network is automatically initialized with the weights
of a ResNet-18 trained on ImageNet.

!!! info "Adding a custom architecture"
    It is possible to add a custom architecture and train it with `clinicadl`.
    Detailed instructions can be found [here](./Custom.md).

## `train slice autoencoder` - Train autoencoders using 2D slices

The objective of an autoencoder is to learn to reconstruct images given in input while performing a dimension reduction. 
Slices at the beginning or at the end of the volume may be excluded using the `discarded_slices` argument.

The difference between the input and the output image is given by the mean squared error (MSE).
In clinicadl, autoencoders are designed [based on a CNN architecture](./Details.md#autoencoders-construction-from-cnn-architectures). 

### Running the task

Here is the command line to train an autoencoder on `t1-linear` outputs with the predefined architecture of ClinicaDL: 
```
clinicadl train slice autoencoder <caps_directory> t1-linear <tsv_path> <output_directory> Conv4_FC3
```
where mandatory arguments are:

- `caps_directory` (str) is the input folder containing the neuroimaging data in a [CAPS](http://www.clinica.run/doc/CAPS/Introduction/) hierarchy.
- `tsv_path` (str) is the input folder of a TSV file tree generated by `clinicadl tsvtool {split|kfold}`.
- `output_directory` (str) is the folder where the results are stored.

!!! info "Common options"
    Options that are common to all `train` input and network types can be found in the introduction of 
    [`clinicadl train`](./Introduction.md#running-the-task).

The options specific to this task are the following: 

- `--slice_direction` (int) axis along which the MR volume is sliced. Default: `0`.
    - 0 corresponds to the sagittal plane,
    - 1 corresponds to the coronal plane,
    - 2 corresponds to the axial plane.
- `--discarded_slices` (list of int) number of slices discarded from respectively the beginning and the end of the MRI volume. 
If only one argument is given, it will be used for both sides. Default: `20`.
- `--use_extracted_slices` (bool) if this flag is given, the outputs of `clinicadl extract` are used.
Otherwise, the whole 3D MR volumes are loaded and slices are extracted on-the-fly.- `--visualization` (bool) if this flag is given, inputs of the train and
the validation sets and their corresponding reconstructions are written in `autoencoder_reconstruction`.
Inputs are reconstructed based on the model that obtained the [best validation loss](./Details.md#model-selection).
- `--transfer_learning_path` (str) is the path to a result folder (output of `clinicadl train`). 
The best model of this folder will be used to initialize the network as 
explained in the [implementation details](./Details.md#transfer-learning). 
If nothing is given the initialization will be random.

### Outputs

The complete output file system is the following (the folder `autoencoder_reconstruction` is created only if the 
flag `--visualization` was given):

<pre>
results
├── commandline.json
├── environment.txt
└── fold-0
    ├── autoencoder_reconstruction
    │   ├── train
    │   │   ├── input-0.nii.gz
    │   │   ├── ...
    │   │   ├── input-&lt;N&gt;.nii.gz
    │   │   ├── output-0.nii.gz
    │   │   ├── ...
    │   │   └── output-&lt;N&gt;.nii.gz
    │   └── validation
    │        ├── input-0.nii.gz
    │        ├── ...
    │        ├── input-&lt;N&gt;.nii.gz
    │        ├── output-0.nii.gz
    │        ├── ...
    │        └── output-&lt;N&gt;.nii.gz
    ├── models
    │    └── best_loss
    │        └── model_best.pth.tar
    └── tensorboard_logs
         ├── train
         │    └── events.out.tfevents.XXXX
         └── validation
              └── events.out.tfevents.XXXX
</pre>

`autoencoder_reconstruction` contains the reconstructions of all the slices of the first image of the dataset.
The number of slices `N` depends on the `slice_direction` and the number of `discarded_slices`.

## `train slice cnn` - Train classification CNN using all 2D slices

The objective of this unique CNN is to learn to predict labels associated to images.
The set of images used corresponds to all the possible slice locations in MR volumes.
Slices at the beginning or at the end of the volume may be excluded using the `discarded_slices` argument.

The output of the CNN is a vector of size equal to the number of classes in this dataset.
This vector can be preprocessed by the [softmax function](https://pytorch.org/docs/master/generated/torch.nn.Softmax.html) 
to produce a probability for each class. During training, the CNN is optimized according to the cross-entropy loss. 
Its value becomes null for a subset of images if the probability of the CNN is 1, 
with respect to the true class (ground truth) of each image in the subset.

### Running the task

Here is the command line to train a CNN on `t1-linear` outputs with the predefined architecture of ClinicaDL: 
```
clinicadl train slice cnn <caps_directory> t1-linear <tsv_path> <output_directory> resnet18
```
where mandatory arguments are:

- `caps_directory` (str) is the input folder containing the neuroimaging data in a [CAPS](http://www.clinica.run/doc/CAPS/Introduction/) hierarchy.
- `tsv_path` (str) is the input folder of a TSV file tree generated by `clinicadl tsvtool {split|kfold}`.
- `output_directory` (str) is the folder where the results are stored.

!!! info "Common options"
    Options that are common to all `train` input and network types can be found in the introduction of 
    [`clinicadl train`](./Introduction.md#running-the-task).

The options specific to this task are the following:

- `--slice_direction` (int) axis along which the MR volume is sliced. Default: `0`.
    - 0 corresponds to the sagittal plane,
    - 1 corresponds to the coronal plane,
    - 2 corresponds to the axial plane.
- `--discarded_slices` (list of int) number of slices discarded from respectively the beginning and the end of the MRI volume. 
If only one argument is given, it will be used for both sides. Default: `20`.
- `--use_extracted_slices` (bool) if this flag is given, the outputs of `clinicadl extract` are used.
Otherwise, the whole 3D MR volumes are loaded and slices are extracted on-the-fly.
- `--transfer_learning_path` (str) is the path to a result folder (output of `clinicadl train`). 
The best model of this folder will be used to initialize the network as 
explained in the [implementation details](./Details.md#transfer-learning). 
If nothing is given the initialization will be random.
- `--transfer_learning_selection` (str) corresponds to the metric according to which the 
[best model](./Details.md#model-selection) of `transfer_learning_path` will be loaded. 
This argument will only be taken into account if the source network is a CNN. 
Choices are `best_loss` and `best_balanced_accuracy`. Default: `best_balanced_accuracy`.
- `--selection_threshold` (float) threshold on the balanced accuracies to compute the 
[image-level performance](./Details.md#soft-voting). 
Slices are selected if their balanced accuracy is greater than the threshold. Default corresponds to no selection.

### Outputs

The complete output file system is the following:

<pre>
results
├── commandline.json
├── environment.txt
└── fold-0
    ├── cnn_classification
    │   ├── best_balanced_accuracy
    │   │   ├── train_image_level_metrics.tsv
    │   │   ├── train_image_level_prediction.tsv
    │   │   ├── train_slice_level_metrics.tsv
    │   │   ├── train_slice_level_prediction.tsv
    │   │   ├── validation_image_level_metrics.tsv
    │   │   ├── validation_image_level_prediction.tsv
    │   │   ├── validation_slice_level_metrics.tsv
    │   │   └── validation_slice_level_prediction.tsv
    │   └── best_loss
    │       └── ...
    ├── models
    │   ├── best_balanced_accuracy
    │   │   └── model_best.pth.tar
    │   └── best_loss
    │       └── model_best.pth.tar
    └── tensorboard_logs
         ├── train
         │    └── events.out.tfevents.XXXX
         └── validation
              └── events.out.tfevents.XXXX
</pre>

!!! note "Level of performance"
    The performance metrics are obtained at two different levels: slice-level and image-level. 
    Slice-level performance corresponds to an evaluation in which all slices are considered to be independent. 
    However it is not the case, and what is more interesting is the evaluation at the image-level, 
    for which the predictions of slice-level were [assembled](./Details.md#soft-voting).

## `train slice multicnn` - Train one classification CNN per slice

Contrary to the preceding network in which all slices were used as input of a unique CNN, with this option
a CNN is trained per slice. Then the predictions of the CNNs are [assembled](./Details.md#soft-voting) to determine
the label at the image level.

The output of each CNN is a vector of size equals to the number of classes in this dataset.
This vector can be preprocessed by the [softmax function](https://pytorch.org/docs/master/generated/torch.nn.Softmax.html) 
to produce a probability for each class. During training, the CNN is optimized according to the cross-entropy loss. 
Its value becomes null for a subset of images if the probability of the CNN is 1, 
with respect to the true class (ground truth) of each image in the subset.

### Running the task

Here is the command line to train a CNN on `t1-linear` outputs with the predefined architecture of ClinicaDL: 
```
clinicadl train slice multicnn <caps_directory> t1-linear <tsv_path> <output_directory> resnet18
```
where mandatory arguments are:

- `caps_directory` (str) is the input folder containing the neuroimaging data in a [CAPS](https://aramislab.paris.inria.fr/clinica/docs/public/latest/CAPS/Introduction/) hierarchy.
- `tsv_path` (str) is the input folder of a TSV file tree generated by `clinicadl tsvtool {split|kfold}`.
- `output_directory` (str) is the folder where the results are stored.

!!! info "Common options"
    Options that are common to all `train` input and network types can be found in the introduction of 
    [`clinicadl train`](./Introduction.md#running-the-task).

The options specific to this task are the following:

- `--slice_direction` (int) axis along which the MR volume is sliced. Default: `0`.
    - 0 corresponds to the sagittal plane,
    - 1 corresponds to the coronal plane,
    - 2 corresponds to the axial plane.
- `--discarded_slices` (list of int) number of slices discarded from respectively the beginning and the end of the MRI volume. 
If only one argument is given, it will be used for both sides. Default: `20`.
- `--use_extracted_slices` (bool) if this flag is given, the outputs of `clinicadl extract` are used.
Otherwise, the whole 3D MR volumes are loaded and slices are extracted on-the-fly.
- `--transfer_learning_path` (str) is the path to a result folder (output of `clinicadl train`). 
The best model of this folder will be used to initialize the network as 
explained in the [implementation details](./Details.md#transfer-learning). 
If nothing is given the initialization will be random.
- `--transfer_learning_selection` (str) corresponds to the metric according to which the 
[best model](./Details.md#model-selection) of `transfer_learning_path` will be loaded. 
This argument will only be taken into account if the source network is a CNN. 
Choices are `best_loss` and `best_balanced_accuracy`. Default: `best_balanced_accuracy`.
- `--selection_threshold` (float) threshold on the balanced accuracies to compute the 
[image-level performance](./Details.md#soft-voting). 
Slices are selected if their balanced accuracy is greater than the threshold. Default corresponds to no selection.

### Outputs

The complete output file system is the following:

<pre>
results
├── commandline.json
├── environment.txt
└── fold-0
    ├── cnn_classification
    │   ├── best_balanced_accuracy
    │   │   ├── train_image_level_metrics.tsv
    │   │   ├── train_image_level_prediction.tsv
    │   │   ├── train_slice_level_metrics.tsv
    │   │   ├── train_slice_level_prediction.tsv
    │   │   ├── validation_image_level_metrics.tsv
    │   │   ├── validation_image_level_prediction.tsv
    │   │   ├── validation_slice_level_metrics.tsv
    │   │   └── validation_slice_level_prediction.tsv
    │   └── best_loss
    │       └── ...
    ├── models
    │   ├── best_balanced_accuracy
    │   │   └── model_best.pth.tar
    │   └── best_loss
    │       └── model_best.pth.tar
    └── tensorboard_logs
         ├── train
         │    └── events.out.tfevents.XXXX
         └── validation
              └── events.out.tfevents.XXXX
</pre>

!!! note "Level of performance"
    The performance metrics are obtained at two different levels: slice-level and image-level. 
    Slice-level performance corresponds to an evaluation in which all slices are considered to be independent. 
    However it is not the case, and what is more interesting is the evaluation at the image-level, 
    for which the predictions of slice-level were [assembled](./Details.md#soft-voting).
