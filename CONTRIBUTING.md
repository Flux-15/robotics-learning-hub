# 🤝 Contributing to The Robotics Learning Hub

First off, thank you for taking the time to contribute! 🎉 

The **Robotics Learning Hub** is a 100% free, community-driven, open-source knowledge base and engineering matrix. Whether you are fixing a typo, updating an obsolete ROS 2 command, reviewing an ICRA paper, or submitting a full robot build blueprint, your contribution makes robotics education more rigorous and accessible to engineers worldwide.

---

## 🧭 Code of Conduct

This project adheres to the **Contributor Covenant 2.1**. By participating, you are expected to uphold this code. Please review [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before contributing.

---

## 🚀 How Can I Contribute?

### 1. 🔬 Review a Landmark Paper
Read a foundational or recent research paper in SLAM, motion planning, manipulation, or robot learning?
* Create a markdown file under `research_papers/`.
* Follow our standard paper review template: Metadata, Mathematical Foundation, System Architecture diagram, and Real-World Impact.

### 2. 🛠️ Submit a Hardware / Simulation Project Blueprint
Built an open-source rover, arm, drone, or quadruped?
* Create a guide under `robotics_projects/beginner/`, `intermediate/`, or `advanced/`.
* Must include: Complete **Bill of Materials (BOM)** with voltage/price, electrical pinout wiring table, system architecture diagram, and tested, working source code.

### 3. 📑 Expand Cheatsheets & Formula Sheets
Add field-tested terminal one-liners, real-time C++ snippets, or kinematics formulas to `cheatsheets/`.

### 4. 🐛 Fix Bugs & Update Broken Links
Submit quick fixes for dead links, typos, or outdated software package flags.

---

## 🛠️ Step-by-Step Git Workflow

### 1. Fork & Clone
1. Fork the repository on GitHub: `https://github.com/Flux-15/robotics-learning-hub/fork`.
2. Clone your fork locally:
   ```bash
   git clone https://github.com/<your-username>/robotics-learning-hub.git
   cd robotics-learning-hub
   ```

### 2. Create a Feature Branch
Create a branch with a descriptive name:
```bash
git checkout -b feat/add-fast-lio2-review
# or
git checkout -b fix/ros2-qos-syntax
```

### 3. Commit with Conventional Commits
We follow standard [Conventional Commits](https://www.conventionalcommits.org/):
* `feat:` A new project, paper review, or domain module.
* `fix:` A bug fix, corrected mathematical formula, or dead link repair.
* `docs:` Documentation improvements or typos.
* `refactor:` Restructuring files or improving layout.

Example:
```bash
git commit -m "feat(projects): add 12-DoF quadruped robot dog build blueprint"
```

### 4. Push and Open a Pull Request
```bash
git push origin feat/add-fast-lio2-review
```
Go to your fork on GitHub and click **Compare & pull request**. Fill out the pull request template and submit!

---

## ⚖️ Open Source License & Intellectual Property

By submitting a pull request to **Robotics Learning Hub**, you agree that:
1. Your contributions will be released under the repository's **[MIT License](LICENSE)**.
2. You hold the necessary rights to share the submitted code, diagrams, and text, and that the content does not infringe upon any third-party copyrights or proprietary licenses.
3. Any hardware schematics or designs submitted are shared under open-source terms for community educational use.

---

## 🌟 Recognition & Community
All contributors will be recognized and celebrated in our repository documentation! If you contribute, feel free to add your name and GitHub profile link to our contributors showcase.

Thank you for building the future of robotics with us! 🤖
