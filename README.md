# MVDNet
Reference: Robust Multimodal Vehicle Detection in Foggy Weather Using Complementary Lidar and Radar Signals, CVPR 2021.

This repo have some update from the original paper

## Get Started

### Prerequisites

- Python 3.7
- Pytorch 1.9.1
- Numpy 1.16.4
- Detectron2 (modified version)
- Pycocotools
- pip install setuptools==59.5.0
- pip install psutil

__step 0__ in case you want plan to work on conda environment. Download and install fron the [official website](https://docs.conda.io/en/latest/miniconda.html)

```
conda create --name MVDNet python=3.7 -y
conda activate MVDNet
```

__step 1__ install pytorch. This work uses cuda [11.1](https://developer.nvidia.com/cuda-11.1.0-download-archive) and Pytorch [1.9.1](https://pytorch.org/get-started/previous-versions/)

> **_NOTE:_** Currently the model can only train and evaluate on GPU (CUDA enabled)

### Installation

__Step 0__ install specific numpy version
```
conda install -c conda-forge numpy=1.16.4
# or 
pip install numpy==1.16.4
```

__Step 1__ install other related package
```
pip install pycocotools
pip install psutil
pip install setuptools==59.5.0
```

__Step 2__ MVDNet uses an old version of Detectron2 (i.e., 0.1.1) with [minor modifications](https://github.com/qiank10/detectron2/commit/370700b01be5ce401a1803af70d3e4c0471858c5). To download and install the compatible version:
```
git clone https://github.com/qiank10/detectron2.git
cd detectron2
git checkout alt-0.1.1
pip install -e .
```

Install MVDNet with some modification from
```
git clone https://github.com/MaiRajborirug/MVDNet.git
cd MVDNet && pip install -e .
```

### Prepare Data

__Step 0__ download partually processed dataset

This repo already partually processed the Oxford Radar RobotCar Dataset and can be downloaded from [CMU box](https://cmu.box.com/s/f4v6lwdi4civuz7d127uym7v74e1p2lw). If you are using the default configure, replace the github subfolder folder `MVDNet/data/RobotCar` with this unzipped folder `RobotCar`.

> **_NOTE:_** The author of MVDNet create a 2D and 3D bounding box for data from [Oxford Radar RobotCar Dataset](https://oxford-robotics-institute.github.io/radar-robotcar-dataset) in the first record (Date: 10/01/2019, Time: 11:46:21 GMT). If your want to adjust the lidar fog density. You can follow the step from the original [MVDNet](https://github.com/qiank10/MVDNet/blob/main/README.md#prepare-data) github

```
# Oxford Radar RobotCar Data Record
|-- RobotCar
    |-- lidar_fog_0 # Lidar data with no fog effect
        |-- lidar
            |-- 1547120787645464.bin
            |-- ...
        |-- lidar_history
            |-- 1547120788638924_1.bin # Symlink to the k-th lidar frame preceding (in `lidar` folder) the frame at the timestamp 1547120788638924, k=1,2,3,4. The reason MVDNet uses symlink is because it requires to much memory ot pepeat the actual pointcloud files
            |-- 1547120788638924_1_T.bin # Transform matrix between the k-th preceding lidar frame and the current frame.
            |-- ...
    |-- lidar_fog_006 # Foggy lidar data with fog density as 0.06
            |-- ...
    |-- lidar_fog_mix # Randomly select clear weather lidar and 0.06 desnsity foggy lidar with each probability 50%
            |-- ... 
    |-- ImageSets
        |-- train.txt # list of lidar and radar frame in training set
        |-- eval.txt # list of lidar and radar frame in evaluation set
        
    |-- object
        |-- radar
            |-- 1547120788638924.jpg
            |-- ...
        |-- radar_history
            |-- 1547120788638924_1.jpg # The k-th radar frame preceding the frame at the timestamp 1547120789640420, k=1,2,3,4.
            |-- ...
        |-- lidar_history # currently empty folder 
        |-- lidar # currently empty folder 
        |-- label_3d
            |-- ...
        |-- label_2d
            |-- 1547120787645464.txt # 2D label in format: [class(only 'car')] [car_id] [top_left_x] [top_left_y] [width] [height] [angle] 
            |-- ...
    |-- ...
```

__Step 1__ select fog lidar

There are lidar datasets with three different fog augmentations in `/RobotCar/{lidar_fog_0, lidar_fog_006, lidar_fog_mix}/{lidar, lidar_history}`. Choose the fog condition and move `lidar` and `lidar_history` folder to `/RobotCar/objects/`

__Step 2__ adjust symlink (optional)
We need to adjust symlinks' target so that the symlink file in `./data/RobotCar/object/lidar_history/___.bin` point to in `./data/RobotCar/object/lidar/___.bin` The jupyter notebook function to adjust the symlink if in `./helper_functions/change_symlink.ipynb`


## Train and Evaluation

### Pretrain weight

The pretrain weight for `clear_weather`, `mix_weather`, and `fog_weather` can be download from [CMU box](https://cmu.box.com/s/myfk2bxzq8bzqheroex3qwce35e7plrw)

### Train MVDNet
```
python ./tools/train.py --config ./configs/train_config.yaml
```
The output directory can be adjusted in `MVDNet/configs/train_config.yaml`

### Evaluate MVDNet
```
python ./tools/eval.py --config ./configs/eval_config.yaml
```

The model weight directory can be adjusted in `MVDNet/configs/eval_config.yaml`
