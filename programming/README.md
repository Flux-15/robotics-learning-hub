# 💻 Programming & Linux for Robotics

<div align="center">

[ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 🗺️ Full Roadmap ](../robotics-roadmap.md) &nbsp;•&nbsp; [ 📑 Linux & Git Cheat Sheet ](../cheatsheets/linux-and-git-cheatsheet.md)

</div>

---

## 📌 Overview & Scope
In modern robotics, software is the nervous system. Robotics software engineering requires an uncompromising dual-paradigm skillset:
1. **Low-latency, deterministically fast real-time code (C++20)** for closed-loop motor control, kinematics, state estimation, and sensor drivers.
2. **High-velocity scientific and AI programming (Python 3.12)** for rapid prototyping, deep learning perception, reinforcement learning, and data analysis.
3. **Rock-solid Linux Systems Engineering** to handle real-time kernels (PREEMPT_RT), hardware device drivers, networking protocols, and containerized deployment.

```mermaid
graph LR
    subgraph REALTIME["Hard Real-Time Layer (C++20)"]
        A["Motor Control Loops (1kHz)"]
        B["IMU / Sensor Sampling (500Hz)"]
        C["Kinematics & Dynamics (250Hz)"]
        D["LiDAR Point Cloud Ingestion"]
    end

    subgraph HIGHLEVEL["High-Level Autonomy (Python / C++)"]
        E["Spatial Perception & SAM 2"]
        F["Global Path Planning (Nav2)"]
        G["Diffusion Policy / VLA (PyTorch)"]
        H["Fleet Management & Dashboards"]
    end

    REALTIME <==>|DDS / Zero-Copy Shared Memory| HIGHLEVEL

    style REALTIME fill:#1e1e2e,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
    style HIGHLEVEL fill:#181825,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
```

---

## 🔑 Core Technical Concepts

### 1. Modern C++ for Robotics (C++17/20)
* **Resource Acquisition Is Initialization (RAII):** Never use manual `new` and `delete`. Use `std::unique_ptr` for exclusive hardware handle ownership and `std::shared_ptr` for shared message pipelines.
* **Real-Time Safety Rules:** In a 1 kHz control loop, NEVER allocate heap memory (`malloc`, `new`, resizing `std::vector`), acquire non-bounded mutex locks, or perform disk/terminal I/O (`std::cout`). Pre-allocate buffers upfront.
* **Modern Threading & Concurrency:** Utilize `std::jthread` (C++20 auto-joining thread), `std::atomic<bool>` for thread-safe state flags, and lock-free ring buffers for passing sensor samples from ISR/driver threads to estimation threads.
* **Modern CMake Targets:** Structure packages cleanly with target-based modern CMake:
  ```cmake
  add_library(robot_core src/kinematics.cpp)
  target_include_directories(robot_core PUBLIC include)
  target_link_libraries(robot_core PUBLIC Eigen3::Eigen)
  ```

### 2. High-Performance Python in Robotics
* **Vectorized Computing:** Replace slow Python `for` loops with vectorized NumPy matrix operations.
* **C++ Bindings (pybind11 / nanobind):** Write computationally intense algorithms (e.g., custom collision checkers or ray casters) in C++ and expose them seamlessly to Python.
* **Interprocess Communication:** Use shared memory (`multiprocessing.shared_memory`) or zero-copy IPC when passing high-bandwidth 4K camera frames between Python vision nodes.

### 3. Linux Systems Mastery
* **Hardware Device Rules (`udev`):** USB devices can change port names upon reboot (`/dev/ttyUSB0` vs `/dev/ttyUSB1`). Create persistent symlinks based on vendor and product IDs:
  ```bash
  # /etc/udev/rules.d/99-robot-sensors.rules
  SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", SYMLINK+="rplidar"
  ```
* **Real-Time Kernel (PREEMPT_RT):** Replace standard Linux kernel scheduling with PREEMPT_RT to guarantee deterministic thread wake-ups within microseconds.
* **systemd Robot Daemons:** Ensure automatic robot software stack initialization on boot with dependency-ordered systemd unit files.

---

## 🛠️ Industry Standard Toolchain

