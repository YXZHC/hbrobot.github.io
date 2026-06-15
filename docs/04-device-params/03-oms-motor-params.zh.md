# 🧠 OMS 执行机构电机参数配置指南

本文档详细介绍如何配置 OMS 系统中的**举升电机（lift_motor）**与**旋转电机（rotate_motor）**的参数，包括限位开关、回零（找原点）策略、PID 控制参数等。

---

## 📁 配置文件位置

OMS 电机参数配置文件位于：

```bash
/home/vmx/WSR_HB_Robot/install/oms_motor/share/oms_motor/config/param.yaml
```

> 💡 **提示**：修改参数后需要重启 `oms_motor` 节点或重启机器人方可生效。


## 完整配置参数示例

??? example "🔧 oms_motor 配置参数（`param.yaml`）"

    ```yaml
    motor_names:
      - rotate_motor
      - lift_motor

    motor:
      #举升电机
      lift_motor:
        set_topic: '/vmxpi/titan_quad0/set_motor2'
        feedback_topic: '/vmxpi/titan_quad0/titan/encoder2'
        limit_sw_topic: '/vmxpi/titan_quad0/limsw2'
        use_limitsw_h: True   #使用限位H(Titan)
        invert_enc: False     #反转编码器

        #找原点参数
        origin_param:
          offset_tick: -50     #限位脉冲偏移:防止顶死限位时为原点
          back_vel: -30.0      #找原点速度(rpm)
          find_cnt: 1         #找原点次数
          approix_vel: -15.0   #逼近原点速度(rpm)
          approix_step: -50   #逼近原点步长(脉冲)
          mode: 0             #找原点模式0:常规单方向 1:双向扫描找原点
          max_step: 1000      #高速找原点最大脉冲步长
        
        #控制参数
        ctrl_param:
          #位置环PID参数
          kp: 0.2
          ti: 0.0
          td: 0.0
          #输出速度滤波器参数
          max_vel: 100.0      #最大速度rpm
          max_acc: 600.0      #最大加速度(脉冲/s^2)
          low_pass: 0.85      #速度低通滤波系数[0,1]
          ek: 10              #误差因子(脉冲)
          steady_clk: 5      #稳态周期(1clk=20ms)	

      #旋转电机
      rotate_motor:
        set_topic: '/vmxpi/titan_quad0/set_motor3'
        feedback_topic: '/vmxpi/titan_quad0/titan/encoder3'
        limit_sw_topic: '/vmxpi/titan_quad0/limsw3'
        use_limitsw_h: True
        #限位脉冲偏移:防止顶死限位时为原点
        origin_offset_tick: 0
        #反转编码器
        invert_enc: False

        #找原点参数
        origin_param:
          offset_tick: 0      #限位脉冲偏移:防止顶死限位时为原点
          back_vel: 50.0      #找原点速度(rpm)
          find_cnt: 1         #找原点次数
          approix_vel: 35.0    #逼近原点速度(rpm)
          approix_step: 150   #逼近原点步长(脉冲)
          mode: 1             #找原点模式0:常规单方向 1:双向扫描找原点
          max_step: 1500      #高速找原点最大脉冲步长
        
        #控制参数
        ctrl_param:
          #位置环PID参数
          kp: 0.28
          ti: 0.0
          td: 0.0
          #输出速度滤波器参数
          max_vel: 100.0      #最大速度rpm
          max_acc: 800.0      #最大加速度(脉冲/s^2)
          low_pass: 0.85      #速度低通滤波系数[0,1]
          ek: 8               #误差因子(脉冲)
          steady_clk: 5      #稳态周期(1clk=20ms)
    ```

---


## 🧩 参数结构概览

配置文件包含两个主要电机：`lift_motor`（举升）和 `rotate_motor`（旋转）。每个电机均包含以下四大模块：

- **基础通信配置**（话题、编码器、限位开关）
- **找原点参数**（origin_param）
- **控制参数**（ctrl_param）
- **辅助选项**（编码器反转、限位使用等）

---

## 📌 1. 举升电机 (lift_motor)

举升电机控制 OMS 机构的垂直升降运动。

### 1.1 基础通信配置

