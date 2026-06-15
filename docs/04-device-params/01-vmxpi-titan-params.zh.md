# ⚙️ VMX_Pi / Titan 设备参数配置

为了兼容不同的设备平台，需要根据实际使用需求调整硬件参数。默认示例参数通常可用，但可能与您的硬件平台存在差异，请重点关注独立列出的参数项。

**📁 配置文件位置：**

```bash
/home/vmx/WSR_HB_Robot/install/vmxpi_ros2/share/vmxpi_ros2/config/config.yaml
```

??? example "🔧 点击查看：默认示例参数文件（`config.yaml`）"

    ```yaml
    vmxpi:
    realtime: True            # 使用实时时钟
    update_rate: 50           # AHRS 更新频率 (Hz)
    enable_i2c: False         # 使能 I²C

    vmxpi_update_rate: 50       # VMXPi 通讯线程频率 (Hz)

    emg_topic: "/chassis/emg_status"   # 急停话题

    # IO 引脚配置
    io:
    di: [9, 11]                     # 数字输入引脚
    do: [19, 20, 21]                # 数字输出引脚
    ai: [22, 23, 24, 25]            # 模拟输入引脚
    pwm: [14, 15, 16, 17, 18]       # PWM 输出引脚
    pwm_default_fre: 200            # PWM 默认频率 (Hz)

    # 超声波引脚配置
    sonar:
    trig_pins: [12, 13]             # 触发引脚
    echo_pins: [8, 10]              # 回响引脚

    # VMXPi 编码器引脚
    encoder:
    phase_a_pins: [0, 2, 4, 6]      # A 相引脚
    phase_b_pins: [1, 3, 5, 7]      # B 相引脚

    # Titan 驱动器名称列表（单 Titan 示例）
    # titan_quad_name: ['titan_quad0']
    # 若为双 Titan，则使用：['titan_quad0', 'titan_quad1']
    titan_quad_name: ['titan_quad0', 'titan_quad1']

    # ========== Titan 0 配置 ==========
    titan_quad0:
    id: 32                          # CAN ID（默认 42）
    fre: 15600                      # PWM 频率 15.6 kHz
    stop_mode: 0                    # 刹车模式：0 = COAST, 1 = BREAK
    titan_update_rate: 50           # Titan 通讯线程频率 (Hz)

    motor_controller:
        encoder_ppi: 1464.0           # 编码器一圈脉冲数
        auto_enable_titan: True       # 自动使能 Titan
        use_vmx_encoder: True         # 使用 VMXPi 编码器（否则使用 Titan 编码器）

        # 电机配置（索引 0~3 对应四个电机）
        motor:
        index0:
            encoder_port: 0			    # 编码器端口
            motor_port: 0			      # titan电机端口
            kp: 0.3					        # KP
            ti: 0.0002				      # Ti
            td: 0.0					        # Td
            invert: False			      # 反相电机旋转ccw<->cw
        index1:
            encoder_port: 1
            motor_port: 1
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False
        index2:
            encoder_port: 2
            motor_port: 2
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False
        index3:
            encoder_port: 3
            motor_port: 3
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False

        # 速度平滑器参数
        velocity_filter:
        index0:
            enable: True			       # 使能速度平滑器
            max_vel: 100.0           # 最大速度约束 (r/min)
            max_acc: 600.0           # 最大加速度 (r/min²)
            lowpass_coeff: 0.8       # 低通滤波系数 (0~1)
        index1:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
        index2:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
        index3:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8

    # ========== Titan 1 配置 ==========
    titan_quad1:
    id: 42                          # CAN ID（默认 42）
    fre: 15600                      # PWM 频率 15.6 kHz
    stop_mode: 0                    # 刹车模式：0 = COAST, 1 = BREAK
    titan_update_rate: 50           # Titan 通讯线程频率 (Hz)

    motor_controller:
        encoder_ppi: 1464.0           # 编码器一圈脉冲数
        auto_enable_titan: True       # 自动使能 Titan
        use_vmx_encoder: True         # 使用 VMXPi 编码器（否则使用 Titan 编码器）

        # 电机配置（索引 0~3 对应四个电机）
        motor:
        index0:
            encoder_port: 0			    # 编码器端口
            motor_port: 0			      # titan电机端口
            kp: 0.3					        # KP
            ti: 0.0002				      # Ti
            td: 0.0					        # Td
            invert: False			      # 反相电机旋转ccw<->cw
        index1:
            encoder_port: 1
            motor_port: 1
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False
        index2:
            encoder_port: 2
            motor_port: 2
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False
        index3:
            encoder_port: 3
            motor_port: 3
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False

        # 速度平滑器参数
        velocity_filter:
        index0:
            enable: True			       # 使能速度平滑器
            max_vel: 100.0           # 最大速度约束 (r/min)
            max_acc: 600.0           # 最大加速度 (r/min²)
            lowpass_coeff: 0.8       # 低通滤波系数 (0~1)
        index1:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
        index2:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
        index3:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8

    ```

