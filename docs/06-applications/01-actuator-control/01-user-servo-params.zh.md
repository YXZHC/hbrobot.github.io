# 用户舵机参数配置 🛠️

## 📖 概述

在让机器人执行精准的抓取、旋转与伸缩动作之前，**参数配置是至关重要的一步**！🔧 就像给乐器调音一样，只有参数配置准确，机器人的动作才能精准无误。

本节教程将详细解析 `wsc_robotic` SDK 中末端执行机构（OMS）舵机的底层参数字典。理解这些参数不仅有助于您在常规场景下顺利控制机器人，更是实现**极限位置软限位保护**、**零位校准**与**行程映射**的必修课。

## 📌 硬件引脚映射

首先，我们需要明确舵机物理端口与主控 PWM 通道的对应关系。在默认配置中，引脚映射如下：

| 物理端口 | PWM 通道 | 默认对应机构 | 功能描述 |
| :--- | :--- | :--- | :--- |
| PIN 0 | PWM 14 | `预留` | 预留 |
| PIN 1 | PWM 15 | `gripper_ry` | 卡爪绕Y轴旋转舵机（摆臂） |
| PIN 2 | PWM 16 | `telescopic` | 伸缩舵机（伸缩） |
| PIN 3 | PWM 17 | `gripper` | 卡爪舵机（开合） |
| PIN 4 | PWM 18 | `gripper_rz` | 卡爪绕Z轴旋转舵机（旋臂） |

---

## 📐 参数字典详解

根据不同的运动特性，舵机参数被划分为**旋转类舵机**（基于角度与占空比插值）和**线性类舵机**（基于行程与占空比映射）。

### 1. 旋转类舵机参数 (角度控制)

对于 `gripper_ry`（摆臂）和 `gripper_rz`（旋臂），SDK 采用两点校准法（0度与90度占空比）来插值计算任意角度的 PWM 占空比。

```python
# 卡爪绕Y轴旋转舵机  [摆臂]
gripper_ry = {
    'pin': 1,               # 端口号 (对应 PIN 1)
    'min': -10,             # 最小角度 (软限位，单位：度)
    'max': 95,              # 最大角度 (软限位，单位：度)
    'zero_duty': 29.6,      # 零位时占空比 (%)
    'deg90_duty': 41.55     # 90度时占空比 (%)
}

# 卡爪绕Z轴旋转舵机  [旋臂]
gripper_rz = {
    'pin': 4,               # 端口号 (对应 PIN 4)
    'min': -95,             # 最小角度 (软限位，单位：度)
    'max': 95,              # 最大角度 (软限位，单位：度)
    'zero_duty': 29.2,      # 零位时占空比 (%)
    'deg90_duty': 17.4      # 90度时占空比 (%)
}
```

**🔑 关键字段解析：**

- `min` / `max`：**软限位**。即使舵机物理上能转更宽的角度，SDK 也会在软件层拦截超出此范围的指令，防止机械碰撞损坏设备！🛡️
- `zero_duty`：0° 时的 PWM 占空比。这是舵机的基准零位。
- `deg90_duty`：90° 时的 PWM 占空比。注意 `gripper_rz` 的 `deg90_duty` (17.4) 小于 `zero_duty` (29.2)，这说明该舵机为**反向安装**，增大角度会导致占空比减小。

### 2. 线性类舵机参数 (行程/距离控制)

对于 `telescopic`（伸缩）和 `gripper`（卡爪），SDK 将物理行程（如直线距离）映射到占空比区间。

```python
# 伸缩舵机参数       [伸缩]
telescopic_param = {
    'pin': 2,               # 端口号 (对应 PIN 2)
    'min_duty': 11.0,       # 最小占空比 (%) -> 对应行程 0 cm
    'max_duty': 49.0,       # 最大占空比 (%) -> 对应行程 18.0 cm
    'itinerary': 18.0       # 物理行程 (单位：cm)
}

# 卡爪舵机参数       [卡爪]
gripper_param = {
    'pin': 3,               # 端口号 (对应 PIN 3)
    'min_dis': 0.0,         # 最小距离 (软限位，单位：cm)
    'max_dis': 31.0,        # 最大距离 (软限位，单位：cm)
    'min_duty': 10.0,       # 最小占空比 (%) -> 对应卡爪完全闭合
    'max_duty': 50.0,       # 最大占空比 (%) -> 对应卡爪最大行程
    'itinerary': 15.5       # 物理行程 (单位：cm)
}
```

**🔑 关键字段解析：**

- `min_duty` / `max_duty`：定义了舵机从一端走到另一端的 PWM 占空比边界，是距离换算的基石。
- `itinerary`：实际的物理行程长度。SDK 会根据 `itinerary` 和占空比区间计算出一个比例系数（如 `k = (max_duty - min_duty) / itinerary`），从而将用户输入的距离（cm）精确转化为 PWM 信号。
- `min_dis` / `max_dis`：卡爪特有的**距离软限位**，防止夹坏物品或过度张开。

---

## 💻 运行示例：交互式舵机调试与零位校准

