# 🚗 基础底盘速度控制

## 📖 概述

底盘运动控制是自主移动机器人最基础且核心的功能。本节教程将基于 `wsc_robotic` SDK，详细介绍如何通过 Python 程序向底盘发送速度指令（`set_cmdvel`），实现对机器人在二维平面内的运动控制。我们将深入解析线速度与角速度的物理意义，并扩充展示机器人在 X轴（前后）、Y轴（左右平移，适用于全向轮底盘）以及 Z轴（原地旋转）的运行示例，帮助您彻底掌握底盘的基础运动控制。

## ✅ 前置条件

在开始本节教程前，请确保您已完成以下准备：

- **🛠️ 硬件准备**：移动机器人底盘已组装完毕，主控与底盘驱动板通信正常，电源供电稳定。
- **💻 软件环境**：已安装 Python 3.x 及 `wsc_robotic` 功能包。
- **📚 基础知识**：了解机器人坐标系约定（通常采用 ROS 标准：X轴向前，Y轴向左，Z轴向上）。

## 📌 核心接口说明

底盘控制的核心在于 `set_cmdvel` 指令，它遵循标准的运动学速度分解模型。

| 方法名 | 参数说明 | 描述 |
| :--- | :--- | :--- |
| `Chassis(robot)` | `robot`: 主控实例 | 实例化底盘控制对象，内部会订阅底盘状态话题。 |
| `set_cmdvel(v_x, v_y, w_z)` | `v_x`: X轴线速度<br>`v_y`: Y轴线速度<br>`w_z`: Z轴角速度 | 下发底盘速度指令。单位通常为 $m/s$ 和 $rad/s$。 |
| `destory()` | 无 | 销毁底盘实例，内部取消订阅，释放网络带宽与计算资源。 |

### 🧭 运动学坐标系解析

- **X轴线速度 (`v_x`)**：正值代表前进 🚀，负值代表后退 🔙。
- **Y轴线速度 (`v_y`)**：正值代表向左平移 ⬅️，负值代表向右平移 ➡️（*注：仅限麦克纳姆轮或全向轮底盘，普通两轮 差速底盘此参数无效*）。
- **Z轴角速度 (`w_z`)**：正值代表逆时针旋转 🔄，负值代表顺时针旋转 🔁。

---

## 💻 代码实现：全轴向运动示例

以下示例展示了机器人依次执行 前进、左平移、原地旋转 的组合运动，最后安全停止。

```python
import time
from wsc_robotic.robotics import *
from wsc_robotic.chassis.chassis import *

def main():
    # 1. 初始化机器人主控实例
    robot = Robotics()

    # 2. 实例化底盘控制对象
    chassis = Chassis(robot)

    # ==================== X轴：前后运动 ====================
    # 给定线速度 x=0.1m/s (向前直行)
    print(">> 机器人开始向前直行... ⬆️")
    chassis.set_cmdvel(0.1, 0, 0)
    time.sleep(3)

    # 给定线速度 x=-0.1m/s (向后倒车)
    print(">> 机器人开始向后倒车... ⬇️")
    chassis.set_cmdvel(-0.1, 0, 0)
    time.sleep(3)

    # ==================== Y轴：左右平移 (需全向底盘) ====================
    # 给定线速度 y=0.1m/s (向左平移)
    print(">> 机器人开始向左平移... ⬅️")
    chassis.set_cmdvel(0, 0.1, 0)
    time.sleep(3)

    # 给定线速度 y=-0.1m/s (向右平移)
    print(">> 机器人开始向右平移... ➡️")
    chassis.set_cmdvel(0, -0.1, 0)
    time.sleep(3)

    # ==================== Z轴：原地旋转 ====================
    # 给定角速度 z=0.5rad/s (逆时针旋转)
    print(">> 机器人开始逆时针旋转... 🔄")
    chassis.set_cmdvel(0, 0, 0.5)
    time.sleep(3)

    # ==================== 复合运动：弧线行驶 ====================
    # 给定线速度 x=0.1m/s, 角速度 z=0.3rad/s (边走边转，画弧线)
    print(">> 机器人开始弧线行驶... 🌪️")
    chassis.set_cmdvel(0.1, 0, 0.3)
    time.sleep(3)

    # ==================== 停止与资源释放 ====================
    # 3秒后停止运动 (速度归零)
    print(">> 机器人停止运动... 🛑")
    chassis.set_cmdvel(0, 0, 0)

    # 销毁底盘实例 (内部取消订阅, 降低性能及网络带宽占用)
    chassis.destory()

    # 关闭机器人
    robot.stop()
    print(">> 程序执行完毕！ ✅")

if __name__ == '__main__':
    main()
```

## 👀 运行与验证

1. **启动程序**
   将代码保存在项目`/wsc_hbrobotv2_python/user`目录中，命名为 `basic_velocity_control.py`，然后执行程序

2. **预期现象**

    - 机器人先以 0.1m/s 的速度向前直行 3 秒，随后向后倒车 3 秒。
    - 接着尝试左右平移（*若为差速底盘，此步原地不动*），随后逆时针旋转 3 秒。
    - 最后执行弧线行驶 3 秒，随后稳稳停住。

## 💡 进阶扩展与常见问题

???+ tip "为什么要调用 `destory()` 销毁实例？"
    在 ROS 或类似的分布式机器人通信架构中，实例化 `Chassis` 本质上是订阅了底盘的实时状态话题（如里程计、IMU数据等）。如果您在后续逻辑中不再需要读取底盘状态，务必调用 `destory()` 释放订阅。这能显著降低内网通信带宽占用和主控 CPU 负载 📉，尤其在多模块协同工作时非常重要。

???+ warning "常见问题：发送了 Y轴速度 (`v_y`)，但底盘原地不动"
    - **原因分析** 🧐：这通常是因为您的机器人是**两轮差速底盘**（如普通的扫地机或巡检车），其运动学模型天然不支持侧向平移。只有装配了麦克纳姆轮或全向轮的**全向移动底盘**，才能响应 `v_y` 指令实现平移。
    - **替代方案** 🛠️：差速底盘如需横向移动，只能通过组合“旋转+直行”的走位来近似替代。

???+ warning "常见问题：指令下发后底盘运动方向与预期相反"
    - **排查1** 🔄：检查电机接线顺序。若左右轮电机线接反，会导致左右轮旋转逻辑反转，造成严重的控制混乱。
    - **排查2** 📐：确认坐标系方向。部分底盘驱动板底层的坐标系定义可能与上层 SDK 不一致，需在底层驱动或参数配置中修正 `axis_direction`。


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