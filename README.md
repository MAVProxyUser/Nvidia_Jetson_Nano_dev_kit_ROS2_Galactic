```
Waveshare JETSON-NANO-DEV-KIT (16gb EMMC) with JetPack 4.5.1 (rev1)
https://www.waveshare.com/jetson-nano-dev-kit-a.htm
https://www.amazon.com/Jetson-Developer-onboard-Machine-Learning/dp/B09Y94MGRZ
https://developer.nvidia.com/embedded/jetpack-sdk-46
https://developer.nvidia.com/embedded/jetpack-sdk-451-archive

Switch to USB boot:

root@ubuntu:/home/ubuntu# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p1   14G  4.9G  8.2G  38% /

ubuntu@ubuntu:~/rootOnUSB$ sudo apt-get install nano -y 

ubuntu@ubuntu:~$ git clone https://github.com/JetsonHacksNano/bootFromUSB.git
ubuntu@ubuntu:~$ cd bootFromUSB/
ubuntu@ubuntu:~/bootFromUSB$  mkdir /tmp/z
ubuntu@ubuntu:~/librealsense/bootFromUSB$ sudo mkfs.ext4 /dev/sda1
ubuntu@ubuntu:~/bootFromUSB$ sudo mount /dev/sda1 /tmp/z
ubuntu@ubuntu:~/bootFromUSB$ ./copyRootToUSB.sh -p /dev/sda1 
ubuntu@ubuntu:~/bootFromUSB$ ./partUUID.sh 
ubuntu@ubuntu:~/bootFromUSB$ sudo pico /boot/extlinux/extlinux.conf 
ubuntu@ubuntu:~/bootFromUSB$ sudo reboot 

ubuntu@ubuntu:~$ df -h / 
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       116G  5.0G  105G   5% /

Upgrade Swap RAM:

ubuntu@ubuntu:~$ free -h 
              total        used        free      shared  buff/cache   available
Mem:           3.9G        595M        2.8G         19M        480M        3.1G
Swap:          1.9G          0B        1.9G

ubuntu@ubuntu:~$ git clone https://github.com/JetsonHacksNano/resizeSwapMemory
ubuntu@ubuntu:~$ cd resizeSwapMemory/
ubuntu@ubuntu:~/resizeSwapMemory$ ./setSwapMemorySize.sh -g 16
ubuntu@ubuntu:~/resizeSwapMemory$ sudo reboot

ubuntu@ubuntu:~$ free -h 
              total        used        free      shared  buff/cache   available
Mem:           3.9G        716M        2.7G         19M        479M        3.0G
Swap:           15G          0B         15G


Update apt, and upgrade the system, clean unused packages, and purge left overs:

ubuntu@ubuntu:~$ sudo apt-get update; sudo apt upgrade -y
ubuntu@ubuntu:~$ sudo apt autoremove -y
ubuntu@ubuntu:~$ sudo apt-get purge $(dpkg -l | grep '^rc' | awk '{print $2}') -y

Compile libraries for the Intel Realsense: 

ubuntu@ubuntu:~$ git clone https://github.com/IntelRealSense/librealsense.git 
ubuntu@ubuntu:~$ cd librealsense/
ubuntu@ubuntu:~/librealsense$ ./scripts/patch-realsense-ubuntu-L4T.sh  
ubuntu@ubuntu:~/librealsense$ mkdir build
ubuntu@ubuntu:~/librealsense$ cd build/
ubuntu@ubuntu:~/librealsense/build$ cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=release -DFORCE_RSUSB_BACKEND=false -DBUILD_WITH_CUDA=false && make -j$(($(nproc)-1)) && sudo make install
ubuntu@ubuntu:~/librealsense/build$ cd ..
ubuntu@ubuntu:~/librealsense$ sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
ubuntu@ubuntu:~/librealsense$ sudo udevadm control --reload-rules && udevadm trigger
ubuntu@ubuntu:~/librealsense/build$ rs-enumerate-devices 


Install a bunch of stuff for compiling ROS2:

ubuntu@ubuntu:~$ sudo apt-get install bison libzmq3-dev  libzmqpp-dev libczmq-dev  
libfreeimage-dev libfreeimageplus-dev openjdk-11-jre-headless libboost-program-options-dev 
libgvc6 libcgraph6 libcdt5 libgraphviz-dev liboctovis-dev bzip2 graphviz libprotobuf-dev 
libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev 
libiberty-dev autoconf llvm git build-essential qt5-default bison libacl1-dev curl gnupg2 
lsb-release build-essential git wget python3-rosdep libopencv-dev libzmq3-dev   
libgtest-dev cmake libacl1-dev libsqlite3-dev libtinyxml2-dev libbullet-dev   
libncurses-dev libbison-dev libcurl4-openssl-dev bison liblog4cxx-dev libeigen3-dev 
libasio-dev python3-dev libboost-all-dev libglib2.0-dev libprotobuf-dev   
libprotoc-dev protobuf-compiler protobuf-compiler-grpc libasound2-dev   
libconsole-bridge-dev libmpg123-dev libv4l-dev libssl-dev python3-pip libx11-dev libxext-dev 
libxtst-dev libxrender-dev libxmu-dev libxmuu-dev libgl1-mesa-dev libglu1-mesa-dev 
freeglut3-dev libboost-all-dev libeigen3-dev libflann-dev libglew-dev libpcap-dev 
libusb-1.0-0-dev libopenni-dev libopenni2-dev clang-format libqhull-dev  libpng-dev 
libxslt1-dev libxml2-dev nano libxrandr-dev libxinerama-dev libxcursor-dev 
git cmake libssl-dev freeglut3-dev libusb-1.0-0-dev pkg-config libgtk-3-dev libxaw7-dev python3-pyqt5 
sip-dev pyqt5-dev python3-sip python3-sip-dev pyqt5-dev-tools libglfw3 -y

ubuntu@ubuntu:~$  pip3 install Cython
ubuntu@ubuntu:~$  pip3 install vcstool argcomplete flake8 flake8-blind-except   
flake8-builtins flake8-class-newline flake8-comprehensions flake8-deprecated   
flake8-docstrings flake8-import-order flake8-quotes pytest-repeat pytest-rerunfailures   
pytest pytest-cov pytest-runner lark ifcfg netifaces boost numpy rosdep lxml 
colcon-common-extensions importlib-resources

Manually compile VTK & PCL libs because Ubuntu packages are not new enough: 

ubuntu@ubuntu:~$ git clone https://github.com/Kitware/VTK.git -b v8.2.0
ubuntu@ubuntu:~$ cd VTK
ubuntu@ubuntu:~$ mkdir build; cd build
ubuntu@ubuntu:~/VTK/build$ cmake -DVTK_USE_SYSTEM_PNG=ON ..
ubuntu@ubuntu:~/VTK/build$ make -j4
ubuntu@ubuntu:~/VTK/build$ sudo make install 

ubuntu@ubuntu:~$ git clone https://github.com/PointCloudLibrary/pcl.git -b pcl-1.11.1
ubuntu@ubuntu:~$ cd pcl
ubuntu@ubuntu:~/pcl$ mkdir build; cd build
ubuntu@ubuntu:~/pcl/build$ cmake ..
ubuntu@ubuntu:~/pcl/build$ make -j4
ubuntu@ubuntu:~/pcl/build$ sudo make install

Pulldown the Xiaomi ROS2 repo for Cyberdog (modified ROS2 galactic for Ubuntu 20, instead of Ubuntu 18):

ubuntu@ubuntu:~$ git clone https://github.com/MiRoboticsLab/cyberdog_ros2.git
ubuntu@ubuntu:~$ mkdir -p ros_src/src
ubuntu@ubuntu:~$ cp cyberdog_ros2/tools/ros2_fork/* ros_src
ubuntu@ubuntu:~$ cd ros_src
ubuntu@ubuntu:~/ros_src$ export PATH=$PATH:/home/ubuntu/.local/bin/
ubuntu@ubuntu:~/ros_src$ vcs import src < mini.repos

Fix a small error in the repo: 

ubuntu@ubuntu:~/ros_src$ cat > fix_extend.diff
diff --git a/extend.repos b/extend.repos
index 6ae5252..d7828e3 100644
--- a/extend.repos
+++ b/extend.repos
@@ -23,7 +23,7 @@ repositories:
     type: git
     url: https://github.com/ros-planning/navigation_msgs.git
     version: 2.1.0
-   ros-visualization/interactive_markers:
+  ros-visualization/interactive_markers:
     type: git
     url: https://github.com/
^C

ubuntu@ubuntu:~/ros_src$ cat fix_extend.diff | patch -p1
ubuntu@ubuntu:~/ros_src$ vcs import src < extend.repos

Set environment variables for the compile:

ubuntu@ubuntu:~/ros_src$ export ROS_VERSION=2
ubuntu@ubuntu:~/ros_src$ sudo mkdir -p /opt/ros2/galactic
ubuntu@ubuntu:~/ros_src$ sudo chown $USER /opt/ros2/galactic
ubuntu@ubuntu:~/ros_src$ export OPENBLAS_CORETYPE=ARMV8
ubuntu@ubuntu:~/ros_src$ colcon build --parallel-workers 20 --event-handlers console_direct+
...

When compile is complete is should have compiled 288 packages, ignore the stderr output. 

Summary: 288 packages finished [3h 40min 24s]
  79 packages had stderr output: ament_clang_format ament_clang_tidy ament_copyright ament_cppcheck ament_cpplint ament_flake8 ament_index_python ament_lint ament_lint_cmake ament_mypy ament_package ament_pclint ament_pep257 ament_pycodestyle ament_pyflakes ament_uncrustify ament_xmllint domain_coordinator examples_tf2_py foonathan_memory_vendor google_benchmark_vendor iceoryx_posh launch launch_ros launch_testing launch_testing_ros launch_xml launch_yaml mimick_vendor osrf_pycommon rcl_lifecycle rcutils ros2action ros2bag ros2cli ros2component ros2doctor ros2interface ros2launch ros2lifecycle ros2multicast ros2node ros2param ros2pkg ros2run ros2service ros2test ros2topic ros2trace rosidl_cli rosidl_runtime_py rpyutils rqt rqt_action rqt_bag rqt_bag_plugins rqt_console rqt_graph rqt_gui rqt_gui_py rqt_msg rqt_plot rqt_publisher rqt_py_console rqt_reconfigure rqt_service_caller rqt_shell rqt_srv rqt_top rqt_topic sensor_msgs_py sros2 test_launch_ros tf2_ros_py tf2_tools tracetools_launch tracetools_read tracetools_trace uncrustify_vendor

ubuntu@ubuntu:~/ros_src$ colcon list | wc -l
288

Install ROS realsense & RPLidar
ubuntu@ubuntu:~$ cd ~/ros_src/src/
ubuntu@ubuntu:~/ros_src/src$ git clone  https://github.com/IntelRealSense/realsense-ros.git
ubuntu@ubuntu:~/ros_src/src$ git clone https://github.com/ros-perception/vision_opencv.git -b 2.2.1
ubuntu@ubuntu:~/ros_src/src$ git clone https://github.com/ros/diagnostics.git -b galactic 
ubuntu@ubuntu:~/ros_src/src$ git clone https://github.com/Slamtec/rplidar_ros.git -b ros2
ubuntu@ubuntu:~/ros_src/src$ cd ../
ubuntu@ubuntu:~/ros_src/src$ colcon build --parallel-workers 20 --event-handlers console_direct+ --packages-skip-build-finished

Source the ros2 install!
ubuntu@ubuntu:~/ros_src$ . install/local_setup.bash

Lauch realsense cam:
ubuntu@ubuntu:~/ros_src$ realsense-viewer
ubuntu@ubuntu:~/ros_src$ ros2 launch realsense2_camera rs_launch.py

Launch RPLidar s1:
ubuntu@ubuntu:~/ros_src$ sudo chmod 777 /dev/ttyUSB0 
ubuntu@ubuntu:~/ros_src$ ros2 launch rplidar_ros2 rplidar_s1_launch.py

Launch RPLidar s1 with rviz2:
ubuntu@ubuntu:~/ros_src$ ros2 launch rplidar_ros2 view_rplidar_s1_launch.py
```
