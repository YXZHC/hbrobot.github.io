# ROS 2 概述

> 本教程系列的第 00 篇，旨在帮助读者建立对 ROS 2 的宏观认知，为后续深入学习打下基础。

---

## 1. 什么是 ROS 2？

**ROS 2（Robot Operating System 2）** 是专为机器人系统设计的**分布式中间件框架**。它通过标准化接口与通信协议，解决复杂机器人系统中多节点协同、实时性保障、跨平台兼容等关键问题。

需要明确的是，ROS 2 **并非传统意义上的操作系统**，而是一个**机器人软件中间件层**，提供硬件抽象、设备控制、消息传递、包管理等功能。其设计哲学是 **“分层解耦”** ，允许开发者基于标准化接口自由组合功能模块，同时支持异构计算环境（如 x86、ARM、GPU）的协同工作。

从技术视角看，ROS 2 本质上是一个**分布式进程通信框架**，通过发布-订阅（Pub/Sub）模式实现节点间数据交换，结合服务调用（Service）与动作接口（Action）支持复杂交互逻辑。

---

## 2. 为什么需要 ROS 2？——从 ROS 1 到 ROS 2 的演进

### 2.1 ROS 1 的局限性

ROS 1 自 2007 年发布以来，已成为机器人领域的事实标准，但其架构缺陷在工业级场景中逐渐暴露：

| 问题 | 说明 |
|------|------|
| **单点故障风险** | ROS Master 作为中央节点，一旦崩溃会导致整个系统瘫痪 |
| **实时性不足** | 基于 TCP 的通信协议难以满足低延迟要求（如机械臂控制需毫秒级响应） |
| **跨平台限制** | 主要支持 Linux 系统，对 Windows/macOS/嵌入式设备的兼容性差 |
| **安全性缺失** | 缺乏身份认证、数据加密等基础安全机制 |

此外，ROS 1 最初是为 PR2 家庭服务机器人设计的——该机器人搭载工作站级计算平台、主要依赖有线通信、成本高昂且仅用于学术研究。随着 ROS 普及到嵌入式系统、自动驾驶汽车、航天机器人等场景，原有架构已无法满足需求。

### 2.2 ROS 2 的核心改进

为解决上述问题，ROS 2 于 2017 年发布，其核心改进包括：

- **去中心化网络**：通过 DDS 实现节点自动发现与故障恢复，彻底抛弃 ROS Master
- **QoS 策略**：支持消息传输的可靠性、截止时间等参数配置
- **跨平台支持**：原生兼容 Linux、Windows、macOS 及 RTOS
- **安全扩展**：集成 DDS-Security 标准，支持 TLS/DTLS 加密
- **实时性保障**：满足机器人运动控制等场景的实时性需求
- **多机器人支持**：为多机器人系统的通信与协作提供标准方法和机制

> **ROS 2 是一个全新的机器人操作系统**，在借鉴 ROS 1 成功经验的基础上，对系统架构和软件代码全部进行了重新设计和实现。

---

## 3. ROS 2 核心架构

ROS 2 采用**分层架构设计**，主要分为以下几个层次：

### 3.1 通信中间件层（Middleware Layer）

ROS 2 构建在 **DDS/RTPS** 之上作为其中间件（RMW），提供发现、序列化和传输功能。

**DDS（Data Distribution Service）** 的核心优势包括：

- **动态发现**：节点启动后自动注册服务，无需中央目录
- **发布/订阅模型**：支持多对多通信，降低耦合度
- **QoS 策略**：可配置消息可靠性（Best Effort/Reliable）、历史深度等参数

DDS 是 OMG 的行业标准，有多种实现可供选择：

| DDS 实现 | 说明 |
|----------|------|
| **Fast DDS** | eProsima 开发，ROS 2 默认实现 |
| **Cyclone DDS** | Eclipse 项目，轻量级实现 |
| **Connext** | RTI 公司商业实现 |
| **OpenSplice** | ADLINK 实现 |

为了兼容多种 DDS 实现，ROS 2 设计了 **RMW（ROS Middleware）层**作为统一接口标准，确保上层编程使用的函数接口一致。

### 3.2 计算图层（Computational Graph）

计算图是 ROS 2 系统的核心，指的是 ROS 系统中节点的网络以及它们之间的通信连接。核心概念包括：

- **节点（Node）** ：最小执行单元，每个节点独立管理生命周期
- **话题（Topic）** ：异步通信通道，适用于高频传感器数据流
- **服务（Service）** ：同步请求-响应模式，用于低频控制指令
- **动作（Action）** ：支持取消、反馈的长周期任务

### 3.3 工具链层

ROS 2 提供了丰富的开发与调试工具：

- **命令行工具**：`ros2 topic`、`ros2 node`、`ros2 service` 等命令用于系统调试
- **可视化工具**：RViz2 支持 3D 场景渲染，RQt 提供插件化界面开发
- **日志系统**：集成 rcl_logging 模块，支持多级别日志输出

---

## 4. ROS 2 核心概念详解

### 4.1 节点（Node）

