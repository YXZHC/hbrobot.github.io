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

### 🚀 [01-快速上手](01-quick-start/01-设备检查与启动.md)

开箱验收、充电规范、开关机流程、急停操作，零基础快速启动机器人。

- 📝 [设备检查与启动](01-quick-start/01-设备检查与启动.md)
- ✅ [连接到机器人](01-quick-start/02-连接到机器人.md) 
- ✅ [基本功能检测](01-quick-start/03-基本功能检测.md) （SSH、急停、电池电压、按键指示灯）
- 🚧 [本地开发调试](01-quick-start/04-本地开发调试.md)

---

### 🛠️ [02-硬件介绍](02-hardware/01-整机结构概述.md)

> ⚠️ 该板块内容正在准备中，敬请期待后续更新。

整机结构拆解、运动底盘、OMS执行机构、传感器外设（激光雷达、视觉相机、IMU等）。

- 🚧 [整机结构概述](02-hardware/01-整机结构概述.md)
- 🚧 [运动底盘](02-hardware/02-运动底盘.md)
- 🚧 [OMS执行机构](02-hardware/03-OMS执行机构.md)
- 🚧 [传感器外设](02-hardware/04-传感器外设.md)

---

### 💻 [03-软件与开发环境](03-software/00-preparation.zh.md)

车载系统架构、ROS/ROS2 环境部署、软件安装、固件升级、调试工具使用教程。

- ✅ [准备工作](03-software/00-preparation.zh.md)   
- ✅ [安装 Ubuntu](03-software/01-install-ubuntu.zh.md) 
- ✅ [安装 ROS2](03-software/02-install-ros2.zh.md) 
- ✅ [安装 VS Code](03-software/03-install-vscode.zh.md) 
- ✅ [安装 Python 依赖](03-software/04-install-python-deps.zh.md) 
- ✅ [安装 NVIDIA 驱动](03-software/05-install-nvidia-driver.zh.md) 
- ✅ [安装 Docker + NVIDIA Container](03-software/06-install-docker-nvidia.zh.md) 
- ✅ [安装 Git](03-software/08-install-git.zh.md)

---

### 🎮 [04-基础操控](04-basics/ROS2/00-ROS2.zh.md)

> ⚠️ 该板块内容正在准备中，敬请期待后续更新。

键盘、手柄手动遥控，程序启停，设备指示灯状态解读，以及 ROS2 和 Python 基础。

#### **ROS2 基础**

- 🚧 [ROS2 快速了解](04-basics/ROS2/00-ROS2.zh.md)
- 🚧 [Topic](04-basics/ROS2/01-ROS2-Topic.zh.md)
- 🚧 [Service](04-basics/ROS2/02-ROS2-Service.zh.md)
- 🚧 [Action](04-basics/ROS2/03-ROS2-Action.zh.md)
- 🚧 [Launch](04-basics/ROS2/04-ROS2-Launch.zh.md)
- 🚧 [Parameter](04-basics/ROS2/05-ROS2-Parameter.zh.md)


#### **Python 基础**

- 🚧 [Python 快速入门](04-basics/Python/1-python.md)

---

### ⚙️ [05-设备参数](05-device-params/01-VMXPi_Titan参数.md)

> ⚠️ 该板块内容正在准备中，敬请期待后续更新。

主控参数、底盘运动学、电机控制参数、激光雷达驱动、IMU标定及无线Wi-Fi配置。

- 📝 [VMXPi_Titan 参数](05-device-params/01-VMXPi_Titan参数.md)
- 📝 [底盘运动学参数](05-device-params/02-底盘运动学参数.md)
- 📝 [OMS电机控制参数](05-device-params/03-OMS电机控制参数.md)
- 📝 [Lidar驱动参数](05-device-params/04-Lidar驱动参数.md)
- 🚧 [IMU水平标定](05-device-params/05-IMU水平标定.md)
- 📝 [无线Wi-Fi参数](05-device-params/06-无线Wi-Fi参数.md)

---

### 🗺️ 06-自主导航功能

> ⚠️ 该板块内容正在准备中，敬请期待后续更新。

SLAM 建图、地图存储加载、路径规划、动态避障、导航点位设定实操步骤。

---

### 🔒 07-安全规范

> ⚠️ 该板块内容正在准备中，敬请期待后续更新。

场地使用要求、安全操作距离、故障紧急处置方案，规避设备与人身风险。

> 📌 详细内容请参考各操作章节中的「安全注意事项」小节。

---

### ❓ 08-附录 FAQ

> ⚠️ 该板块内容正在准备中，敬请期待后续更新。

通信连接失败、导航漂移、传感器无数据、程序闪退等高频问题排查方案。

> 📌 FAQ 内容会随用户反馈持续更新，建议查看在线最新版本。

---

## 📚 使用建议

- **初次使用** 👶：请严格按照「[01-快速上手](01-quick-start/01-设备检查与启动.md)」章节顺序操作，切勿跳过安全检查步骤。
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
