---
title: SLAM部署与imu标定
categories: 
  - 技术
  - SLAM
tags: [技术博文,ROS1/2,SLAM]
date: 2025-03-19
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  SLAM部署与imu标定
</div>

# 前言
- 笔者在[上一条帖子](https://tlf-nagisa-blog.com/2025/03/15/docker1/)时已经在尝试运行若干SLAM算法试图择优观察部署效果
(包括[FAST_LIO](https://github.com/hku-mars/FAST_LIO)，[LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)，[Point-LIO](https://github.com/hku-mars/Point-LIO)，[rtabmap](https://github.com/introlab/rtabmap)等笔者看过的建图，定位算法）

- 笔者最终聚焦在[LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)的建图(意即先导地图的获取)，随后利用[liorf_localization](https://github.com/YJZLuckyBoy/liorf_localization)的[LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)衍生纯定位算法实现机器人在已知地图的重定位

- 可能有的开发者想问SLAM已经自带了同时建图与定位功能，为什么还要将建图和定位隔开进行开发。笔者在单跑LIO-SAM时发现过其轨迹与实际录取数据包时的误差，有时甚至会平地起飞（漂移）

- 为了解决这一问题，笔者尝试了隔离建图定位，进行imu内参标定的尝试，将其记录下来(~~其实就是臭掉包工程师~~)

# 基础知识扫盲
首先，我们要了解一下SLAM和传感器的有关术语

## SLAM，定位与建图
- 1.建图：简而言之，就是收集正在运作的传感器数据（本文中提到的均为激光SLAM，所以传感器基本上是激光雷达+imu），利用建图算法将这段时间的传感器数据记录为一张地图  

- 2.定位：实际上就是确定机器人从规定的地图原点（通常也是机器人的上电原点），到机器人任何一时刻运行时刻的位置改变，确定这个改变也就是给机器人定位了

- 或者，有了解过tf树的同学也可以通过几大坐标系更轻松的理解。map：地图原点；odom：机器人上电原点，里程计的初始点；base_link：机器人在运行过程中的任意一点，定位，就是时刻刷新odom到base_link的tf树变化

- 3.SLAM（同时建图与定位）：能够依靠高计算量算法，做到机器人首次在新环境中同时完成前两项的工作

- 需要注意的是，前面的三个概念，在实际运行的时候所需的数据资源一模一样，除了定位需要多用到一个建图结束生成的先导地图，三者均只需要提供雷达和imu数据即可。

## SLAM的分类
### 松、紧耦合传感器SLAM
- 1.紧，即传感器数据处理方式。将多个传感器的原始数据直接融合到同一个优化框架中，共同完成状态估计（如位姿、地图等）。
​所有传感器的原始数据在同一个数学模型中被联合优化

- 需注意，本帖中所提到的所有SLAM均为紧耦合SLAM，紧耦合最大的好处是充分发挥各传感器之间位姿带来的数据融合互补，配合算法纠偏

- 2.松，即多传感器分模块单独处理，各个传感器独立处理数据，生成局部估计结果（如相机的位姿、IMU的位姿），再通过后处理（如滤波或优化）进行融合

### 视觉与激光SLAM
- 1.激光SLAM即主要运用激光类传感器，如激光雷达。主要传感器数据为3D点云。目前被广泛运用，且有很多牛b的实验室算法

- 同样，本帖中所提到的所有SLAM均为激光SLAM，这也是大多数情况下使用的SLAM类型

- 2.视觉SLAM主要运用相机（如双目等）。主要传感器数据为2D图像。不如激光的发展迅速和好用，但在一些特定情况下能发挥其可以捕捉细节纹理的优势。

## <u>***跑定位前，你需要做什么***</u>
### 传感器标定
#### 缘由
- 传感器的标定，直接影响了传感器在运行时数据的质量

- 传感器在运行时存在噪声，但是可以通过标定算法来平衡或者减小这一噪声带来的数据误差

#### 噪声标定分类
- 内参标定：标定传感器自身运行时的数据，自我校准，解决自身数据噪声。

- 外参标定：多传感器处于机器人不同的方位，外参标定就是因为人为测量计算两传感器之间的旋转矩阵存在误差，用外参标定包来进行二次校准，可以让两传感器之间的相对位置表示更加准确，方便后续的数据对齐（算法里的数据需要再同一坐标系下）

#### 标定的分类（若想更详细的了解可以点击这篇[优质文章](https://blog.csdn.net/zardforever123/article/details/130030767)）

##### 相机标定
- 消除畸变：
相机镜头存在光学畸变（如径向畸变、切向畸变），导致图像边缘的直线弯曲。标定可以量化并校正这些畸变

- ​获取相机内参：
内参包括焦距（fx、fy）、主点（u0、v0）和畸变系数（k1、k2、p1、p2），这些参数决定了相机如何将3D世界投影到2D图像平面

- ​确定相机外参：
外参包括旋转矩阵 R 和平移向量 T，描述相机坐标系与世界坐标系之间的相对位置关系

- 使用的方法一般是棋盘格法标定，或采用OpenCV库函数正畸来进行标定

##### <u>***IMU标定***</u>
- 知晓IMU在运行时加速度仪和陀螺仪的白噪声和随机噪声，用得到的平均噪声去优化算法对于IMU数据的处理

- 一般采取IMU内参标定包如[imu_utils](https://github.com/gaowenliang/imu_utils)，在IMU静态或极缓状态下移动，并记录较长时间段（1h到3h不等），采集数据包后运行该包进行噪声计算

##### 联合标定：标定传感器之间位置关系的外参
- 相机与IMU，IMU与Lidar，相机与Lidar之间如果可以，建议都进行标定。原理这里不多赘述，理论上而言外参如果相信自己的测量且精度要求没有那么高的情况下，标定与不标定的区别对最终结果的影响没有那么大。

- 附若干外参标定工程项目：LiDAR与IMU联合标定浙大开源的[lidar_IMU_calib](https://github.com/APRIL-ZJU/lidar_IMU_calib)，相机雷达联合标定[calibration_camera_lidar
](https://github.com/XidianLemon/calibration_camera_lidar)等


# 跑通SLAM与定位的流程（以[LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)建图和[liorf_localization](https://github.com/YJZLuckyBoy/liorf_localization)定位为例）

## 概述：仅仅只是单论跑起来的步骤很简单
- 首先写好各个硬件的驱动模块，让所有传感器开始工作，并发布要求的话题，并在话题中发布传感器数据（IMU和Lidar传感器驱动，对于Mid360用户而言，这个激光雷达中自带了一个精度还行的IMU所以只需要一个硬件）

- 然后再运用上述提到的标定包标定对应传感器的内参，得到内参标定结果，写入SLAM算法的config文件的对应传感器配置项。

- 随后只需要调整SLAM原始包的参数文件，修改订阅的传感器话题和前述自己传感器发布的话题一致，并启动SLAM的launch文件，理论上而言就可以跑起来了

- 但仅仅跑起来远远不够，在没有调整其他参数，如地图降采样率，传感器数据置信度，数据单位等（官方包数据和你的传感器数据单位不一定一样）。你会发现你的建图虽然正常打开了，但是机器人一开起来就去外太空旅游了，根本不是实际你采集数据时候的轨迹

## 重点记录1：参数文件的修改
这也是困扰了笔者很久很久的调参环节，想要看懂yaml文件里的参数，要去回查源码和了解SLAM的部分运行原理是最好，包括算法的模块设计思路
### 参数详解
- 笔者整理了大部分的lio-sam的yaml中的参数含义并添加了大段注释，知道了参数含义，才能做出正确尝试性的改参，详细看下文注释
```bash
lio_sam:

  # Topics
  pointCloudTopic: "points_raw" # Point cloud data 
  imuTopic: "imu_raw"  # IMU data
  odomTopic: "odometry/imu" # 这个是imu数据经过处理，代码自动发布的，
  #IMU pre-preintegration odometry, same frequency as IMU
  gpsTopic: "odometry/gpsz" # 若要使用GPS，GPS发布的话题
  #odometry topic from navsat, see module_navsat.launch file

  # Frames tf树形态
  lidarFrame: "base_link"
  baselinkFrame: "base_link"
  odometryFrame: "odom"
  mapFrame: "map"

  # GPS Settings
  useImuHeadingInitialization: true # 是否使用IMU的航向角（偏航角）​来初始化系统的初始方向，
  #但需要九轴IMU，if using GPS data, set to "true"
  useGpsElevation: false  # if GPS elevation is bad, set to "false" 
  #是否使用GPS的高度数据作为定位的垂直方向（Z轴）输入
  gpsCovThreshold: 2.0    # m^2, threshold for using GPS data 
  #GPS数据协方差的阈值（单位：平方米），用于判断当前GPS数据是否可靠，若大于该数值则认为数据不可靠，不会使用gps数据
  poseCovThreshold: 25.0  # m^2, threshold for using GPS data 
  #系统当前位姿估计的协方差阈值（单位：平方米），用于判断是否触发GPS修正
  
  # Export settings
  savePCD: false  # https://github.com/TixiaoShan/LIO-SAM/issues/3
  savePCDDirectory: "/Downloads/LOAM/" # in your home folder, starts and ends with "/". Warning: the code deletes "LOAM" folder then recreates it. See "mapOptimization" for implementation 
  #将建图结果先导地图保留路径，程序结束或ctrl+c后自动触发保存
  
  # Sensor Settings
  sensor: velodyne # lidar sensor type, 'velodyne' or 'ouster' or 'livox' 
  #源码中有数据之间的转换，这几种雷达会有不一样的传感器点云数据构成
  N_SCAN: 16  # number of lidar channel (i.e., Velodyne/Ouster: 16, 32, 64, 128, Livox Horizon: 6)
  Horizon_SCAN: 1800  # lidar horizontal resolution 
  #(Velodyne:1800, Ouster:512,1024,2048, Livox Horizon: 4000)
  #激光雷达的水平分辨率（每圈扫描的水平点数），决定点云的列数
  downsampleRate: 1  # default: 1. 
  #Downsample your data if too many points. i.e., 16 = 64 / 4, 16 = 16 / 1 
  #点云下采样率，降低数据量以提升处理速度
  lidarMinRange: 1.0 # default: 1.0, minimum lidar range to be used
  lidarMaxRange: 1000.0 # default: 1000.0, 
  #maximum lidar range to be used 
  #采集雷达数据的视在有效范围，采集数据的简单滤波

  # IMU Settings
  imuAccNoise: 3.9939570888238808e-03  # IMU的加速度计测量噪声
  imuGyrNoise: 1.5636343949698187e-03  # IMU的陀螺仪测量噪声
  imuAccBiasN: 6.4356659353532566e-05  # IMU的加速度计白噪声（零飘）
  imuGyrBiasN: 3.5640318696367613e-05  # IMU的陀螺仪白噪声
  imuGravity: 9.80511  # IMU的重力加速度值​（单位：m/s²），注意有的IMU（如Mid360的IMU该数据单位为g所以该项数据为1左右）
  imuRPYWeight: 0.01   # IMU姿态（Roll、Pitch、Yaw）在多传感器融合中的权重，表示对IMU的信任程度，越高越信任

  # Extrinsics: T_lb (lidar -> imu) 传感器外参的标定矩阵
  extrinsicTrans: [0.0, 0.0, 0.0]  # 平移矩阵：雷达坐标系 到 ​IMU坐标系 的 ​原点偏移量
  extrinsicRot: [-1, 0, 0,
                  0, 1, 0,
                  0, 0, -1]  # 旋转矩阵：定义从 ​雷达坐标系 到 ​IMU坐标系 的 ​旋转变换
  extrinsicRPY: [0, -1, 0,
                 1, 0, 0,
                 0, 0, 1]  # 通过欧拉角旋转矩阵（滚转、俯仰、偏航）描述 ​雷达→IMU 的旋转变换，并以矩阵形式存储和extrinsicRot其实存储的信息一致
  # extrinsicRot: [1, 0, 0,
  #                 0, 1, 0,
  #                 0, 0, 1]
  # extrinsicRPY: [1, 0, 0,
  #                 0, 1, 0,
  #                 0, 0, 1]

  # LOAM feature threshold
  edgeThreshold: 1.0  # 边缘特征曲率阈值，用于判断一个点是否属于边缘特征
  surfThreshold: 0.1  # 平面特征曲率阈值，用于筛选平坦区域的特征点
  edgeFeatureMinValidNum: 10   # 单帧点云中边缘特征的最小有效数量  
  #​动态场景​（如行人密集）：增加该值（如 20），确保足够特征点匹配。
  #​低线数雷达​（如16线）：减少该值（如 5），避免频繁丢帧
  surfFeatureMinValidNum: 100  # ​单帧点云中平面特征的最小有效数量  
  #​大范围场景​（如高速公路）：提高该值（如 150），增强匹配鲁棒性。
  #​狭窄场景​（如走廊）：降低该值（如 50），避免因平面点不足导致匹配失败

  # voxel filter paprams 值相当于定义体素大小，点云被划分成正方体，值越小越精细
  odometrySurfLeafSize: 0.4  # default: 0.4 - outdoor, 0.2 - indoor
  #控制里程计（Odometry）模块中表面（Surf）点云的下采样体素大小。
  #​数值含义：体素边长为 0.4 米，即点云会被划分为 0.4m 的立方体，每个立方体内的点用一个代表点（如重心）替代
  mappingCornerLeafSize: 0.2 # default: 0.2 - outdoor, 0.1 - 
  #indoor控制建图（Mapping）模块中表面（Surf）点云的下采样体素大小
  mappingSurfLeafSize: 0.4   # default: 0.4 - outdoor, 0.2 - indoor

  # robot motion constraint (in case you are using a 2D robot)
  z_tollerance: 1000         # meters 控制机器人的两帧之间z轴位移大小，当值为很大时就是完全不限定
  #为0时强制其在一平面上运动，但当确实有z轴微小偏移却强行设0会引发漂移
  rotation_tollerance: 1000  # radians 控制机器人的两帧之间的转动最大角（弧度单位）

  # CPU Params
  numberOfCores: 4  # number of cores for mapping optimization 
  #指定用于建图优化（Mapping Optimization）的CPU核心数为4
  mappingProcessInterval: 0.15 # seconds, regulate mapping frequency 
  #建图进程的执行间隔为0.15秒（即每秒约6.6次更新

  # Surrounding map
  surroundingkeyframeAddingDistThreshold: 1.0  # meters, regulate keyframe adding threshold 
  #设置关键帧添加的距离阈值
  surroundingkeyframeAddingAngleThreshold: 0.2 # radians, regulate keyframe adding threshold 
  #设置关键帧添加的角度阈值
  surroundingKeyframeDensity: 2.0 # meters, downsample surrounding keyframe poses  
  #控制周围关键帧的下采样密度，越多越准，但cpu负荷更大
  surroundingKeyframeSearchRadius: 50.0 # meters, within n meters scan-to-map optimization (when loop closure disabled) 
  #设置局部地图优化的搜索半径

  # Loop closure
  loopClosureEnableFlag: true
  loopClosureFrequency: 1.0    # Hz, regulate loop closure constraint add frequency
  surroundingKeyframeSize: 50  # submap size (when loop closure enabled) 
  #定义用于构建局部子地图（Submap）的关键帧数量
  historyKeyframeSearchRadius: 15.0  # meters, key frame that is within n meters from current pose will be considerd for loop closure 
  #设定搜索历史关键帧的空间半径（狭窄环境要减小，防止误匹配）
  historyKeyframeSearchTimeDiff: 30.0  # seconds, key frame that is n seconds older will be considered for loop closure 
  #设置历史关键帧的时间差阈值（高速机器人减少，低速增长）
  historyKeyframeSearchNum: 25  # number of hostory key frames will be fused into a submap for loop closure 
  #指定融合到子地图中的历史关键帧数量（低的实时性好，高的鲁棒性强）
  historyKeyframeFitnessScore: 0.3  # icp threshold, the smaller the better alignment 
  #设定ICP（迭代最近点）匹配的误差阈值，数值越小匹配越精准

  # Visualization
  globalMapVisualizationSearchRadius: 1000.0 # meters, global map visualization radius 
  #设置全局地图可视化的搜索半径（单位：米），即仅显示机器人当前位姿周围 ​1000 米范围内 的地图数据 
  #大范围建图​（如户外）：保持默认值，确保全局地图完整显示。
  #​实时避障：可缩小范围（如 500.0），减少渲染数据量以提升帧率。
  globalMapVisualizationPoseDensity: 10.0    # meters, global map visualization keyframe density 
  #高精度调试：设为较小值（如 5.0），显示更多关键帧轨迹细节。
  #​轻量化显示：增大值（如 20.0），减少轨迹点数量以降低GPU负载。
  globalMapVisualizationLeafSize: 1.0        # meters, global map visualization cloud density 
  #设置全局地图点云的 ​下采样体素尺寸​（单位：米），即每个 1.0×1.0×1.0 米体素内保留一个点 
  #​降低点云冗余：减少重复点（如墙面、地面），提升渲染效率。​保持特征结构：合理值（如 0.5-2.0）可在降噪与保留几何细节间平衡。




# Navsat (convert GPS coordinates to Cartesian)
navsat:
  frequency: 50
  wait_for_datum: false  # 无需等待基准点（Datum），适合动态启动的场景（如车载系统即时定位
  delay: 0.0
  magnetic_declination_radians: 0 # 磁偏角补偿
  yaw_offset: 0
  zero_altitude: true  # 将高度值强制置零，适用于平面导航（如室内或忽略高程的2D地图）
  broadcast_utm_transform: false
  broadcast_utm_transform_as_parent_frame: false
  publish_filtered_gps: false

# EKF for Navsat
ekf_gps:
  publish_tf: false
  map_frame: map
  odom_frame: odom
  base_link_frame: base_link
  world_frame: odom

  frequency: 50
  two_d_mode: false
  sensor_timeout: 0.01
  # -------------------------------------
  # External IMU:
  # -------------------------------------
  imu0: imu_correct
  # make sure the input is aligned with ROS REP105. "imu_correct" is manually transformed by myself. EKF can also transform the data using tf between your imu and base_link
  imu0_config: [false, false, false,
                true,  true,  true,
                false, false, false,
                false, false, true,
                true,  true,  true]
  imu0_differential: false
  imu0_queue_size: 50 
  imu0_remove_gravitational_acceleration: true
  # -------------------------------------
  # Odometry (From Navsat):
  # -------------------------------------
  odom0: odometry/gps
  odom0_config: [true,  true,  true,
                 false, false, false,
                 false, false, false,
                 false, false, false,
                 false, false, false]
  odom0_differential: false
  odom0_queue_size: 10
```

## 重点记录2：imu_utils的imu标定包的使用
### <u>***注意***</u>
- 笔者很想喷人的是，github上开源的imu_utils包的内容不能过编译，其中的源码有需要修改的部分，笔者自己在原包的基础上进行了二次开发，并封装了docker容器

- 项目地址：https://github.com/njustup70/nagisa_little_kit
### 包的使用
- 首先拉取项目
```bash
git clone https://github.com/njustup70/nagisa_little_kit.git
```
- 配置好docker-compose环境，注意也要提前下载好docker（对docker下载和docker-compose容器管理不太清楚的话可以看看我之前的[帖子](https://tlf-nagisa-blog.com/2025/03/15/docker1/)，这里默认用户已下载好了docker
```bash
sudo apt install docker-compose
```
- 编译并打开容器
```bash
nagisa_little_kit/IMU_init/.devcontainer$ docker-compose up --b && docker-compose up --d
```

- 进入容器（再开一个终端）
```bash
nagisa_little_kit/IMU_init/.devcontainer$ docker exec -it littlekit-comtainer bash
```
- 进入容器后，先source（这里牵扯到ROS1的source工作空间覆盖问题，***但是笔者已经解决，按照步骤来，不要做多余步骤，就可以解决这个问题***，ROS开发中的一些小坑会在之后的博客中更新，这里暂时先不说原理了）
```bash
cd ~/packages/ros_manager && source devel/setup.bash # 这步就可以解决source问题
```
- 启动标定launch文件
```bash
cd ../nagisa_ws && roslaunch my_imu_init A3.launch
```
- 随后启动IMU数据包即可，但是注意，***IMU数据包中的IMU话题需要与A3.launch中定义的一样，如果不一样请先在拉取工程后修改为一样（ROS1中的launch和yaml文件修改后不用catkin_make和source）***

# 文末
- 附LIO-SAM的[官方论文](https://drive.google.com/file/d/16uXlxT91tk-mtIE-lI0_GFF6qfnseC0k/view)

