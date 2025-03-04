# MMStereo

Learned stereo training code for the paper:

```
Krishna Shankar, Mark Tjersland, Jeremy Ma, Kevin Stone, and Max Bajracharya. A Learned Stereo Depth System for Robotic Manipulation in Homes. ICRA 2022 submission.
```

## Setup

### Install apt dependencies
```
sudo apt install libturbojpeg virtualenv python3.8
```

### Anaconda environment
`conda` lets you install Python packages without conflicting with other workspaces.

#### Create the virtual environment
```
conda create -n mmstereo Python=3.8
```

#### Activate the virtual environment
```
conda activate mmstereo
```
Make sure you activate it each time you use the training code.

### Install PyTorch
```
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.3 -c pytorch
```

### Install other dependencies
```
pip install -r requirements.txt
```

## Sceneflow Dataset Training

### Get dataset
Download RGB and disparity images for Sceneflow Flying Things dataset from https://lmb.informatik.uni-freiburg.de/resources/datasets/SceneFlowDatasets.en.html.

Extract the downloaded files to `datasets/sceneflow/raw`.


### Prepare dataset
The dataset needs to be converted into a format that is compatible with the training code.

```
python prepare_sceneflow.py
```

### Train
The configuration is set up to work with a Titan RTX GPU with 24GB of VRAM. For other GPUs with less VRAM, you may need to reduce the batch size.
```
python train.py --config config_sceneflow.yaml
```

### Visualize with tensorboard
In a new terminal (since your other terminal is running the training script),
```
source venv/bin/activate
tensorboard --logdir output
```
and then open the url it gives you in a browser.

# Inference
You can run inference on a pair of images using the Torchscript output from training.
```
python run.py --script output/sceneflow/version_0/checkpoints/model.pt --left datasets/sceneflow/flying_things/val/left/0000000.png --right datasets/sceneflow/flying_things/val/right/0000000.png
```

# Dataset
Each sample is dataset is made up of a pair of left and right RGB images, a left disparity image, and an optional right
disparity image.

## RGB images
The RGB images should be 3-channel 8-bit images as either JPG or PNG format.

## Disparity images
The disparity images should a floating point Numpy ndarray stored in NPZ format with the same height and width as the
RGB images. If there is no right disparity, training is possible but the horizontal flip data augmentation can't be
used.

## Directory structure
The each element of a sample should be stored in a certain directory with the same base filename and appropriate
extension.

Example structure:
* left
  * sample0.png
  * sample1.png
* left_disparity
  * sample0.npz
  * sample1.npz
* right
  * sample0.png
  * sample1.png


# Running on OAK-D device

The learned model can be exported to run on an OAK-D device. After training a model, run:
```
./compile_model.sh <path-to-onnx-file>
```
where `<path-to-onnx-file>` is the path to the `model.onnx` file in the training checkpoint directory, e.g. `output/sceneflow/version_0/checkpoints/model.onnx`.

The `compile_model.sh` script uses an Intel Openvino [Docker](https://docs.docker.com/engine/install/ubuntu/) image to compile the model for the Myriad processor, so make sure you have docker installed and running.

The script produces a `model.blob` blob file, that can be loaded by the depthai library.

With your device connected, you can test it with `python depthai_test.py`.



