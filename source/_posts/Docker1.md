---
title: Docker踩坑与总结(含有ROS联合开发)
categories: 
  - 技术
  - Docker
tags: [技术博文,ROS1/2,SLAM,Docker]
date: 2025-03-16
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  Docker踩坑与总结(含有ROS联合开发)
</div>

# 前言
- 笔者最初是在工控机上进行ROS/ROS2的开发，吃了无数配置环境的史

- ROS环境下的依赖安装与环境隔离在开发中需要在开发中实际考虑的重要一项

- 并且在以几大实验室开源的Star数极多的github有关SLAM项目(包括[FAST_LIO](https://github.com/hku-mars/FAST_LIO)，[LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)，[Point-LIO](https://github.com/hku-mars/Point-LIO)，[rtabmap](https://github.com/introlab/rtabmap)等笔者看过的建图，定位算法），其环境配置ROS1/2参差不齐，在同台宿主机上反复的重新配置ROS1和2的环境极其繁琐

- 在工作室同学的推荐下，接触到了Docker开发容器，发现其是解决这一环境配置问题和环境隔离问题的好东西

- 经过一段时间的开发和踩坑，遂将一些心得体会分享出来。

## Docker的优势运用
- 隔离不同容器间的环境，不同容器可以有截然不同的环境配置和依赖（请记住容器和镜像这两个词，可以类比C++的类的实例化与类来理解）

- 容器可以无限创建，并且理论上而言可以将一切的环境配置用镜像记录下来，一旦构建好镜像，以此生成的容器可以被任何人轻松构建，自动完成多个相同环境的容器配置

- Docker的容器与宿主机（即你的主机），在容器创建开启时自动配置网络桥接（关于网络，自动化专业的笔者也不是特别的清楚，具体可以看看这篇[帖子](https://blog.csdn.net/succing/article/details/122433770)），大致可以理解成相当于宿主机和各个容器全连接到docker自己生成的虚拟交换机，然后各个容器的网络请求被docker虚拟交换机传输到宿主机，走宿主机的网络传输，这个模型中可以将宿主机看作连接容器的转发路由器。

## 我们的选择
 第三点的网络特点在同样支持局域网协议的ROS是极其有益的，但这种情况下ROS1中的Master中心化给局域网配置带来极大的不便，而ROS2同一局域网下天然支持节点相互通讯，则给前述的容器-ROS2开发带来了更多的通讯选择，笔者和同学在实际开发时也遇到过不少ROS1的局域网通信问题，我们采用的是ROS2容器-ROS1/2桥接容器的方法。

| 使用内容                                               | 说明         |
| -------------------------------------------------- | ------------ |
| 容器                 | 解决单机上各个不同容器通信 |
| ROS2 | 解决同一局域网下不同主机上的ROS2容器的通信   |
| [ROS1-2桥接](https://github.com/TommyChangUMD/ros-humble-ros1-bridge-builder)（我们找到的github项目原始版)   | 让ROS1/2之间的话题可以互相发现

# Docker的基础使用
## 下载Docker(鱼香ros一键安装)
```bash
wget http://fishros.com/install -O fishros && . fishros ##然后根据终端提示项一键下载docker
```

## 你需要构建/拉取一个镜像
### 从[DockerHub](https://hub.docker.com/)开源镜像网站拉取镜像（需要科学上网）
```bash
docker pull 镜像名/Tags(在官网查看)
```
### 自己构建镜像
- 同样需要先拉取一个镜像作为基础镜像
- 将这个基础镜像在你的Dockerfile以FROM xxx（image）的格式，以该镜像作为基础镜像开始构建，具体的攥写这里不多赘述，如果是着急使用可以修改一下他人已经攥写好的Dockerfile，格式大差不差

## 管理镜像
可以通过docker-compose工具来管理镜像
- 这个工具是一个用于定义和运行多容器应用程序的工具
- 通过过 Compose，可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务

Compose 使用的三个步骤：
- 使用 Dockerfile 定义应用程序的环境
- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行
- 最后，执行 docker-compose up 命令来启动并运行整个应用程序
```bash
sudo apt install docker-compose 
```

# 重点解析：Dockerfile和docker-compose.yml文件的语法和文件结构，运行逻辑
## 文件结构
- 附笔者某github的docker仓库结构
```text
.
├── .devcontainer
│   ├── docker-compose.yaml
│   └── Dockerfile
├── .gitignore
├── nagisa_ws
│   ├── build
│   ├── .catkin_workspace
│   ├── devel
│   ├── .gitignore
│   └── src
├── README.md
├── ros_manager
│   ├── build
│   ├── .catkin_workspace
│   ├── devel
│   ├── .gitignore
│   └── src
└── walking_dataset.bag
```
- 文件结构最好与笔者保持一致(如果是docker-compose管理)
- 两个容器重要文件存放在.devcontainer下的原因是如果使用vscode的dev container扩展项，必须要求文件夹名为.devcontainer，在其中加入vscode的指定格式.json可以方便的配置容器内vscode,编辑容器内文件（此处没有添加，感兴趣可以去搜一艘）

## Dockerfile语法格式常用
- 此处粘贴一篇笔者Dockerfile文件，根据注释可以了解到笔者对每条语法指令的观点（如果要直接使用请删除笔者某些注释）
```bash
# 指定基础镜像，以此为基础镜像来构建容器
FROM osrf/ros:kinetic-desktop-full-xenial 

# 定义用户和用户ID，为容器中的用户名，用户UID，和用户组GID
ARG USERNAME=Nagisa
ARG USER_UID=1000
ARG USER_GID=$USER_UID


# 创建容器的用户和组
# 创建一个新用户组，并指定固定 GID。（第一条）
​意义：确保容器内用户组 ID 与宿主机用户组 ID 一致，解决跨系统文件权限问题，所以请务必确保宿主机用户所属GID和此处定义的GID一致，否则会有奇怪的权限报错。
RUN groupadd -g $USER_GID $USERNAME && \
    useradd -m -u $USER_UID -g $USER_GID -s /bin/bash $USERNAME
第二条：
-m：自动创建用户家目录（如 /home/Nagisa），避免权限问题。
-u 和 -g：​固定 UID 和 GID，与宿主机用户匹配。
-s /bin/bash：设置默认 Shell，方便进入容器后操作。
​意义：容器内用户与宿主机用户共享相同的 UID/GID，使得容器内生成的文件在宿主机上可直接读写，无需 chown。


# 切换到 root 用户来执行系统软件包安装
USER root

RUN apt-get update \
    && apt-get install -y curl \
    && curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add - \
    && apt-get update \
    && apt-get install -y ros-kinetic-navigation \
    && apt-get install -y ros-kinetic-rqt-tf-tree \
    && apt-get install -y ros-kinetic-robot-localization \
    && apt-get install -y ros-kinetic-robot-state-publisher \
    && rm -rf /var/lib/apt/lists/*

# 安装该容器中所需依赖项
RUN apt-get update \
    && apt install -y software-properties-common \
    && add-apt-repository -y ppa:borglab/gtsam-release-4.0 \
    && apt-get update \
    && apt install -y libgtsam-dev libgtsam-unstable-dev \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install -y nautilus \
    && apt-get install -y gedit \
    && rm -rf /var/lib/apt/lists/*

# 添加 ROS 环境设置到 .bashrc
RUN echo "source /opt/ros/kinetic/setup.bash" >> /home/${USERNAME}/.bashrc

# 切换回 Nagisa 用户
USER $USERNAME

# 创建工作空间并克隆代码库
RUN mkdir -p ~/liosam_ws/src \
    && cd ~/liosam_ws/src \
    && git config --global http.postBuffer 524288000 \
    && git config --global http.timeout 300 \
    && git clone --depth 1 --branch master https://github.com/TixiaoShan/LIO-SAM.git 

# 编译clone下的源代码
RUN cd ~/liosam_ws \
    && /bin/bash -c "source /opt/ros/kinetic/setup.bash && catkin_make "
# /bin/bash -c是以为source和catkin的指令必须指定bash为shell来运行

# 切换回 Nagisa 用户
USER $USERNAME

# 设置工作目录
WORKDIR /home/$USERNAME

# 设置容器启动时默认执行的命令，并自动加载工作空间
CMD ["/bin/bash"]

```
- 上述代码中可能会有冗余，但是一般笔者的dockerfile都是类似这样的结构编写

常见指令字
- FROM：用于指定dockerfile文件的镜像起点，以什么为基础镜像

- RUN：用于指定构建文件时候要采取的操作，包括各项安装指令，操作指令

- ARG：用于指定变量内容，后续可以引用被赋值变量

- USER：切换 Docker 容器中的运行用户，默认情况下，Docker 容器内的进程以 root 用户 运行。但在某些情况下，你可能希望以非 root 用户运行。

- WORKDIR：设置容器内的工作目录，它影响所有后续指令（如 RUN、COPY、CMD、ENTRYPOINT 等）的默认目录

- CMD：用于指定容器启动时默认执行的命令，但不是强制的，可以被 docker run 提供的命令覆盖。例：CMD ["echo", "Hello, Docker!"]。其为JSON（exec 形式）命令关键字在前，执行内容在后

- COPY：语法格式：
```bash
COPY [源路径] [目标路径] 
```
源路径：本地文件或目录（相对路径或绝对路径）\
目标路径：容器内的路径（必须是绝对路径）\
注意!!!!!COPY 只能复制 构建上下文（build context）内的文件,当使用docker-compose时则只能复制context: .的文件（在笔者的文件路径下只能复制.devcontainer文件夹中的文件）

- ADD：与COPY 一样，只能复制构建上下文内的文件（不会访问宿主机任意路径），但可以复制自动解压 .tar 压缩包（COPY 不会自动解压）

## docker-compose.yml文件的语法格式
- 同附docker-compose.yml文件
 ```bash
 version: '3' ## 指定 Docker Compose 文件格式版本
services:
  lio_sam_container: ## 这里是笔者之前没注意到的问题，此处为服务名，一个服务在没有指定下面的container_name时可以启动多个容器，且下面的所有挂载等的设置全部都设置的是一个服务的参数
    build:
      context: .
      dockerfile: Dockerfile ## 指定参与构建的dockerfile文件名
    image: Nagisa/lio_sam ## 构建出的镜像名，这个是针对dockerfile的
    container_name: lio_sam_container ## 指定构建出的容器名
    environment: ## 环境变量设置
      - DISPLAY=${DISPLAY} ## 允许容器内 GUI 应用显示到宿主机屏幕（需配合 X11 挂载）
      - NVIDIA_VISIBLE_DEVICES=all ## 允许访问所有 NVIDIA GPU
      - NVIDIA_DRIVER_CAPABILITIES=all ## 启用所有 GPU 驱动功能
      - OMP_WAIT_POLICY=passive # 优化 OpenMP 多线程性能
      - TERM=xterm-256color # 终端颜色支持
    volumes: ## 挂载配置
      # - ${XAUTHORITY}:/root/.Xauthority # X11 认证文件，！添加该项后不用每次重启电脑后必须要在主机执行xhost local:才可在容器中使用图形界面
      - /tmp/.X11-unix:/tmp/.X11-unix # X11 套接字挂载
      - ./../..:/home/Nagisa/packages # 挂载项目代码到容器
      - /dev:/dev  # 挂载宿主设备文件
    network_mode: host # 容器共享宿主机的网络命名空间，直接使用宿主机 IP 和端口
    pid: "host" # 添加 pid 命名空间共享，容器内可见宿主机进程，便于调试或与宿主机进程交互
    ipc: "host" # 添加 ipc 命名空间共享，允许容器与宿主机通过共享内存通信   
    privileged: true # 赋予容器所有内核权限（近似于 root 权限）
    stdin_open: true # 保持 STDIN 打开
    tty: true # 分配伪终端（TTY）
    user: "Nagisa" # 以指定用户（而非 root）运行容器进程
    #runtime: nvidia #启用 NVIDIA GPU 支持（需取消注释并安装 NVIDIA Container Toolkit）
    working_dir: "/home/Nagisa" # 指定默认工作目录
```
部分问题来源细节：
- X Server 的安全策略：
X Window 系统默认只允许当前用户连接的客户端显示图形界面。
Docker 容器被视为“外部客户端”，即使它运行在本地，也需要显式授权才能访问 X Server。

- xhost +local: 的作用：
该命令临时允许所有本地用户连接 X Server（相当于关闭访问控制）。
未执行此命令时，容器内的 rviz 会被 X Server 拒绝连接，导致报错，挂载宿主机的 X 认证文件（~/.Xauthority）到容器内后彻底解决
- context: .：必须要这样写的原因，context只能指定一个，即构造目录起始点，与docker的节省缓存构建有关

## 运行逻辑：为什么docker-compose的镜像，服务管理方式具有复用性
Docker Compose的核心思想就是通过服务（service），将镜像构建（images），运行配置和资源管理统一封装（volumes，environment），使其成为可复用的模板

### Docker Compose 的核心逻辑

​服务（Service）​ 是一个 ​容器模板，它定义了：
- ​如何构建镜像​（通过 build + Dockerfile）
- ​如何运行容器​（通过 image、volumes、environment 等配置项
- ​容器（Container）​ 是服务的​实例化对象，一个服务可以启动多个容器
​
### 服务如何复用
#### ​镜像复用
- 如果服务中指定了 build，Docker Compose 会根据 Dockerfile ​生成镜像，并保存到本地（默认名称为 `<project>_<service_name>`，或通过 image 字段自定义名称）
- 该镜像可以被其他服务或其他项目复用
```yaml
# 复用已存在的镜像（无需重新构建）
services:
  another_service:
    image: Nagisa/lio_sam  # 直接使用已构建的镜像
    volumes: ...
```
#### ​配置复用
- 服务的所有配置项（如 volumes、environment、network）都定义在服务块内，​每次启动容器时都会应用这些配置

可以通过以下方式复用服务配置：
- ​横向扩展：使用 docker-compose up --scale service_name=N 启动多个容器实例 

- ​继承扩展：使用 extends 关键字继承其他服务的配置（需在 Compose 文件中定义）