# 🔦 激光雷达 ROS2 驱动

> 本文档对应世界技能大赛自主移动机器人系列教程中 **高级应用** 模块的 **激光雷达 ROS2 驱动** 章节。


## 📌 概述

在完成 YDLidar-SDK 的编译安装后，下一步是为激光雷达安装 **ROS 2 驱动包**。`ydlidar_ros2_driver` 是 YDLIDAR 官方推出的标准 ROS 2 驱动包，支持在 ROS 2 环境下驱动 YDLIDAR 全系列激光雷达设备。

本文档将指导您完成以下工作：

- ✅ 克隆 ROS 2 驱动源码
- ✅ 编译驱动包
- ✅ 配置雷达参数
- ✅ 运行驱动并验证数据

> ⚠️ **前置条件**：请确保已按上一章节完成 YDLidar-SDK 的编译与安装。若尚未安装 SDK，驱动编译将报错。


## 🛠️ 支持的 ROS 2 版本

| ROS 2 版本 | 支持情况 | 分支 |
|------------|---------|------|
| **Humble** | ✅ 官方支持 | `humble` |
| **Jazzy** | ✅ 官方支持 | `humble` |
| **Foxy** | ✅ 官方支持 | `humble` |
| **Galactic** | ✅ 官方支持 | `humble` |
| **Dashing** | ✅ 官方支持 | `master` |

> 💡 对于 Humble、Jazzy 等较新版本，统一使用 `humble` 分支；对于 Dashing 等旧版本，使用 `master` 分支。


## 📥 步骤一：创建工作空间

首先，创建一个用于存放 ROS 2 驱动包的工作空间：

```bash
mkdir -p ~/ydlidar_ros2_ws/src
cd ~/ydlidar_ros2_ws/src
```

> 💡 工作空间名称可自定义，本文统一使用 `ydlidar_ros2_ws` 便于后续说明。


## 📥 步骤二：克隆驱动源码

在 `src` 目录下克隆 YDLIDAR ROS2 驱动仓库。根据您的 ROS 2 版本选择对应的分支：

### Humble / Jazzy / Foxy / Galactic

```bash
git clone -b humble https://github.com/YDLIDAR/ydlidar_ros2_driver.git ydlidar_ros2_ws/src/ydlidar_ros2_driver
```

或进入 `src` 目录后执行：

```bash
cd ~/ydlidar_ros2_ws/src
git clone -b humble https://github.com/YDLIDAR/ydlidar_ros2_driver.git
```

### Dashing（旧版本）

```bash
git clone https://github.com/YDLIDAR/ydlidar_ros2_driver.git ydlidar_ros2_ws/src/ydlidar_ros2_driver
```

