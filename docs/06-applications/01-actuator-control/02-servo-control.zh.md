# 舵机控制

## 📖 概述

在机器人应用中，舵机（Servo）是实现夹爪开合、机械臂伸缩及末端旋转等精细动作的核心执行机构。本节教程将基于 `wsc_robotic` SDK，详细介绍两种舵机控制模式的开发流程：

- **基础模式 (`Servo`)**：提供直接的角度/位置控制与扭矩释放功能，适用于简单的点位控制场景。
- **高级规划模式 (`ServoActuator`)**：内置梯形速度规划器，支持加减速平滑控制与多轴并发执行，适用于对运动轨迹和平稳性要求较高的场景。

帮助您从入门到进阶，全面驾驭机器人的末端执行机构！🤖

## ✅ 前置条件

在开始本节教程前，请确保您已完成以下准备：

- **硬件准备**：舵机及机械臂安装到位，舵机控制板与主控通信正常，舵机电源供电稳定。
- **软件环境**：已安装 Python 3.x 及 `wsc_robotic` 功能包。
- **前置章节**：已阅读并理解《舵机参数配置》章节中的参数配置方法。

## 📌 核心接口说明

### 基础控制接口 (`Servo`)

| 方法名 | 参数说明 | 描述 |
| :--- | :--- | :--- |
| `start()` | 无 | 显式启动舵机状态发布线程。 |
| `stop()` | 无 | 显式停止舵机状态发布线程。 |
| `gripper(value, release=False)` | `value`: 角度/位置<br>`release`: 是否释放扭矩 | 控制夹爪舵机。若 `release=True`，舵机到达目标后将失去扭矩保持力。 |
| `telescopic(value, release=False)` | `value`: 角度/位置<br>`release`: 是否释放扭矩 | 控制伸缩舵机。 |
| `gripper_ry(value, release=False)` | `value`: 角度/位置<br>`release`: 是否释放扭矩 | 控制末端绕 Y 轴旋转舵机。 |
| `gripper_rz(value, release=False)` | `value`: 角度/位置<br>`release`: 是否释放扭矩 | 控制末端绕 Z 轴旋转舵机。 |

### 高级规划接口 (`ServoActuator`)

| 方法名 | 参数说明 | 描述 |
| :--- | :--- | :--- |
| `start()` | 无 | 启动内部 PWM 发布线程。 |
| `stop()` | 无 | 停止内部 PWM 发布线程。 |
| `[axis].start(pos, vel, acc, decel)` | `pos`: 目标位置<br>`vel`: 速度<br>`acc`: 加速度<br>``decel`: 减速度 | 针对指定轴（如 `gripper`, `telescopic` 等）启动梯形速度规划运动。返回 `(ret, info)`。 |
| `[axis].wait()` | 无 | 阻塞当前线程，等待该轴规划运动完成。 |

---

## 🤖 基础舵机控制范例 (`Servo`)

本范例演示 `Servo` 模块的两种使用方式：显式启停控制与优雅的上下文管理器（`with...as`）控制。

```python
import time

# 导入机器人库及舵机模块
from wsc_robotic.robotics import *
from wsc_robotic.oms.servo import *

# 方式1：显式启停控制
def mode1(robot):
    # 实例化舵机,并启动发布
    oms_servo = Servo(robot)
    oms_servo.start()   # 显示启动发布线程

    # 控制舵机指定角度后释放舵机
    oms_servo.gripper(5)
    oms_servo.telescopic(0)
    oms_servo.gripper_ry(0)
    oms_servo.gripper_rz(0)
    time.sleep(3)
    
    # 到达目标后释放扭矩（适用于抓取易碎物品或需手动调整的姿态）
    oms_servo.gripper(0, True)
    oms_servo.telescopic(0, True)
    oms_servo.gripper_ry(0, True)
    oms_servo.gripper_rz(0, True)

    # 关闭舵机,停止发布
    time.sleep(0.1)
    oms_servo.stop()    # 显示关闭发布线程

# 方式2：上下文管理器 (推荐✨)
def mode2(robot):
    """
        使用with...as..方式, 内部自动调用start()和stop()方法
        无需显式调用，代码更安全简洁
    """
    with Servo(robot) as oms_servo:
        oms_servo.gripper(10)
        time.sleep(2)
        oms_servo.gripper(5)
        time.sleep(1)
        oms_servo.gripper(0, True)

        oms_servo.gripper_ry(90)
        time.sleep(2)
        oms_servo.gripper_ry(0)
        time.sleep(2)

def main():
    robot = Robotics()
    
    # mode1(robot)
    mode2(robot)
    
    robot.stop()

if __name__ == '__main__':
    main()