```yaml
lift_motor:
  set_topic: '/vmxpi/titan_quad0/set_motor2'          # 速度/位置指令发布话题
  feedback_topic: '/vmxpi/titan_quad0/titan/encoder2' # 编码器反馈话题
  limit_sw_topic: '/vmxpi/titan_quad0/limsw2'         # 限位开关状态话题
  use_limitsw_h: True                                 # 是否使用限位开关（True=使用）
  invert_enc: False                                   # 是否反转编码器计数方向
```

| 参数 | 说明 |
| :--- | :--- |
| `set_topic` | 向 Titan 驱动器发送电机转速/位置指令的 ROS 话题。 |
| `feedback_topic` | 获取电机编码器实时计数的 ROS 话题。 |
| `limit_sw_topic` | 获取限位开关状态（触发/未触发）的话题。 |
| `use_limitsw_h` | 是否利用H限位开关信号进行回零保护。 |
| `invert_enc` | 若电机转向与编码器读数相反，设为 `True` 反转计数。 |

### 1.2 找原点参数 (origin_param)

举升电机回零（寻找机械原点）的配置。

```yaml
origin_param:
  offset_tick: -50      # 限位脉冲偏移：防止顶死限位时恰好停在原点
  back_vel: -30.0       # 找原点速度（rpm，负号表示方向）
  find_cnt: 1           # 找原点次数（通常 1 次足够）
  approix_vel: -15.0    # 逼近原点速度（rpm）
  approix_step: -50     # 逼近原点步长（脉冲数）
  mode: 0               # 找原点模式：0=常规单方向，1=双向扫描
  max_step: 1000        # 高速找原点最大脉冲步长
```

| 参数 | 单位 | 说明 |
| :--- | :---: | :--- |
| `offset_tick` | 脉冲 | 限位触发后回退的脉冲数，避免卡死在限位处。 |
| `back_vel` | rpm | 向限位方向寻找原点的速度。负值表示运动方向。 |
| `find_cnt` | 次 | 重复寻找原点的次数（提高可靠性）。 |
| `approix_vel` | rpm | 接近原点时的低速逼近速度。 |
| `approix_step` | 脉冲 | 从限位触发点回退的脉冲距离。 |
| `mode` | - | 0=单方向寻找（向限位移动），1=双向扫描（先正向再反向）。 |
| `max_step` | 脉冲 | 防止超程的安全步长限制。 |

> ⚠️ **注意**：`back_vel` 和 `approix_vel` 的正负号应与限位开关的安装方向匹配。通常情况下，举升电机向上为正，向下为负。

### 1.3 控制参数 (ctrl_param)

举升电机位置闭环控制参数。

```yaml
ctrl_param:
  kp: 0.2           # 比例系数
  ti: 0.0           # 积分时间（0 表示无积分）
  td: 0.0           # 微分时间（0 表示无微分）
  max_vel: 100.0    # 最大速度限制（rpm）
  max_acc: 600.0    # 最大加速度（脉冲/s²）
  low_pass: 0.85    # 速度低通滤波系数 [0,1]
  ek: 10            # 误差因子（脉冲）
  steady_clk: 5     # 稳态周期（1 clk = 20 ms）
```

| 参数 | 说明 |
| :--- | :--- |
| `kp` | 位置环比例增益。越大响应越快，但可能引起震荡。 |
| `ti` | 积分时间常数。增大积分可消除静差，但会降低响应速度。 |
| `td` | 微分时间常数。抑制超调，但对噪声敏感。 |
| `max_vel` | 电机允许的最大转速（r/min）。 |
| `max_acc` | 最大加速度（脉冲/秒²）。限制启动/停止时的冲击。 |
| `low_pass` | 一阶低通滤波系数。值越小速度曲线越平滑，但滞后越大。 |
| `ek` | 位置误差容限（脉冲）。当误差小于该值时认为已到位。 |
| `steady_clk` | 稳态判定周期。连续多个控制周期误差小于 `ek` 即判为稳定。 |

---

## 🔄 2. 旋转电机 (rotate_motor)

旋转电机控制 OMS 机构的水平旋转运动。参数结构与举升电机类似，仅数值不同。

### 2.1 基础通信配置

