# 🛤️ 连续轨迹运动控制

## 📖 概述

在前一节《基础直行与旋转控制》中，我们掌握了基于**相对位置**的增量运动。然而，当面临复杂的站点搬运或巡检任务时，我们需要机器人在**世界坐标系**下沿着连续的路径点自主移动。本节教程将深入探讨 `BaseMotionKit` 的进阶功能——**连续轨迹运动控制** 🌍。

与相对运动不同，轨迹控制依赖于里程计的世界坐标系绝对位姿（`Pose2D`）。SDK 内置了三种不同特性的轨迹驱动器，以适应不同的底盘构型与运动需求：

    1. **点到点（PTP）运动** 🎯：定位最准，先转后走。
    2. **差速跟随运动** 🏎️：边走边转，轨迹丝滑，适合差速底盘连续巡航。
    3. **全向轮连续运动** 🕹️：全自由度平移，仅限全向轮底盘，多维度预规划。

> ⚠️ **重要前提**：执行任何轨迹运动前，**必须确保里程计已初始化**（`init_odometry`），否则世界坐标原点将不可知，导致轨迹混乱！

## ✅ 前置条件

- **🛠️ 硬件准备**：底盘编码器及 IMU 工作正常，已标定轮径与轴距参数。
- **💻 软件环境**：已安装 Python 3.x 及 `wsc_robotic` 功能包。
- **📚 前置章节**：已学习《基础直行与旋转控制》，理解相对运动与世界坐标的区别。

---

## 🎯 1. 点到点运动 (PTP)

### 📌 运动特性
PTP（Point-to-Point）是**定位精度最高**的轨迹模式。它的核心逻辑是**“先旋转至下一个目标航向，然后再直线运动到目标点”**。由于不会出现折返轨迹，且直线行走阶段不进行复合解算，因此它能够达到极高的停靠精度，非常适合对位要求严苛的工位对接。

### 💻 代码实现：1*1 正方形轨迹

```python
import time
from wsc_robotic.robotics import *
from wsc_robotic.chassis.chassis import *
from wsc_robotic.navigation.base_motion_kit import *

def ptp_demo():
    robot = Robotics()
    
    # 🚨 关键：必须初始化世界坐标系原点
    with Chassis(robot) as chassis:
        chassis.init_odometry(pose=[0,0,0], mode=OdomClearMode.CLEAR_ALL)
        print('当前电池电压:', chassis.get_battery_vol(), 'v')
    time.sleep(0.6)

    base_motion_kit = BaseMotionKit(robot)

    print(">> PTP 点到点运动模式 - 1*1 正方形 🎯")
    # 设定 1*1 正方形的世界坐标路径点
    path = []
    path.append(Pose2D(x=1.0, y=0.0))  # 目标点1：直行1米
    path.append(Pose2D(x=1.0, y=1.0))  # 目标点2：左侧1米
    path.append(Pose2D(x=0.0, y=1.0))  # 目标点3：后退1米
    path.append(Pose2D(x=0.0, y=0.0))  # 目标点4：右侧1米，回到原点

    # 配置 PTP 驱动器参数
    ptp_param = PTPParam(
        linear_vel   = 0.67,        # 最大线速度 m/s
        angular_vel  = 4.1,         # 最大角速度 rad/s
        linear_acc   = 1.675,       # 直线加速度 m/s^2
        linear_decel = 0.5695,      # 直线减速度 m/s^2
        rotate_acc   = 8.2,         # 旋转加速度 rad/s^2
        rotate_decel = 3.28,        # 旋转减速度 rad/s^2
    )
    
    # 设置参数：目标路径、最终朝向0度(回到初始朝向)、禁止倒退、使用自定义参数
    base_motion_kit.set_ptp_driver_param(points=path, final_heading=0.0, back=False, param=ptp_param)

    base_motion_kit.start()
    base_motion_kit.wait_finished()              # 等待运动结束

    robot.stop()

if __name__ == '__main__':
    ptp_demo()
```
将代码保存在项目`/wsc_hbrobotv2_python/user`目录中，命名为 `ptp_demo.py`，然后执行程序

