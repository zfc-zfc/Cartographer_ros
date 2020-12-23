# Cartographer-using-Delta_III-Lidar
@[toc]
## Cartographer安装
```bash
sudo apt install ros-melodic-cartographer
sudo apt install ros-melodic-cartographer-ros
sudo apt install ros-melodic-cartographer-ros-msgs
```
之后可以在  `/opt/ros/melodic/share` 找到这三个包，若找不到可以用`rospack find packagename` 找到路径。
### 跑一个bag试试
#### 下载官网示例bag
```
https://storage.googleapis.com/cartographer-public-data/bags/backpack_2d/cartographer_paper_deutsches_museum.bag
```
复制此网址到浏览器，下载bag（470MB），名字为 `cartographer_paper_deutsches_museum.bag`，存储路径默认为 /home/username/Downloads
#### 运行bag
```bash
roslaunch cartographer_ros demo_backpack_2d.launch bag_filename:=${HOME}/Downloads/cartographer_paper_deutsches_museum.bag
```
可以看到雷达的实时数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223164011157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)
点击左下角"Add"，可以添加一个名为 "/map" 的topic，即可看到建立的地图，如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223164128517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)
至此说明安装cartographer成功。

## 让雷达动起来
这一步涉及LiDAR的驱动安装，各种不同的雷达不太相同。此处以杉川的Delta_III为例，
产品链接
```
http://www.3irobotics.com/pro_s.php?id=168
```
驱动安装和运行方法，详情见delta_III_ros

### 雷达基本信息获取
启动雷达 Delta_III
```
roslaunch delta_lidar view_delta_lidar.launch 
```
可以在rviz中看到雷达扫描数据，然后用 `rostopic list` 查看雷达发布出来的话题名
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223165054327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)

此处本Delta_III雷达的输出topic为 /scan
使用以下语句打印当前雷达输出:
```
rostopic echo /scan
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223165257268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbmdfY2hlbmdf,size_16,color_FFFFFF,t_70)

在打印信息中找到坐标系名字frame_id: “laser”,此处坐标系名字为 laser
## 文件修改
### lua文件修改
在 `/opt/ros/melodic/share/cartographer_ros/launch` 中找到revo_lds.lua 文件, 复制一份, 改名为my_revo_lds.lua
由于权限原因，可以用如下方式复制文件：
在此目录打开终端
```bash
sudo cp revo_lds.lua my_revo_lds.lua
```
如下命令修改内容
```bash
sudo gedit my_revo_lds.lua
```
打开my_revo_lds.lua, 修改以下两行为我们自己的坐标系名字:
```cpp
//两行"horizontal_laser_link"均改为我们当前的"laser"
tracking_frame = "laser",
published_frame = "laser",
```
### launch文件修改
在`/opt/ros/melodic/share/cartographer_ros/configuration_files`, 复制一份, 改名为my_demo_revo_lds.launch
修改以下内容:
```cpp
//因为非bag仿真,将以下true改为false
<param name="/use_sim_time" value="false" />

//使用我们的配置文件,将revo_lds.lua改为my_revo_lds.lua
<node name="cartographer_node" pkg="cartographer_ros"
      type="cartographer_node" args="
          -configuration_directory $(find cartographer_ros)/configuration_files
          -configuration_basename my_revo_lds.lua"

//将horizontal_laser_2d改为我们的输出话题scan
<remap from="scan" to="scan" />

//不使用bag,删去以下内容
<node name="playbag" pkg="rosbag" type="play"
      args="--clock $(arg bag_filename)" />
```

## 运行与建图
运行时, 先运行雷达的相关topic, 保证雷达已有有效数据通过topic输出后, 运行我们的新launch文件:
```bash
roslaunch cartographer_ros my_demo_revo_lds.launch
```
即可开始建图.

## 地图保存
打开另一个终端, 输入以下语句:

//结束第一条轨迹,之后的数据不再添加.
`rosservice call /finish_trajectory 0`


//cartographer序列化当前状态,形成pbstream文件
`rosservice call /write_state "{filename: '${HOME}/Downloads/mymap.pbstream'}"`

//transform pbstream to pgm and yaml
```
rosrun cartographer_ros cartographer_pbstream_to_ros_map -map_filestem=/home/kai/Downloads/mymap -pbstream_filename=/home/kai/Downloads/mymap.pbstream -resolution=0.05
```
地址和文件名根据实际进行修改.

