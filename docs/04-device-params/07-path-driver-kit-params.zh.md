# 🛣️ 路径驱动与运动控制套件参数配置

为了实现高精度的导航与轨迹跟踪，需要根据实际机器人的运动学模型和动力学特性调整运动控制参数。本文档详细介绍了路径驱动套件 ([`param.yaml`](./res/base_motion_kit/param.yaml)) 的各项配置，涵盖 PTP（点到点）控制、差速追踪、基础运动及全向轮连续运动驱动。

**📁 配置文件位置：**

```bash
/home/vmx/WSR_HB_Robot/install/path_driver/share/path_driver/config/param.yaml
```

??? example "🔧 点击查看：默认示例参数文件（[`param.yaml`](./res/base_motion_kit/param.yaml)）"

    ```yaml
    topic_odom: '/chassis/odom/user'      # odom topic
    topic_cmd_vel: '/chassis/cmd_vel'     # 线速度 topic 
    topic_emg: '/chassis/emg_status'      # 紧急停止状态 topic

    # |-------------> [PTP 控制器参数] <-------------|
    ptp:
      #直线运动默认参数
      line_param:
        pid_param: [2.1, 0.0, 0.0]
        max_vel: 0.67               # 最大速度m/s
        max_acc: 1.34               # 最大加速度
        low_pass: 1.0               # 低通平滑系数[0,1]
        ek: 0.007                   # 误差阈值m
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms

      #旋转运动默认参数:直线运动过程中使用
      roate_param_move:
        pid_param: [2.8, 0.0, 0.0]
        max_vel: 4.2                # 最大速度rad/s
        max_acc: 8.4                # 最大加速度
        low_pass: 1.00              # 低通平滑系数[0,1]
        ek: 0.010                   # 误差阈值rad  0.008726389 = 0.5deg
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms

      #旋转运动默认参数:初始对准及旋转到位使用
      roate_param:
        pid_param: [3.2, 0.0, 0.0]
        max_vel: 4.2                # 最大速度rad/s
        max_acc: 8.4                # 最大加速度
        low_pass: 0.95              # 低通平滑系数[0,1]
        ek: 0.010                   # 误差阈值rad  0.008726389 = 0.5deg
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms  ***


    # |-------------> [差速追踪参数] <-------------|
    follow:
      # 路径跟踪参数
      pursuit_param:
        L: 0.1                      # 轴距m
        ek: 0.01                    # 误差跟踪结束m

        #自适应前瞻距离参数 ld = kv *v + ld0
        ld0: 0.15                   # 预瞄距离最小值m
        kv: 0.1                     # 速度比例系数 kv *v
        ld_max: 0.5                 # 最大预瞄距离m

      #直线运动默认参数
      line_param:
        pid_param: [2.1, 0.0, 0.0]
        max_vel: 0.67               # 最大速度m/s
        max_acc: 1.072              # 最大加速度
        low_pass: 0.9               # 低通平滑系数[0,1]
        ek: 0.015                   # 误差阈值m
        steady_clk: 5               # 误差跳出稳定周期1clk=20ms

      #旋转运动默认参数:跟踪时旋转控制使用
      roate_param:
        pid_param: [3.2, 0.0, 0.0]
        max_vel: 4.2                # 最大速度rad/s
        max_acc: 8.4                # 最大加速度
        low_pass: 0.8               # 低通平滑系数[0,1]
        ek: 0.017                   # 误差阈值rad  0.008726389 = 0.5deg
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms

      #旋转运动默认参数:初始对准及旋转到位使用
      roate_planner_param:
        pid_param: [3.1, 0.0, 0.0]
        max_vel: 4.2                # 最大速度rad/s
        max_acc: 8.4                # 最大加速度
        low_pass: 0.95              # 低通平滑系数[0,1]
        ek: 0.017                   # 误差阈值rad  0.008726389 = 0.5deg
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms  ***

      init_heading_thres: 0.174     # 初始对准航向误差阈值(rad),大于该阈值先执行旋转对准航向
      kv_heading: 1.6               # 转弯时线速度减速系数(min=1), 调大时航向角偏差越大,线速度减速越大.
      low_pass_heading: 1.0         # 航向角低通滤波系数[0,1]


    # |-------------> [基础运动参数] <-------------|
    base_motion:
      # 直线运动默认参数
      line_param:
        pid_param: [2.3, 0.0, 0.0]
        max_vel: 0.67               # 最大速度m/s
        max_acc: 1.34               # 最大加速度
        low_pass: 0.9               # 低通平滑系数[0,1]
        ek: 0.01                    # 误差阈值m
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms ***

      # 旋转运动默认参数
      rotate_param:
        pid_param: [2.6, 0.0, 0.0]
        max_vel: 240.0              # 最大速度deg/s
        max_acc: 480.0              # 最大加速度
        low_pass: 0.9               # 低通平滑系数[0,1]
        ek: 1.0                     # 误差阈值deg  
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms ***

    # |-------------> [连续运动驱动器 (全向轮)] <-------------|
    path_driver_3wd:
      #直线运动默认参数
      line_param:
        pid_param: [3.1, 0.0, 0.0]
        max_vel: 0.67               # 最大速度m/s
        max_acc: 1.34               # 最大加速度
        low_pass: 1.0               # 低通平滑系数[0,1]
        ek: 0.003                   # 误差阈值m
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms

      #旋转运动默认参数
      roate_param:
        pid_param: [3.2, 0.0, 0.0]
        max_vel: 3.28               # 最大速度rad/s
        max_acc: 6.56               # 最大加速度
        low_pass: 0.95              # 低通平滑系数[0,1]
        ek: 0.0043633               # 误差阈值rad  0.008726389 = 0.5deg
        steady_clk: 10              # 误差跳出稳定周期1clk=20ms  ***
    ```