```

### 运行预期

- 夹爪先张开至角度 10，随后闭合至 5，最终归零并释放扭矩（手可轻松拨动）；
- 末端 Y 轴旋转舵机旋转至 90°，停留 2 秒后归位；
- 退出 `with` 代码块后，发布线程自动安全关闭。

---

## 🚀 高级舵机执行器控制范例 (`ServoActuator`)

本范例演示 `ServoActuator` 的梯形速度规划与多轴并发执行能力。通过设定加减速参数，让机械臂动作如丝般顺滑！

```python
import time

# 导入机器人库及舵机执行器模块
from wsc_robotic.robotics import *
from wsc_robotic.oms.servo_actuator import *

# 方式一：上下文管理器
def mode1(robot):
    """
        使用with...as..方式, 内部自动调用start()和stop()方法
    """
    with ServoActuator(robotics=robot) as servo_actuator:
        # 夹爪以 20速度, 150加减速 闭合至 15
        servo_actuator.gripper.start(15, 20, 150, 150)
        servo_actuator.gripper.wait()
        
        # 夹爪以相同参数回归至 0
        servo_actuator.gripper.start(0, 20, 150, 150)
        servo_actuator.gripper.wait()

# 方式二：多线程规划执行 (进阶🌟)
def mode2(robot):
    ###############初始化舵机执行器##############
    servo_actuator = ServoActuator(robot)
    servo_actuator.start()  # 内部启动舵机PWM发布线程

    ##############多线程规划执行OMS#################
    # 设定目标参数 后台启动规划
    # 始位置:0.0 目标位置:15 速度:18 加速度:45 减速度:10
    ret, info = servo_actuator.telescopic.start(15, 18, 45, 10)
    if ret:
        print('规划信息: ', info)
    servo_actuator.telescopic.wait()

    # 以下三个轴将同时启动规划（并发执行），无需等待前一轴完成
    servo_actuator.gripper.start(15, 20, 150, 150)
    servo_actuator.gripper_ry.start(90, 200, 700, 700)
    time.sleep(1)  # 延时1秒后再启动rz轴，形成时序差
    servo_actuator.gripper_rz.start(90, 300, 2000, 2000)

    # 等待上述运动全部完成
    servo_actuator.gripper.wait()
    servo_actuator.gripper_ry.wait()
    servo_actuator.gripper_rz.wait()

    # 启动回零运动
    servo_actuator.gripper.start(0, 20, 150, 150)
    servo_actuator.gripper_ry.start(0, 200, 700, 700)
    servo_actuator.gripper_rz.start(0, 300, 2000, 2000)

    servo_actuator.telescopic.start(0, 18, 45, 45)
    servo_actuator.telescopic.wait()

    # 控制完成后关闭执行器后台发布线程
    servo_actuator.stop()

def main():
    robot = Robotics()

    # mode1(robot)
    mode2(robot)

    robot.stop()

if __name__ == '__main__':
    main()
```

### 运行预期

- 伸缩臂平滑伸出至指定位置；
- 夹爪、Y轴旋转、Z轴旋转根据各自的规划参数**同时或错时**运转，展现出复杂的多轴协同动作；
- 控制台输出 `规划信息: ...` 表明梯形规划器成功启动；
- 所有动作完成后，各轴平滑返回零位。

---

## 💡 进阶扩展与常见问题

???+ tip "何时使用 `Servo` 与 `ServoActuator`？"
    - **`Servo`**：适用于对运动过程无平滑度要求、只需简单触发目标角度的场景（如简单的开关、无需考虑惯性的轻微开合）。它的 `release` 参数非常适合需要柔顺抓取或手动示教的场景。
    - **`ServoActuator`**：适用于对运动轨迹有严格要求、负载较大容易产生冲击的场景。通过设定速度与加减速，它能生成梯形速度曲线，保护机械结构，并支持多轴并发运行，效率更高。

???+ warning "常见问题：多轴并发时为什么有时动作会卡顿或不同步？"
    - **排查1**：检查是否在启动规划（`.start()`）后立即调用了 `.wait()`。`.wait()` 会阻塞当前线程直到该轴运动完毕。如果希望多轴同时运动，应该**先依次调用所有轴的 `.start()`，再统一调用 `.wait()`**。
    - **排查2**：供电不足。多舵机同时瞬间启动时电流峰值极大，若电源功率不够会导致电压骤降，引发控制板重启或舵机卡顿，建议确保电源余量充足。

???+ warning "常见问题：`ServoActuator` 的 `start()` 返回的 `ret` 为 False 是什么原因？"
    - **排查**：通常是因为传入的参数超出限制（如速度设置过高、加速度为0或负数），或上一个规划动作还未结束又向同一轴发送了新指令。建议在调用时检查返回值 `ret`，并打印 `info` 查看具体错误提示。

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
        <span>⬅️ <strong>上一节</strong><br><a href="../01-servo-params.zh">舵机参数配置</a></span>
        <span>📑 <strong>上级目录</strong><br><a href="../02-servo-control.zh">servo-control</a></span>
        <span>➡️ <strong>下一节</strong><br><a href="../03-user-lift-rotate-params.zh">用户举升与旋转参数配置</a></span>
    </p>