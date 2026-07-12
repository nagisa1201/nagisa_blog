---
title: ROS2 Middelware通信
categories:
  - 技术
  - ROS2
tags: [ROS2, 通信]
date: 2026-07-12
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  ROS2 Middleware通信
</div>

本文用于记录笔者在使用ROS2开发时，在通信问题上的踩坑与总结。

# ROS2 Middleware通信件
- ROS2通信的核心是Middleware，传递消息。ROS2的体系结构是采取分层设计的，主要由自下而上的**Operationg System层、Communication Layer层、Middleware层、ROS2 Client Wrapper层与User Application层组成**。 ***Middleware层用于提供ROS2到Communication Layer的翻译接口，进而调用Communication Layer的功能。***
![ROS2分层架构图](/blog-img/ROS2_Communication/image.png)

- 笔者使用过的Communication Layer的通信件有：
  - **Fast DDS**，同时也是ROS2官方推荐的通信件，基于UDP组播
  - **Cyclone DDS**，更加轻量化，也更加灵活的通信件，基于UDP组播
  - **Zenoh**，由Rust语言开发的通信件，支持多种传输协议，基于TCP

- 我们使用的通信件经历了从官方默认的Fast DDS，到Cyclone DDS，再到Zenoh的过程。每个通信件都有其优缺点，选择合适的通信件对于ROS2应用的性能和稳定性至关重要。

# 通信件使用的演化
## 从最初的Fast DDS
- Fast DDS是**ROS2官方默认的通信件**，**基于UDP组播**，适用于大多数场景。它的优点是成熟、稳定，并且有广泛的社区支持。然而，在某些场景下，Fast DDS可能会遇到性能瓶颈，尤其是在高频率、大数据量的通信需求下。
- 让我决定换用Cyclone DDS的原因是，在使用Navigation2时，默认的Fast DDS通信件使得我们**标称10Hz的SLAM的TF变换发布频率由于通信阻塞**变为$< 10 \text{Hz}$，且可能导致了一个**未知的终端Warning**的持续刷新（*是的，是可能，换通信件后不再warning，但也不可排除是否是这个问题*）
```bash
[rviz2-5] [INFO] [1674612885.459650696] [rviz]: Message Filter dropping message: frame 'base_scan' at time 140.564 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612885.523260716] [rviz]: Message Filter dropping message: frame 'odom' at time 136.749 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612885.523386495] [rviz]: Message Filter dropping message: frame 'odom' at time 140.667 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612885.650110920] [rviz]: Message Filter dropping message: frame 'base_scan' at time 140.763 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612885.747162271] [rviz]: Message Filter dropping message: frame 'odom' at time 140.871 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612885.875493196] [rviz]: Message Filter dropping message: frame 'base_scan' at time 140.963 for reason 'discarding message because the queue is full'
[component_container_isolated-6] [INFO] [1674612885.932109373] [global_costmap.global_costmap]: Timed out waiting for transform from base_link to map to become available, tf error: Invalid frame ID "map" passed to canTransform argument target_frame - frame does not exist
[rviz2-5] [INFO] [1674612885.939653241] [rviz]: Message Filter dropping message: frame 'odom' at time 141.041 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612886.070139848] [rviz]: Message Filter dropping message: frame 'base_scan' at time 141.163 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612886.131094368] [rviz]: Message Filter dropping message: frame 'odom' at time 137.334 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612886.131269299] [rviz]: Message Filter dropping message: frame 'odom' at time 141.245 for reason 'discarding message because the queue is full'
[rviz2-5] [INFO] [1674612886.259987441] [rviz]: Message Filter dropping message: frame 'base_scan' at time 141.364 for reason 'discarding message because the queue is full'
[component_container_isolated-6] [WARN] [1674612886.279423396] [amcl]: AMCL cannot publish a pose or update the transform. Please set the initial pose...
```
- 似乎是因为Fast DDS在高频率的消息传递下，导致了消息队列的溢出，从而引发了这些Warning信息，因此，换通信件的尝试开始了。

