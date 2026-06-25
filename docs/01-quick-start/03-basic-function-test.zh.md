# 基本功能检测

## 1. 概述

为了避免因本地环境不完善而无法进行测试，本检测方案通过 **SSH** 直接连接到机器人，在机器人端运行多个示例程序，依次验证底盘急停、控制器电池电压、面板按键与指示灯等功能是否正常。后续可根据本流程再配置本地开发环境。

## 2. 连接信息

- **SSH**：`ssh vmx@10.12.34.2`
- **SFTP**：`sftp://10.12.34.2/`

> [!IMPORTANT]
> 1. SSH 与 SFTP 连接时登录用户名为：`vmx`，密码为：`password`
> 2. 所有操作均在机器人本体上执行，请确保机器人电源接通，网络可达。

## 3. 检测步骤

### 3.1 SSH 登录机器人

打开终端（Linux/macOS）或 PowerShell（Windows），执行：

```bash
ssh vmx@10.12.34.2
```

输入密码 `password` 后成功登录。

### 3.2 切换到 root 用户

```bash
sudo su
```

> ⚠️ 提示：执行 `sudo su` 需要输入当前用户 `vmx` 的密码，同样为 `password`。

### 3.3 进入工作空间

```bash
cd /home/vmx/User_WSR_HB_Robot/wsc_hbrobotv2_python/
```

### 3.4 加载环境变量

```bash
source /home/vmx/User_WSR_HB_Robot/wsc_hbrobotv2_python/set_vmx_env.sh
```

> 📌 说明：该脚本会设置 ROS 环境变量、Python 路径等，必须执行后才能正确运行示例程序。

### 3.5 运行底盘急停状态获取程序

```bash
python3 example/01-底盘控制范例/02-底盘急停状态获取.py
```

> [!IMPORTANT]
>
> 运行程序后，程序会通过ROS2系统读取设备返回数据，终端日志会提示我们**“请拍下急停!”**，我们可以按下急停按键，如果正常触发，则会反馈相应日志信息，并结束程序；

#### 预期行为与操作指引

1. 程序启动后，终端会输出提示信息：**“请拍下急停!”**。
2. 此时请手动按下机器人底盘上的**急停按钮**（红色大按钮）。
3. 若急停功能正常，程序将检测到状态变化，并在终端输出类似以下内容的日志：
   ```
   [INFO] 急停已触发，状态：按下
   [INFO] 程序正常退出
   ```
4. 程序自动结束。

---

### 3.6 控制器电池电压获取

```bash
python3 example/01-底盘控制范例/04-电池电压获取.py
```

> [!IMPORTANT]
> 运行程序后，程序会通过ROS2系统读取设备返回数据，终端日志将打印主控制器电池电压信息，读取 **10 次**后程序自动结束。

#### 预期输出示例

```
[INFO] 电池电压: 12.15 V
[INFO] 电池电压: 12.147 V
...
[INFO] 已读取10次，程序正常退出
```

---

### 3.7 面板按键与指示灯测试

```bash
python3 example/02-IO控制范例/02-按键与指示灯控制.py
```

> [!IMPORTANT]
> 1. 运行程序后，机器人上的 LED 指示灯会开始闪烁（频率约 1Hz）。
> 2. 约 5 秒后，终端会输出提示 **“请按下按键关闭程序.”**。
> 3. 此时请按下机器人上的**启动按键**（绿色或带有标识的物理按键）。
> 4. 按下按键后，程序检测到按键事件，LED 停止闪烁，程序正常退出。

#### 预期行为

- LED 周期性闪烁。
- 按下启动按键后，终端输出类似：
  ```
  [INFO] 检测到按键按下，程序退出
  ```
- LED 熄灭，程序结束。

---

### 3.8 底盘基础直线运动

```bash
python3 example/06-导航范例/02-基础直行.py
```

> [!IMPORTANT]
> 运行程序后，程序会通过运动学闭环控制系统，驱动机器人底盘直线行走1m，到达目标后会主动结束程序。

#### 预期行为

- 运行程序，建立连接
- 机器人直线行走1m
- 到达目标，程序结束

---


### 3.9 底盘基础旋转运动

```bash
python3 example/06-导航范例/01-基础旋转.py
```

> [!IMPORTANT]
> 运行程序后，程序会通过运动学闭环控制系统，驱动机器人底盘旋转180度，到达目标后会主动结束程序。

#### 预期行为

- 运行程序，建立连接
- 机器人原地旋转180度
- 到达目标，程序结束

---

## 4. 完整检测流程小结

建议按顺序执行以下检测：

1. **急停功能**（3.5）→ 确认急停响应正常
2. **电池电压**（3.6）→ 确认电源系统通信正常
3. **按键与指示灯**（3.7）→ 确认人机交互硬件正常
4. **基础直线运动** （3.8）→ 验证运动学模型配置
5. **基础旋转运动** （3.9）→ 验证IMU数据源

## 5. 常见问题及处理

| 现象 | 可能原因 | 解决方法 |
|------|----------|----------|
| SSH 连接超时 | IP 地址错误或网络不通 | 确认机器人 IP 为 `10.12.34.2`，且本机与机器人在同一网段 |
| `sudo su` 认证失败 | 密码错误 | 确认 `vmx` 用户密码为 `password` |
| 环境变量加载报错 | 文件路径不存在 | 使用 `ls` 检查 `/home/vmx/User_WSR_HB_Robot/wsc_hbrobotv2_python/` 是否存在 |
| Python 程序报错 `ModuleNotFoundError` | 未加载环境变量 | 重新执行 `source set_vmx_env.sh` |
| 急停程序无响应 | 急停信号未传递或程序未正确监听 | 检查急停按钮是否按下到位，可重启程序重试 |
| 电池电压程序无输出 | 通信异常或未加载驱动 | 确认机器人主控已上电，重试或查看系统日志 |
| 按键与指示灯程序 LED 不闪烁 | GPIO 权限或设备树问题 | 确保以 root 运行（`sudo su` 后执行），检查硬件连接 |

## 6. 检测结论

- **急停功能**：程序按预期输出日志并退出 → 正常  
- **电池电压**：连续输出 10 次电压值并退出 → 正常  
- **按键与指示灯**：LED 闪烁，按下按键后程序退出 → 正常  
- **基础直线运动**：正常直线运动1m → 正常 
- **基础旋转运动**：正常原地旋转180度 → 正常   

> ✅ 通过全部示例检测后，即可进行后续本地环境配置与更多功能测试。

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