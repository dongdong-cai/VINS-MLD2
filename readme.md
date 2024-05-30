# MLD2-VINS

## VINS-MLD2: Monocular Visual-Inertial SLAM With Multi-level Detector and Descriptor

VINS-MLD2 is a robust real-time SLAM framework by integrating feature extraction network MLD2 with VINS-Mono. Our work primarily focuses on improving the speed of feature extraction network and the robustness of feature tracking. 

**The code will be open source soon！！！**



## 1.Dependencies

- [OpenCV](https://opencv.org/) python >= 4.9
- [PyTorch](https://pytorch.org/) >= 1.8

Note that in some versions of OpenCV, such as 4.2, frequent switching between CPU and GPU can cause congestion, resulting in optical flow matching times of tens of milliseconds (normal should be less than 10 milliseconds).



## 2.Build

```sh
cd ~/catkin_ws/src
git clone .....
cd ../
catkin build
source devel/setup.bash
```



## 3.Run on the dataset

We provide guidelines for running on the dataset including [EuRoC](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) and [TUM VI](https://vision.in.tum.de/data/datasets/visual-inertial-dataset).

EuRoC:

```sh
# Terminal 1
cd ~/catkin_ws
source devel/setup.bash
roslaunch vins_estimator euroc.launch
# Terminal 2
cd ~/catkin_ws
source devel/setup.bash
roslaunch vins_estimator vins_rviz.launch
# Terminal 3
cd ~/catkin_ws/VINS_MLD2/src/VINS_MLD2/MLD2
python feature_tracker.py --mld2 ./config/EuRoc.yaml
# Terminal 4
cd your_euroc_path
rosbag play MH_01_easy.bag
```

TUM VI:

```sh
# Terminal 1
cd ~/catkin_ws
source devel/setup.bash
roslaunch vins_estimator tum.launch
# Terminal 2
cd ~/catkin_ws
source devel/setup.bash
roslaunch vins_estimator vins_rviz.launch
# Terminal 3
cd ~/catkin_ws/VINS_MLD2/src/VINS_MLD2/MLD2
python feature_tracker.py --mld2 ./config/TUM.yaml
# Terminal 4
cd your_tumvi_path
rosbag play dataset-magistrale1_512_16.bag
```



## 4.Deployed on your device

We provide methods for deployment on the D435i camera, and similar methods can be applied to other cameras as well.

D435i:

```sh
# 1.modify configuration file, including necessary parameters such as camera topic name, imu topic name, camera internal parameters, camera-imu extrinsic parameters, and IMU internal parameters
cd ~/catkin_ws/VINS_MLD2/src/VINS_MLD2
gedit ./VINS-Mono/config/realsense/d435i_config.yaml
gedit ./MLD2/config/D435i.yaml

# 2.launch your sensor_ros_package
roslaunch realsense2_camera rs_stere_rgbd_depth_imu.launch

# 3. launch VINS-MLD2
cd ~/catkin_ws
source devel/setup.bash
roslaunch vins_estimator d435i.launch
roslaunch vins_estimator vins_rviz.launch
cd ~/catkin_ws/VINS_MLD2/src/VINS_MLD2/MLD2
python feature_tracker.py --mld2 ./config/D435i.yaml
```



## 5.Evaluate on the HPatches dataset

The evaluation is based on the code from [R2D2](https://github.com/naver/r2d2?tab=readme-ov-file) and [ASLFeat](https://github.com/lzx551402/ASLFeat).

```sh
# 1.Use the R2D2 script to extract features and save them to a local file. Be careful to replace the network structure in the script with the MLD2 network structure file.
git clone https://github.com/naver/r2d2.git
python extract.py --model mld2_v1.pt --images d2-net/image_list_hpatches_sequences.txt --top-k 2000
# 2.Use ASLFeat's evaluation indicator script to calculate the corresponding indicators. Note that the ASLFeat feature extraction part in the script should be changed to read the MLD2 result file in the previous step.
git clone https://github.com/lzx551402/ASLFeat.git
python hseq_eval.py --config configs/hseq_eval.yaml
```

At the end of running, we report the average number of features, repeatability, precision, matching score, recall and mean matching accuracy (a.k.a. MMA). The evaluation results will be displayed as:

```sh
----------i_eval_stats----------
....
----------v_eval_stats----------
....
----------all_eval_stats----------
avg_n_feat 2000
avg_rep 0.6137987
avg_precision 0.77793497
avg_matching_score 0.29641402
avg_recall 0.45165068
avg_MMA 0.76863936
avg_homography_accuracy 0.7114814
```



## 6.Acknowledgements

Thanks to the open sources of [R2D2](https://github.com/naver/r2d2?tab=readme-ov-file) and [VINS-Mono](https://github.com/HKUST-Aerial-Robotics/VINS-Mono), it is possible to build our algorithm quickly within the VINS system.

**The reference:**

VINS-Mono:

```sh
@ARTICLE{VINS-Mono,
	author={Qin, Tong and Li, Peiliang and Shen, Shaojie},
	journal={IEEE Trans. Robot.}, 
	title={{VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator}}, 
	year={2018},
	volume={34},
	number={4},
	pages={1004-1020},
	doi={10.1109/TRO.2018.2853729}}
```

R2D2:

```sh
@inproceedings{4,
     author= {Revaud, Jerome and De Souza, Cesar and Humenberger, Martin and Weinzaepfel, Philippe},
     booktitle= {Advances in Neural Information Processing Systems},
     pages= {},
     publisher= {Curran Associates, Inc.},
     title= {{R2D2: Reliable and Repeatable Detector and Descriptor}},
     url= {https://proceedings.neurips.cc/paper_files/paper/2019/file/3198dfd0aef271d22f7bcddd6f12f5cb-Paper.pdf},
     volume= {32},
     year= {2019}
}
```