---

## 1️⃣ VMX 通讯参数

| 参数名称 | 最小值 | 默认值 | 最大值 | 参数说明 |
| :--- | :---: | :---: | :---: | :--- |
| **`update_rate`** | 25 Hz | 50 Hz | 100 Hz | 板载 IMU 通过 AHRS 与主控制器通讯。可根据需要调整 IMU 通讯频率，默认 50 Hz；控制器负载过高时可降低至 25 Hz。 |
| **`enable_i2c`** | / | `False` | `True` | 板载 I²C 组件开启时会占用较多主控制器资源。不使用 QTI/LSB 传感器或 I²C 设备时应保持关闭，以降低资源占用。 |
| **`vmxpi_update_rate`** | 25 Hz | 50 Hz | 100 Hz | VMXPi 主通讯线程频率，影响外部 IO 响应速率（同时影响 ROS 数据发布频率）。默认 50 Hz，负载过高时可降至 25 Hz。 |

!!! tip "优化建议"
    若无特殊需求，保持默认值即可；若出现控制延迟或资源不足，优先降低 `update_rate` 和 `vmxpi_update_rate`。

---

## 2️⃣ Titan 控制参数

| 参数名称 | 最小值 | 默认值 | 最大值 | 参数说明 |
| :--- | :---: | :--- | :---: | :--- |
| **`titan_quad_name`** | / | `['titan_quad0', 'titan_quad1']` | / | CAN 总线可挂载多个 Titan 控制器。默认挂载两个，每个控制器有独立参数组（通过 CAN ID 识别，默认为 42）。若只有一个 Titan 设备，需修改为单 Titan 配置，例如：`titan_quad_name: ['titan_quad0']`。 |
| **`titan_quad0.id`** | 0 | 42 | 63 | CAN ID 默认为 42。部分双 Titan 设备分别配置为 32 和 42，请根据设备实际配置修改。 |
| **`titan_update_rate`** | 25 Hz | 50 Hz | 100 Hz | Titan 设备通过 CAN 总线交互，默认采用 50 Hz 的通讯闭环控制频率（影响相关 ROS 话题频率）。多设备时 CAN 总线负荷增加，可降至 25 Hz。 |
| **`encoder_ppi`** | / | 1464.0 | / | 对应直流电机编码器一圈的脉冲数：6 × 61 × 4 = 1464。 |
| **`auto_enable_titan`** | `False` | `True` | / | Titan 控制器建立通讯后需使能信号才能驱动。默认自动使能；若不需要，请设为 `False`。 |
| **`use_vmx_encoder`** | `False` | `True` | / | 直流电机闭环控制依赖编码器信息。可选 VMXPi 编码器或 Titan 编码器。默认使用 VMXPi 编码器；如需使用 Titan 编码器，请设为 `False`。 |

!!! warning "安全警告"
    修改 CAN ID 或编码器来源前，请确认硬件实际接线与固件配置，否则可能导致电机失控。

---

## 3️⃣ 完整配置示例（单 Titan 设备）

以下为仅挂载一个 Titan 设备时 `config.yaml` 的完整内容，您可以根据实际硬件平台修改对应字段。

