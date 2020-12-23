## 1.How to build 3iRobotics lidar ros package:

1. Clone this package to your catkin's workspace src folder
2.  Please select correct serial in “src/node.cpp”：/dev/ttyUSB0(default)
3.  `cd catkin`
4.  `catkin_make` (to build delta_lidar_node and delta_lidar_node_client)
5.  `source devel/setup.bash`
6.  `sudo chmod 777 /dev/ttyUSB0`

## 2.How to run 3iRobotics lidar ros package:	

#### 1st step:

```
roslaunch delta_lidar delta_lidar.launch
```

the launch file listed above includes publish-node and subscribe_node.

#### 2nd step:

```
rosrun rviz rviz
```

#### 3rd step:

Add "LaserScan", then type **laser** into the Fixed Frame . Then you can see the laser scan info on the grid.



## 3. Directly run all

`view_delta_lidar.launch`   includes `delta_lidar.launch`，and will open rviz automatically

```
roslaunch delta_lidar view_delta_lidar.launch
```