---

## 1️⃣ ROS 话题与通信参数

| 参数名称 | 类型 | 参数说明 |
| :--- | :---: | :--- |
| **`topic_odom`** | string | 里程计数据订阅话题，用于获取当前机器人位姿与速度。 |
| **`topic_cmd_vel`** | string | 底盘线速度/角速度指令发布话题。 |
| **`topic_emg`** | string | 紧急停止状态订阅话题，用于安全监控。 |

!!! info "通信说明"
    路径驱动套件通过 `topic_odom` 获取机器人实时状态，经过算法处理后生成控制指令发布至 `topic_cmd_vel`，同时监听 `topic_emg` 以便在异常时及时中断运动。

---

## 2️⃣ PTP (Point-to-Point) 控制器参数

PTP 控制器用于点到点的精准移动，包含直线运动、直线过程中的姿态维持以及原地对准/旋转到位控制。

| 参数名称 | 单位 | 参数说明 |
| :--- | :---: | :--- |
| **`pid_param`** | - | PID 参数数组 `[P, I, D]`，决定响应速度与稳态精度。 |
| **`max_vel`** | m/s 或 rad/s | 运动过程中的最大速度约束。 |
| **`max_acc`** | m/s² 或 rad/s² | 最大加速度，影响启停平滑度。 |
| **`low_pass`** | - | 低通滤波系数 (0~1)，值越小滤波越强，滞后越大。 |
| **`ek`** | m 或 rad | 误差阈值，当误差小于该值时进入稳定周期判断。 |
| **`steady_clk`** | clk | 稳定周期数 (1clk=20ms)，连续满足阈值该周期数后判定运动完成。 |

!!! tip "PTP 调优建议"
    - `line_param`：控制直线行走的响应。若行走轨迹呈波浪状，可适当降低 P 值或减小 `low_pass`。
    - `roate_param_move`：直线运动中抵抗偏航的旋转微调参数，通常 P 值比 `line_param` 略高。
    - `roate_param`：原地旋转参数。若到位时震荡，降低 P 值或增加 D 值。

---

## 3️⃣ 差速追踪参数

差速追踪基于 Pure Pursuit（纯追踪）算法，适用于路径跟踪场景。

### 路径跟踪参数 (`pursuit_param`)

