# 🚀 基础直行与旋转控制

## 📖 概述

在上一节中，我们掌握了通过 `set_cmdvel` 下发持续速度指令的基础控制。然而，在实际应用中，我们更常需要的是让机器人**精准地前进指定距离**或**旋转指定角度**。本节教程将引入 `BaseMotionKit`（基础运动套件），它基于底盘里程计提供了**相对位置**的闭环控制能力，支持带梯形速度规划的直线与旋转运动。无论是前进 1 米还是旋转 90 度，都是基于机器人**当前时刻的位置和朝向**作为基准进行增量运动的 🎯。

> 💡 **核心概念**：直线与旋转控制属于**相对位置控制**（增量运动），不依赖于全局世界坐标原点；而后续的《连续轨迹运动控制》则是基于里程计的**世界坐标系**（绝对坐标）来运行的，请务必区分两者差异。

## ✅ 前置条件

在开始本节教程前，请确保您已完成以下准备：

- **🛠️ 硬件准备**：底盘编码器及 IMU 工作正常，能够提供稳定的里程计数据。
- **💻 软件环境**：已安装 Python 3.x 及 `wsc_robotic` 功能包。
- **📚 前置章节**：已学习《基础底盘速度控制》，理解了线速度与角速度的概念。

## 📌 核心接口说明

`BaseMotionKit` 封装了底层的 PID 控制与速度规划逻辑，提供了基于当前姿态的“增量驱动型”API。

### 运动参数配置

| 方法名 | 参数说明 | 描述 |
| :--- | :--- | :--- |
| `set_line_driver_param(distance, vel, acc, decel)` | `distance`: 相对距离<br>`vel`: 速度<br>`acc`: 加速度<br>`decel`: 减速度 | 配置直线运动参数。`distance` 为负数时表示后退。单位：米, 米/秒, 米/秒²。 |
| `set_rotate_driver_param(angle, vel, acc, decel)` | `angle`: 相对角度<br>`vel`: 角速度<br>`acc`: 角加速度<br>`decel`: 角减速度 | 配置旋转运动参数。`angle` 为负数时表示顺时针旋转。单位：度, 度/秒, 度/秒²。 |

### 运动控制与状态反馈

| 方法名 | 描述 |
| :--- | :--- |
| `start()` | 开始执行配置好的运动指令。 |
| `wait_finished()` | 阻塞当前线程，等待相对运动执行完毕。 |
| `is_moving()` | 判断当前是否正在运动中，返回布尔值（常用于非阻塞监控）。 |
| `get_feedback()` | 获取实时运动反馈数据（包含当前速度、位置等）。 |
| `cancel()` | 取消当前正在进行的运动（紧急停止）。 |

---

## 💻 代码实现：精准直线与旋转实战

以下示例展示了直线前进后退、相对旋转，并结合实时状态监控的完整流程。

```python
import time
from wsc_robotic.robotics import *
from wsc_robotic.chassis.chassis import *
from wsc_robotic.navigation.base_motion_kit import *

def main():
    # 1. 初始化机器人主控实例
    robot = Robotics()

    # 2. 实例化基础运动套件
    base_motion_kit = BaseMotionKit(robot)

    # (可选) 获取当前电池电压 📊
    with Chassis(robot) as chassis:
        vol = chassis.get_battery_vol()
        print('当前电池电压:', vol, 'v')
    time.sleep(0.6)

    # ==================== 直线运动模式 (相对位置) ====================
    print(">> 基础运动套件 - 相对当前位置前进 1.0 米 🚗")
    # 相对距离 1.0m, 速度 0.5m/s, 加速度 0.6m/s², 减速度 0.4m/s²
    base_motion_kit.set_line_driver_param(distance=1.0, vel=0.5, acc=0.6, decel=0.4)
    base_motion_kit.start()
    base_motion_kit.wait_finished() # 阻塞等待到达相对 1.0m 位置

    print(">> 基础运动套件 - 相对当前位置后退 0.5 米 🔙")
    # 相对距离 -0.5m (负数代表后退)
    base_motion_kit.set_line_driver_param(distance=-0.5, vel=0.3, acc=0.6, decel=0.35)
    base_motion_kit.start()
    base_motion_kit.wait_finished()

    # ==================== 旋转运动模式 (相对位置) ====================
    print(">> 基础运动套件 - 相对当前朝向逆时针旋转 90 度 🔄")
    # 相对角度 90.0deg, 速度 30.0deg/s, 加速度 50.0deg/s², 减速度 50.0deg/s²
    base_motion_kit.set_rotate_driver_param(angle=90.0, vel=30.0, acc=50.0, decel=50.0)
    base_motion_kit.start()
    base_motion_kit.wait_finished()

    # ==================== 带实时监控的旋转运动 🛠️ ====================
    print(">> 基础运动套件 - 相对当前朝向顺时针旋转 90 度 (带实时监控) 🔁")
    # 相对角度 -90.0deg (负数代表顺时针)
    base_motion_kit.set_rotate_driver_param(angle=-90.0, vel=30.0, acc=50.0, decel=50.0)
    base_motion_kit.start()

    # 使用 is_moving() 进行非阻塞监控，并获取实时反馈
    while base_motion_kit.is_moving():
        feedback = base_motion_kit.get_feedback()
        print(f"   实时状态反馈: {feedback}")
        time.sleep(0.05) # 建议监控周期不要太短，避免占用过多CPU
        
    print(">> 旋转运动完成！ ✅")

    # ==================== 超时取消机制 (演示用法) ====================
    # 如果担心运动卡死，可以使用带超时的循环，超时后调用 cancel()
    # base_motion_kit.start()
    # for i in range(5):
    #     if not base_motion_kit.is_moving():
    #         break
    #     time.sleep(1)
    # else:
    #     base_motion_kit.cancel()  # 5秒未完成则强制取消运动 🛑
    #     print("运动超时，已强制取消！")

    # 关闭机器人连接
    robot.stop()

if __name__ == '__main__':
    main()
```

