# ubuntu18.04安装image_geometry，解决与OpenCV版本直接的冲突

#################################
#    V1.0 -- wz -- 2023/4/26    #
#################################

1. 下载源码
$ git clone https://github.com/ros-perception/vision_opencv.git
>> 切换到ros-melodic对于的分支
$ git checkout origin/melodic

2. 进入image_geometry文件夹，修改CMakeLists.txt中OpenCV路径及版本，改为指定版本号
>> find_package(OpenCV REQUIRED COMPONENTS opencv_core opencv_imgproc opencv_imgcodecs CONFIG)
>> 改为以下内容：
set(OpenCV_DIR "/usr/local/lib/cmake/opencv4") #wz
find_package(OpenCV 4.5.3 REQUIRED COMPONENTS opencv_core opencv_imgproc opencv_imgcodecs CONFIG) #wz

3. 编译
$ cd ~/3rdparty/vision_opencv-melodic/image_geometry/
$ mkdir build
$ cd build
$ cmake ..
$ make -j12
$ sudo make install

4. 在开源代码需要的cmaklist.txt中添加image_geometry的cmake路径
>> set(image_geometry_DIR /usr/local/share/image_geometry/cmake)  //在find_package前面


**NOTE:**
1）建议用以下指令直接替换ros自带的image_geometry包，或者CMakeLists.txt中没有调 image_geometry 包，但在 manifest.xml 文件中调用 image_geometry 包，会出现调用不同opencv版本问题，可能会引起 seg fault（段错误）
>> 解决办法：将 /usr/local/share/image_geometry 和 /usr/local/lib/libimage_geometry.so 等替换 /opt/ros/melodic/ 中对应的文件夹和文件：
$ sudo cp -r /usr/local/share/image_geometry /opt/ros/melodic/share/

$ sudo cp /usr/local/lib/libimage_geometry.so /opt/ros/melodic/lib/

$ sudo cp /usr/local/lib/pkgconfig/image_geometry.pc /opt/ros/melodic/lib/pkgconfig/

$ sudo cp -r /usr/local/lib/python2.7/dist-packages/image_geometry /opt/ros/melodic/lib/python2.7/dist-packages/
$ sudo cp /usr/local/lib/python2.7/dist-packages/image_geometry-1.13.1.egg-info /opt/ros/melodic/lib/python2.7/dist-packages/

$ sudo cp -r /usr/local/include/image_geometry /opt/ros/melodic/include/


2）或者直接拷贝 ~/3rdparty/vision_opencv-melodic/image_geometry 文件夹到需要编译的ROS工作空间，一起编译，优先使用该ROS工作空间的image_geometry生成的so库.