??? example "📄 点击展开：单 Titan 完整配置示例"

    ```yaml
    vmxpi:
    realtime: True            # 使用实时时钟
    update_rate: 50           # AHRS 更新频率 (Hz)
    enable_i2c: False         # 使能 I²C

    vmxpi_update_rate: 50       # VMXPi 通讯线程频率 (Hz)

    emg_topic: "/chassis/emg_status"   # 急停话题

    # IO 引脚配置
    io:
    di: [9, 11]                     # 数字输入引脚
    do: [19, 20, 21]                # 数字输出引脚
    ai: [22, 23, 24, 25]            # 模拟输入引脚
    pwm: [14, 15, 16, 17, 18]       # PWM 输出引脚
    pwm_default_fre: 200            # PWM 默认频率 (Hz)

    # 超声波引脚配置
    sonar:
    trig_pins: [12, 13]             # 触发引脚
    echo_pins: [8, 10]              # 回响引脚

    # VMXPi 编码器引脚
    encoder:
    phase_a_pins: [0, 2, 4, 6]      # A 相引脚
    phase_b_pins: [1, 3, 5, 7]      # B 相引脚

    # Titan 驱动器名称列表（单 Titan 示例）
    titan_quad_name: ['titan_quad0']
    # 若为双 Titan，则使用：['titan_quad0', 'titan_quad1']

    # ========== Titan 0 配置 ==========
    titan_quad0:
    id: 42                         # CAN ID（默认 42）
    fre: 15600                     # PWM 频率 15.6 kHz
    stop_mode: 0                   # 刹车模式：0 = COAST, 1 = BREAK
    titan_update_rate: 50          # Titan 通讯线程频率 (Hz)

    motor_controller:
        encoder_ppi: 1464.0          # 编码器一圈脉冲数
        auto_enable_titan: True      # 自动使能 Titan
        use_vmx_encoder: True        # 使用 VMXPi 编码器（否则使用 Titan 编码器）

        # 电机配置（索引 0~3 对应四个电机）
        motor:
        index0:
            encoder_port: 0			 # 编码器端口
            motor_port: 0			 # titan电机端口
            kp: 0.3					 # KP
            ti: 0.0002				 # Ti
            td: 0.0					 # Td
            invert: False			 # 反相电机旋转ccw<->cw
        index1:
            encoder_port: 1
            motor_port: 1
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False
        index2:
            encoder_port: 2
            motor_port: 2
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False
        index3:
            encoder_port: 3
            motor_port: 3
            kp: 0.3
            ti: 0.0002
            td: 0.0
            invert: False

        # 速度平滑器参数
        velocity_filter:
        index0:
            enable: True			 # 使能速度平滑器
            max_vel: 100.0           # 最大速度约束 (r/min)
            max_acc: 600.0           # 最大加速度 (r/min²)
            lowpass_coeff: 0.8       # 低通滤波系数 (0~1)
        index1:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
        index2:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
        index3:
            enable: True
            max_vel: 100.0
            max_acc: 600.0
            lowpass_coeff: 0.8
    ```

!!! info "参数调优提示"
    - **PID 参数**（`kp`, `ti`, `td`）：影响电机响应速度和稳态精度。增大 `kp` 可提高响应，但过大会引起震荡；`ti` 积分时间越小积分作用越强；`td` 微分可抑制超调。
    - **速度平滑器**：`max_acc` 越大响应越快，但超调也越大；`lowpass_coeff` 越小滤波越平滑，但滞后增加。

---

## 4️⃣ 修改配置后的操作

完成配置文件修改后，需要重启 ROS 节点或机器人以使新参数生效：

```bash
# 重启机器人主节点（推荐）
sudo /home/vmx/WSR_HB_Robot/robot_manager.py restart
```

!!! success "验证配置"
    启动后可通过 `ros2 param list` 查看当前运行参数，或观察电机行为确认配置正确。

---

## ❓ 常见问题

| 问题 | 可能原因 | 解决方法 |
| :--- | :--- | :--- |
| **电机不转** | `auto_enable_titan` 为 `False` 或 CAN ID 错误 | 检查配置，手动使能或修正 ID |
| **编码器读数异常** | `encoder_ppi` 不匹配或 `use_vmx_encoder` 设置错误 | 核对电机型号，修改为正确脉冲数 |
| **CAN 通信超时** | `titan_update_rate` 过高导致总线拥塞 | 降低频率至 25 Hz |
| **I²C 设备无响应** | `enable_i2c` 为 `False` | 改为 `True` 并重启 |

!!! summary "总结"
    合理调整 VMXPi 和 Titan 参数，可以显著提升系统稳定性与响应性能。建议先在空载环境下测试参数变化，再应用到实际任务中。

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