## Cyclone DDS
- Cyclone DDS是一个轻量级的DDS实现，**同样基于UDP组播**，它的设计目标是提供一个更灵活和可扩展的通信件。Cyclone DDS在某些场景下表现出更好的性能，换用后，在**同样默认配置下，可以明显感觉到通信容量的提升**，在高频数据流入时，消息丢失的情况减少了，系统的稳定性也有所提高。
- 由于我们使用的是Docker的多容器通信，我们有如下配置：
```yaml
version: '3'
services:
  ros2driver-service:
    # build:
    #   context: ..
    #   dockerfile: .devcontainer/Dockerfile
    image: elainasuki/rc2025:ros2_driver
    container_name: ros2driver-container
    environment:
      - DISPLAY=${DISPLAY}
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
      - OMP_WAIT_POLICY=passive
      - TERM=xterm-256color
      - RMW_IMPLEMENTATION=rmw_cyclonedds_cpp # 使用CycloneDDS就取消注释此行
      - ROS_LOCALHOST_ONLY=1

    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix
      - ./..:/home/Elaina/ros2_driver
      - /var/run/docker.sock:/var/run/docker.sock
      - /dev:/dev
    network_mode: host
    pid: "host" # 添加 pid 命名空间共享
    ipc: "host" # 添加 ipc 命名空间共享    
    privileged: true
    stdin_open: true
    tty: true
    user: "Elaina"
    # runtime: "nvidia"
    working_dir: "/home/Elaina/ros2_driver" # 指定默认工作目录
    # entrypoint: [ "/bin/bash", "-i", "-c", " source ~/.bashrc && ros2 launch rc_bringup didi.launch.py "]
```
- environment中添加了`RMW_IMPLEMENTATION=rmw_cyclonedds_cpp`，即用Middleware层的RMW接口指定使用Cyclone DDS作为通信件，并采用了`ROS_LOCALHOST_ONLY=1`，即只允许本地通信，避免了**UDP组播的网络干扰**，**此问题将会在后续的通信问题中详细说明**。
## DDS带来的发现灾难
- Fast DDS和Cyclone DDS都是基于DDS（Data Distribution Service）标准的通信件。虽然Cyclone DDS提升了Fast DDS带来的通信效率问题，但两者由于其UDP组播的特性，在我们的**Docker多容器通信或同局域网多主机环境下，出现薛定谔发现式的通信问题**
  - 具体而言，当使用DDS通信时，存在跨容器、容器与宿主间
    - 1.**ros2 topic list无法发现任何已有话题**，单个容器中的topic疑似隔离问题（即使加上了“host”模式）
    - 2.ros2 topic list可以发现实际存在的话题，但**ros2 topic echo死活找不到实际正在发布的话题数据**的问题
    - 3.使用cyclone时，若不设置`ROS_LOCALHOST_ONLY=1`而采用默认值，会受到同局域网下其他主机ROS2话题的串扰污染。
## DDS问题的解决方式
- 当我们发现了上述问题后，便开始着手解决。
  - 为解决**同局域网不同主机的串扰，且防止同主机内不同容器找不到话题的问题**，需在docker-compose.yml中添加`environment`配置项`ROS_LOCALHOST_ONLY=1`，只允许本地通信。
  - 为了解决上述1、2的话题、数据发布通信问题，我们需要在每次启动主机时，运行如下指令，之后可确保
    - 1. ros2 topic list可以发现所有话题
    - 2. ros2 topic echo可以正常接收话题数据
```bash
sudo ip link set lo multicast on
```
> Q: 为什么设置lo的multicast为on即可解决UDP组播发现的问题？
**A：由于绝大多数 Linux 发行版（如 Ubuntu），默认是关闭 lo 网卡的组播（Multicast）功能的**，而Cyclone DDS 及其他的 DDS 协议寻找其他节点的第一步（SPDP 发现阶段），一定都要依赖 UDP 组播。
- 导致启动的流程中：
  - 当 Cyclone DDS 启动时，它看到 ROS_LOCALHOST_ONLY=1，于是**使用本机 lo 网卡**。
  - 然后它试图在 lo 网卡上**发送组播包寻找其他节点**。
  - 结果：因为 Linux 内核**默认关闭了 lo 的组播**，这些发现包就像撞上了一堵无形的墙，直接被内核丢弃。没有报错，也没有警告，就是默默地发不出去。