### 👀 运行预期
1. 机器人在原点原地旋转 0°（已对准），直行至 `(1.0, 0.0)` 停下。
2. 在 `(1.0, 0.0)` 原地旋转 90° 对准下一点，直行至 `(1.0, 1.0)` 停下。
3. 在 `(1.0, 1.0)` 原地旋转 90°，直行至 `(0.0, 1.0)` 停下。
4. 在 `(0.0, 1.0)` 原地旋转 90°，直行回到原点 `(0.0, 0.0)`。
5. 最终在原点再次原地旋转，调整最终朝向至 `0.0` 度。**（整条轨迹呈现棱角分明的标准正方形）** 🔲

---

## 🏎️ 2. 差速跟随运动

### 📌 运动特性
差速跟随模式专为连续轨迹巡航设计。与 PTP 的“先转后走”不同，它采用**实时跟踪算法，边走边转**，不会产生生硬的停顿折返。由于是连续的切向跟随，轨迹更加平滑流畅，但在多个路径点交汇处的定位精度略逊于 PTP。**仅限两轮差速底盘使用**。

### 💻 代码实现：1*1 正方形轨迹

```python
import time
from wsc_robotic.robotics import *
from wsc_robotic.chassis.chassis import *
from wsc_robotic.navigation.base_motion_kit import *

def diff_follow_demo():
    robot = Robotics()
    
    # 🚨 关键：必须初始化世界坐标系原点
    with Chassis(robot) as chassis:
        chassis.init_odometry(pose=[0,0,0], mode=OdomClearMode.CLEAR_ALL)
    time.sleep(0.6)

    base_motion_kit = BaseMotionKit(robot)

    print(">> 差速跟随运动模式 - 1*1 正方形 🏎️")
    # 设定 1*1 正方形的世界坐标路径点
    path = []
    path.append(Pose2D(x=1.0, y=0.0))
    path.append(Pose2D(x=1.0, y=1.0))
    path.append(Pose2D(x=0.0, y=1.0))
    path.append(Pose2D(x=0.0, y=0.0))

    # 配置差速跟随驱动器参数
    diff_follow_param = DiffFollowParam(
        linear_vel  = 0.5695,   # 最大线速度 m/s
        angular_vel = 3.28,     # 最大角速度 rad/s
        rotate_acc  = 3.28,     # 旋转加速度 rad/s^2
        rotate_decel= 2.624,    # 旋转减速度 rad/s^2
    )
    
    # 设置参数：目标路径、最终朝向0度、不倒退、使用自定义参数
    base_motion_kit.set_diff_follow_param(points=path, final_heading=0.0, back=False, param=diff_follow_param)

    base_motion_kit.start()
    base_motion_kit.wait_finished()              # 等待运动结束

    robot.stop()

if __name__ == '__main__':
    diff_follow_demo()
```
将代码保存在项目`/wsc_hbrobotv2_python/user`目录中，命名为 `diff_follow_demo.py`，然后执行程序

### 👀 运行预期
1. 机器人起步直行，接近 `(1.0, 0.0)` 时不会停下，而是开始提前打方向盘，平滑画弧切入下一段轨迹 🌊。
2. 在四个顶点处，机器人都会以圆弧过渡，边走边转，不会出现原地停顿旋转的情况。
3. 最终平滑收敛回原点，并将朝向顺滑调整至 `0.0` 度。**（整条轨迹呈现边角被圆润化的正方形）** 🏁

---

## 🕹️ 3. 全向轮连续运动

### 📌 运动特性
全向轮驱动器是专为**麦克纳姆轮或全向轮底盘**打造的多自由度预规划器。它允许机器人在 X、Y 轴上独立平移，同时进行 Z 轴旋转。轨迹由底层预规划解算，支持三轴同步控制，可以实现真正的“横行”与“斜行”，多自由度极高。但受限于全向轮的机械打滑特性，其绝对位姿精度一般。

### 💻 代码实现：1*1 正方形轨迹 (侧移展示)

