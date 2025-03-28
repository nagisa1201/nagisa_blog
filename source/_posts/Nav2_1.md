---
title: Navigation2开发心得
categories: 
  - 技术
  - Docker
tags: [技术博文,ROS1/2,SLAM,Nav2]
date: 2025-03-28
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  Navigation2开发心得
</div>

```yaml
bt_navigator:
  ros__parameters:
    use_sim_time: True  # 使用Gazebo等仿真器的时间而非系统时间
    global_frame: map  # 全局定位使用的地图坐标系
    robot_base_frame: base_link  # 机器人基坐标系
    odom_topic: /odom  # 里程计话题
    bt_loop_duration: 10  # 行为树循环周期(ms)
    default_server_timeout: 20 # 默认服务超时时间(s)
    wait_for_service_timeout: 1000  # 等待服务启动超时(ms)
    # 'default_nav_through_poses_bt_xml' and 'default_nav_to_pose_bt_xml' are use defaults:
    # nav2_bt_navigator/navigate_to_pose_w_replanning_and_recovery.xml
    # nav2_bt_navigator/navigate_through_poses_w_replanning_and_recovery.xml
    # They can be set here or via a RewrittenYaml remap from a parent launch file to Nav2.
    plugin_lib_names: # 行为树插件库列表（包含所有预定义的行为树节点）
      - nav2_compute_path_to_pose_action_bt_node
      - nav2_compute_path_through_poses_action_bt_node
      - nav2_smooth_path_action_bt_node
      - nav2_follow_path_action_bt_node
      - nav2_spin_action_bt_node
      - nav2_wait_action_bt_node
      - nav2_assisted_teleop_action_bt_node
      - nav2_back_up_action_bt_node
      - nav2_drive_on_heading_bt_node
      - nav2_clear_costmap_service_bt_node
      - nav2_is_stuck_condition_bt_node
      - nav2_goal_reached_condition_bt_node
      - nav2_goal_updated_condition_bt_node
      - nav2_globally_updated_goal_condition_bt_node
      - nav2_is_path_valid_condition_bt_node
      - nav2_initial_pose_received_condition_bt_node
      - nav2_reinitialize_global_localization_service_bt_node
      - nav2_rate_controller_bt_node
      - nav2_distance_controller_bt_node
      - nav2_speed_controller_bt_node
      - nav2_truncate_path_action_bt_node
      - nav2_truncate_path_local_action_bt_node
      - nav2_goal_updater_node_bt_node
      - nav2_recovery_node_bt_node
      - nav2_pipeline_sequence_bt_node
      - nav2_round_robin_node_bt_node
      - nav2_transform_available_condition_bt_node
      - nav2_time_expired_condition_bt_node
      - nav2_path_expiring_timer_condition
      - nav2_distance_traveled_condition_bt_node
      - nav2_single_trigger_bt_node
      - nav2_goal_updated_controller_bt_node
      - nav2_is_battery_low_condition_bt_node
      - nav2_navigate_through_poses_action_bt_node
      - nav2_navigate_to_pose_action_bt_node
      - nav2_remove_passed_goals_action_bt_node
      - nav2_planner_selector_bt_node
      - nav2_controller_selector_bt_node
      - nav2_goal_checker_selector_bt_node
      - nav2_controller_cancel_bt_node
      - nav2_path_longer_on_approach_bt_node
      - nav2_wait_cancel_bt_node
      - nav2_spin_cancel_bt_node
      - nav2_back_up_cancel_bt_node
      - nav2_assisted_teleop_cancel_bt_node
      - nav2_drive_on_heading_cancel_bt_node
      - nav2_is_battery_charging_condition_bt_node

bt_navigator_navigate_through_poses_rclcpp_node:
  ros__parameters:
    use_sim_time: True

bt_navigator_navigate_to_pose_rclcpp_node:
  ros__parameters:
    use_sim_time: True

controller_server:
  ros__parameters:
    use_sim_time: True
    controller_frequency: 20.0 # 控制频率(Hz)
    min_x_velocity_threshold: 0.001 # X方向最小速度阈值
    min_y_velocity_threshold: 0.5 # Y方向最小速度阈值
    min_theta_velocity_threshold: 0.001 # 角度最小速度阈值
    failure_tolerance: 0.3 # 控制器失败容忍度
    progress_checker_plugin: "progress_checker" # 进度检查插件
    goal_checker_plugins: ["general_goal_checker"] # "precise_goal_checker"目标到达判定插件
    controller_plugins: ["FollowPath"] # 此处为DWB局部规划器，启用的控制器插件

    # Progress checker parameters 插件参数配置
    progress_checker:
      plugin: "nav2_controller::SimpleProgressChecker"
      required_movement_radius: 0.5
      movement_time_allowance: 10.0
    # Goal checker parameters
    #precise_goal_checker:
    #  plugin: "nav2_controller::SimpleGoalChecker"
    #  xy_goal_tolerance: 0.25
    #  yaw_goal_tolerance: 0.25
    #  stateful: True
    general_goal_checker: # 目的达到检查插件
      stateful: True
      plugin: "nav2_controller::SimpleGoalChecker"
      xy_goal_tolerance: 0.25
      yaw_goal_tolerance: 0.25
    # DWB parameters
    FollowPath:  # 控制器插件参数配置
      plugin: "dwb_core::DWBLocalPlanner"
      debug_trajectory_details: True
      min_vel_x: -2.6 # 最小X线速度（后退速度，单位：m/s）
      max_vel_x: 2.6  # 最大X线速度（前进速度，单位：m/s）
      min_vel_y: -2.6 # 最小Y线速度（左移速度，全向轮机器人有效）
      max_vel_y: 2.6  # 最大Y线速度（右移速度，全向轮机器人有效）
      max_vel_theta: 2.6  # 最大角速度（单位：rad/s）
      min_speed_xy: 0.0   # XY平面最小合成速度（m/s）
      max_speed_xy: 0.26  # XY平面最大合成速度（m/s）
      min_speed_theta: 0.0 # 最小旋转速度（rad/s）
      # Add high threshold velocity for turtlebot 3 issue.
      # https://github.com/ROBOTIS-GIT/turtlebot3_simulations/issues/75
      acc_lim_x: 2.5    # X方向加速度上限（m/s²）
      acc_lim_y: 2.5    # Y方向加速度上限（全向轮机器人需调整）
      acc_lim_theta: 3.2 # 角加速度上限（rad/s²）
      decel_lim_x: -2.5 # X方向减速度下限（m/s²）
      decel_lim_y: -2.5 # Y方向减速度下限
      decel_lim_theta: -3.2  # 角减速度下限（rad/s²）
      vx_samples: 20 # X线速度采样数量
      vy_samples: 5  # Y线速度采样数量（差速机器人设为0）
      vtheta_samples: 20 # 角速度采样数量
      sim_time: 1.7  # 轨迹预测时长（秒）
      linear_granularity: 0.05  # 线性位移分辨率（米）
      angular_granularity: 0.025 # 角度分辨率（弧度）
      xy_goal_tolerance: 0.25    # 目标点XY位置容差（米）
      trans_stopped_velocity: 0.25  # 判定静止的线速度阈值（m/s）
      transform_tolerance: 0.2   # 坐标变换超时容差（秒）
      short_circuit_trajectory_evaluation: True  # 启用轨迹评估短路优化
      stateful: True  # 保持上次规划的最佳轨迹
      critics: ["RotateToGoal", "Oscillation", "BaseObstacle", "GoalAlign", "PathAlign", "PathDist", "GoalDist"]
      # 轨迹评分器(Critics)配置
      BaseObstacle.scale: 0.02    # 避障评分权重（低权重可能不安全）
      PathAlign.scale: 32.0       # 路径方向对齐权重
      PathAlign.forward_point_distance: 0.1  # 前瞻点距离（米）
      GoalAlign.scale: 24.0       # 目标方向对齐权重
      GoalAlign.forward_point_distance: 0.1  
      PathDist.scale: 32.0        # 路径距离评分权重
      GoalDist.scale: 24.0        # 目标距离评分权重
      RotateToGoal.scale: 32.0    # 末端旋转优化权重
      RotateToGoal.slowing_factor: 5.0  # 减速曲线系数
      RotateToGoal.lookahead_time: -1.0 # 动态前瞻时间（-1=自动计算）

local_costmap:
  local_costmap:
    ros__parameters:
      update_frequency: 5.0  # 代价地图更新频率(Hz)
      publish_frequency: 2.0 # 地图数据发布频率(Hz)
      global_frame: odom     # 全局坐标系（使用里程计坐标系）
      robot_base_frame: base_link  # 机器人本体坐标系
      use_sim_time: True     # 使用仿真时间
      rolling_window: true   # 启用滚动窗口模式
      width: 3    # 地图宽度3米
      height: 3   # 地图高度3米
      resolution: 0.05     # 分辨率5cm/格
      robot_radius: 0.22   # 机器人碰撞半径22cm
      plugins: ["voxel_layer", "inflation_layer"] # 激活的层级
      inflation_layer:   # 膨胀层
        plugin: "nav2_costmap_2d::InflationLayer"
        cost_scaling_factor: 3.0
        inflation_radius: 0.55  # 膨胀半径(m)
      voxel_layer:  # 3D体素层
        plugin: "nav2_costmap_2d::VoxelLayer"
        enabled: True # 启用该层
        publish_voxel_map: True # 发布3D体素地图
        origin_z: 0.0      # Z轴原点
        z_resolution: 0.05 # 垂直分辨率5cm
        z_voxels: 16  # 垂直方向体素数（16×0.05=0.8米高度）
        max_obstacle_height: 2.0  # 最大障碍物高度2米
        mark_threshold: 0  # 体素标记阈值（0表示1个点即标记）
        observation_sources: scan # 数据源名称
        scan:
          topic: /laser/sd/raw # 激光话题
          max_obstacle_height: 2.0 # 数据类型（实际为2D扫描）
          clearing: True # 用于标记障碍物
          marking: True # 用于清除空闲区域
          data_type: "LaserScan"
          raytrace_max_range: 3.0 # 光线追踪最大距离3米
          raytrace_min_range: 0.0
          obstacle_max_range: 2.5 # 有效障碍检测最大距离2.5米
          obstacle_min_range: 0.0
      static_layer:
        plugin: "nav2_costmap_2d::StaticLayer"
        map_subscribe_transient_local: True # 使用临时本地订阅
      always_send_full_costmap: True # 始终发布完整地图

global_costmap:
  global_costmap:
    ros__parameters:
      update_frequency: 1.0
      publish_frequency: 1.0
      global_frame: map
      robot_base_frame: base_link
      use_sim_time: True
      robot_radius: 0.22 # 机器人碰撞半径0.22米，用于膨胀层生成避障区域
      resolution: 0.05 # 地图分辨率为5厘米/像素
      track_unknown_space: true # 将未知区域视为可通行区域
      plugins: ["static_layer", "obstacle_layer", "inflation_layer"]
      obstacle_layer: # 启用的地图层级，按顺序叠加（静态层 → 障碍层 → 膨胀层）
        plugin: "nav2_costmap_2d::ObstacleLayer"
        enabled: True #  启用障碍物层
        observation_sources: scan # 障碍物检测的数据源
        scan:
          topic: /laser/sd/raw
          max_obstacle_height: 2.0
          clearing: True # 清除激光路径上的自由区域
          marking: True # 标记激光点云中的障碍物
          data_type: "LaserScan"
          raytrace_max_range: 3.0
          raytrace_min_range: 0.0
          obstacle_max_range: 2.5 # 有效障碍物检测最大距离2.5米
          obstacle_min_range: 0.0
      static_layer:  # 静态地图层
        plugin: "nav2_costmap_2d::StaticLayer"
        map_subscribe_transient_local: True
      inflation_layer:
        plugin: "nav2_costmap_2d::InflationLayer"
        cost_scaling_factor: 3.0 # 代价值随距离的衰减系数，值越大衰减越快
        inflation_radius: 0.55
      always_send_full_costmap: True

map_server:
  ros__parameters:
    use_sim_time: True
    # Overridden in launch by the "map" launch configuration or provided default value.
    # To use in yaml, remove the default "map" value in the tb3_simulation_launch.py file & provide full path to map below.
    yaml_filename: "" # 地图文件路径，通常通过启动参数传入（如map:=/path/map.yaml

map_saver:
  ros__parameters:
    use_sim_time: True
    save_map_timeout: 5.0
    free_thresh_default: 0.25 # 自由区域阈值，概率值小于0.25的栅格视为自由
    occupied_thresh_default: 0.65 # 占据区域阈值，概率值大于0.65的栅格视为障碍
    map_subscribe_transient_local: True

planner_server:
  ros__parameters:
    expected_planner_frequency: 20.0
    use_sim_time: True
    planner_plugins: ["GridBased"]
    GridBased:
      plugin: "nav2_navfn_planner/NavfnPlanner"
      tolerance: 0.5 # 目标点容差0.5米，允许规划路径终点在此范围内
      use_astar: true # ！！！！是否启动A*算法，若false则启动Dijkstra算法，A*快速响应动态障碍，适合人员流动频繁的区域
      allow_unknown: true # 允许在位置区域通行

smoother_server:
  ros__parameters:
    use_sim_time: True
    smoother_plugins: ["simple_smoother"]
    simple_smoother:
      plugin: "nav2_smoother::SimpleSmoother"
      tolerance: 1.0e-10
      max_its: 1000
      do_refinement: True

behavior_server:
  ros__parameters:
    costmap_topic: local_costmap/costmap_raw # 订阅本地代价地图的原始数据话题
    footprint_topic: local_costmap/published_footprint # 机器人轮廓话题
    cycle_frequency: 10.0
    behavior_plugins: ["spin", "backup", "drive_on_heading", "assisted_teleop", "wait"]
    spin:
      plugin: "nav2_behaviors/Spin"
    backup:
      plugin: "nav2_behaviors/BackUp"
    drive_on_heading:
      plugin: "nav2_behaviors/DriveOnHeading"
    wait:
      plugin: "nav2_behaviors/Wait"
    assisted_teleop:
      plugin: "nav2_behaviors/AssistedTeleop"
    global_frame: odom
    robot_base_frame: base_link
    transform_tolerance: 0.1 #  坐标变换容忍时间
    use_sim_time: true
    simulate_ahead_time: 2.0
    max_rotational_vel: 5.0 # ！！最大旋转速度1.0 rad/s
    min_rotational_vel: 0.4
    rotational_acc_lim: 3.2

robot_state_publisher:
  ros__parameters:
    use_sim_time: True

waypoint_follower:
  ros__parameters:
    use_sim_time: True
    loop_rate: 20
    stop_on_failure: false
    waypoint_task_executor_plugin: "wait_at_waypoint"
    wait_at_waypoint:
      plugin: "nav2_waypoint_follower::WaitAtWaypoint"
      enabled: True
      waypoint_pause_duration: 200

velocity_smoother:
  ros__parameters:
    use_sim_time: True
    smoothing_frequency: 20.0
    scale_velocities: False
    feedback: "OPEN_LOOP"
    max_velocity: [2.6, 2.6, 5.0] # 最大速度限制
    min_velocity: [-2.6, -2.6, -5.0] # 最小速度限制
    max_accel: [2.5, 2.5, 3.2] # 最大加速度限制
    max_decel: [-2.5, -2.5, -3.2]
    odom_topic: "odom"
    odom_duration: 0.1 # 里程计数据的有效期0.1秒
    deadband_velocity: [0.0, 0.0, 0.0] # 速度死区，忽略微小速度指令
    velocity_timeout: 1.0
```