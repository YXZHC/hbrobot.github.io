# 🔦 激光雷达 SDK 安装

> 本文档对应世界技能大赛自主移动机器人系列教程中 **高级应用** 模块的 **激光雷达 SDK 安装** 章节。


## 📌 概述

激光雷达（LiDAR，Light Detection and Ranging）是自主移动机器人实现**即时定位与地图构建**（SLAM）、**导航避障**和**环境感知**的核心传感器。YDLIDAR 系列雷达以其高性价比和稳定的性能，在机器人竞赛和教学场景中被广泛应用。

YDLIDAR 官方提供了跨平台的 **YDLidar-SDK**，该 SDK 封装了雷达数据采集、解析和控制的底层接口，是上层 ROS 驱动功能包的**基础依赖**。在安装 ROS 驱动包之前，必须**优先完成 SDK 的编译与安装**。

本文档将指导您完成 YDLidar-SDK 的完整安装流程。

> ⚠️ **前置条件**：请确保您的机器已经搭建好了基础的 ROS2 环境。


## 🛠️ 安装前准备

### 系统要求

| 项目 | 要求 |
|------|------|
| **操作系统** | Ubuntu 18.04 / 20.04 / 22.04（推荐） |
| **ROS 版本** | Melodic / Noetic / Humble |
| **CMake** | 2.8.2 或更高版本 |
| **编译器** | GCC 5.4+ |

### 安装依赖包

YDLidar-SDK 需要 CMake 和 pkg-config 作为编译依赖，请先执行以下命令安装：

```bash
sudo apt update
sudo apt install cmake pkg-config
```

> 💡 部分环境可能还需要 `libreadline-dev`、`libeigen3-dev` 等开发库，若后续编译报错可按需补充安装。


## 📥 步骤一：克隆 SDK 仓库

从 YDLIDAR 官方 GitHub 仓库克隆 SDK 源码：

```bash
git clone https://github.com/YDLIDAR/YDLidar-SDK.git
```

执行后，当前目录下会生成 `YDLidar-SDK` 文件夹。

> 🔗 官方仓库地址：[https://github.com/YDLIDAR/YDLidar-SDK.git](https://github.com/YDLIDAR/YDLidar-SDK.git)


## 🔨 步骤二：编译 SDK

进入 SDK 目录，创建 `build` 文件夹并执行 CMake 编译：

```bash
cd YDLidar-SDK
mkdir build
cd build
cmake ..
make
```

### 编译参数说明

| 命令 | 作用 |
|------|------|
| `mkdir build` | 创建独立的编译目录，保持源码整洁 |
| `cd build` | 进入编译目录 |
| `cmake ..` | 生成 Makefile，检测系统依赖 |
| `make` | 执行编译，生成库文件和可执行文件 |

> 🚀 **加速编译**：如需加快编译速度，可使用多核编译：`make -j4`（4 核）或 `make -j8`（8 核）。

### 编译输出

正常情况下，编译过程会在终端打印大量信息，最终显示 **100%** 完成，无 `error` 字样。


## 📦 步骤三：安装 SDK

编译完成后，执行以下命令将库文件安装到系统目录：

```bash
sudo make install
```

> 🔑 `sudo` 是必需的，因为安装过程会将库文件复制到 `/usr/local/lib` 等系统目录。

安装成功后，SDK 的动态库（如 `libydlidar_sdk.so`）和头文件将被部署到系统中，供后续 ROS 驱动包调用。


## ✅ 验证安装

### 方法一：运行测试程序

YDLidar-SDK 提供了测试程序 `tri_test`，可用于验证 SDK 是否安装成功：

```bash
cd ~/YDLidar-SDK/build
./tri_test
```

如果流程正确且雷达已正确连接，终端将打印雷达扫描数据。

### 方法二：检查库文件

```bash
ls -la /usr/local/lib | grep ydlidar
```

应能看到 `libydlidar_sdk.so` 等相关库文件。


## 📦 可选：打包为 DEB 安装包

YDLidar-SDK 支持使用 `cpack` 打包为 DEB 安装包，便于在多台机器上分发安装：

```bash
cd ~/YDLidar-SDK/build
cpack
# 如遇权限问题，可使用 sudo cpack
```

生成的 `.deb` 包可直接通过 `sudo dpkg -i` 安装。


## ⚠️ 常见问题与故障排除

???+ warning "Q1：`cmake ..` 报错 \"CMake 版本过低\"？"
    - **原因**：系统自带的 CMake 版本低于 2.8.2。
    - **解决方案**：升级 CMake。可通过 `sudo apt install cmake` 更新，或从 CMake 官网下载最新版本源码编译安装。

???+ warning "Q2：`make` 编译过程中出现 undefined reference 错误？"
    - **原因**：缺少必要的系统开发库依赖。
    - **解决方案**：安装缺失的依赖包，常见的有：
      ```bash
      sudo apt install libreadline-dev libeigen3-dev libsuitesparse-dev
      ```
      清理 build 目录后重新执行 `cmake .. && make`。

???+ tip "Q3：`sudo make install` 提示权限不足？"
    - **解决方案**：确保命令前加 `sudo`。若仍失败，检查 `/usr/local/lib` 目录的写入权限，或使用 `sudo chown` 调整目录归属。

???+ warning "Q4：运行 `./tri_test` 无法检测到雷达设备？"
    - **排查1**：检查雷达 USB 连接是否正常，执行 `lsusb` 查看是否有 YDLIDAR 设备。
    - **排查2**：确认 USB 串口权限，执行 `sudo chmod 666 /dev/ttyUSB*` 或将自己加入 `dialout` 组：
      ```bash
      sudo usermod -a -G dialout $USER
      ```
      然后**重新登录**生效。
    - **排查3**：重新插拔雷达 USB 线缆。

???+ tip "Q5：如何卸载 YDLidar-SDK？"
    - **解决方案**：进入 build 目录执行 `sudo make uninstall`（若支持）。若不支持，可手动删除相关库文件和头文件：
      ```bash
      sudo rm -rf /usr/local/lib/libydlidar_sdk*
      sudo rm -rf /usr/local/include/YDlidarSDK*
      ```

???+ tip "进阶扩展：如何指定安装路径？"
    - 在 `cmake` 阶段可通过 `-DCMAKE_INSTALL_PREFIX` 参数指定安装目录，例如：
      ```bash
      cmake .. -DCMAKE_INSTALL_PREFIX=/opt/ydlidar_sdk
      ```
      后续 `sudo make install` 会将文件安装到指定路径。使用时需在编译 ROS 驱动包时通过 `-DCMAKE_PREFIX_PATH` 指定该路径。


## 📚 参考资料

- [YDLidar-SDK 官方 GitHub 仓库](https://github.com/YDLIDAR/YDLidar-SDK.git)


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