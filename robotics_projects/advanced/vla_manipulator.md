# 🤖 Open-Source Embodied AI Manipulator with Hugging Face LeRobot

<div align="center">

[ ⬅️ Back to Projects Directory ](../README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../../README.md)

</div>

---

## 📌 Project Overview
Build and deploy a real-world **Embodied AI robotic arm system** powered by **Hugging Face LeRobot** for under **\$250**. 

Using a 3D-printed **SO-ARM100** (or Koch v1.1) 6-DoF arm driven by serial bus servomotors, this project demonstrates end-to-end modern robot learning: collecting human demonstrations via a 1:1 leader-follower teleoperation rig, recording multimodal datasets (dual cameras + joint proprioception), training a **Diffusion Policy** or **ACT (Action Chunking with Transformers)**, and deploying the trained policy for fully autonomous vision-guided manipulation.

```mermaid
graph TD
    subgraph PHASE1["1. HUMAN TELEOPERATION & DATASET RECORDING"]
        LEADER["Leader Arm (Passive 6-DoF) Hand-operated by human"]
        FOLLOWER["Follower Arm (Active 6-DoF) Mirrors leader in real time"]
        CAMS["Dual USB Cameras<br/>(1x Overhead Global View + 1x Wrist Camera)"]
        LEADER -->|Joint Angles (30 Hz)| FOLLOWER
        FOLLOWER --> DATASET["LeRobot Dataset Recorder<br/>(Stores RGB frames + joint state + action chunk)"]
        CAMS --> DATASET
    end

    subgraph PHASE2["2. NEURAL NETWORK POLICY TRAINING"]
        DATASET --> TRAIN["Train Diffusion Policy / ACT in PyTorch<br/>(Conditioned on Image Embeddings)"]
        TRAIN --> MODEL["Saved PyTorch Policy Checkpoint (.pt)"]
    end

    subgraph PHASE3["3. AUTONOMOUS INFERENCE DEPLOYMENT"]
        CAMS2["Dual Cameras"] --> MODEL
        STATE2["Arm Proprioception"] --> MODEL
        MODEL -->|Predicted 30 Hz Action Chunk| FOLLOWER2["Follower Arm Executes Task Autonomously!"]
    end

    style PHASE1 fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style PHASE2 fill:#181825,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
    style PHASE3 fill:#11111b,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4
```

---

## 📋 Bill of Materials (BOM)

| Component | Specification | Quantity | Interface / Voltage | Est. Cost (USD) |
| :--- | :--- | :---: | :--- | :---: |
| **Arm Servomotors** | Feetech STS3215 High-Torque Serial Bus Servos | 12 (6 for Leader, 6 for Follower) | 7.4V–12V Half-Duplex Serial UART | \$150.00 |
| **USB Serial Adapters**| Waveshare USB-to-UART / Bus Servo Control Board | 2 | USB-C to 3-pin servo bus | \$16.00 |
| **Cameras** | 2x Wide-Angle 1080p USB Webcams | 2 | USB 2.0 (UVC) | \$30.00 |
| **3D Printed Parts** | SO-ARM100 Mechanical CAD (PETG or PLA+) | 2 Sets | ~1.2 kg filament | \$25.00 |
| **Power Supply** | 7.4V 5A or 12V 5A DC Power Brick | 1 | Dedicated servo power | \$14.00 |
| **Fasteners** | M3 & M4 Socket Cap Screws + Bearings | 1 Kit | Hardware pack | \$12.00 |

---

## 🛠️ Step-by-Step Software Setup

### 1. Install Hugging Face LeRobot
```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
conda create -y -n lerobot python=3.10
conda activate lerobot
pip install -e ".[feetech]"
```

### 2. Configure Motors & Find Device Ports
Identify your USB serial adapters on Linux:
```bash
ls -l /dev/ttyUSB*
# Assign persistent udev symlinks for /dev/ttyFollower and /dev/ttyLeader
```

Calibrate motor zero-offsets:
```bash
python lerobot/scripts/control_robot.py \
    --robot-type=so100 \
    --robot-path=/dev/ttyFollower \
    --leader-path=/dev/ttyLeader \
    --teleoperate
```

### 3. Record 50 Human Teleoperation Demonstrations
Demonstrate the task (e.g., picking up an object and placing it in a container) 50 times:
```bash
python lerobot/scripts/record.py \
    --robot-type=so100 \
    --repo-id=local/so100_pick_and_place \
    --num-episodes=50 \
    --run-compute-stats=true
```

### 4. Train the Diffusion Policy
Train for 25,000 steps (takes ~1.5 hours on an NVIDIA RTX 3080/4080):
```bash
python lerobot/scripts/train.py \
    --dataset.repo_id=local/so100_pick_and_place \
    --policy.type=diffusion \
    --output_dir=outputs/train/diffusion_so100 \
    --training.batch_size=32 \
    --training.steps=25000 \
    --device=cuda
```

### 5. Deploy & Run Autonomously!
Remove the leader arm, power up the cameras, and watch the robot execute the learned behavior entirely from visual feedback:
```bash
python lerobot/scripts/eval.py \
    --robot-type=so100 \
    --policy-dir=outputs/train/diffusion_so100/checkpoints/last/pretrained_model
```

---

## 🔧 Pro-Tips for Real-World Policy Success
* **Consistent Lighting & Table Texture:** Avoid shiny tables that reflect harsh LED glare into the wrist camera. Use a matte colored silicone work mat.
* **Camera Rigidity:** Ensure camera mounts do NOT flex or vibrate during arm motion. A $1\text{cm}$ displacement in camera position between training and evaluation can degrade success rates significantly.
* **Gripper Finger Pads:** Cast or 3D print soft TPU / silicone pads onto the gripper jaws to dramatically improve grip friction on smooth objects.