| 参数名称 | 单位 | 参数说明 |
| :--- | :---: | :--- |
| **`L`** | m | 机器人轴距。 |
| **`ek`** | m | 终点跟踪误差阈值。 |
| **`ld0`** | m | 前瞻距离最小值。 |
| **`kv`** | - | 速度比例系数，前瞻距离 `ld = kv * v + ld0`。 |
| **`ld_max`** | m | 前瞻距离最大值约束。 |

### 运动学控制参数
包含 `line_param`、`roate_param` 和 `roate_planner_param`，参数定义与 PTP 控制器一致，用于跟踪过程中的速度与姿态闭环。

### 航向与转弯控制

| 参数名称 | 单位 | 参数说明 |
| :--- | :---: | :--- |
| **`init_heading_thres`** | rad | 初始对准航向误差阈值。若偏差大于此值，优先原地旋转对准。 |
| **`kv_heading`** | - | 转弯减速系数 (≥1)。值越大，航向偏差越大时线速度衰减越剧烈。 |
| **`low_pass_heading`** | - | 航向角低通滤波系数，平滑航向反馈。 |

!!! warning "路径跟踪调优"
    前瞻距离 (`ld0` 和 `kv`) 是纯追踪算法的核心：增大 `ld0` 可提高高速行驶稳定性但会降低拐弯灵活性；减小 `ld0` 可提高跟踪精度但易引发震荡。

---

## 4️⃣ 基础运动参数

用于直接下达指令的非连续性基础位移操作。

| 模块 | 参数特点 |
| :--- | :--- |
| **`line_param`** | 直线行走，`max_vel` 默认 0.67 m/s。 |
| **`rotate_param`** | 原地旋转。注意单位为 **deg/s** 与 **deg**，与上层规划角度单位保持一致。 |

---

## 5️⃣ 连续运动驱动器 (全向轮 `path_driver_3wd`)

专用于三轮（或多轮）全向轮底盘的连续轨迹驱动。

| 模块 | 单位 | 参数说明 |
| :--- | :---: | :--- |
| **`line_param`** | m/s, m/s² | 全向直线运动 PID 及动力学约束。 |
| **`roate_param`** | rad/s, rad/s² | 全向移动过程中的自转控制。相比基础运动，具有更严格的误差阈值 (`ek: 0.0043633` rad)。 |

!!! tip "全向轮控制优化"
    全向轮由于存在辊子滑动，里程计精度通常低于差速底盘。建议在调参时适当放宽 `ek` 误差阈值，并增大 `low_pass` 以避免电机高频抖动。

---

## 6️⃣ 修改配置后的操作

完成 `param.yaml` 修改后，必须重启 ROS 节点或机器人主节点使参数生效：

```bash
# 重启机器人主节点（推荐）
sudo /home/vmx/WSR_HB_Robot/robot_manager.py restart
```

!!! success "验证配置"
    可以使用 `ros2 param list` 确认参数是否已加载，或发布简单的速度指令测试响应是否符合预期。

---

## ❓ 常见问题

| 现象 | 可能原因 | 解决方法 |
| :--- | :--- | :--- |
| **机器人走不直，左右摇摆** | PTP `roate_param_move` 的 P 值过大或 `low_pass` 过高 | 降低 P 值或减小 `low_pass` 滤波系数 |
| **路径跟踪切弯不及（外抛）** | `pursuit_param` 前瞻距离过大或速度过快 | 减小 `ld0` 或降低最大线速度 `max_vel` |
| **到达目标点后反复震荡** | `steady_clk` 设置过小或 `ek` 阈值过严 | 增大 `ek` 误差阈值或增加 `steady_clk` 周期数 |
| **全向轮横移时姿态偏转** | `path_driver_3wd` 旋转控制 P 值偏低 | 适当增大 `roate_param.pid_param` 的 P 值 |

!!! summary "总结"
    路径驱动套件参数繁多，建议从默认参数起步，先在空载环境下逐项测试 PTP 和追踪功能。重点把握 PID 响应、前瞻距离平滑器及加速度约束三者间的平衡。

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