```python
import time
from wsc_robotic.robotics import *
from wsc_robotic.chassis.chassis import *
from wsc_robotic.navigation.base_motion_kit import *

def omni_wheel_demo():
    robot = Robotics()
    
    # 🚨 关键：必须初始化世界坐标系原点
    with Chassis(robot) as chassis:
        chassis.init_odometry(pose=[0,0,0], mode=OdomClearMode.CLEAR_ALL)
    time.sleep(0.6)

    base_motion_kit = BaseMotionKit(robot)

    print(">> 全向轮连续运动模式 - 1*1 正方形 🕹️")
    # 全向轮支持在每个点指定独立的朝向 theta
    # 为体现全向特性，我们让机器人始终保持车头朝向 0 度（正前方）走完正方形
    path = []
    path.append(Pose2D(x=1.0, y=0.0, theta=0.0))   # 正向直行
    path.append(Pose2D(x=1.0, y=1.0, theta=0.0))   # 左侧平移 (车头依旧朝前)
    path.append(Pose2D(x=0.0, y=1.0, theta=0.0))   # 后退平移
    path.append(Pose2D(x=0.0, y=0.0, theta=0.0))   # 右侧平移回原点

    # 配置全向轮驱动器参数
    omni_wheel_param = OmniWheelParam(
        velocity_x = 0.55,      # X轴最大线速度 m/s
        velocity_y = 0.525,     # Y轴最大线速度 m/s
        velocity_w = 3.69,      # 最大角速度 rad/s
        start_tacc = 0.7,       # 起始加速度时间 s
        middle_tacc = 1.2,      # 中间加速度时间 s
        end_tacc = 1.5,         # 结束加速度时间 s
        start_tacc_w = 1.0,     # 起始角加速度时间 s
        end_tacc_w = 1.32,      # 结束角加速度时间 s
        sync_rotate = True      # 同步旋转
    )
    
    # 设置参数
    base_motion_kit.set_omni_wheel_param(points=path, param=omni_wheel_param)

    base_motion_kit.start()
    base_motion_kit.wait_finished()              # 等待运动结束

    robot.stop()

if __name__ == '__main__':
    omni_wheel_demo()
```
将代码保存在项目`/wsc_hbrobotv2_python/user`目录中，命名为 `omni_wheel_demo.py`，然后执行程序

### 👀 运行预期
1. 机器人车头朝前，正向直行至 `(1.0, 0.0)` ⬆️。
2. 随后**不改变车头朝向**，直接向左侧横向平移至 `(1.0, 1.0)` ⬅️。
3. 接着保持车头朝前，向后倒退平移至 `(0.0, 1.0)` ⬇️。
4. 最后向右侧横向平移回原点 `(0.0, 0.0)` ➡️。**（整条轨迹是标准的正方形，且机器人全程不转弯，展示了极致的侧移能力）** 🪄

---

## 💡 进阶扩展与常见问题

???+ tip "如何选择合适的轨迹驱动器？ 🤔"
    - **需要精准对接充电桩/工位** 👉 选择 **PTP**。它先转后走，到达终点时的姿态最标准。
    - **差速底盘巡线巡航，要求轨迹丝滑** 👉 选择 **差速跟随**。边走边转，如行云流水。
    - **全向底盘需要侧移、斜行入位** 👉 选择 **全向轮**。释放底盘的全部自由度，灵活走位。

???+ warning "常见问题：为什么轨迹执行一段时间后，机器人越来越“跑偏”？ 🧐"
    - 这是典型的**里程计漂移**现象。无论是 PTP 还是跟随，底层都依赖轮式里程计推算位置。当车轮打滑或地面不平时，累计误差会越来越大。
    - **解决方案** 🛠️：长距离轨迹巡航不能单纯依赖开环里程计，必须在关键节点结合激光雷达、视觉等传感器进行全局定位校正（SLAM）。

???+ warning "常见问题：差速底盘误用了全向轮驱动器，会发生什么？ 🚨"
    - 指令不会报错，但底盘运动会**完全错乱**。当底层下发 Y 轴平移速度时，普通差速车轮无法侧向滚动，只能原地卡死或摩擦拖行。请务必根据实际硬件型号选择对应的驱动器！

???+ warning "常见问题：`back=False` 参数的作用是什么？ 🔙"
    - 在 PTP 和差速跟随中，`back` 参数决定了机器人在面对“需要后退才能到达的路径”时的行为。若设为 `False`，机器人会自动先旋转车身至前进方向，再正向驶向目标；若设为 `True`，则允许机器人倒车行驶至后方目标点。


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
        <span>⬅️ <strong>上一节</strong><br><a href="../02-straight-rotate-control.zh">基础直行与旋转控制</a></span>
        <span>📑 <strong>上级目录</strong><br><a href="../01-basic-velocity-control.zh">基础底盘速度控制</a></span>
        <span>➡️ <strong>下一节</strong><br><a href="../../03-sensor-application/01-ranging-sensors.zh">测距传感器合集</a></span>
    </p>