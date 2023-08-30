# ubuntu18.04安装cv_bridge，解决与OpenCV版本直接的冲突

#################################
#    V1.0 -- wz -- 2023/4/26    #
#################################

1. 下载源码
$ cd ~/3rdparty/
$ git clone https://github.com/ros-perception/vision_opencv.git
>> 切换到ros-melodic对于的分支
$ git checkout origin/melodic

2. 进入cv_bridge文件夹中，修改CMakeLists.txt中OpenCV路径及版本，改为指定版本号
>> find_package(OpenCV REQUIRED COMPONENTS opencv_core opencv_imgproc opencv_imgcodecs CONFIG)
>> 改为以下内容：
set(OpenCV_DIR "/usr/local/lib/cmake/opencv4") #wz
find_package(OpenCV 4.5.3 REQUIRED COMPONENTS opencv_core opencv_imgproc opencv_imgcodecs CONFIG) #wz

3. 编译
$ cd ~/3rdparty/vision_opencv/cv_bridge/
$ mkdir build
$ cd build
$ cmake ..
$ make -j12
>> 编译问题1：/home/labpano/3rdparty/vision_opencv-melodic/cv_bridge/src/module_opencv2.cpp:151:16: error: cannot declare variable ‘g_numpyAllocator’ to be of abstract type ‘NumpyAllocator’
 NumpyAllocator g_numpyAllocator;
>> 解决办法：（参考https://github.com/ros-perception/vision_opencv/issues/272）
I was able to successfully compile cv_bridge with opencv4 below are the rough notes of what i did:
1）Add set (CMAKE_CXX_STANDARD 11) to your top level cmake
2）In cv_bridge/src CMakeLists.txt line 35 change to if (OpenCV_VERSION_MAJOR VERSION_EQUAL 4)
3）In cv_bridge/src/module_opencv3.cpp change signature of two functions
3.1) UMatData* allocate(int dims0, const int* sizes, int type, void* data, size_t* step, int flags, UMatUsageFlags usageFlags) const
to
UMatData* allocate(int dims0, const int* sizes, int type, void* data, size_t* step, AccessFlag flags, UMatUsageFlags usageFlags) const
3.2) bool allocate(UMatData* u, int accessFlags, UMatUsageFlags usageFlags) const
to
bool allocate(UMatData* u, AccessFlag accessFlags, UMatUsageFlags usageFlags) const
>> 重新编译即可

$ sudo make install

3. 在开源代码需要的cmaklist.txt中添加cv_bridge的cmake路径
>> set(cv_bridge_DIR /usr/local/share/cv_bridge/cmake)  //在find_package前面

4. python调用cv_bridge相关的工作，在 ~/.bashrc 中添加：
>> export LD_LIBRARY_PATH=/usr/local/lib/


**NOTE:**
1）CMakeLists.txt中没有调 cv_bridge 包，但在 manifest.xml 文件中调用 cv_bridge 包时，出现调用不同opencv版本问题，可能会引起 seg fault（段错误）
>> 解决办法：将 /usr/local/share/cv_bridge 和 /usr/local/lib/libcv_bridge.so 等替换 /opt/ros/melodic/ 中对于文件夹和文件
$ sudo cp -r /usr/local/share/cv_bridge /opt/ros/melodic/share/
$ sudo cp /usr/local/lib/libcv_bridge.so /opt/ros/melodic/lib/
$ sudo cp /usr/local/lib/pkgconfig/cv_bridge.pc /opt/ros/melodic/lib/pkgconfig/
$ sudo cp -r /usr/local/lib/python2.7/dist-packages/cv_bridge /opt/ros/melodic/lib/python2.7/dist-packages/
$ sudo cp /usr/local/lib/python2.7/dist-packages/cv_bridge-1.13.1.egg-info /opt/ros/melodic/lib/python2.7/dist-packages/
$ sudo cp -r /usr/local/include/cv_bridge /opt/ros/melodic/include/

