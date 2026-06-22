![](images/61d7f15b093ae.png)
# <div style="text-align:center">🤖 世界技能大赛自主移动机器人使用说明文档</div>

## 📖 文档概述

本文档为专用**自主移动机器人**配套的完整操作、硬件、软件及导航开发手册，面向高校课程实验、课堂实操演示、课程设计与课外科创实践人员编写。

文档基于 **MkDocs** 构建，支持在线网页浏览，源码托管于 **GitHub**，可随时迭代更新、协作修订。

---

## 📊 文档建设进度看板

> 我们正日夜兼程地完善这份文档，以下是各板块的最新状态。感谢您的耐心等待！🚀  

| 状态图标 | 含义 | 说明 |
| :---: | :--- | :--- |
| ✅ | **稳定完善** | 内容已校验，功能正常，推荐阅读。 |
| 📝 | **持续优化** | 核心内容已即将就绪，细节正在微调中。 |
| 🚧 | **火热施工中** | 正在紧张筹备，即将上线，敬请期待！ |

---
> *点击章节标题可直接跳转至对应详细文档。*

### 🚀 [01-快速上手](01-quick-start/01-device-check-and-boot.zh.md)

开箱验收、充电规范、开关机流程、急停操作，零基础快速启动机器人。

- 📝 [设备检查与启动](01-quick-start/01-device-check-and-boot.zh.md)
- ✅ [连接到机器人](01-quick-start/02-connect-to-robot.zh.md) 
- ✅ [基本功能检测](01-quick-start/03-basic-function-test.zh.md) 
- ✅ [本地开发调试](01-quick-start/04-local-dev-debug.zh.md)

---

### 💻 [02-开发环境配置](02-software/00-preparation.zh.md)

ROS2 环境部署、软件安装、固件升级、调试工具使用教程。

- ✅ [环境准备](02-software/00-preparation.zh.md)   
- ✅ [安装 Ubuntu](02-software/01-install-ubuntu.zh.md) 
- ✅ [安装 ROS 2](02-software/02-install-ros2.zh.md) 
- ✅ [安装 VS Code](02-software/03-install-vscode.zh.md) 
- ✅ [Python 依赖](02-software/04-install-python-deps.zh.md) 
- ✅ [NVIDIA 显卡驱动](02-software/05-install-nvidia-driver.zh.md) 
- ✅ [Docker 与 NVIDIA](02-software/06-install-docker-nvidia.zh.md) 
- ✅ [安装与配置 Git](02-software/08-install-git.zh.md)

---

### 🛠️ [03-硬件与结构](03-hardware/01-robot-overview.zh.md)

整机结构解析、运动底盘、OMS执行机构、传感器外设（激光雷达、视觉相机、IMU等）。

- 🚧 [整机结构概述](03-hardware/01-robot-overview.zh.md)
- 🚧 [运动底盘](03-hardware/02-motion-chassis.zh.md)
- 🚧 [OMS 执行机构](03-hardware/03-oms-actuator.zh.md)
- 🚧 [传感器外设](03-hardware/04-sensor-peripherals.zh.md)

---

### ⚙️ [04-参数配置与标定](04-device-params/01-vmxpi-titan-params.zh.md)

主控参数、底盘运动学、电机控制参数、激光雷达驱动、IMU标定及无线Wi-Fi配置。

- ✅ [VMXPi & Titan 参数](04-device-params/01-vmxpi-titan-params.zh.md)
- ✅ [底盘运动学参数](04-device-params/02-chassis-kinematics-params.zh.md)
- ✅ [OMS 电机参数](04-device-params/03-oms-motor-params.zh.md)
- 📝 [激光雷达参数](04-device-params/04-lidar-driver-params.zh.md)
- 🚧 [IMU 水平标定](04-device-params/05-imu-calibration.zh.md)
- ✅ [无线网络参数](04-device-params/06-wifi-params.zh.md)

---

### 🎮 [05-基础教学](05-basics/ros2/00-ros2-overview.zh.md)

ROS2 和 Python 基础。

#### **ROS 2 基础**
- 🚧 [ROS 2 概述](05-basics/ros2/00-ros2-overview.zh.md)
- 🚧 [话题 (Topic)](05-basics/ros2/01-topic.zh.md)
- 🚧 [服务 (Service)](05-basics/ros2/02-service.zh.md)
- 🚧 [动作 (Action)](05-basics/ros2/03-action.zh.md)
- 🚧 [启动文件 (Launch)](05-basics/ros2/04-launch.zh.md)
- 🚧 [参数 (Parameter)](05-basics/ros2/05-parameter.zh.md)

#### **Python 基础**
- 🚧 [Python 基础](05-basics/python/01-python.zh.md)

---

### 🤖 [06-控制与应用](06-applications/01-actuator-control/01-user-servo-params.zh.md)

执行机构控制、底盘运动控制、传感器与视觉应用及综合任务实战。

