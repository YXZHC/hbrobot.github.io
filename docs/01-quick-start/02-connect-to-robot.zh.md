# 连接到机器人

本章节将详细介绍如何与机器人设备建立通信连接，并验证网络与 ROS 2 系统的运行状态。

我们将涵盖两种常用连接方式：**Wi‑Fi 无线网络**与**以太网有线连接**，包括默认网络参数、登录凭证及配置注意事项。同时，您还将学习如何使用命令行工具检测设备连通性，以及如何通过 `scp` 命令将机器人上的控制程序下载至本地进行开发。

完成本章后，您将能够：

- ✅ 成功通过无线或有线方式连接到机器人设备。
- ✅ 使用 `ping` 与 `ros2 topic list` 检查通信状态。
- ✅ 使用 `scp` 命令将机器人控制程序复制到本地进行开发与调试。

---

## 一、连接到机器人设备

机器人通常支持以下两种连接方式。请根据您的现场环境选择其一。

### 1. 通用连接信息

无论使用哪种方式连接，机器人的默认登录凭证如下：

| 项目 | 值 |
| :--- | :--- |
| **用户名** | `vmx` |
| **密码** | `password` |

> ⚠️ **注意**：SSH 与 SFTP 连接均使用上述凭证。

---

### 2. Wi‑Fi 无线网络连接

通过 Wi‑Fi 直连机器人热点是最便捷的连接方式。

#### 连接参数
| 参数 | 值 |
| :--- | :--- |
| **SSID** | `WSC_ROS2_Robot_HB01` |
| **Password** | `hb123456` |
| **IP 地址** | `10.12.34.2` |

#### 远程访问地址
- **SSH**: `ssh vmx@10.12.34.2`
- **SFTP**: `sftp://10.12.34.2/`

---

### 3. 以太网（网线）连接

使用网线连接可获得更稳定的通信质量，适用于大数据传输场景。

#### 连接参数
| 参数 | 值 |
| :--- | :--- |
| **IP 地址** | `192.168.123.3` |

#### 远程访问地址
- **SSH**: `ssh vmx@192.168.123.3`
- **SFTP**: `sftp://192.168.123.3/`

#### ⚠️ 重要配置
使用以太网连接前，**必须将电脑设置为静态 IP**，并确保其位于同一网段：

- **IP 地址**: `192.168.123.XXX` (例如 `192.168.123.16`)
- **子网掩码**: `255.255.255.0`

---

## 二、设备通讯检查

成功接入网络后，请按照以下步骤验证设备连通性及 ROS 2 运行状态。

### 1. 检查网络连通性 (Ping)

打开命令行终端，根据连接方式执行以下命令：

#### Wi‑Fi 连接检查
```bash
ping 10.12.34.2
```

#### 以太网连接检查
```bash
ping 192.168.123.3
```

**结果判定**：
- ✅ **正常**：终端持续显示回复时间（如 `64 bytes from ... time=xx ms`）。
- ❌ **异常**：显示 `Destination Host Unreachable` 或请求超时。请检查 IP 设置或物理连接。

### 2. 检查 ROS 2 运行状态

若网络连通，进一步检查 ROS 2 分布式通信是否正常：

```bash
ros2 topic list
```

**结果判定**：
- ✅ **正常**：列出当前机器人发布的 Topic 列表（如 `/cmd_vel`, `/scan` 等）。
- ❌ **异常**：无任何输出或报错。请检查机器人端 ROS 2 环境是否已启动，以及 `ROS_DOMAIN_ID` 是否与机器人一致。

---

## 三、文件传输（下载控制程序）

确认连接无误后，您可以使用 `scp` 命令将机器人上的代码复制到本地进行开发。

### 示例：下载控制程序
假设机器人上的控制程序位于 `/home/vmx/User_WSR_HB_Robot`，使用以下命令将其下载至本地当前目录：

```bash
# 语法：scp -r <用户名>@<机器人IP>:<远程路径> <本地路径>
scp -r vmx@10.12.34.2:/home/vmx/User_WSR_HB_Robot ./
```

> 💡 **提示**：如果使用以太网连接，请将 IP 替换为 `192.168.123.3`。

---

### 附录：常用命令速查表

| 功能 | 命令 |
| :--- | :--- |
| **Wi-Fi SSH 登录** | `ssh vmx@10.12.34.2` |
| **有线 SSH 登录** | `ssh vmx@192.168.123.3` |
| **Wi-Fi Ping 测试** | `ping 10.12.34.2` |
| **有线 Ping 测试** | `ping 192.168.123.3` |
| **查看 ROS 话题** | `ros2 topic list` |
| **下载文件夹** | `scp -r vmx@<IP>:/remote/path /local/path` |

---

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
            <a href="https://github.com/其他用户" style="text-decoration: none;">HBRobot</a>
        </div>
    </div>
</div>
---
🤝 **欢迎参与共建：**

[:fontawesome-brands-github: 提交 Issue](https://github.com/hbrobot/hbrobot.github.io/issues/new/choose){: .md-button }
[:octicons-git-pull-request-24: 提交 PR](https://github.com/hbrobot/hbrobot.github.io/compare){: .md-button .md-button--primary }