在实际部署中，由于装配误差，每台机器人的 `zero_duty` 或 `min_duty` 可能略有不同。以下示例提供了一个交互式的命令行辅助脚本，支持选择特定舵机端口、重复输入 PWM 值进行实时调试、随时切换端口以及安全退出，帮助您快速找到正确的零位或行程边界占空比。

将以下代码保存在项目 `/wsc_hbrobotv2_python/user` 目录中，命名为 `straight_rotate_control.py`，然后执行程序：

```python
import time
from wsc_robotic.robotics import *
from wsc_robotic.oms.servo import *


if __name__ == '__main__':
    robot = Robotics()
    # 实例化舵机，并启动发布
    oms_servo = Servo(robot)
    oms_servo.start()
    print("=== 舵机调试程序已启动 ===")

    exit_program = False

    try:
        while not exit_program:
            # 选择调试机构（舵机接口）
            print("\n--- 请选择要调试的舵机端口 ---")
            print("0: SERVO0 | 1: SERVO1 | 2: SERVO2 | 3: SERVO3 | 4: SERVO4")
            port_input = input("输入端口号 (输入 'q' 退出程序): ").strip()
            
            if port_input.lower() == 'q':
                exit_program = True
                break
                
            try:
                port_num = int(port_input)
                if port_num not in [s.value for s in ServoPort]:
                    print("错误：端口号必须在 0 到 4 之间！")
                    continue
                
                current_port = ServoPort(port_num)
                print(f"\n>>> 已切换至舵机端口: {current_port.name} <<<")
                
            except ValueError:
                print("输入无效，请输入数字或 'q'。")
                continue

            # 重复输入 PWM 值进行调试
            while not exit_program:
                pwm_input = input(f"输入 PWM 占空比 (0-50) [输入 'c' 切换端口, 'q' 退出]: ").strip()
                
                if pwm_input.lower() == 'q':
                    exit_program = True
                    break
                elif pwm_input.lower() == 'c':
                    print("正在返回端口选择...")
                    break
                
                try:
                    duty = int(pwm_input)
                    if 0 <= duty <= 50:
                        # 向选定舵机写入占空比
                        oms_servo.write_duty(current_port, duty)
                        print(f"[执行成功] 端口 {current_port.name} | 占空比: {duty}")
                        time.sleep(0.05)  # 极短延时保证指令下发
                    else:
                        print("错误：PWM 占空比必须在 0 到 50 之间！")
                except ValueError:
                    print("输入无效，请输入数字、'c' 或 'q'。")
                    
    except KeyboardInterrupt:
        print("\n检测到手动中断(Ctrl+C)...")
    finally:
        # 确保无论正常退出还是异常中断，都能关闭发布线程
        time.sleep(0.1)
        oms_servo.stop()
        print("=== 舵机发布线程已关闭，程序安全退出 ===")
```

**🔑 脚本功能说明：**

- **端口选择**：程序启动后，可输入 `0-4` 选择对应的舵机端口进行调试。
- **实时调试**：选定端口后，可连续输入 `0-50` 之间的 PWM 占空比，观察舵机运动状态，便于精准标定零位或极限位置。
- **灵活切换**：在调试过程中，输入 `c` 可随时返回端口选择菜单，切换其他舵机。
- **安全退出**：输入 `q` 或触发 `Ctrl+C` 中断，程序均会在 `finally` 块中安全关闭发布线程，避免资源泄露。


---

## 💡 进阶扩展与常见问题

???+ tip "为什么旋转舵机需要 `zero_duty` 和 `deg90_duty` 两个参数？"
    由于模拟舵机或硬件电路的非线性，PWM 占空比与角度之间并非完美的绝对线性关系。使用两点校准法（0度和90度）可以计算出一条基准斜率 `k = (deg90_duty - zero_duty) / 90`。SDK 在收到目标角度（如 45°）时，会通过公式 `duty = zero_duty + k * 45` 实时计算占空比，这比单纯的线性比例映射精度更高！🎯

???+ warning "常见问题：修改了参数字典中的限位，但机器人还是能运动到更远的位置？"
    - **排查**：请确保您修改的是**正在被实例化类加载**的参数文件。如果只是在副本上修改，SDK 运行时依然读取的是默认配置。此外，软限位仅在 SDK 逻辑层生效，如果是通过底层 PWM 直接发送指令绕过 SDK 校验，则软限位无效。

???+ warning "常见问题：卡爪控制输入的是距离（cm），这和行程（itinerary）有什么关系？"
    - **理解**：`itinerary` 是舵机从 `min_duty` 运动到 `max_duty` 所代表的实际物理位移。而输入的 `min_dis` 和 `max_dis` 是您设定的安全工作区间。输入的距离值会先通过比例换算成占空比：`目标占空比 = min_duty + (目标距离 / itinerary) * (max_duty - min_duty)`。

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
        <span>⬅️ <strong>上一节</strong><br><a href="../00-hardware-overview.zh">硬件概览与组装</a></span>
        <span>📑 <strong>上级目录</strong><br><a href="../02-servo-control.zh">servo-control</a></span>
        <span>➡️ <strong>下一节</strong><br><a href="../02-servo-control.zh">舵机控制教程</a></span>
    </p>