#### **执行机构控制**
- ✅ [用户伺服舵机参数](06-applications/01-actuator-control/01-user-servo-params.zh.md)
- 📝 [伺服舵机控制](06-applications/01-actuator-control/02-servo-control.zh.md)
- 📝 [用户举升与旋转参数](06-applications/01-actuator-control/03-user-lift-rotate-params.zh.md)
- ✅ [举升与旋转控制](06-applications/01-actuator-control/04-lift-rotate-control.zh.md)

#### **底盘运动控制**
- ✅ [基础底盘速度控制](06-applications/02-chassis-control/01-basic-velocity-control.zh.md)
- ✅ [基础直行与旋转控制](06-applications/02-chassis-control/02-straight-rotate-control.zh.md)
- ✅ [连续轨迹运动控制](06-applications/02-chassis-control/03-trajectory-control.zh.md)

#### **传感器应用**
- 🚧 [测距传感器合集](06-applications/03-sensor-application/01-ranging-sensors.zh.md)
- 🚧 [IMU 数据读取与应用](06-applications/03-sensor-application/02-imu-data-usage.zh.md)
- 🚧 [激光雷达数据处理](06-applications/03-sensor-application/03-lidar-data-processing.zh.md)

#### **视觉应用**
- 🚧 [图像采集](06-applications/04-vision-application/01-camera-capture.zh.md)
- 🚧 [目标识别基础](06-applications/04-vision-application/02-object-detection-basics.zh.md)
- 🚧 [深度学习目标检测](06-applications/04-vision-application/03-deep-learning-detection.zh.md)

#### **综合任务实战**
- 🚧 [视觉追踪与抓取](06-applications/05-integration-tasks/01-visual-tracking-grasp.zh.md)
- 🚧 [多模块联动示例](06-applications/05-integration-tasks/02-multi-module-task.zh.md)

---

### 🚀 [07-进阶内容](07-advanced/01-udev-rule.zh.md)

UDEV规则配置、激光雷达SDK与驱动安装、激光定位与自主导航。

- 🚧 [UDEV 规则配置](07-advanced/01-udev-rule.zh.md)
- 🚧 [激光雷达 SDK 安装](07-advanced/02-lidar-sdk-install.zh.md)
- 🚧 [激光雷达 ROS2 驱动](07-advanced/03-lidar-ros2-driver.zh.md)
- 🚧 [激光定位与导航](07-advanced/04-lidar-localization-navigation.zh.md)

---

## 📚 使用建议

- **初次使用** 👶：请严格按照「[01-快速上手](01-quick-start/01-device-check-and-boot.zh.md)」章节顺序操作，切勿跳过安全检查步骤。
- **课程实验** 🧪：可按需截取对应章节打印或导出 PDF 作为实验指导书。
- **协作修订** 🤝：文档支持多人协作修改，可 Fork 仓库提交修订建议。
- **快速导航** 🔍：使用文档顶部的搜索功能（MkDocs 支持）或左侧目录树跳转。

---

## 🔧 版本信息

| 项目 | 内容 |
| :--- | :--- |
| **文档版本** | V1.0 |
| **适用机型** | 自主移动机器人 |
| **更新日期** | 2026.06 |
| **维护方** | 汇博机器人世界技能大赛移动机器人项目组 |

---

> 📌 **提示**：本文档中所有相对链接均基于 MkDocs 站点根目录，若在本地查看请确保文件路径与目录结构一致。

---
## 👥 贡献者
本项目离不开每一位提交 PR、提 Issue、优化文档的开发者，由衷致谢！
<div style="display: flex; flex-wrap: wrap; gap: 30px; margin-top: 20px; margin-bottom: 20px;">
    <div style="text-align: center;">
        <a href="https://github.com/yxzhc">
            <img src="https://avatars.githubusercontent.com/u/80094007?size=120" style="border-radius: 50%; width: 80px; height: 80px; object-fit: cover;" alt="yxzhc">
        </a>
        <div style="margin-top: 8px; font-weight: 600;">
            <a href="https://github.com/yxzhc" style="text-decoration: none;">YXZHC</a>
        </div>
    </div>
    <div style="text-align: center;">
        <a href="https://github.com/hbrobot">
            <img src="https://avatars.githubusercontent.com/u/292023923?v=4?size=120" style="border-radius: 50%; width: 80px; height: 80px; object-fit: cover;" alt="HBRobot">
        </a>
        <div style="margin-top: 8px; font-weight: 600;">
            <a href="https://github.com/hbrobot" style="text-decoration: none;">HBRobot</a>
        </div>
    </div>
</div>
---
🤝 **欢迎参与共建：**

[:fontawesome-brands-github: 提交 Issue](https://github.com/hbrobot/hbrobot.github.io/issues/new/choose){: .md-button }
[:octicons-git-pull-request-24: 提交 PR](https://github.com/hbrobot/hbrobot.github.io/compare){: .md-button .md-button--primary }