| Category | Gold-Standard Tool | Why It Is Used |
| :--- | :--- | :--- |
| **Language** | C++20 / GCC 13+ & Clang 17+ | Maximum deterministic execution speed and memory efficiency |
| **Language** | Python 3.10 – 3.12 | Scientific computing, PyTorch models, scripting |
| **Linear Algebra** | [Eigen 3](https://eigen.tuxfamily.org/) | Header-only C++ template library for matrix operations and transformations |
| **Build System** | [CMake](https://cmake.org/) & [Colcon](https://colcon.readthedocs.io/) | Industry standard C++ build definition and ROS 2 workspace orchestrator |
| **Containers** | [Docker](https://www.docker.com/) + NVIDIA Container Toolkit | Consistent execution across development laptops, cloud, and edge SBCs (Jetson) |
| **Version Control**| Git & [Git LFS](https://git-lfs.com/) | Version control with large file storage for CAD meshes (.stl, .dae) and weights |

---

## 📚 Curated Learning Media

### 🎓 Premier Courses
- [The Missing Semester of Your CS Education (MIT)](https://missing.csail.mit.edu/) — Essential mastery of the command line, Vim, Git, SSH, and shell scripting.
- [C++ for C Programmers (Coursera / UC Santa Cruz)](https://www.coursera.org/learn/c-plus-plus-a) — Object-oriented and generic programming.
- [Introduction to Linux (Linux Foundation / edX)](https://www.edx.org/course/introduction-to-linux) — Foundational OS internals.

### 📖 Essential Books
- *A Tour of C++ (3rd Edition)* by Bjarne Stroustrup — Concise overview of modern C++20 by the creator of the language.
- *Effective Modern C++* by Scott Meyers — Best practices covering lambdas, rvalue references, move semantics, and smart pointers.
- *How Linux Works (3rd Edition)* by Brian Ward — Deep dive into the kernel, processes, storage, and networking.
- *Hands-On System Programming with C++* by Rian Quinn — Low-level memory, POSIX threads, and system calls.

### 🎥 High-Yield Video Channels
- [CppCon (YouTube)](https://www.youtube.com/@CppCon) — Watch talks on "Real-Time Programming in C++" and "Back to Basics: Smart Pointers".
- [The Cherno: C++ Series (YouTube)](https://www.youtube.com/playlist?list=PLlrATfBNZ98dudnM48yfGUldqGD0S4G5b) — Crystal-clear visual breakdowns of memory, pointers, and compilation.
- [Corey Schafer: Python & Git (YouTube)](https://www.youtube.com/@coreyms) — Practical software developer workflows.

### 💻 Open-Source Repositories to Study
- [isocpp/CppCoreGuidelines](https://github.com/isocpp/CppCoreGuidelines) — The canonical set of guidelines for modern C++.
- [google/googletest](https://github.com/google/googletest) — Unit testing framework essential for reliable robotics code.
- [pybind/pybind11](https://github.com/pybind/pybind11) — Seamless interoperability between C++11/17 and Python.

---

## ⚡ Quick-Reference Snippet: Real-Time Safe Thread in C++

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <atomic>
#include <sched.h>
#include <pthread.h>

std::atomic<bool> is_running{true};

void real_time_control_loop(int loop_hz) {
    // Set thread scheduling to real-time FIFO priority
    sched_param sch_params;
    sch_params.sched_priority = 80; // High real-time priority
    if (pthread_setschedparam(pthread_self(), SCHED_FIFO, &sch_params) != 0) {
        std::cerr << "Warning: Could not set SCHED_FIFO (requires root/CAP_SYS_NICE)\n";
    }

    const auto period = std::chrono::nanoseconds(1000000000 / loop_hz);
    auto next_wake_time = std::chrono::steady_clock::now();

    while (is_running.load(std::memory_order_relaxed)) {
        next_wake_time += period;

        // --- HARD REAL-TIME CRITICAL SECTION (NO HEAP ALLOCATIONS!) ---
        // 1. Read sensor buffers
        // 2. Compute state estimate & control effort (LQR / PID)
        // 3. Write motor PWM command
        // -------------------------------------------------------------

        std::this_thread::sleep_until(next_wake_time);
    }
}
```

---

<div align="center">

[ ⬅️ Return to Hub Home ](../README.md) &nbsp;•&nbsp; [ Next: Mathematics for Robotics ➡️ ](../mathematics/README.md)

</div>
