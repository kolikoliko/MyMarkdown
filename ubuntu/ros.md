# ROS学习

## 创作工作空间

1. 创作工作空间
```shell
mkdir -p catkin_ws/src
cd catkin_ws/src
catkin_init_workspace
```
2. 编译工作空间
```shell
cd catkin_ws/
catkin_make install
```
3. 设置环境变量
```shell
echo $ROS_PACKAGE_PATH
```

## 创建功能包
1. 创建功能包
```shell
cd catkin_ws/src
catkin_create_pkg learning_topic roscpp rospy std_msgs geometry_msgs turtlesim
```
2. 编译功能包   
```shell
cd catkin_ws
catkin_make
source catkin_ws/devel/setup.bash
```

3. 在功能包的src文件里面创建文件，例程如下
```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>
 
int main(int argc, char **argv)
{
  //ROS节点初始化
  ros::init(argc, argv, "velocity_publisher");
  
  //创建节点句柄
  ros::NodeHandle n;  
  
  //创建一个Publisher,发布名为/turtle1/cmd_vel的topic，消息类型为geometry_msgs::Twist，队列长度为10
  ros::Publisher turtule_vel_pub = n.advertise<geometry_msgs::Twist>("/turtle1/cmd_vel", 10);  
 
  //设置循环的频率
  ros::Rate loop_rate(10); 
 
  int count = 0;
  while (ros::ok())
  {
    //初始化geometry_msgs::Twist的类型的消息
    geometry_msgs::Twist vel_msg;
    vel_msg.linear.x=0.5;
    vel_msg.linear.y=0.2;

    //发布消息
    turtle_vel_pub.publish(vel_msg);
    ROS_INFO("[%0.2f m/s %0.2f rad/s]",vel_msg.linear.x,vel_msg.angular.z);

    //按频率循环
    loop_rate.sleep();
  }
 
  return 0;
}
```
红色报错解决方案
`/opt/ros/noetic/include/**`


4. 配置发布者代码编译规则
```cmake
add_executable(velocity_publisher src/velocity_publisher.cpp)
target_link_libraries(velocity_publisher ${catkin_LIBRARIES})
```

5. 编译并运行发布者
```shell
catkin_make
source devel/setup.bash
roscore
rosrun turtlesim turtlesim_node
rosrun learning_topic velocity_publisher
```

发布者参考连接
https://blog.csdn.net/qq_44989881/article/details/118568922

订阅发布与客户端服务端建议直接看代码
课程源码连接在gitee上找了一个
https://gitee.com/guyuehome/ros_21_tutorials

## 参数的使用
使用`rosparam`命令可以有很多操作

```shell
Commands:
	rosparam set	set parameter
	rosparam get	get parameter
	rosparam load	load parameters from file
	rosparam dump	dump parameters to file
	rosparam delete	delete parameter
	rosparam list	list parameter names
```
这里的file文件类型一般采用`.yaml`文件格式

也可以采用cpp,py等文件进行操作，具体在代码中也有

## ros中的坐标管理系统
```shell
sudo apt-get install ros-melodic-turtle-tf
roslaunch turtle_tf turtle_tf_demo.launch 
```

一些查看数据的方法
```shell
rosrun tf view_frames
rosrun tf tf_echo turtle1 turtle2
0
```

0