## Zenoh
- Zenoh是一个由Rust语言开发的通信件，**支持多种传输协议，基于TCP**，它的通信有些类似于ROS1的roscore，采用了**中心化的通信模式**。
  - 他可以通过开启一个Zenoh Router来实现多容器、多主机的通信，帮助各个`rmw`设置为`rmw_zenoh_cpp`的ROS2节点进行**星型拓扑的通信**，极大节省了管理开销。
  - 极其适配**单机共享内存 (SHM)**，业务节点只需通过标准的 TCP 单播 或者 共享内存 (SHM) 去连接本地的 127.0.0.1 Router。而 TCP 单播在 lo 网卡上是永远默认畅通无阻的。他能够让需要通信的数据只在内存中写一次。
  - **同时这也成为他的一大缺点**，当通信数据量过大时，假设**不额外设置共享内存或设置的共享内存过小**，Zenoh Router的TCP单播通信会出现阻塞，导致**ROS2话题数据发布严重延迟，甚至出现数据丢失（别问真的吗，问就是真的调试时出现过定位数据延后从十几秒到几分钟）。**
- Zenoh用于Docker的设置如下所示：

```yaml
name: driver
x-common-env: &common_env
  DISPLAY: ${DISPLAY}
  NVIDIA_VISIBLE_DEVICES: all
  NVIDIA_DRIVER_CAPABILITIES: all
  OMP_WAIT_POLICY: passive
  TERM: xterm-256color
  RMW_IMPLEMENTATION: rmw_zenoh_cpp
  ROS_LOCALHOST_ONLY: 1
  ZENOH_CONFIG_OVERRIDE: 'transport/shared_memory/enabled=true'
  ZENOH_SHM_ALLOC_SIZE: 67108864
x-common: &common
  tty: true
  network_mode: host
  pid: "host"
  ipc: "host"
  privileged: true
  user: "Elaina"
  stdin_open: true
  ulimits:
    memlock:
      soft: -1
      hard: -1
services:
  zenoh_service:
    image: elainasuki/rc2026:driverLogic
    container_name: zenoh_runtime
    restart: always
    <<: *common
    entrypoint:
      - /bin/bash
      - -i
      - -c
      - |
        if pgrep -x 'rmw_zenohd' >/dev/null; then
          echo '[zenoh] rmw_zenohd already running'
          exec sleep infinity
        else
          echo '[zenoh] starting rmw_zenohd...'
          exec ros2 run rmw_zenoh_cpp rmw_zenohd
        fi
  driverLogic_service:
    image: elainasuki/rc2026:driverLogic
    container_name: driverLogic_runtime
    restart: always
    <<: *common
    group_add:
      - "${DOCKER_GID:-999}"
    environment:
      <<: *common_env
      PYTHONPATH: /home/Elaina/ros2_ws/src:$$PYTHONPATH
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix
      - driverLogic_volume:/home/Elaina/ros2_ws
      - /var/run/docker.sock:/var/run/docker.sock
      - /dev:/dev
    working_dir: "/home/Elaina/ros2_ws"
    entrypoint: ["/bin/bash", "-i", "-c", "source ~/.bashrc && ros2 launch rc_bringup R2n.launch.py"]
  ros2yolo-service:
    image: elainasuki/rc2026:yolo-ros #cpu镜像
    container_name: yolo_ros_runtime
    restart: always
    <<: *common
    environment:
      <<: *common_env
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix
      - ros2_opencv_volume:/home/Elaina/yolo
      - /dev:/dev
    working_dir: "/home/Elaina/yolo"
    entrypoint: ["/bin/bash", "-i", "-c", "python3 /home/Elaina/yolo/spear/ros2_arucopnp_serial_node.py"]
  voxel_service:
    image: elainasuki/rc2026:voxel_slam_ros2
    container_name: voxel_slam_ros2_runtime
    restart: always
    <<: *common
    environment:
      <<: *common_env
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix
      - voxel_ros2_volume:/home/Elaina/ros2_ws
    working_dir: "/home/Elaina/slam/VoxelSlam"
    entrypoint: ["/bin/bash", "-i", "-c", "source ~/ros2_ws/install/setup.bash && ros2 launch voxel_slam vxlm_airy.launch.py"]
volumes:
  ros2_opencv_volume:
    external: true
    name: ros2_opencv_volume
  driverLogic_volume:
    external: true
    name: driverLogic_volume
  voxel_ros2_volume:
    external: true
    name: voxel_ros2_volume
```