```yaml
rotate_motor:
  set_topic: '/vmxpi/titan_quad0/set_motor3'
  feedback_topic: '/vmxpi/titan_quad0/titan/encoder3'
  limit_sw_topic: '/vmxpi/titan_quad0/limsw3'
  use_limitsw_h: True
  origin_offset_tick: 0      # 原点偏移脉冲（直接配置，非子项）
  invert_enc: False
```

> 📌 **特殊参数**：`origin_offset_tick` 是旋转电机的独立原点偏移，与举升电机的 `offset_tick` 作用相同，但放置位置不同。

### 2.2 找原点参数

```yaml
origin_param:
  offset_tick: 0
  back_vel: 50.0
  find_cnt: 1
  approix_vel: 35.0
  approix_step: 150
  mode: 1                # 双向扫描找原点
  max_step: 1500
```

- `mode: 1` 表示双向扫描：先正向移动寻找限位，若未触发则反向移动，提高回零成功率。
- 旋转电机的回零速度较高（`back_vel: 50 rpm`），适用于大角度旋转。

### 2.3 控制参数

```yaml
ctrl_param:
  kp: 0.28
  ti: 0.0
  td: 0.0
  max_vel: 100.0
  max_acc: 800.0         # 加速度比举升电机更大
  low_pass: 0.85
  ek: 8                  # 误差容限更小
  steady_clk: 5
```

---

## ⚙️ 参数调优建议

### 🔧 调整回零可靠性

- 若回零时经常冲过限位导致卡死 → 降低 `back_vel` 或 `approix_vel`，增大 `offset_tick`。
- 若回零精度不足 → 减小 `approix_step`，降低 `approix_vel`。
- 若回零过程时间过长 → 适当提高 `back_vel`，或使用双向模式（`mode: 1`）。

### 🎛️ 调整位置控制响应

- **响应慢** → 增大 `kp`，适当减小 `low_pass`（加快滤波响应）。
- **超调大/震荡** → 减小 `kp`，增加 `td`（微分项），减小 `max_acc`。
- **到位后抖动** → 增大 `ek`（误差容限），或增加 `steady_clk` 周期。

### 🧪 编码器方向校验

- 若电机实际转向与指令相反 → 修改 `invert_enc: True`。
- 若编码器读数异常（静止时跳变） → 检查硬件接线或反馈话题是否对应。

---

## 🚀 修改配置后的操作

完成参数修改后，必须使配置生效：

```bash
# 重启机器人主节点（推荐）
sudo /home/vmx/WSR_HB_Robot/robot_manager.py restart
```

验证配置是否生效：

```bash
# 查看电机节点是否正常运行
ros2 node list | grep oms

# 监听电机状态话题（假设有 /motor_state）
ros2 topic echo /oms_motor/state
```

---

## ❓ 常见问题

| 现象 | 可能原因 | 解决方法 |
| :--- | :--- | :--- |
| 电机不转 | `set_topic` 错误或 Titan 未使能 | 检查话题名称，确认 `titan_quad0/enable` 服务已调用 |
| 回零时直接冲过限位 | 限位开关极性错误或 `use_limitsw_h` 配置不符 | 检查 `limit_sw_topic` 数据是否为触发状态，修改 `use_limitsw_h` |
| 位置控制震荡 | PID 参数过强 | 降低 `kp`，增加 `low_pass` 滤波 |
| 到位后误差保持不为零 | 积分项未启用或静摩擦过大 | 设置 `ti: 0.01` 启用积分，或增大 `ek` |
| 电机发热严重 | `max_acc` 过大或频繁启停 | 降低加速度，提高 `low_pass` 滤波系数 |

---

## 📚 总结

- OMS 电机配置分为**举升**和**旋转**两个独立 section，参数体系相同但数值不同。
- 关键参数包括 **找原点策略**、**PID 参数**、**速度/加速度限制**。
- 合理调参可显著提升 OMS 机构的位置精度与运行平稳性。

> 🎯 **下一步**：完成电机参数配置后，可运行 `ros2 run oms_motor oms_node` 并调用 `/oms_motor/go_to_position` 服务进行点位运动测试。

**相关文档**：
- [Titan 驱动器参数配置](01-vmxpi-titan-params.zh.md)

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