# UR5 REALSENSE CALIBRATION


### Requires

Make sure the following environment:

* ROS Noetic
* MoveIt 
* librealsense & realsense-ros

### Prepare
1. Build catkin workspace

```
mkdir -p ~/catkin_ws/src
cd  ~/catkin_ws
catkin_make
```

2. Install package dependencies:

```
$ sudo apt install ros-noetic-visp
$ sudo apt install ros-noetic-industrial-msgs
$ sudo apt install ros-noetic-ros-control ros-noetic-ros-controllers
$ sudo apt install ros-noetic-realsense2-camera
$ sudo apt install ros-noetic-rgbd-launch
```
Setup your UR robot packages following the readme [here](https://github.com/montrealrobotics/REAL_ur_robots)


3. Download the calibration lib
```
$ cd ~/catkin_ws/src
$ git clone https://github.com:montrealrobotics/ur5_realsense_calibration.git
```
or
```
$ git clone https://github.com:montrealrobotics/ur5_realsense_calibration.git
$ cd  ur5_realsense_calibration
$ git submodule foreach --recursive git submodule init
$ git submodule foreach --recursive git submodule update
```
4. Perform catkin make
```
$ catkin_make
```

:star: Before excuting roslaunch command, you should remember to perform the cmd below, **Always remember it !**

```
cd ~/catkin_ws
source ./devel/setup.bash
```
or a convenient way（if you have at least two workspaces, not recommended)
```
$ echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
$ source ~/.bashrc
```

### Aruco markers
You can print markers fom [here](https://tn1ck.github.io/aruco-print/). The default marker ID specified in the launch file is 582 and the size is 0.1m.

### Excution

* launch communication with real ur5 robot using IP
```
$ roslaunch real_ur_robots_bringup real_ur5e_bringup.launch sim:=false robot_ip:=192.168.1.102
```

* launch moveit 
```
$ roslaunch ur5e_moveit_config moveit_planning_execution.launch sim:=false
```
* launch point cloud
```
$ roslaunch realsense2_camera rs_rgbd.launch
```
* launch calibration program
```
$ roslaunch easy_handeye ur5_realsense_handeyecalibration.launch
```
Arguments can be added to this launch to change the marker size or the marker ID. For example:
```
$ roslaunch easy_handeye ur5_realsense_handeyecalibration.launch marker_size:=0.15 marker_id:=0
```

After that, you will see **three** GUI following(make sure your librealsense was installed at single path and the version accord with environment):

1. ![Image text](./images/automatic_movement.png)

2. ![Image text](./images/easy_handeye.png)

3. ![Image text](./images/rviz.png)

In the GUI window 'rqt_easy_handeye.perspective -rqt': 

Plugins ->  Visualization -> Image View
(you may maximize this window to see the options on the left-top)

![Image text](./images/rqt_image_viewer.jpg)

select "/aruco_tracker/result" in listbox

Let the  broad in the center of your camera screen

![Image text](./images/center_of_screen.jpg)

See the Calibrator automatic movement 

Begin start, Pleas slow down the speed of your robot

![Image text](./images/automatic_movement.png)

Click Check starting Pose Button

Take (Next Pose) (Plan) (Excute) (Take Sample) as a loop

** Just remember that, robot move a new pose than Take  Sample **

** If board move out of sight, do not press (Take Sample) button, it may occur bug **

** Plz take more than 8 sample, then press (Compute) button to get 7-element-matrix **

 Save ! Finish!(The .yaml file will save at ~/.ros/easy_handeye/)

![Image text](./images/loop.png)

You can have a look by publishing your calibration result.

1. Edit launch file and roslaunch it(without close other program window).
 1.1 copy parameters in .yaml to ur5_realsense_handeyecalibration_after2.launch(arg:x,y,z,qx,qy,qz,qw)
 ```
 sudo gedit ~/catkin_ws/src/ur5_realsense_calibarion/easy_handeye/easy_handeye/launch/ur5_realsense_handeyecalibration_after2.launch
 sudo gedit ~/.ros/easy_handeye/ur5_realsense_handeyecalibration_eye_on_base.yaml
 ```
 ![Image text](./images/copy_matrix_to_launch1.png)
 ![Image text](./images/copy_matrix_to_launch2.png)
 
 1.2 now roslaunch the launch file
```
(do not run this command)roslaunch  easy_handeye  publish.launch eye_on_hand:=false namespace_prefix:=ur5_realsense_handeyecalibration
(run this)roslaunch easy_handeye ur5_realsense_handeyecalibration_after2.launch 
```
2.Add TF to see calibration result(in rviz)
 if you find TF flash in rviz
```
sudo gedit ~/catkin_ws/src/ur5_realsense_calibarion/easy_handeye/easy_handeye/launch/calibrate.launch
```
comment out this code patch and re-launch
![Image text](./images/calibrate_launch_comment_out.png)

using easy_handeye publish result

```
roslaunch easy_handeye publish.launch eye_on_hand:=true

```


### Trouble Shooting

1. ERROR: 'module 'catkin_pkg' not found', try command below to install the corresponding package.
```
sudo pip install -U catkin_pkg
```
In case you experience error with command above, you might want to run it without -U option. see [ROSWiki](http://wiki.ros.org/catkin_pkg)
 
2. Revise the content of ur5_realsense_handeyecalibration.launch
[ur5_realsense_handeyecalibration.launch](https://github.com/portgasray/easy_handeye/blob/5ee30dd50f250452cdc56bfe8f4a7597f9d0b6d6/easy_handeye/launch/ur5_realsense_handeyecalibration.launch) in easy_handeye/launch changed for calibration, you may need to revise to yourself. 

3. Built target ddynamic_reconfigure
$ sudo apt-get install ros-kinetic-ddynamic_reconfigure

4. ~/calibration_ws/build/ur5_realsense_calibarion/aruco_ros/aruco_ros/setup_custom_pythonpath.sh: 5: exec: ~/calibration_ws/src/ur5_realsense_calibarion/aruco_ros/aruco_ros/cfg/ArucoThreshold.cfg: Permission denied

```
chmod +x [relevent .cfg]
```