## 👀 运行与验证

1. **启动程序**
   将代码保存在项目`/wsc_hbrobotv2_python/user`目录中，命名为 `straight_rotate_control.py`，然后执行程序

2. **预期现象**：

    - 终端首先打印当前电池电压。
    - 机器人先平滑加速前进 1 米，然后后退 0.5 米（此时距离起点净位移 0.5 米）。
    - 接着机器人相对当前朝向逆时针精准旋转 90°。
    - 最后顺时针旋转 90° 回到原朝向，此过程中终端高速刷新实时状态反馈数据。

## 💡 进阶扩展与常见问题

???+ tip "相对位置与绝对坐标的区别 🌍"
    本节介绍的直线与旋转控制是**相对增量运动**：无论机器人在世界坐标的何处，执行 `distance=1.0` 都只是向前走 1 米。而在后续的《连续轨迹运动控制》中，我们将使用**世界坐标系**，那时发送的目标将是地图上的绝对坐标（如 `x=2.0, y=3.0`），底盘会根据当前里程计位置自动规划路径前往。

???+ warning "常见问题：相对运动前需要调用 `init_odometry` 清零吗？"
    - **不是强制必须的** 📏。因为直线与旋转控制是基于当前时刻的相对增量，即使里程计当前坐标是 `(5.0, 5.0, 45°)`，执行前进 1 米后，里程计坐标会变为 `(5.7, 5.7, 45°)`，控制逻辑完全正确。
    - **但是推荐在任务起点清零** 🧹：保持里程计状态机的干净是个好习惯，这能确保底盘的位姿状态与实际环境对齐，为后续切换到基于世界坐标的连续轨迹控制做准备。

???+ warning "常见问题：设定的距离是 1 米，但实际总是走多或走少"
    - **原因分析** 🧐：这属于里程计漂移现象。由于地面打滑、轮径参数不准或 IMU 未标定，导致底盘计算出的内部位移与实际物理位移存在偏差。
    - **解决方案 🛠️**：
      1. 检查底盘配置中的轮径参数是否与实际轮胎匹配（磨损会导致轮径变小）。
      2. 确保底层驱动正确融合了 IMU 数据进行航迹推算。
      3. 避免在易打滑的地面（如光滑瓷砖上有水）进行高精度的里程运动。

???+ warning "常见问题：如何安全地中断正在执行的旋转/直行？"
    如果在运动过程中遇到障碍物或突发情况，不能仅仅依赖 `wait_finished()`。此时应该使用 `cancel()` 方法。它会立即将底盘速度目标置零并中断当前的运动规划，是保障机器人安全的重要手段 🛑。


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

!!! tip "🔗 本节导航"

    <p style="display: flex; justify-content: space-between; flex-wrap: wrap;">
        <span>⬅️ <strong>上一节</strong><br><a href="../01-basic-velocity-control.zh">基础底盘速度控制</a></span>
        <span>📑 <strong>上级目录</strong><br><a href="../../01-actuator-control/04-lift-rotate-control.zh">举升与旋转控制</a></span>
        <span>➡️ <strong>下一节</strong><br><a href="../03-trajectory-control.zh">连续轨迹运动控制</a></span>
    </p>
