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

## Requirements
- Docker, NVIDIA Container Toolkit  

## How to use
1. `docker pull eehantiming/livox:{tagnumber}`. 
2. Edit run.sh with the appropriate tag number, and (optional) any volumes that you want to mount 
3. Connect the LIDAR via ethernet and set your IP manually to 192.168.1.50 (assuming LIDAR with default static IP)
4. If required, connect IMU via USB and start clock master with `sudo ptpd -M -i eth0 -C` (check ifconfig and change eth0)
5. `./run.sh`. There are currently 2 workspaces, ros_ws and fastlioslam_ws. `source {ws}/devel/setup.bash` as required  

Note: tmux is included in the docker image  

## FAST_LIO  
1. start clock master with `sudo ptpd -M -i eth0 -C` check ifconfig and change eth0  
2. find imu tty with `dmesg | grep tty`  
3. edit mapping_avia.launch. add apm.launch for the mavros, use ACM0 or ACM1 from above  
4. edit config. change imu topic to /mavros/imu/data  
5. Rosservice call mavros/set_stream_rate. message rate 50 and true  

## Image versions
Tag | Description (Packages added on top of all previous versions)
---|---
1.2 |  livox_mapping <br> livox detection <br> 
1.3 | FAST_LIO <br> loam_livox <br> ceres, gtsam <br> FAST_LIO_SLAM
1.4 | Edited imu topic in FAST_LIO/config/avia.yaml. <br> Included apm.launch and rosservice call in FAST_LIO/launch/mapping_avia.launch <br> LIO-SAM <br> ssl_slam2

## Demo
<img src="https://raw.githubusercontent.com/eehantiming/livox/main/media/run2_hwsync.png" width="571" height="314" />