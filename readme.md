# ROS environment for testing Livox MID70
- Docker container for testing SLAM packages with Livox MID70
- Starts a ROS melodic container with the following packages
    - [x] livox viewer
    - [x] livox SDK
    - [x] livox ros driver
    - [x] livox detection
    - [x] livox mapping 
    - [x] [FAST-LIO](https://github.com/DinoHub/FAST_LIO) 
    - [x] [FAST-LIO-SLAM](https://github.com/gisbi-kim/FAST_LIO_SLAM) | Not tested. Ceres, gtsam
    - [ ] [R2LIVE](https://github.com/hku-mars/r2live) | make error
    - [x] mavros
    - [x] PTP master (PTPD on host)
    - [x] [loam livox](https://github.com/hku-mars/loam_livox) | rviz hangs after awhile  
    - [x] [LIO-SAM](https://github.com/Innopolis-UAV-Team/LIO-SAM/tree/livox)  
    - [x] [ssl-slam2](https://github.com/wh200720041/ssl_slam2)
    - [ ] [SFA3D](https://github.com/maudzung/SFA3D/tree/ea0222c1b35489dc35d8452c989c4b014e20e0da)

## Requirements
- PTPD (sudo apt install ptpd), Docker, NVIDIA Container Toolkit  

## How to use
1. `docker pull eehantiming/livox:{tagnumber}`  
2. Edit run.sh with the appropriate tag number, and (optional) any volumes that you want to mount 
3. Connect the LIDAR via ethernet and set your IP manually to 192.168.1.50 (assuming LIDAR with default static IP). Note: you must connect before starting the container
4. If required, connect IMU via USB and start clock master with `sudo ptpd -M -i eth0 -C` (check ifconfig and change eth0)
5. `./run.sh`. There are currently 2 workspaces, ros_ws and fastlioslam_ws. `source {ws}/devel/setup.bash` as required  
6. Use `docker cp {file} livox:/root` to copy rosbag or other files in. You can also use this to copy files out  

### TMUX
Activate with `tmux`. The basic controls are:  
- ctrl-a followed by / to split and create another pane on the right  
- ctrl-a followed by . to split and create another pane below  
- hold alt and press arrow keys to navigate between the panes  
- ctrl-d (with nothing typed) to close unwanted panes  


### Livox ROS driver
`roslaunch livox_ros_driver livox_lidar.launch`  
This package is required when working with actual LIDAR data instead of with rosbags. There are 2 main launch files which publishes lidar cloud to /livox/lidar.  
- livox_lidar.launch publishes with PointCloud2 type  
- livox_lidar_msg.launch publishes with a custom msg type which is required by some packages such as FAST-LIO  

### FAST_LIO  
This package requires IMU data from /mavros/data/imu and custom lidar msg type  
1. Find imu tty with `dmesg | grep tty`  
3. In mapping_avia.launch, the included file apm.launch is required for the IMU data from px4, edit param ACM0 or ACM1 from step 1  
4. Edit config/avia, change imu topic to /mavros/imu/data. change T to [0.080, 0, -0.080]. Change 'dense_publish_en' to false  
5. Start the node with `roslaunch fast_lio mapping_avia.launch`  
6. Turn on IMU stream with `rosservice call mavros/set_stream_rate`. message rate 200 and 'true'  
7. Start the lidar launch file or rosbag  

Note: Before replaying rosbag with mapping.avia.launch, remove /path and /cloud_registered from the .bag files OR only record livox/lidar and mavros/imu/data  

### loam_livox
Does not require IMU and uses PointCloud2 msg. 
1. Change livox.launch to use config/performance_precision.yaml
2. Edit .yaml, move max_allow_incre_R, max_allow_incre_T and max_allow_final_cost from mapping to optimization. Change if_enable_loop_closure to 1  
3. `roslaunch loam_livox livox.launch` and start the lidar launch file or rosbag  

### [SFA3D](https://github.com/maudzung/SFA3D/tree/ea0222c1b35489dc35d8452c989c4b014e20e0da)
1. clone and `git checkout -b` the branch with ros.
2. in ros/src/super/src/rosInference.py, change sub topic to livox/lidar, PointCloud2. Run this node for detection
3. `roslaunch detected_objects_visualizer detected_objects_vis.launch` to process the detections into rviz. 
4. `rosrun rviz/rviz` and add the marker and cloud. May need to change frame from 'map' if there is an error.
5. play rosbag 

### [R2live](https://github.com/borongyuan/r2live)
Main branch have cmake error. use the fork instead and build. 


## Results
- FAST-LIO: works well
- loam_livox: works but sometimes DC and odometry fails. loop closure didnt do well when tested indoors. yet to test outdoors.
- livox_mapping: basic
- LIO-SAM: can't get livox supported fork to work. No map generated
- r2live: tested rosbag, have not hook up to camera
- ssl_slam2: tested rosbag

## Image versions
Tag | Description (Packages added on top of all previous versions)
---|---
1.2 |  livox_mapping <br> livox detection <br> 
1.3 | FAST_LIO <br> loam_livox <br> ceres, gtsam <br> FAST_LIO_SLAM
1.4 | Edited imu topic in FAST_LIO/config/avia.yaml. <br> Included apm.launch and rosservice call in FAST_LIO/launch/mapping_avia.launch <br> LIO-SAM <br> ssl_slam2

## Demo
<img src="https://raw.githubusercontent.com/eehantiming/livox/main/media/20211124_fastlio_3.png" width="571" height="314" />