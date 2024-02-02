# tiago_openday_static_obst_avoidance

## Table of Contents
* [General info](#general-info)
* [Setup](#setup)
    - [TIAGo Workspace](#tiago-workspace)
    - [Acados](#acados)
    - [Scikit-learn](#scikit-learn)
* [Launch](#launch)
    - [Gazebo Simulation](#gazebo-simulations)
    - [Real Robot Experiment](#real-robot-experiment)

## General info
Demo for DIAG Open Day in which TIAGo avoids static obstacles using laser measurements and MPC. The Constraints of the optimization procedure are written in terms of Discrete Time Control Barrier Functions (DT-CBF). The communication with the robot is establish through wifi and the commands are published on a ROS topic using ROs messages. 
	
## Setup
For running this project, the aforemetnioned lybraries must be installed, as well as the catkin workspace of the robot

### TIAGo workspace
All information are in the [TIAGo ROS tutorial](http://wiki.ros.org/Robots/TIAGo/Tutorials/Installation/InstallUbuntuAndROS). The code has been tested for ROS noetic in ubuntu 20.04

### Acados
The installation guide can be found at [this page](https://docs.acados.org/installation/). Clone acados and its submodules by running:
```
git clone https://github.com/acados/acados.git
cd acados
git submodule update --recursive --init
cd acados
mkdir -p build
cd build
cmake -DACADOS_PYTHON=ON ..
make -j4
sudo make install -j4
```
for the python interface run the following commands from the `build` folder
```
pip install -e ../interfaces/acados_template
```
Note: The option `-e` makes the installation editable, so you can seamlessly switch to a later `acados` version and make changes in the Python interface yourself.

Add the path to the compiled shared libraries `libacados.so`, `libblasfeo.so`, `libhpipm.so` to `LD_LIBRARY_PATH` (default path is <acados_root/lib>) by running:
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:"<acados_root>/lib"
export ACADOS_SOURCE_DIR="<acados_root>"
```
you can put them directly in the `~/.bashrc` file.

### Scikit-learn
The program makes use of the predictive data analysis scikit-learn. You can install it by simply running the following command
```
pip install -U scikit-learn
```

  
## Launch
In order to launch the simulation, you need to source the tiago_ws commands by running `source <tiago_publi_ws root>/devel/setup.bash`.

At this point for compiling the workspace in the *Release* mode, use the command
```
catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
```
and then compile using the command
```
catkin build
```

### Gazebo Simulations
Once the compilation has succeed the gazebo simulation starts through the command
```
roslaunch tiago_openday_static_obst_avoidance tiago_gazebo.launch world:=<WORLD>
```
The previous command will launch the `<WORLD>.world` file in the `labrob_gazebo_worlds/worlds` package that you can obtain [here](https://github.com/DIAG-Robotics-Lab/labrob_gazebo_worlds) cloning the repository
```
git clone git@github.com:DIAG-Robotics-Lab/labrob_gazebo_worlds.git
``` 

Then the nmpc and object detection module must be started
```
roslaunch tiago_openday_static_obst_avoidance setup_controller.launch
```
A desired position can be send to the robot using
```
roslaunch tiago_openday_static_obst_avoidance send_desired_target_position.launch x_des:=<X> y_des:=<Y>
```

### Real Robot Experiment
Connect to the robot through the wifi and run the following command
```
export ROS_MASTER_URI=http://tiago-<NUMBER>c:<PORT>
export ROS_IP=10.68.0.128
rosservice call /pal_navigation_sm "input: 'MAP'" # if map not found
rosrun rviz rviz -d `rospack find tiago_2dnav`/config/rviz/advanced_navigation.rviz
rosservice call /pal_navigation_sm "input: 'LOC'" # if not already localized
```
Once the robot is in the desired initial position you will start the experiment in the same way as before