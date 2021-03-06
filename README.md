# Introduction
It's a TensorFlow based inplementation of [VoxelNet: End-to-End Learning for Point Cloud Based 3D Object Detection]  
(https://arxiv.org/abs/1711.06396). This inplementation is based on a previous version from Qiangui [here](https://github.com/qianguih/voxelnet). I made some necessary modifications according to the paper, and I complete the implementation of Pedestrian and Cyclist detection. Thanks to [@Qiangui](https://github.com/qianguih) for the work. 

For the basic set up , we can directly follow Qiangui's instruction. 


# Dependencies
- `python3.5+`
- `TensorFlow` (tested on 1.4.1)
- `opencv`
- `shapely`
- `numba`
- `easydict`

# Installation
1. Clone this repository.
2. Compile the Cython module
```bash
$ python3 setup.py build_ext --inplace
```
3. Compile the evaluation code
```bash
$ cd kitti_eval
$ g++ -o evaluate_object_3d_offline evaluate_object_3d_offline.cpp
```
4. grant the execution permission to evaluation script
```bash
$ cd kitti_eval
$ chmod +x launch_test.sh
```

# Data Preparation
1. Download the 3D KITTI detection dataset from [here](http://www.cvlibs.net/datasets/kitti/eval_object.php?obj_benchmark=3d). Data to download include:
    * Velodyne point clouds (29 GB): input data to VoxelNet
    * Training labels of object data set (5 MB): input label to VoxelNet
    * Camera calibration matrices of object data set (16 MB): for visualization of predictions
    * Left color images of object data set (12 GB): for visualization of predictions

2. In this project, we use the cropped point cloud data for training and validation. Point clouds outside the image coordinates are removed. Update the directories in `data/crop.py` and run `data/crop.py` to generate cropped data. Note that cropped point cloud data will overwrite raw point cloud data.

2. Split the training set into training and validation set according to the protocol [here](https://xiaozhichen.github.io/files/mv3d/imagesets.tar.gz). And rearrange the folders to have the following structure:
```plain
└── DATA_DIR
       ├── training   <-- training data
       |   ├── image_2
       |   ├── label_2
       |   └── velodyne
       └── validation  <--- evaluation data
       |   ├── image_2
       |   ├── label_2
       |   └── velodyne
```
        
3. Update the dataset directory in `config.py` and `kitti_eval/launch_test.sh`

# Train
1. Specify the GPUs to use in `config.py`
2. run `train.py` with desired hyper-parameters to start training:
```bash
$ python3 train.py --alpha 1 --beta 10
```
Note that the hyper-parameter settings introduced in the paper are not able to produce high quality results. So, a different setting is specified here.

Training on two Nvidia 1080 Ti GPUs takes around 3 days (160 epochs as reported in the paper). During training, training statistics are recorded in `log/default`, which can be monitored by tensorboard. And models are saved in `save_model/default`. Intermediate validation results will be dumped into the folder `predictions/XXX/data` with `XXX` as the epoch number. And metrics will be calculated and saved in  `predictions/XXX/log`. If the `--vis` flag is set to be `True`, visualizations of intermediate results will be dumped in the folder `predictions/XXX/vis`.

3. When the training is done, executing `parse_log.py` will generate the learning curve.
```bash
$ python3 parse_log.py predictions
```

4. There is a pre-trained model for car in `save_model/pre_trained_car`.


# Evaluate
1. run `test.py -n default` to produce final predictions on the validation set after training is done. Change `-n` flag to `pre_trained_car` will start testing for the pre-trained model (only car model provided for now).
```bash
$ python3 test.py
```
results will be dumped into `predictions/data`. Set the `--vis` flag to True if dumping visualizations and they will be saved into `predictions/vis`.

2. run the following command to measure quantitative performances of predictions:
```bash
$ ./kitti_eval/evaluate_object_3d_offline [DATA_DIR]/validation/label_2 ./predictions
```


# TODO
- [] Train the network. 


