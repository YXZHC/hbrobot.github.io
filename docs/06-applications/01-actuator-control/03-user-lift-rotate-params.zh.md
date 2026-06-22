# 用户举升与旋转参数配置

## 📖 概述

在机器人执行机构的实际控制中，软件接口发送的指令最终需要转换为底层电机驱动器能识别的脉冲信号。不同型号的丝杠导程、电机编码器线数以及传动比，会导致相同的脉冲数对应不同的物理位移。

本节教程将深入剖析 `wsc_robotic` SDK 中 `LiftMotor` 与 `RotateMotor` 类的核心参数配置，帮助您理解物理单位（`cm`/`deg`）与底层单位（`tick` 脉冲）之间的换算关系，确保在实际硬件更换或改装时能够正确修改参数。

## 📌 举升电机参数说明

`LiftMotor` 类主要负责将用户输入的距离（`cm`）转换为电机脉冲（`tick`）。其核心参数定义在类属性中。

### 核心参数表

| 参数名 | 默认值 | 单位 | 描述 |
| :--- | :---: | :---: | :--- |
| `max_distance` | `28.0` | cm | 举升机构允许的最大物理行程。超出此值将被限幅。 |
| `min_distance` | `0.0` | cm | 举升机构允许的最小物理行程。低于此值将被限幅。 |
| `cail_step` | `3000.0` | tick | 标定脉冲数，用于换算物理距离与脉冲的比例关系。 |
| `cail_dis` | `30.0` | cm | 标定距离，与 `cail_step` 配合使用。 |

### 换算原理解析

在控制指令下发前，SDK 会通过 `_proces_position` 方法进行限位检查与单位换算：

```python
# 限位处理
if distance > self.max_distance:
    distance = self.max_distance
if distance < self.min_distance:
    distance = self.min_distance
# 单位转换 cm->脉冲
coeff = self.cail_step / self.cail_dis * -1.0
count = distance * coeff
```

???+ tip "为什么换算系数 `coeff` 要乘以 `-1.0`？"
    电机正反转的方向取决于接线与驱动器配置。乘以 `-1.0` 是为了将**物理上的向上抬起**映射为**电机编码器的负向计数**（或反之）。如果发现电机运动方向与预期相反，可修改此处的符号，或调整电机驱动器端的接线。

## 🔄 旋转电机参数说明

`RotateMotor` 类负责将用户输入的角度（`deg`）转换为电机脉冲（`tick`）。

### 核心参数表

| 参数名 | 默认值 | 单位 | 描述 |
| :--- | :---: | :---: | :--- |
| `max_range` | `230.0` | deg | 旋转机构允许的最大旋转角度。 |
| `min_range` | `-90.0` | deg | 旋转机构允许的最小旋转角度。 |

### 换算原理解析

旋转电机的换算依赖于底层编码器的线数。在代码中，硬编码了编码器的分辨率：

```python
# 单位转换 角度->脉冲
enc_ppi = 1750.0 * 4.0  # 7000 tick/圈
count = angle * (enc_ppi / 360.0) * -1.0
```

- `enc_ppi`：编码器每圈脉冲数（Pulses Per Revolution）。此处 `1750.0 * 4.0` 表示编码器线数为 1750，经过 4 倍频处理后为 7000 脉冲/圈。
- `(enc_ppi / 360.0)`：计算出每度对应的理论脉冲数。

???+ warning "如何修改旋转电机编码器参数？"
    如果您更换了不同线数的电机，需要修改 `enc_ppi` 的计算公式。建议将 `enc_ppi` 提取为类属性（如 `enc_ppi = 7000.0`），方便统一管理与适配。

## 🛠️ 参数修改与适配指南

当您使用非原厂标配的电机或传动机构时，必须重新标定这些参数，否则会导致位置控制严重偏差甚至机械损坏。

### 步骤 1：确定机械限位
查阅机械图纸或手动推拉机构，测量其安全的物理行程边界，更新 `max_distance` / `min_distance` 或 `max_range` / `min_range`。

### 步骤 2：标定脉冲当量（举升机构）
1. 在安全环境下，控制电机缓慢运动。
2. 记录一段已知距离的运动（如运动 `10cm`）。
3. 读取底层驱动器反馈的脉冲差值（假设为 `1000 tick`）。
4. 则新的 `cail_dis = 10.0`，`cail_step = 1000.0`。

### 步骤 3：更新编码器线数（旋转机构）
查阅新电机数据手册，获取编码器线数 `PPR`。更新代码中的 `enc_ppi = PPR * 4.0`（绝大多数情况采用4倍频）。

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
