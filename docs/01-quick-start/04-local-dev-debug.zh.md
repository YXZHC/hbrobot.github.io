# 本地开发与调试

本章节将指导您如何将机器人的控制程序下载到本地电脑，配置 VSCode 开发环境，并直接在本地运行各个轴向的运动示例程序进行调试。通过本地开发，您可以更高效地编写、修改和测试代码。

完成本章后，您将能够：

- ✅ 使用 `scp` 命令将机器人控制程序下载至本地。
- ✅ 配置 VSCode 环境，实现一键加载运行环境。
- ✅ 在本地运行基础 IO 与底盘各轴向运动控制示例，验证功能。

---

## 一、环境准备与程序下载

### 1.1 下载机器人控制程序

机器人配套程序存储在机器人内部，我们可以连接到机器人网络，并将其下载到我们电脑当中。参考如下示例，使用 SCP 远程下载程序，过程中根据提示输入确认信息。

#### 1.1.1 Wi-Fi 连接下载

```shell
scp -r vmx@10.12.34.2:/home/vmx/User_WSR_HB_Robot ./
```

#### 1.1.2 有线连接下载

```shell
scp -r vmx@192.168.123.3:/home/vmx/User_WSR_HB_Robot ./
```

> [!NOTE]
> SCP 连接时登陆用户名为：`vmx`    密码为：`password` 🛡️

程序下载成功后，可以进入程序目录并使用 VSCode 编辑器打开，参考快捷指令如下：

```shell
cd ./User_WSR_HB_Robot/wsc_hbrobotv2_python/
code ./
```

### 1.2 配置编辑器启动脚本

> [!WARNING]
> 部分系统镜像文件中缺少 `.vscode` 文件，需要手动添加启动 `.vscode` 脚本；请参考 A1 小节步骤。⚠️

#### A1、添加编辑器启动脚本

在项目根目录创建 `.vscode` 文件夹，并在其中新建 `settings.json` 文件。

> [!CAUTION]
> 注意文件夹名称前的点号 `.`，这是 Linux 下的隐藏文件夹！完整名称应为“ `.vscode` ”！！！🚨

**文件结构：**
```
wsc_hbrobotv2_python/
├── .vscode/
│   └── settings.json   👈 新建这个
├── example/
├── ...
└── set_vmx_env.sh
```
**`.vscode/settings.json` 内容：**
```json
{
    "terminal.integrated.profiles.linux": {
        "bash": {
            "path": "/bin/bash",
            "args": ["-c", "source ./set_env.sh && exec bash"]
        }
    },
    "terminal.integrated.defaultProfile.linux": "bash"
}
```

---

## 二、基本示例程序验证

在本小节我们将连接到机器人设备，并在本地 VSCode 中进行基本 IO 与状态检查。请确保您的电脑已连接到机器人的 Wi-Fi 或有线网络。🌐

### 2.1 急停按键基本测试 🛑

在编辑器中打开 `example/01-底盘控制范例` 目录中的 `02-底盘急停状态获取.py` 示例程序；点击编辑器右上角运行箭头 ▶️，运行程序。

> [!IMPORTANT]
> 运行程序后程序将会通过分布式系统读取设备返回数据，终端日志会提示我们 **“请拍下急停!”**。此时请手动按下机器人底盘上的急停按键，如果正常触发，则会反馈相应日志信息，并结束程序。

### 2.2 控制器电池电压获取 🔋

在编辑器中打开 `example/01-底盘控制范例` 目录中的 `04-电池电压获取.py` 示例程序；点击编辑器右上角运行箭头 ▶️，运行程序。

> [!IMPORTANT]
> 运行程序后程序将会通过分布式系统读取设备返回数据，终端日志将打印主控制器电池电压信息，读取 **10 次**后程序将主动结束。

### 2.3 面板按键与指示灯测试 💡

在编辑器中打开 `example/02-IO控制范例` 目录中的 `02-按键与指示灯控制.py` 示例程序；点击编辑器右上角运行箭头 ▶️，运行程序。

> [!IMPORTANT]
> 运行程序后程序将会通过分布式系统向机器人设备发送 LED 闪烁指令。约 5 秒后，日志输出窗口会显示 **“请按下按键关闭程序.”**。我们需要按下机器人上的启动按键，按下后程序结束。

---

## 三、底盘运动控制示例 (各轴向验证) 🚗

为了确保机器人的运动学模型与底盘配置正确，我们需要在本地运行底盘运动控制示例，全面验证各个轴向的运动情况。请在本地 VSCode 中打开并运行以下示例。

### 3.1 X轴：基础直线运动 (前进/后退) ⬆️⬇️

在编辑器中打开 `example/06-导航范例` 目录中的 `02-基础直行.py` 示例程序；点击运行箭头 ▶️。

> [!IMPORTANT]
> 运行程序后，程序会通过运动学闭环控制系统，驱动机器人底盘沿 **X 轴正向**直线行走 1m。到达目标后程序会主动结束。

**预期行为**：

- 机器人平稳向前移动 1 米。
- 到达目标位置后停止，终端显示程序正常退出。

> [!TIP]
> 💡 **扩充测试**：您可以在代码中修改目标距离参数为负值（如 `-1.0`），重新运行程序以测试机器人的 **X 轴后退** 运动。

### 3.2 Z轴：基础旋转运动 (左转/右转) 🔄

在编辑器中打开 `example/06-导航范例` 目录中的 `01-基础旋转.py` 示例程序；点击运行箭头 ▶️。

> [!IMPORTANT]
> 运行程序后，程序会通过运动学闭环控制系统，驱动机器人底盘绕 **Z 轴**原地旋转 180 度。到达目标后程序会主动结束。

**预期行为**：

- 机器人原地旋转 180 度。
- 旋转完成后停止，终端显示程序正常退出。

> [!TIP]
> 💡 **扩充测试**：修改代码中的目标角度参数，设置为 `90` 度可测试逆时针旋转，设置为 `-90` 度可测试顺时针旋转。


## 四、常见问题及处理 🛠️

| 现象 | 可能原因 | 解决方法 |
|------|----------|----------|
| 本地运行无法连接机器人 | 网络未连通或 `ROS_DOMAIN_ID` 不一致 | 检查 Wi-Fi/有线连接，确保本地与机器人在同一网段且 `ROS_DOMAIN_ID` 一致 |
| VSCode 终端报错 `ModuleNotFoundError` | 未正确加载环境变量 | 检查 `.vscode/settings.json` 配置，确保终端启动时执行了 `source ./set_env.sh` |
| 运动控制程序无响应 | 机器人底盘未上电或急停未释放 | 确认底盘电源已开启，急停按钮处于弹起状态 |
| SCP 下载失败 | IP 地址错误或网络不稳定 | 确认机器人 IP 地址，检查网络连接后重试 |

---

> ✅ 通过本地环境配置与各轴向示例验证后，您就可以开始自定义您的机器人控制程序了！🚀

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