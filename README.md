# 使用CHAMP开源算法,实现宇树机器狗go2的ign-gazebo仿真与导航

## 环境依赖

### 系统与基础

- Ubuntu 22.04
- ROS2 Humble
- Ignition Gazebo（Fortress，对应 `ros-humble-ros-gz`）

### ROS2 功能包
```bash
sudo apt update
sudo apt install -y \
    python3-colcon-common-extensions \
    python3-rosdep \
    ros-humble-ros-gz \
    ros-humble-ros-gz-sim \
    ros-humble-ros-gz-bridge \
    ros-humble-ros2-control \
    ros-humble-ros2-controllers \
    ros-humble-joint-state-publisher \
    ros-humble-joint-state-publisher-gui \
    ros-humble-robot-state-publisher \
    ros-humble-xacro \
    ros-humble-navigation2 \
    ros-humble-nav2-bringup \
    ros-humble-nav2-common \
    ros-humble-slam-toolbox \
    ros-humble-cartographer-ros \
    ros-humble-tf2-ros \
    ros-humble-tf2-tools \
    ros-humble-rviz2 \
    ros-humble-joy \
    ros-humble-pointcloud-to-laserscan
```
### 初始化 rosdep

```bash
sudo rosdep init || true
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```
## 编译与安装

```bash
cd ~/ign_dog_stu
colcon build --symlink-install
source install/setup.bash
```

```bash
colcon build --packages-select go2_description sim_ign_dog go2_config \
             champ_msgs champ champ_base champ_bringup champ_description \
             champ_config champ_gazebo champ_navigation \
             champ_teleop sim_navigation2 slam_cartographer
```

### 1. 仅在 RViz 中查看模型（无仿真）

```bash
ros2 launch go2_description dog_description.launch.py
```
### 2. 启动完整仿真环境（Gazebo + RViz + 控制器 + CHAMP + Nav2）
```bash
ros2 launch sim_ign_dog gazebo_sim_dog.launch.py
```
### 3.启动键盘控制节点
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```
### 4.手柄遥操作机器狗
```bash
ros2 launch champ_teleop teleop.launch.py use_joy:=true
```

## 功能特性
```bash
- Ignition Gazebo 室内房屋仿真环境（含家具模型）
- Go2 机器狗 URDF/Xacro 模型（含碰撞、惯性、传动、传感器）
- ros2_control 关节轨迹控制器（`joint_state_broadcaster` + `legs_controller`）
- CHAMP 四足步态控制框架（站立、行走、转向、姿态调节）
- ROS↔Gazebo 桥接（时钟、里程计、激光、深度相机、图像、`/cmd_vel`）
- Cartographer 2D SLAM 建图
- Navigation2 自主导航（行为树、代价地图、规划器、控制器、速度平滑）
- 键盘 / 手柄遥操作
- RViz2 可视化（机器人模型、传感器、地图、导航目标）
```
## 代码结构

```bash
ign_dog_stu/
├── ign_models/                       # Gazebo 模型库（Desk, Sofa, Bed, Pickup ...）
├── nav2_params.yaml                  # 顶层 nav2 参数备份
├── src/
│   ├── sim_dog/
│   │   ├── go2_description/          # Go2 机器人描述包（URDF/Xacro/meshes）
│   │   │   ├── urdf/
│   │   │   │   ├── go2_description.urdf        # 用于 CHAMP（base 链接）
│   │   │   │   └── go2_description_ign.urdf    # 用于 Ignition Gazebo
│   │   │   ├── xacro/                          # robot/leg/transmission/gazebo/materials
│   │   │   ├── meshes/                         # DAE 网格
│   │   │   ├── launch/
│   │   │   │   ├── dog_description.launch.py        # RViz 单独可视化（带 JSP）
│   │   │   │   ├── dog_description_ign.launch.py    # 仿真用 robot_state_publisher
│   │   │   │   └── go2_rviz.launch                  # 旧版 ROS1 风格 launch（参考）
│   │   │   └── config/
│   │   ├── go2w_description/         # Go2 轮式变体描述（可选）
│   │   └── sim_ign_dog/              # 仿真主启动包
│   │       ├── launch/
│   │       │   ├── gazebo_sim_dog.launch.py    # 总启动入口
│   │       │   └── old_gazebo_sim_dog.launch.py
│   │       ├── world/
│   │       │   ├── house.sdf                   # 空房屋
│   │       │   └── house_add.sdf               # 房屋 + 家具
│   │       ├── config/go2_controllers.yaml     # ros2_control 控制器配置
│   │       └── rviz/
│   │           ├── ign_dog.rviz
│   │           └── nav2.rviz
│   ├── go2_config/                   # Go2 专属 CHAMP 配置
│   │   ├── config/
│   │   │   ├── gait/gait.yaml                  # 步态参数
│   │   │   ├── joints/joints.yaml             # 关节映射
│   │   │   ├── links/links.yaml               # 链接映射
│   │   │   └── autonomy/                       # slam.yaml / navigation.yaml
│   │   ├── maps/                               # 已建地图
│   │   └── launch/                             # bringup/slam/navigate/gazebo
│   ├── champ/                        # CHAMP 四足控制器框架
│   │   ├── champ/                   # C++ 控制库
│   │   ├── champ_base/              # 节点：quadruped_controller / state_estimator
│   │   ├── champ_bringup/           # 顶层 bringup.launch.py
│   │   ├── champ_description/       # CHAMP 默认 URDF/rviz
│   │   ├── champ_config/            # CHAMP 默认配置（参考）
│   │   ├── champ_gazebo/            # CHAMP 自带 Gazebo 启动
│   │   ├── champ_msgs/              # 自定义消息（Joints/Pose/Contacts ...）
│   │   └── champ_navigation/        # CHAMP 自带 slam/navigate 启动
│   ├── champ_teleop/                 # 键盘 / 手柄遥操作节点
│   ├── nav2/
│   │   └── sim_navigation2/          # 仿真用 Navigation2 启动与参数
│   │       ├── launch/
│   │       │   ├── nav2_base.launch.py         # 各 nav2 子节点
│   │       │   └── nav2_bringup.launch.py      # nav2_base + cartographer
│   │       ├── params/                         # 各 nav2 子模块 yaml
│   │       └── bts/                            # 自定义行为树 xml
│   └── slam/
│       └── slam_cartographer/        # Cartographer SLAM 启动
└── install/                          # colcon 编译产物
```
## 参考仓库
```bash
anujjain-dev/unitree-go2-ros2
chvmp/champ
```

