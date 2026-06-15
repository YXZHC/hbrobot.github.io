# 📡 配置机器人无线热点（AP）

本文档介绍如何修改机器人自带无线热点的 **Wi-Fi 名称（SSID）** 和 **密码**，以便您自己的设备能够安全、稳定地连接。

> 📌 **适用场景**：机器人作为 AP（Access Point）发射 Wi-Fi 信号，电脑、手机等终端通过连接该热点与机器人通信。

---

## 🔧 准备工作

- 机器人已正常启动，AP 热点功能已安装并配置（默认系统已集成 `create_ap`）。
- 您有一台电脑（Windows / macOS / Linux），并已安装 SSH 客户端（如 OpenSSH、PuTTY）。

---

## 📶 1. 连接到机器人网络

在开始配置之前，您的电脑需要先连接到机器人的网络。有两种方式：

### 方式一：无线连接（推荐）

1. 打开电脑的 Wi-Fi 列表。
2. 找到机器人当前发射的热点名称（出厂默认：`WSC_ROS2_Robot_HB01`）。
3. 输入密码进行连接（出厂默认：`hb123456`）。

### 方式二：有线连接

使用网线直接将电脑的以太网口与机器人的网口（`eth0`）相连。

> [!NOTE]
> - 无线连接时，机器人的 AP IP 地址为：`10.12.34.2`
> - 有线连接时，机器人的有线网口 IP 地址为：`192.168.123.3`

---

## 🖥️ 2. SSH 登录到机器人

打开终端（Linux/macOS）或 PowerShell（Windows），执行对应连接方式的命令。

### 无线连接

```bash
ssh vmx@10.12.34.2
```

### 有线连接

```bash
ssh vmx@192.168.123.3
```

> [!NOTE]
> - **用户名**：`vmx`
> - **密码**：`password`

登录成功后，切换到 `root` 用户以获取配置文件修改权限：

```bash
sudo su
```

> [!NOTE]
> - `root` 密码与 `vmx` 用户密码相同：`password`

---

## ✏️ 3. 编辑 AP 配置文件

机器人的 AP 配置存储在 `/etc/create_ap.conf` 文件中。使用 `nano` 编辑器打开：

```bash
nano /etc/create_ap.conf
```

??? example "🔧 默认配置参数示例（`create_ap.conf`）"

    ```ini
    CHANNEL=default
    GATEWAY=10.12.34.2
    WPA_VERSION=2
    ETC_HOSTS=0
    DHCP_DNS=gateway
    NO_DNS=0
    NO_DNSMASQ=0
    HIDDEN=0
    MAC_FILTER=0
    MAC_FILTER_ACCEPT=/etc/hostapd/hostapd.accept
    ISOLATE_CLIENTS=0
    SHARE_METHOD=nat
    IEEE80211N=0
    IEEE80211AC=1
    HT_CAPAB=[HT40+]
    VHT_CAPAB=
    DRIVER=nl80211
    NO_VIRT=0
    COUNTRY=CN
    FREQ_BAND=5
    NEW_MACADDR=
    DAEMONIZE=0
    NO_HAVEGED=0
    WIFI_IFACE=wlan0
    INTERNET_IFACE=eth0
    SSID=WSC_ROS2_Robot_HB01
    PASSPHRASE=hb123456
    USE_PSK=0
    ```

在文件中找到以下两行（通常位于文件末尾附近）：

```ini
SSID=WSC_ROS2_Robot_HB01
PASSPHRASE=hb123456
```

> [!IMPORTANT]
> **nano 编辑器操作提示**：
>
> - 使用 **上下左右箭头** 移动光标。
> - 按 **退格键（Backspace）** 删除字符。
> - 修改完成后，按 `Ctrl + S` 保存文件。
> - 按 `Ctrl + X` 退出编辑器。

将 `SSID` 和 `PASSPHRASE` 的值修改为您自定义的名称和密码，例如：

```ini
SSID=MyRobot_Hotspot
PASSPHRASE=MySecurePwd123
```

> ⚠️ 注意：
> - SSID 和密码均不支持中文或特殊符号（建议仅使用字母、数字）。
> - 密码长度至少 8 位。

修改完成后，保存并退出。

---

## 🔄 4. 使配置生效

修改配置文件后，需要重启 AP 服务或重启机器人。

### 方法一：重启 AP 服务（推荐，无需重启机器人）

```bash
sudo systemctl restart create_ap
```

> 📌 重启服务后，热点会立即使用新的 SSID 和密码。

### 方法二：重启机器人

```bash
reboot
```

等待机器人启动完成（约 50 秒），即可用新的 SSID 和密码连接热点。

---

## 🛠️ 5. 热点服务管理（可选）

您可以使用 `systemctl` 命令控制热点服务的启动、停止、开机自启等行为。

| 操作             | 命令                                       |
| :--------------- | :----------------------------------------- |
| **开启热点**     | `sudo systemctl start create_ap`           |
| **关闭热点**     | `sudo systemctl stop create_ap`            |
| **设置开机自启** | `sudo systemctl enable create_ap.service`  |
| **关闭开机自启** | `sudo systemctl disable create_ap.service` |
| **查看服务状态** | `sudo systemctl status create_ap`          |

> 💡 **提示**：若您暂时不需要机器人发射热点，可以使用 `stop` 命令关闭；需要时再用 `start` 开启。

---

## ❓ 常见问题

### Q1：修改后无法连接到新热点？

- 检查电脑是否忘记旧的 Wi-Fi 网络，重新搜索并连接新 SSID。
- 确认密码输入正确（区分大小写）。
- 在机器人上执行 `sudo systemctl status create_ap` 查看服务是否正常运行。

### Q2：忘记自己设置的密码怎么办？

- 重新 SSH 登录机器人（通过有线连接或使用其他可用网络），再次编辑 `/etc/create_ap.conf` 查看或重置密码。

### Q3：如何恢复出厂设置？

- 将 `/etc/create_ap.conf` 中的 SSID 和 PASSPHRASE 改回默认值：
  ```ini
  SSID=WSC_ROS2_Robot_HB01
  PASSPHRASE=hb123456
  ```
  然后重启 AP 服务。

---

## 📚 附录：配置文件完整参数说明（参考）

`/etc/create_ap.conf` 中的其他参数一般无需修改，下表列出常用字段含义：

| 参数             | 说明                       | 示例值                |
| :--------------- | :------------------------- | :-------------------- |
| `SSID`           | Wi-Fi 热点名称             | `WSC_ROS2_Robot_HB01` |
| `PASSPHRASE`     | Wi-Fi 密码                 | `hb123456`            |
| `WIFI_IFACE`     | 无线网卡接口               | `wlan0`               |
| `INTERNET_IFACE` | 互联网上行接口             | `eth0`                |
| `CHANNEL`        | Wi-Fi 信道（default 自动） | `default`             |
| `FREQ_BAND`      | 频率波段（2.4 / 5）        | `5`                   |
| `COUNTRY`        | 国家代码（影响可用信道）   | `CN`                  |

> 📌 除非您熟悉无线网络配置，否则不建议修改上述参数。

---

✅ **配置完成**！现在您可以使用自定义的 Wi-Fi 名称和密码连接机器人热点，继续后续的开发与调试工作。

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