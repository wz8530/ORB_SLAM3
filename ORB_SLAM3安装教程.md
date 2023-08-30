# Ubuntu18.04 安装 ORB_SLAM3

#===========================#
# V1.0 -- WZ -- 2023/7/12   #
#===========================#

1. 安装依赖库
$ sudo apt-get install libglew-dev ffmpeg libavcodec-dev libavutil-dev libavformat-dev libswscale-dev libdc1394-22-dev libraw1394-dev libjpeg-dev libpng-dev libtiff5-dev libopenexr-dev

2. 安装Pangolin-0.5，请参考 ~/3rdparty/Pangolin-0.5中的readme-wz.md文档

3. 安装Opencv-4.5.3，请参考 ~/3rdparty/opencv-4.5.3/ 中的readme-wz.md文档（opencv必须大于4.4.0）,修改ORB_SLAM3目录下CMakeLists.txt和Thirdparty/DBoW2中的CMakeLists.txt中OpenCV的路径。

4. 安装Eigen3

5. 编译ORB_SLAM3
$ cd ~/project/
$ git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git
$ git checkout 4452a3c4ab75b1cde34e5505a36ec3f9edcdc4c4   ### 对应v1.0版本
$ cd ORB_SLAM3
$ ./build.sh


6. 编译ROS ORB_SLAM3
1) 下载vision_opencv，修改源码适配opencv-4.5.3版本，参考 ~/3rdparty/vision_opencv-melodic/vision_opencv使用说明.md，将文件夹放置根目录下（~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/）；

2) 修改 ROS/ORB_SLAM3/CMakeLists.txt，修改后如下：
```
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
add_subdirectory(vision_opencv) #wz-new add
rosbuild_init()
```
```
set(OpenCV_DIR "/usr/local/lib/cmake/opencv4") #wz
find_package(OpenCV 4 QUIET) #wz
```
3) 在 ROS/ORB_SLAM3/vision_opencv/ 下新建一个CMakeLists.txt，添加以下内容：
```
add_subdirectory(./cv_bridge)
add_subdirectory(./image_geometry)
add_subdirectory(./opencv_tests)
add_subdirectory(./vision_opencv)
```

4) 执行build_ros.sh脚本，开始编译
$ cd ~/project/ORB_SLAM3
$ ./build_ros.sh


7. 运行 `ORB_SLAM3 双目 + EuRoC数据库`
```
Usage: 
./stereo_inertial_euroc 
path_to_vocabulary  # 字典文件  
path_to_settings  # 参数设置文件
path_to_sequence_folder_1  # 影像序列文件夹路径
path_to_times_file_1   # 对应的时间戳文件
save_file_name
```

$ /home/labpano/project/ORB_SLAM3/Examples/Stereo/stereo_euroc /home/labpano/project/ORB_SLAM3/Vocabulary/ORBvoc.txt /home/labpano/project/ORB_SLAM3/Examples/Stereo/EuRoC.yaml /home/labpano/data/MH_01_easy /home/labpano/project/ORB_SLAM3/Examples/Stereo/EuRoC_TimeStamps/MH01.txt stereo_MH01


8. 运行 `ROS ORB_SLAM3 双目 + EuRoc数据集`
## 三个终端
$ roscore 

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 Stereo ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Stereo/EuRoC.yaml true

$ rosbag play --pause ~/data/V1_03_difficult.bag /cam0/image_raw:=/camera/left/image_raw /cam1/image_raw:=/camera/right/image_raw /imu0:=/imu


9. 运行 `ROS ORB_SLAM3 双目 + EuRoC数据集转成d435i数据`
$ roscore

# 修改RealSense_D435i.yaml参数
$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash 
$ rosrun ORB_SLAM3 Stereo ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Stereo/RealSense_D435i.yaml false

$ rosbag play --pause --clock ~/data/MH_01_easy.bag /cam0/image_raw:=/camera/left/image_raw /cam1/image_raw:=/camera/right/image_raw /imu0:=/imu


10. 运行 `ROS ORB_SLAM3(v0.4) 双目 + realsense d435i + imu`
$ source ~/project/ws_realsense_ros/devel/setup.bash
$ roslaunch realsense2_camera rs_camera.launch align_depth:=true enable_align:=true depth_width:=640 depth_height:=480 depth_fps:=30 color_width:=640 color_height:=480 color_fps:=30 enable_infra:=true enable_infra1:=true enable_infra2:=true infra_width:=640 infra_height:=480 infra_fps:=30 enable_gyro:=true enable_accel:=true unite_imu_method:="linear_interpolation" gyro_fps:=200 accel_fps:=200 emitter_enable:=true

### 转换ros topic主题
$ rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 30.0 /camera/left/image_raw
$ rosrun topic_tools throttle messages /camera/infra2/image_rect_raw 30.0 /camera/right/image_raw
$ rosrun topic_tools throttle messages /camera/imu 200.0 /imu

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 Stereo_Inertial ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Stereo-Inertial/RealSense_D435i.yaml false


