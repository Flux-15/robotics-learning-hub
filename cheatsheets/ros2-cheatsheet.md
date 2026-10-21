# 🤖 ROS 2 Engineering CLI & Syntax Cheat Sheet

<div align="center">

[ ⬅️ Back to Cheatsheets ](README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 🤖 ROS 2 Module ](../ros/README.md)

</div>

---

## 🛠️ Essential CLI Commands (Humble & Jazzy)

### 1. Nodes & Graph Introspection
```bash
ros2 node list                              # List all active nodes
ros2 node info /my_robot_node               # Detailed subscriptions, publications, services, actions
```

### 2. Topics & Streaming Data
```bash
ros2 topic list                             # List active topics
ros2 topic list -t                          # List topics with their message types
ros2 topic echo /odom                       # Stream message contents to terminal
ros2 topic hz /scan                         # Measure publishing frequency
ros2 topic bw /camera/image_raw             # Measure bandwidth consumed by topic
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.5}, angular: {z: 0.2}}" -r 20 # Pub at 20 Hz
ros2 topic type /tf                         # Output message type definition
```

### 3. Services (Request / Response)
```bash
ros2 service list                           # List available services
ros2 service type /reset_odometry           # Get service interface type
ros2 service call /clear_costmaps nav2_msgs/srv/ClearEntireCostmap "{request: {}}" # Trigger service call
```

### 4. Actions (Goal / Feedback / Result)
```bash
ros2 action list                            # List available action servers
ros2 action info /navigate_to_pose          # Inspect action feedback/goal types
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
    "{pose: {header: {frame_id: 'map'}, pose: {position: {x: 1.0, y: 2.0}}}}" --feedback
```

### 5. Parameters & Dynamic Reconfiguration
```bash
ros2 param list                             # List parameters across all nodes
ros2 param get /controller_server max_vel_x # Query specific parameter value
ros2 param set /controller_server max_vel_x 0.8 # Dynamically update parameter at runtime
ros2 param dump /controller_server          # Dump parameters to YAML file
```

---

## 🧭 Coordinate Transforms (TF2)

```bash
# View the live transform tree as a PDF diagram
ros2 run tf2_tools view_frames

# Output real-time transform between two coordinate frames
ros2 run tf2_ros tf2_echo map base_link

# Broadcast a static coordinate transform (x y z yaw pitch roll parent child)
ros2 run tf2_ros static_transform_publisher 0.15 0.0 0.25 0 0 0 base_link laser_link
```

---

## 📦 Workspace Build Commands (`colcon`)

```bash
# Build all packages with symlinked install (no rebuild needed for Python edits)
colcon build --symlink-install

# Build ONLY a single package and its dependencies
colcon build --packages-select my_robot_bringup --symlink-install

# Build with compiler flags for maximum C++ speed
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release

# Clean and rebuild fresh
rm -rf build/ install/ log/ && colcon build --symlink-install
```

---

## 📼 Data Recording & Playback (rosbag & MCAP)

```bash
# Record all topics into modern MCAP format with zstd compression
ros2 bag record -a -s mcap --compression-mode file --compression-format zstd -o test_run_01

# Record specific high-yield topics
ros2 bag record /scan /tf /tf_static /odom /cmd_vel -s mcap -o mapping_data

# Inspect recorded bag file metadata
ros2 bag info mapping_data.mcap

# Play back at 0.5x speed in loop
ros2 bag play mapping_data.mcap -r 0.5 -l
```

---

## ⚡ Quality of Service (QoS) Cheat Sheet

```cpp
#include <rclcpp/rclcpp.hpp>

// 1. Sensor Profile (Best-Effort, drops packets to eliminate lag)
auto sensor_qos = rclcpp::SensorDataQoS(); // Reliability: Best-Effort, Durability: Volatile, Depth: 5

// 2. Latched Map / Static Frame Profile (Transient Local, late joiners get message)
auto latched_qos = rclcpp::QoS(1)
    .reliable()
    .transient_local();

// 3. Command / Control Profile (Guaranteed delivery)
auto control_qos = rclcpp::QoS(10)
    .reliable()
    .durability_volatile();
```
