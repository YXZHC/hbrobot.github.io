# 举升与旋转控制

## 📖 概述

在自主移动机器人（如仓储AGV、服务机器人）的应用中，举升与旋转模块是实现物体搬运、角度调整及精准对接的核心执行机构。本节教程将基于 `wsc_robotic` SDK，独立且详细地分别介绍举升电机（`LiftMotor`）与旋转电机（`RotateMotor`）的控制流程，涵盖设备实例化、回原点校准、绝对位置控制以及带梯形速度规划的高级运动控制，帮助您精准驾驭机器人的执行机构。

## ✅ 前置条件

在开始本节教程前，请确保您已完成以下准备：

- **硬件准备**：举升与旋转机构安装到位，电机驱动板与主控通信正常，原点限位开关接线无误（至关重要！）。
- **软件环境**：已安装 Python 3.x 及 `wsc_robotic` 功能包。
- **前置章节**：已阅读并理解《用户举升与旋转参数》章节中的参数配置方法。

## 📌 核心接口说明

举升与旋转电机的控制接口设计保持高度一致，均支持直接位置控制与梯形速度规划控制。

| 方法名 | 参数说明 | 描述 |
| :--- | :--- | :--- |
| `back_origin()` | 无 | 控制电机寻找原点限位，触发后清零当前位置，建立绝对坐标基准。 |
| `wait_finished()` | 无 | 阻塞当前线程，直到电机执行完当前指令到达目标位置。 |
| `get_feedback()` | 无 | 获取当前电机的实时位置反馈（举升为 `cm`，旋转为 `deg`）。 |
| `set_position(value, speed)` | `value`: 距离或角度<br>`speed`: 速度 | 以给定速度运动到绝对目标位置。 |
| `set_position_with_plan(...)` | 目标值, 速度, 加速度, 减速度 | 启用内部梯形规划器运行到目标位置，可独立设置加减速，使运动更平滑。 |

---

## ⬆️ 举升电机控制范例

本范例演示举升电机的完整控制流程：回原点 -> 常速举升 -> 获取反馈 -> 梯形规划下降。

```python
from wsc_robotic.robotics import *
from wsc_robotic.oms.lift_motor import *

def main():
    # 1. 初始化机器人主控实例
    robot = Robotics()

    # 2. 实例化举升电机
    lift_motor = LiftMotor(robot)

    # 3. 回原点（建立位置基准，避免累计误差，上电后必须执行！）
    print(">> 举升电机正在回原点...")
    lift_motor.back_origin()
    # 等待回原点动作完成
    lift_motor.wait_finished()

    # 4. 常速位置控制 (目标距离10cm, 速度40rpm)
    print(">> 举升电机运动至 10cm 处...")
    lift_motor.set_position(10, 40)
    lift_motor.wait_finished()

    # 5. 获取当前位置反馈
    print('当前位置: ', lift_motor.get_feedback(), 'cm')

    # 6. 梯形规划位置控制 (距离8cm, 速度50rpm, 加速度3000tick/s^2, 减速度3000tick/s^2)
    print(">> 举升电机按梯形规划平滑运动至 8cm 处...")
    lift_motor.set_position_with_plan(distance=8, velocity=50, acc=3000, decel=3000)
    lift_motor.wait_finished()

    # 7. 安全停止
    robot.stop()
    print(">> 举升任务完成！")

if __name__ == '__main__':
    main()
```

### 运行预期

- 举升机构先执行回原点动作，直到触碰到限位；
- 随后以 40rpm 速度平稳抬起至 10cm 位置停下；
- 终端打印当前反馈位置；
- 最后启用梯形规划，平滑减速下降至 8cm 位置，无机械冲击。

---

## 🔄 旋转电机控制范例

本范例演示旋转电机的完整控制流程：回原点 -> 常速旋转 -> 获取反馈 -> 梯形规划回零。

```python
from wsc_robotic.robotics import *
from wsc_robotic.oms.rotate_motor import *

def main():
    # 1. 初始化机器人主控实例
    robot = Robotics()

    # 2. 实例化旋转电机
    rotate_motor = RotateMotor(robot)

    # 3. 回原点
    print(">> 旋转电机正在回原点...")
    rotate_motor.back_origin()
    rotate_motor.wait_finished()

    # 4. 常速位置控制 (目标角度90deg, 速度100rpm)
    print(">> 旋转电机运动至 90deg 处...")
    rotate_motor.set_position(90, 100)
    rotate_motor.wait_finished()

    # 5. 获取当前位置反馈
    print('当前位置: ', rotate_motor.get_feedback(), 'deg')

    # 6. 梯形规划位置控制 (角度0deg, 速度100rpm, 加速度2000tick/s^2, 减速度2000tick/s^2)
    print(">> 旋转电机按梯形规划平滑回零...")
    rotate_motor.set_position_with_plan(0, 100, 2000, 2000)
    rotate_motor.wait_finished()

    # 7. 安全停止
    robot.stop()
    print(">> 旋转任务完成！")

if __name__ == '__main__':
    main()
```

### 运行预期

- 旋转机构先执行回原点动作，对齐零位；
- 随后以 100rpm 速度旋转至 90° 位置停下；
- 终端打印当前反馈角度；
- 最后启用梯形规划，平滑减速旋转归位至 0°。

---

## 💡 进阶扩展与常见问题

???+ tip "何时使用梯形速度规划 (`set_position_with_plan`)？"
    对于负载较大或存在机械惯性的机构（如载重型举升台），直接使用 `set_position` 瞬间启动/停止会导致机械冲击较大，产生抖动甚至损坏结构。通过 `set_position_with_plan` 设置合适的加速度（`acc`）和减速度（`decel`），可以规划出梯形速度曲线，实现平滑的加减速过程，让机器人动静皆宜。

???+ warning "常见问题：调用 `back_origin()` 后电机一直运动不停"
    - **排查1**：检查原点限位开关是否正常工作，接线是否松脱。电机在触碰限位开关前无法感知原点信号，会一直运动直至触发硬件保护。
    - **排查2**：确认限位开关的逻辑电平是否与驱动配置匹配（常开/常闭配置错误会导致驱动板误以为一直处于原点信号中，从而直接跳过回原点动作）。

???+ warning "常见问题：梯形规划的加速度单位 `tick/s^2` 如何理解？"
    - **理解**：`tick` 在电机控制中通常指编码器的脉冲数。`tick/s^2` 是以脉冲为基准的角加速度/线加速度单位。实际物理加速度（如 $m/s^2$）需要结合丝杠导程或减速比进行换算。
    - **调参建议**：在应用层，不要一上来就设很大！建议先以较小的数值（如 1000）为起点测试，逐步增大直至运动响应既平滑又迅速。

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
        <span>⬅️ <strong>上一节</strong><br><a href="../03-user-lift-rotate-params.zh">用户举升与旋转参数配置</a></span>
        <span>📑 <strong>上级目录</strong><br><a href="../02-servo-control.zh">servo-control</a></span>
        <span>➡️ <strong>下一节</strong><br><a href="../../02-chassis-control/01-basic-velocity-control.zh">基础底盘速度控制</a></span>
    </p>