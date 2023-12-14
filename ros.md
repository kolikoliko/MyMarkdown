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
catkin_make
```
3. 设置环境变量
```shell
echo $ROS_PACKAGE_PATH
```

## 创建功能包
1. 创建功能包
```shell
cd catkin_ws/src
catkin_creat_pkg test_pkg std_msgs rospy roscpp
```
2. 编译功能包   
```shell
cd catkin_ws
catkin_make
source catkin_ws/devel/setup.bash
```