11. 运行 `ROS ORB_SLAM3(v0.4) 单目 + realsense d435i + imu`
$ source ~/project/ws_realsense_ros/devel/setup.bash
$ roslaunch realsense2_camera rs_camera.launch align_depth:=true enable_align:=true depth_width:=640 depth_height:=480 depth_fps:=30 color_width:=640 color_height:=480 color_fps:=30 enable_infra:=true enable_infra1:=true enable_infra2:=true infra_width:=640 infra_height:=480 infra_fps:=30 enable_gyro:=true enable_accel:=true unite_imu_method:="linear_interpolation" gyro_fps:=200 accel_fps:=200 emitter_enable:=true

### 转换ros topic主题，这里采用左目infra1的图像
$ rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 30.0 /camera/image_raw
$ rosrun topic_tools throttle messages /camera/imu 200.0 /imu

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 Mono_Inertial ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Monocular-Inertial/RealSense_D435i.yaml false


12. 运行 `ROS ORB_SLAM3(v0.4) RGBD + realsense d435i + imu`
$ source ~/project/ws_realsense_ros/devel/setup.bash
$ roslaunch realsense2_camera rs_camera.launch align_depth:=true enable_align:=true depth_width:=640 depth_height:=480 depth_fps:=30 color_width:=640 color_height:=480 color_fps:=30 enable_infra:=true enable_infra1:=true enable_infra2:=true infra_width:=640 infra_height:=480 infra_fps:=30 enable_gyro:=true enable_accel:=true unite_imu_method:="linear_interpolation" gyro_fps:=200 accel_fps:=200 emitter_enable:=true

### 转换ros topic主题
$ rosrun topic_tools throttle messages /camera/color/image_raw 30.0 /camera/rgb/image_raw
$ rosrun topic_tools throttle messages /camera/aligned_depth_to_color/image_raw 30.0 /camera/depth_registered/image_raw
$ rosrun topic_tools throttle messages /camera/imu 200.0 /imu

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 RGBD_Inertial ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/RGB-D-Inertial/RealSense_D435i.yaml false
#$ rosrun ORB_SLAM3 RGBD_Inertial ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/RGB-D-Inertial/RealSense_D435i.yaml false /camera/rgb/image_raw:=/camera/color/image_raw /camera/depth_registered/image_raw:=/camera/aligned_depth_to_color/image_raw /imu:=/camera/imu


13. 运行 `ROS ORB_SLAM3(V0.4) 单目 + realsense d435i` 
$ source ~/project/ws_realsense_ros/devel/setup.bash
$ roslaunch realsense2_camera rs_camera.launch

$ rosrun topic_tools throttle messages /camera/color/image_raw 30.0 /camera/image_raw

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 Mono ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Monocular/RealSense_D435i.yaml


14. 运行 `ROS ORB_SLAM3(V0.4) 双目 + realsense d435i`
$ source ~/project/ws_realsense_ros/devel/setup.bash
$ roslaunch realsense2_camera rs_camera.launch depth_width:=640 depth_height:=480 depth_fps:=30 color_width:=640 color_height:=480 color_fps:=30 enable_infra:=true enable_infra1:=true enable_infra2:=true infra_width:=640 infra_height:=480 infra_fps:=30 emitter_enable:=false

### 转换ros topic主题
$ rosrun topic_tools throttle messages /camera/infra1/image_rect_raw 30.0 /camera/left/image_raw
$ rosrun topic_tools throttle messages /camera/infra2/image_rect_raw 30.0 /camera/right/image_raw

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 Stereo ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Stereo/RealSense_D435i.yaml false
#$ rosrun ORB_SLAM3 Stereo ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/Stereo/RealSense_D435i.yaml false /camera/left/image_raw:=/camera/infra1/image_rect_raw /camera/right/image_raw:=/camera/infra2/image_rect_raw


15. 运行 `ROS ORB_SLAM3(V0.4) RGBD + realsense d435i`
$ roscore

$ source ~/project/ws_realsense_ros/devel/setup.bash
$ roslaunch realsense2_camera rs_camera.launch align_depth:=true enable_align:=true depth_width:=640 depth_height:=480 depth_fps:=30 color_width:=640 color_height:=480 color_fps:=30 enable_infra:=true enable_infra1:=true enable_infra2:=true infra_width:=640 infra_height:=480 infra_fps:=30

$ source ~/project/ORB_SLAM3/Examples_old/ROS/ORB_SLAM3/build/devel/setup.bash
$ rosrun ORB_SLAM3 RGBD ~/project/ORB_SLAM3/Vocabulary/ORBvoc.txt ~/project/ORB_SLAM3/Examples_old/RGB-D/RealSense_D435i.yaml /camera/rgb/image_raw:=/camera/color/image_raw /camera/depth_registered/image_raw:=/camera/aligned_depth_to_color/image_raw

