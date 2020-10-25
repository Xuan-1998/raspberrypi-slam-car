# Raspberrypi SLAM car operation guide

> Xuan Jiang  UC Berkeley

# Source code

> The source code is in the Raspberry Pi operating system

python source code:

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/script

C source code:

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/src

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/launch

## Install the virtual machine software VMware

## Download virtual machine system files

Open Vmware, select `Open Virtual Machine`

find `Ubuntu 64 位.vmx`

Shows that the virtual machine seems to be in use...click `take ownership`

Select the bridge mode for the network adapter, check the copy physical network connection status

open virtual machine

I have copied the virtual machine

Unable to connect to virtual device: yes

confirm confrim

## Open the virtual machine Ubuntu system

username: CLB，password: 123456

View-Automatically resize-Automatically adapt to clients Automatically adapt to windows

install: vmware-tools

shortcut：

ctrl + shift + t New command line tab

ctrl + alt + t New command line window

## Build raspberry pi iso

Download the image building tool Etcher or win32img

Download the image file, distinguish 3B and 3B+

Build it into sd card

# Turn on the car

Insert the SD card into the trolley, the power-on voltage should be above 12V, and the display will display when the screen is loaded.

Use flying mouse to set up wifi on the display, computer and raspberry pi are connected to the same wifi, use ifconfig to check the ip address or use LAN ip scanner to check the raspberry pi ip address.

Find the ip address of the Raspberry Pi, for example mine is 192.168.200.29

Enter in the virtual machine

```
sudo gedit /etc/hosts
```

> This file shows the mapping between domain names and URLs. The URL appearing in this file does not need to use the DNS protocol when accessing it, but directly reads the domain name.

Set the ip address in front of the robot to the ip address of the Raspberry Pi

ctrl + s ，close the window

Enter in the virtual machine

```
ssh clbrobot@robot
```

After logging in, enter

```
clbrobot@robot~$ roslaunch clbrobot bringup.launch
```

Start the master node

```
CLB@CLB sudo gedit ~/.bashrc
```

Replace the last line with the ip address of the Raspberry Pi to make the Raspberry Pi point to the master node

```
CLB@CLB source ~/.bashrc
```

Let the changes take effect

```
CLB@CLB rosrun rviz rviz
```

Start the rviz debug window

# Operate the robot

```
CLB@CLB ssh clbrobot@robot
clbrobot@robot sudo nano ~/.bashrc
```

Make sure the last parts are tank and rplidar

> rplidar refers to the laser radar produced by Silan

# Control the robot quickly

```
clbrobot@robot rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

# Shutdown

```
clbrobot@robot sudo halt
```

Chuanglebo logo appears on the display, and it can be shut down

# Restart the robot

Start the virtual machine

```
CLB@CLB ssh clbrobot@robot
```

Start the chassis node

```
clbrobot@robot roslaunch clbrobot bringup.launch
clbrobot@robot roscd clbrobot/
```

Enter the project directory

```
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot$ cd param

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param$ ls
ekf  imu  navigation
```

## Calibrate the IMU

Based on the previous step

```
cd imu

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param$ cd imu

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ ls
imu_calib.yaml

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ sudo nano imu_calib.yaml 
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ rostopic echo /imu/data

##Turn on the real-time gyroscope

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ rosrun imu_calib do_calib
##Turn the different sides up in turn and press Enter to calibrate
```

## Angular velocity correction

```
clbrobot@robot $ rosrun rikirobot_nav calibrate_angular.py
CLB@CLB:~$ rosrun rqt_reconfigure rqt_reconfigure 
```

Open the imu calibration interface

Click start_test to perform a 360-degree steering test. If it turns 365 degrees, fill in odom_angular_scale_coreection with 365/360=1.014, and test again

Turn off all command lines, including the master node

```
clbrobot@robot:~$ roscd clbrobot/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot$ cd launch/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/launch$ sudo nano bringup.launch
```

Find the following line

```
 <param name="angular_scale" value="0.986" />
```

Pass the measured value into value, save and exit

## Linear speed correction

SSH to the Raspberry Pi, start the master node

```
clbrobot@robot~$ rosrun rikirobot_nav calibrate_linear.py
CLB@CLB:~$ rosrun rqt_reconfigure rqt_reconfigure 
```

Choose `calibrate linear`

modify odom_linear_scale_correction

```
clbrobot@robot:~$ roscd clbrobot/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot$ cd launch/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/launch$ sudo nano bringup.launch
```

Open the xml configuration file

modifure the value

```
 <param name="linear_scale" type="double" value="1.045" />
```

# Create a map

ssh to the raspberry pi, open the master node

```
clbrobot@robot:~$ roslaunch clbrobot lidar_slam.launch
```

Open rviz debug window

```
CLB@CLB:~$ rosrun rviz rviz
```

open config

load  /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

Keyboard control robot

```
clbrobot@robot rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

speed sets to 0.3，turn sets to 0.5

## Save the map

```
clbrobot@robot:~$ roscd clbrobot
cd maps
ls
./map.sh
ls -l 即可看到更新后的hous.pgm和house.yaml文件
```

