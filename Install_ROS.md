# Raspbian Buster Image
Raspbian Buster image can be installed in the official [Raspberry Pi site](https://www.raspberrypi.org/downloads/raspbian/).

# Bootable Drive
Bootable Drive are made using 32gb micro sd card with [balena ethcer](https://www.balena.io/etcher/)

# Headless Raspberry Pi
After the bootable drive are created. Some files need to be add so that we can directly ssh to raspberry pi. This simplifies the setup process where there is no need for monitor. The setup process can be read [here](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md).

# Connecting to Raspberry Pi

User can connect to Raspberry Pi through ssh using [putty](https://www.putty.org/)

    Hostname : raspberrypi.local
    Port : 22

# ROS Installation

Here are the whole process of installing ROS Melodic in Raspberry pi

### Add ROS into source list
    sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

    sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

### Update software package

    sudo apt-get update

### Install dependencies
    sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake

### Update rosdep

    sudo rosdep init
    rosdep update
    
### Create workspace to build ROS

    mkdir ~/ros_catkin_ws
    cd ~/ros_catkin_ws

### Dowload all ROS package

These command will download ros-melodic-destop package. For other download option, can see [here](http://wiki.ros.org/melodic/Installation/Source)

    rosinstall_generator desktop --rosdistro melodic --deps --wet-only --tar > melodic-desktop-wet.rosinstall

    wstool init -j8 src melodic-desktop-wet.rosinstall

if the download interrupted, you can resume with following command

    wstool update -j 4 -t src

### Fix some dependency problems

#### collada-urdf dependency issue

    mkdir -p ~/ros_catkin_ws/external_src
    cd ~/ros_catkin_ws/external_src

    wget http://sourceforge.net/projects/assimp/files/assimp-3.1/assimp-3.1.1_no_test_models.zip/download -O assimp-3.1.1_no_test_models.zip

    unzip assimp-3.1.1_no_test_models.zip
    cd assimp-3.1.1
    cmake .
    make
    sudo make install

#### libboost issue

Can refer [here](https://stackoverflow.com/a/53382269) for the fix.
This does not cover here because it seems to be fixed.

### Install the rest of dependencies

    cd ~/ros_catkin_ws
    rosdep install --from-paths src --ignore-src --rosdistro melodic -y

### Build catkin package

    sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic -j2

If the compilation freezes, you need to increase the swap space. By default it's 100Mb, try increasing it to 2048Mb

This installation are done with Rasberry Pi 4 with 4gb of ram. Should be fine.

### Test your build

    echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
    roscore

`roscore` should run successfully