# 💻 Linux, Git & Systems Engineering Cheat Sheet for Robotics

<div align="center">

[ ⬅️ Back to Cheatsheets ](README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 💻 Programming Module ](../programming/README.md)

</div>

---

## 🔌 Hardware Device Management (`udev` Rules)

### 1. Identifying USB Serial Hardware
When plugging in a LiDAR, IMU, or microcontroller:
```bash
# Query hardware vendor and product IDs
udevadm info -a -n /dev/ttyUSB0 | grep -E "idVendor|idProduct|serial"
```

### 2. Creating Persistent Device Symlinks
Create `/etc/udev/rules.d/99-robot.rules`:
```bash
# RPLiDAR A1 / A2 / C1 (Silicon Labs CP2102)
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", SYMLINK+="rplidar", MODE="0666"

# STM32 / Teensy Motor Controller (FTDI / CDC-ACM)
SUBSYSTEM=="tty", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", SYMLINK+="motor_controller", MODE="0666"
```
Reload rules:
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
# Devices will now persistently appear as /dev/rplidar and /dev/motor_controller!
```

---

## 🚗 SocketCAN & CAN-FD Interface Setup

Bring up high-speed CAN bus on a Raspberry Pi or Jetson:
```bash
# Configure CAN0 for standard 1 Mbps CAN 2.0B
sudo ip link set can0 type can bitrate 1000000
sudo ip link set up can0

# Configure CAN-FD (1 Mbps arbitration, 5 Mbps data phase)
sudo ip link set can0 type can bitrate 1000000 dbitrate 5000000 fd on
sudo ip link set up can0

# Sniff and transmit CAN frames
candump can0                             # Monitor all live bus traffic
cansend can0 123#DEADBEEF                # Transmit test frame with ID 0x123
```

---

## ⚙️ systemd Service: Auto-Start Robot Stack on Boot

Create `/etc/systemd/system/robot_bringup.service`:
```ini
[Unit]
Description=Autonomous Robot Bringup Daemon
After=network.target

[Service]
Type=simple
User=ubuntu
Environment="ROS_DOMAIN_ID=42"
Environment="RMW_IMPLEMENTATION=rmw_cyclonedds_cpp"
ExecStart=/bin/bash -c "source /opt/ros/jazzy/setup.bash && source /home/ubuntu/robot_ws/install/setup.bash && ros2 launch robot_bringup bringup.launch.py"
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```
Enable and test service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable robot_bringup.service
sudo systemctl start robot_bringup.service
sudo journalctl -u robot_bringup.service -f # Follow live log output
```

---

## ⚡ Real-Time Thread Priorities (PREEMPT_RT)

Allow user threads to acquire `SCHED_FIFO` real-time scheduling without requiring `sudo`:
Add to `/etc/security/limits.d/99-realtime.conf`:
```
@realtime   soft    rtprio          99
@realtime   soft    priority        99
@realtime   soft    memlock         unlimited
@realtime   hard    memlock         unlimited
```
Add your user: `sudo usermod -a -G realtime $USER`

---

## 🐳 Docker for Robotics with GPU Acceleration

Run GUI-enabled ROS 2 containers with NVIDIA CUDA and host networking:
```bash
docker run -it --rm \
    --net=host \
    --ipc=host \
    --privileged \
    --gpus all \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
    -v /dev:/dev \
    osrf/ros:jazzy-desktop
```

---

## 🌿 Git for Robotics (Submodules & Git LFS)

```bash
# Initialize and pull submodules recursively (e.g. driver libraries)
git submodule update --init --recursive

# Install Git Large File Storage for CAD models (.stl, .step) and neural weights
git lfs install
git lfs track "*.stl" "*.dae" "*.pt" "*.onnx"
git add .gitattributes
```