节点是 ROS 图中的**参与单元**，使用客户端库（client library）与其他节点通信。每个节点应该只做**一件逻辑上独立的事情**。

节点可以：
- **发布（Publish）** 到话题，向其他节点传递数据
- **订阅（Subscribe）** 到话题，从其他节点获取数据
- **提供服务（Service）** 或**使用服务**
- **提供动作（Action）** 或**使用动作**

### 4.2 话题（Topic）

话题是 ROS 2 中**异步通信**的核心机制，充当节点间交换信息的**总线**。

- **发布/订阅模型**：一个节点发布消息到话题，任意数量的节点可以订阅该话题接收消息
- **强类型**：每个话题都有定义好的消息类型
- **匿名通信**：发布者和订阅者互不知晓对方的存在

### 4.3 服务（Service）

服务采用**客户端-服务器模型**，实现**同步请求/响应**通信：

- 客户端节点发送请求，等待服务器节点响应
- 适用于**低频控制指令**（如启动/停止机器人、设置参数）

### 4.4 动作（Action）

动作是 ROS 2 中处理**长周期任务**的通信机制：

- 动作客户端向动作服务器发送**目标（Goal）**
- 服务器确认目标后，返回**反馈（Feedback）** 流和**结果（Result）**
- 支持**任务取消**功能
- 适用于**路径规划、导航**等需要持续反馈的场景

### 4.5 参数（Parameter）

参数是节点的**配置值**，可在运行时动态读取和修改。每个节点可以维护自己的参数集，用于控制节点行为。

### 4.6 生命周期管理（Lifecycle Management）

ROS 2 节点通过**生命周期服务（Lifecycle Service）** 实现状态控制，支持：

- **未配置（Unconfigured）** → **配置（Configuring）** → **激活（Active）** → **清理（Cleaning Up）** → **关闭（Shutting Down）**

这种设计使得节点的启动和关闭更加可控，适用于需要精细状态管理的工业级应用。

---

## 5. ROS 2 发行版与系统要求

### 5.1 主流发行版

ROS 2 采用**发行版（Distro）** 制度管理版本，每个发行版与特定的 Ubuntu 版本绑定：

| ROS 2 发行版 | 支持的 Ubuntu 版本 | 支持期限 |
|-------------|-------------------|---------|
| **Humble Hawksbill** | Ubuntu 22.04 LTS | 2022.05 – 2027.05 |
| **Jazzy Jalisco** | Ubuntu 24.04 LTS | 最新 LTS 版本 |

> **重要提示**：不同 ROS 2 发行版绑定特定的 Ubuntu 版本，**不能随意搭配**。

### 5.2 系统要求

- **操作系统**：Ubuntu 22.04 LTS（推荐）或 Ubuntu 24.04 LTS
- **也支持**：Windows 10/11（需 WSL2）、macOS 及嵌入式 RTOS
- **依赖组件**：编译工具链（gcc/g++、cmake）、Git、Python3 环境

---

## 6. ROS 2 工具链与生态

### 6.1 命令行工具（`ros2`）

`ros2` 是用户管理、内省和交互 ROS 系统的主要工具，支持多个子命令：

```bash
ros2 node list          # 列出所有运行中的节点
ros2 topic list         # 列出所有话题
ros2 topic echo /topic  # 打印话题消息
ros2 service list       # 列出所有服务
ros2 action list        # 列出所有动作
ros2 param list         # 列出节点参数
```

### 6.2 可视化工具

- **RViz2**：3D 可视化工具，用于机器人状态显示、传感器数据可视化
- **RQt**：基于 Qt 的插件化 GUI 框架，提供 rqt_graph（节点关系图）、rqt_console（日志控制台）等插件

### 6.3 开发库

ROS 2 支持多种编程语言：

- **rclcpp**：C++ 客户端库
- **rclpy**：Python 客户端库
- **rcljava**：Java 客户端库（社区支持）

### 6.4 文档生态

ROS 2 官方文档分为几个主要部分：

- **教程（Tutorials）** ：循序渐进的步骤说明，适合初学者按顺序学习
- **概念（Concepts）** ：关键方面的高阶背景信息
- **操作指南（How-to Guides）** ：针对特定问题的模块化答案，适合有基础的开发者

---

## 7. ROS 2 典型应用场景

ROS 2 已广泛应用于多个领域：

| 应用领域 | 说明 |
|----------|------|
| **机器人控制** | 机械臂、移动机器人、人形机器人的运动控制与状态管理 |
| **智能驾驶** | 自动驾驶汽车的感知、决策、规划与控制 |
| **工业自动化** | 工业机械臂、AGV、物流机器人的系统集成 |
| **服务机器人** | 配送机器人、清洁机器人、接待机器人 |
| **多机器人协同** | 多台机器人协同完成共同目标 |
| **航天与特种机器人** | 对可靠性和实时性有极高要求的场景 |

---
## 8. 参考资源

- [ROS 2 官方文档](https://docs.ros.org/en/humble/index.html)
- [ROS 2 GitHub 仓库](https://github.com/ros2/ros2)
- [DDS 基金会](https://www.dds-foundation.org/)


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

---