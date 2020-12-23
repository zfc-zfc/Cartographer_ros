# Cartographer-using-Delta_III-Lidar
[TOC]

## Install Cartographer
```bash
sudo apt install ros-melodic-cartographer
sudo apt install ros-melodic-cartographer-ros
sudo apt install ros-melodic-cartographer-ros-msgs
```
Then you can find these three packages in  `/opt/ros/melodic/share` . If not, type `rospack find packagename` in the terminal to find the path.

### Run a demo bag
#### Install a bag
```
https://storage.googleapis.com/cartographer-public-data/bags/backpack_2d/cartographer_paper_deutsches_museum.bag
```
Copy this link to download the demo bag（470MB）named  `cartographer_paper_deutsches_museum.bag`. The default saving path is `/home/username/Downloads`
#### Run the bag
```bash
roslaunch cartographer_ros demo_backpack_2d.launch bag_filename:=${HOME}/Downloads/cartographer_paper_deutsches_museum.bag
```
You could see the real-time laser scan data in rviz.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223164011157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)
Click the "Add" button, add an topic named "/map", the you can see the real-time updated map as follow.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223164128517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)

This implies you have installed cartographer successfully.

## Start the LiDAR

Weblink of Delta_III Lidar produced by 3irobotix:
```
http://www.3irobotics.com/pro_s.php?id=168
```
To install the driver and make the Lidar run，please reference **delta_III_ros**

### Acquire the needed info of Lidar
Start the Lidar
```
roslaunch delta_lidar view_delta_lidar.launch 
```
You could see the real-time laser scan data in rviz, then type `rostopic list` to acquire the topic name.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223165054327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)

Here, Delta_III publishes a topic named **/scan**
Print the information of /scan
```
rostopic echo /scan
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223165257268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)

Find the frame_id: “laser”. This will be used later.
## Modify Files
### Modify .lua
Find the file named **revo_lds.lua** in `/opt/ros/melodic/share/cartographer_ros/launch` , copy it and rename as `my_revo_lds.lua`
Constrained by the **Insufficient Permissions**, you could complete the tasks as follow:
Open a new terminal and cd `/opt/ros/melodic/share/cartographer_ros/launch`, then:
```bash
sudo cp revo_lds.lua my_revo_lds.lua
```
Modify the file:
```bash
sudo gedit my_revo_lds.lua
```
Open `my_revo_lds.lua`, implement the following modification:
```cpp
//Change the original frame "horizontal_laser_link" into our "laser"
tracking_frame = "laser",
published_frame = "laser",
```
### Modify .launch
Find the file named **demo_revo_leds.launch** in `/opt/ros/melodic/share/cartographer_ros/configuration_files`, copy it and rename as my_demo_revo_lds.launch
修改以下内容:
```cpp
//Since it's not bag simulation, set the following value as false.
<param name="/use_sim_time" value="false" />

//Use our own configuration: change revo_lds.lua into my_revo_lds.lua
<node name="cartographer_node" pkg="cartographer_ros"
      type="cartographer_node" args="
          -configuration_directory $(find cartographer_ros)/configuration_files
          -configuration_basename my_revo_lds.lua"

//Change horizontal_laser_2d into our topic scan
<remap from="scan" to="scan" />

//Delete the following codes.
<node name="playbag" pkg="rosbag" type="play"
      args="--clock $(arg bag_filename)" />
```

## Mapping
Firstly start the lidar and assure that there is effictive output laser data topic.
Then
```bash
roslaunch cartographer_ros my_demo_revo_lds.launch
```
Start to construct a occupancy grid map..

## Save the map
Open another terminal and type: 

//End the first trajectory, and no further data will be added.
`rosservice call /finish_trajectory 0`


//Generate a pbstream file
`rosservice call /write_state "{filename: '${HOME}/Downloads/mymap.pbstream'}"`

//transform pbstream to pgm and yaml
```
rosrun cartographer_ros cartographer_pbstream_to_ros_map -map_filestem=/home/kai/Downloads/mymap -pbstream_filename=/home/kai/Downloads/mymap.pbstream -resolution=0.05
```
The address and file name could be modified according to your preference.

