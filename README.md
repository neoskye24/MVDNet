# MVDNet
Reference: Robust Multimodal Vehicle Detection in Foggy Weather Using Complementary Lidar and Radar Signals, CVPR 2021.

This repo have some update from the original paper

## Get Started

### Prerequisites

- Python 3.7
- Pytorch 1.9.1
- Numpy 1.16.4
- Detectron2 (modified version)
- Pycocotools 2.0

__step 0__ in case you want plan to work on conda environment. Download and install fron the [official website](https://docs.conda.io/en/latest/miniconda.html)

```
conda create --name MVDNet python=3.7 -y
conda activate MVDNet
```

__step 1__ install pytorch. This work uses cuda [11.1](https://developer.nvidia.com/cuda-11.1.0-download-archive) and Pytorch [1.9.1](https://pytorch.org/get-started/previous-versions/)

On GPU

On CPU

### Installation

__Step 0__ install specific numpy version
```
conda install -c conda-forge numpy=1.16.4
# or 
pip install numpy==1.16.4
```

__Step 1__ MVDNet uses an old version of Detectron2 (i.e., 0.1.1) with [minor modifications](https://github.com/qiank10/detectron2/commit/370700b01be5ce401a1803af70d3e4c0471858c5). To download and install the compatible version:
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

## Prepare Data

Download the [Oxford Radar RobotCar Dataset](https://oxford-robotics-institute.github.io/radar-robotcar-dataset). Currently, only the vehicles in the first data record (Date: 10/01/2019, Time: 11:46:21 GMT) are labeled. After unzipping the files, the directory should look like this:
```
# Oxford Radar RobotCar Data Record
|-- DATA_PATH
    |-- gt
    |-- radar
    |-- velodyne_left
    |-- velodyne_right
    |-- vo
    |-- radar.timestamps
    |-- velodyne_left.timestamps
    |-- velodyne_right.timestamps
    |-- ...
```

Prepare the radar data:
```
python data/sdk/prepare_radar_data.py --data_path DATA_PATH --image_size 320 --resolution 0.2
```

Prepare the lidar data:
```
python data/sdk/prepare_lidar_data.py --data_path DATA_PATH
```

Prepare the foggy lidar test set with specified fog density, e.g., 0.05:
```
python data/sdk/prepare_fog_data.py --data_path DATA_PATH --beta 0.05
```

The processed data is organized as follows:
```
# Oxford Radar RobotCar Data Record
|-- DATA_PATH
    |-- processed
        |-- radar
            |-- 1547120789640420.jpg
            |-- ...
        |-- radar_history
            |-- 1547120789640420_k.jpg   # The k-th radar frame preceding the frame at the timestamp 1547120789640420, k=1,2,3,4.
            |-- ...
        |-- lidar
            |-- 1547120789640420.bin
            |-- ...
        |-- lidar_history
            |-- 1547120789640420_k.bin   # Link to the k-th lidar frame preceding the frame at the timestamp 1547120789640420, k=1,2,3,4.
            |-- 1547120789640420_k_T.bin # Transform matrix between the k-th preceding lidar frame and the current frame.
            |-- ...
        |-- lidar_fog_0.05               # Foggy lidar data with fog density as 0.05
            |-- 1547120789640420.bin
            |-- ...
        |-- lidar_history_fog_0.05
            |-- 1547120789640420_k.bin
            |-- 1547120789640420_k_T.bin
            |-- ...
```

Both 2D and 3D labels are in
```
./data/RobotCar/object/
```

## Train MVDNet
```
python ./tools/train.py --config ./configs/train_config.yaml
```

## Evaluate MVDNet
```
python ./tools/eval.py --config ./configs/eval_config.yaml
```