> 🔗 官方仓库地址：[https://github.com/YDLIDAR/ydlidar_ros2_driver](https://github.com/YDLIDAR/ydlidar_ros2_driver)


## 🔨 步骤三：编译驱动包

回到工作空间根目录，执行编译：

```bash
cd ~/ydlidar_ros2_ws
colcon build --symlink-install
```

### 编译参数说明

| 参数 | 作用 |
|------|------|
| `--symlink-install` | 使用符号链接安装，修改源码后无需重新编译即可生效，便于调试 |

> 🚀 **加速编译**：如需加快编译速度，可指定只编译特定包：
> ```bash
> colcon build --symlink-install --packages-select ydlidar_ros2_driver
> ```

### 常见编译错误

若编译时出现以下错误：

```
Could not find a package configuration file provided by "YDLidar-SDK"
```

说明 YDLidar-SDK 尚未安装或未正确安装，请返回上一章节完成 SDK 的编译与安装。


## 📦 步骤四：配置环境变量

编译完成后，需要将工作空间的环境变量添加到当前终端会话：

```bash
source ~/ydlidar_ros2_ws/install/setup.bash
```

### 永久生效（推荐）

将环境变量添加到 `~/.bashrc`，每次打开新终端时自动加载：

```bash
echo "source ~/ydlidar_ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 验证环境配置

执行以下命令确认工作空间路径已正确设置：

```bash
printenv | grep -i ROS
```

应能看到类似输出：

```
OLDPWD=/home/username/ydlidar_ros2_ws/install
```


## 🔌 步骤五：配置串口权限（可选）

为了让 ROS 2 驱动能够正常访问雷达的 USB 串口，需要配置串口权限。驱动包提供了便捷的初始化脚本：

```bash
cd ~/ydlidar_ros2_ws
chmod 0777 src/ydlidar_ros2_driver/startup/*
sudo sh src/ydlidar_ros2_driver/startup/initenv.sh
```

> ⚠️ 执行完成后，**请重新插拔雷达 USB 线缆**，使权限配置生效。

### 手动配置（备选）

若不使用脚本，也可手动添加串口权限：

```bash
sudo chmod 666 /dev/ttyUSB0
```

或将自己加入 `dialout` 组：

```bash
sudo usermod -a -G dialout $USER
```

然后**重新登录**生效。


## ⚙️ 步骤六：配置雷达参数

YDLIDAR ROS2 驱动的参数配置文件位于：

```
~/ydlidar_ros2_ws/src/ydlidar_ros2_driver/params/ydlidar.yaml
```

### 默认参数详解

以下是默认配置文件的核心参数：

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `port` | string | `/dev/ttyUSB0` | 串口设备或 IP 地址（如 `192.168.1.11`） |
| `frame_id` | string | `laser_frame` | TF 坐标系统名称 |
| `baudrate` | int | `230400` | 串口波特率或网络端口 |
| `lidar_type` | int | `1` | 雷达类型：0=TOF，1=三角测距，2=TOF_NET |
| `device_type` | int | `0` | 设备类型：0=串口，1=TCP，2=UDP |
| `ignore_array` | string | `""` | 角度过滤区间，如 `"-90,-80,30,40"` |
| `sample_rate` | int | `9` | 采样率 |
| `abnormal_check_count` | int | `4` | 异常启动数据尝试次数 |
| `fixed_resolution` | bool | `true` | 是否固定角度分辨率 |
| `reversion` | bool | `true` | 是否反转 |
| `inverted` | bool | `true` | 扫描方向：false=顺时针，true=逆时针 |
| `auto_reconnect` | bool | `true` | 是否自动重连（热插拔支持） |
| `isSingleChannel` | bool | `false` | 是否单通道雷达 |
| `intensity` | bool | `false` | 是否支持强度信息（G2 雷达设为 true） |
| `support_motor_dtr` | bool | `false` | 是否通过 DTR 控制电机启停 |
| `angle_min` | float | `-180.0` | 最小有效角度（度） |
| `angle_max` | float | `180.0` | 最大有效角度（度） |
| `range_min` | float | `0.01` | 最小有效距离（米） |
| `range_max` | float | `64.0` | 最大有效距离（米） |
| `frequency` | float | `10.0` | 扫描频率（Hz） |
| `invalid_range_is_inf` | bool | `false` | 无效距离是否设为 inf |

### 根据雷达型号修改参数

不同型号的 YDLIDAR 雷达对应不同的参数配置。以下为常见型号的参数参考：

| 型号 | lidar_type | baudrate | sample_rate | range_max | 特殊说明 |
|------|-----------|----------|-------------|-----------|---------|
| **F4** | 1 | 115200 | 4 | 12.0 | 电压 4.8~5.2V |
| **S4** | 4 | 115200 | 4 | 8.0 | 频率 5~12Hz (PWM) |
| **S4B** | 4/11 | 153600 | 4 | 8.0 | 强度 8bit |
| **G4** | 5 | 230400 | 9/8/4 | 16.0 | 常用型号 |
| **G4PRO** | 7 | 230400 | 9/8/4 | 16.0 | G4 升级版 |
| **X4** | 6 | 128000 | 5 | 10.0 | |
| **R2** | 9 | 230400 | 5 | 16.0 | |
| **G6** | 13 | 512000 | 18/16/8 | 25.0 | 远距离 |
| **G2** | 15 | 230400 | 5 | 16.0 | 强度 8bit |

### 修改配置文件示例

以 **G4 雷达** 为例，配置文件应设置为：

```yaml
ydlidar_ros2_driver_node:
  ros__parameters:
    port: /dev/ttyUSB0
    frame_id: laser_frame
    baudrate: 230400
    lidar_type: 5        # G4 对应 lidar_type = 5
    device_type: 0
    sample_rate: 9
    range_max: 16.0
    frequency: 10.0
    # ... 其他参数保持默认
```


## 🚀 步骤七：运行驱动

### 方式一：使用默认参数启动

```bash
ros2 launch ydlidar_ros2_driver ydlidar_launch.py
```

### 方式二：使用指定参数文件启动

```bash
ros2 launch ydlidar_ros2_driver ydlidar.py
```

### 方式三：同时启动 RVIZ 可视化

```bash
ros2 launch ydlidar_ros2_driver ydlidar_launch_view.py
```

此命令会同时启动雷达驱动和 RVIZ 可视化界面，方便实时查看点云数据。

### Launch 文件说明

| Launch 文件 | 功能说明 |
|-------------|---------------------|
| `ydlidar.py` | 使用默认参数连接雷达，发布 `/scan` 话题 |
| `ydlidar_launch.py` | 使用 `ydlidar.yaml` 配置参数连接雷达，发布 `/scan` 话题 |
| `ydlidar_launch_view.py` | 加载配置文件并启动 RVIZ 可视化 |


## 📊 验证数据

### 查看发布的话题

```bash
ros2 topic list | grep scan
```

应能看到 `/scan` 话题。

### 打印雷达数据

```bash
ros2 topic echo /scan
```

或使用驱动自带的客户端工具：

```bash
ros2 run ydlidar_ros2_driver ydlidar_ros2_driver_client
```

### 查看话题发布频率

```bash
ros2 topic hz /scan
```

正常情况下应显示与配置频率（如 10Hz）一致的发布频率。


## 🛑 控制雷达启停

驱动提供了两个服务，用于控制雷达的启动和停止：

| 服务名 | 类型 | 功能 |
|--------|------|------|
| `stop_scan` | `std_srvs/Empty` | 停止雷达扫描 |
| `start_scan` | `std_srvs/Empty` | 启动雷达扫描 |

调用示例：

```bash
# 停止扫描
ros2 service call /stop_scan std_srvs/Empty

# 启动扫描
ros2 service call /start_scan std_srvs/Empty
```


## ⚠️ 常见问题与故障排除

???+ warning "Q1：编译时报错 \"Could not find YDLidar-SDK\"？"
    - **原因**：YDLidar-SDK 未安装或未正确安装。
    - **解决方案**：返回上一章节完成 YDLidar-SDK 的编译与安装（`sudo make install`）。安装完成后，清理 build 目录重新编译：
      ```bash
      cd ~/ydlidar_ros2_ws
      rm -rf build install log
      colcon build --symlink-install
      ```

???+ warning "Q2：运行 launch 文件后提示 \"Cannot open serial port\"？"
    - **排查1**：检查雷达 USB 是否正确连接，执行 `lsusb` 查看是否有 YDLIDAR 设备。
    - **排查2**：检查串口设备名是否正确，执行 `ls /dev/ttyUSB*` 确认实际设备名。
    - **排查3**：检查串口权限，执行 `sudo chmod 666 /dev/ttyUSB0` 或运行初始化脚本。
    - **排查4**：确认没有其他程序占用该串口（如 `screen`、`minicom` 等）。

???+ tip "Q3：雷达已连接但 `/scan` 话题无数据？"
    - **排查1**：检查配置文件中的 `lidar_type` 和 `baudrate` 是否与您的雷达型号匹配。
    - **排查2**：确认 `angle_min` 和 `angle_max` 设置正确，若设置不当可能导致所有点被过滤。
    - **排查3**：检查 `range_min` 和 `range_max` 是否覆盖了实际测量范围。
    - **排查4**：尝试调用 `start_scan` 服务启动扫描：
      ```bash
      ros2 service call /start_scan std_srvs/Empty
      ```

???+ tip "Q4：如何在多台雷达情况下指定不同的串口？"
    - 修改配置文件中的 `port` 参数为对应的串口设备，如 `/dev/ttyUSB1`。
    - 也可通过创建 udev 规则为不同雷达绑定固定别名，避免设备名变化。

???+ tip "Q5：如何将雷达数据用于 SLAM 建图？"
    - 确认 `/scan` 话题正常发布后，可直接启动 SLAM 算法包（如 `slam_toolbox` 或 `cartographer`），在配置文件中将激光雷达话题设置为 `/scan`。
    - 确保 `frame_id`（默认为 `laser_frame`）与 TF 树中的坐标系名称一致。

???+ tip "进阶扩展：如何自定义雷达的 TF 坐标系？"
    - 修改配置文件中的 `frame_id` 参数，例如改为 `lidar_link`。
    - 在 URDF 或 TF 静态变换中发布从 `base_link` 到 `lidar_link` 的坐标变换，确保机器人定位系统能正确关联雷达数据。


## 📚 参考资料

- [YDLIDAR ROS2 Driver GitHub 仓库](https://github.com/YDLIDAR/ydlidar_ros2_driver)
- [YDLIDAR ROS2 Driver 参数详解](https://github.com/YDLIDAR/ydlidar_ros2_driver/blob/master/details.md)

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