# Automatic navigation

ssh to Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot navigate.launch
CLB@CLB rosrun rviz rviz
```

load

/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/navigate.rviz

# Mouse widget map

ssh to the Raspberry Pi, start the chassis node

launch slam node：

```
clbrobot@robot:~$ roslaunch clbrobot lidar_slam.launch

CLB@CLB rosrun rviz rviz
```

load /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

use 2D Nav Goal to select target directly

# Select area to build a map

ssh to the Raspberry Pi, start the chassis node

launch the auto_slam node：

```
clbrobot@robot:~$ roslaunch clbrobot auto_slam.launch

CLB@CLB rosrun rviz rviz
```

load /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

Use the Publish Point in the toolbar to outline a closed area

# Use hector algorithm to build a map

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot hector_slam.launch

CLB@CLB rosrun rviz rviz
```

load /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

hector algorithm is suitable for open areas, where there will be ghosting in a small space

# Multipoint navigation

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot navigate_multi.launch

CLB@CLB rosrun rviz rviz
```

load /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/multi_goal.rviz

Use publish point tool to set target point

Multipoint navigation source code:

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/script

# Dynamic PID parameter adjustment

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot rosrun riki_pid pid_configure

CLB@CLB rosrun rqt_reconfigure rqt_reconfigure
```

source code：

/home/rikirobot/catkin_ws/src/clbrobot_project/riki_pid

cfg：Default loading parameters

src：Configuration file

Recompile after modification to take effect

# Camera hunting

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot camera.launch

CLB@CLB roslaunch riki_line_follower riki_line.launch
```

The camera is turned on and the red line is recognized

source code

/home/rikirobot/catkin_ws/src/clbrobot_project/riki_line_follower

# Radar follow

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot camera.launch

clbrobot@robot roslaunch riki_lidar_follower laser_follower.launch
```

source code

/home/rikirobot/catkin_ws/src/clbrobot_project/riki_lidar_follower

# karto algorithm to build a map

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot karto_slam.launch

CLB@CLB rosrun rviz rviz
```

load /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

Open keyboard control on virtual machine

```
CLB@CLB rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

The default linear velocity is 0.5 and the angular velocity is 1.0

Because the karto algorithm is more complicated, adjust the linear velocity to about 0.3 and the angular velocity to 0.5.

# Android mobile app and image monitoring

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot appcamera.launch
```

Install the .apk file on the phone, create a new robot, change localhost to the robot's ip address, the default port is 11311

# opencv Image Processing

source code

/home/rikirobot/catkin_ws/src/opencv_apps

View source code:

```
CLB@CLB roscd opencv_apps
cd src
cd nodelet
ls
```

ssh to raspberry pi

```
clbrobot@robot roslaunch clbrobot camera.launch



CLB@CLB roslaunch opencv_apps edge_detection.launch #Edge extraction

CLB@CLB roslaunch opencv_apps hough_lines.launch #Line detection
CLB@CLB roslaunch opencv_apps find_contours.launch #Detect contour
CLB@CLB roslaunch opencv_apps convex_hull.launch #Convex hull detection
CLB@CLB roslaunch opencv_apps general_contours.launch #Oval contour detection
CLB@CLB roslaunch opencv_apps people_detect.launch #Pedestrian detection
CLB@CLB roslaunch opencv_apps goodfeature_track.launch #Sharp corner detection
CLB@CLB roslaunch opencv_apps camshift.launch #Target Tracking
CLB@CLB roslaunch opencv_apps fback_flow.launch #Object movement detection
CLB@CLB roslaunch opencv_apps lk_flow.launch #movement detection2
CLB@CLB roslaunch opencv_apps simple_flow.launch #movement detection3
CLB@CLB roslaunch opencv_apps phase_corr.launch #Phase shift
CLB@CLB roslaunch opencv_apps segment_objects.launch #Object segmentation
CLB@CLB roslaunch opencv_apps rgb_color_filter.launch #Turn black and white
CLB@CLB roslaunch opencv_apps hls_color_filter.launch #Turn black and white
CLB@CLB roslaunch opencv_apps hsv_color_filter.launch #Turn black and white
```

# Android mobile app to build a map

Download on mobile`Make Map.apk`

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot lidar_slam.launch

clbrobot@robot roslaunch clbrobot camera.launch
```

Open the mobile app, set localhost as the robot ip address, click connect

# Android mobile app navigation

Download on mobile`Map Nav`

ssh to the Raspberry Pi, start the chassis node

```
clbrobot@robot roslaunch clbrobot navigate.launch

clbrobot@robot roslaunch clbrobot camera.launch

CLB@CLB rosrun rviz rviz
```

load /home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/navigate.rviz

Open the mobile app, set localhost as the robot ip address, click connect

You can specify the position of the robot, set pose, or navigate the set goal

When setting, long press and drag on the screen to indicate the direction

Only the nearest map is displayed on the app. As the location moves, the map will update accordingly

At the same time, the car will automatically avoid obstacles in the way and is very intelligent.

