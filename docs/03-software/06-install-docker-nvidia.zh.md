# Docker 与 NVIDIA Container Toolkit 安装指南
---
## 概述

本文档提供 Docker 引擎的基础安装步骤，以及 NVIDIA Container Toolkit（用于 GPU 加速容器）的配置流程，适用于需要在容器中使用 NVIDIA GPU 的场景（如 AI 训练、深度学习推理等）。

---

## 一、前置条件

### 1.1 操作系统

64 位 Linux 发行版（Ubuntu 22.04+ / Debian 10+ / CentOS 7+ / RHEL 7+）

### 1.2 硬件要求

| 组件 | 要求 |
|------|------|
| Docker | 无特殊硬件要求（支持虚拟化更佳） |
| NVIDIA Container Toolkit | 需配备 NVIDIA GPU（Compute Capability ≥ 3.5） |

### 1.3 依赖环境

- 已安装 NVIDIA 显卡驱动（版本 ≥ 450.80.02，推荐通过 [官方指南](https://www.nvidia.com/en-us/drivers/) 安装）
- 网络连接正常（用于下载安装包）

---

## 二、安装 Docker 引擎

### 2.1 方法 1：使用官方一键脚本（推荐）

1. 下载官方安装脚本
   ```shell
   curl -fsSL https://get.docker.com -o get-docker.sh
   ```
2. 执行脚本安装 Docker（需管理员权限）
   ```shell
   sudo sh get-docker.sh
   ```
3. （可选）将当前用户添加到 docker 组，避免每次使用 sudo
   ```shell
   sudo usermod -aG docker $USER
   ```
4. 重启终端或注销重新登录，使权限生效  { 临时刷新组权限（部分系统有效）}
   ```shell
   newgrp docker
   ```
5. 验证 Docker 安装成功
   ```shell
   docker --version
   ```
   

### 2.2 方法 2：手动分步安装

适用于需要自定义配置的场景，参考 Docker 官方文档：

- [Ubuntu / Debian](https://docs.docker.com/engine/install/ubuntu/)
- [CentOS / RHEL](https://docs.docker.com/engine/install/centos/)

---

## 三、安装 NVIDIA Container Toolkit

遵循 [NVIDIA 官方最新指南](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)（适配 Docker Engine ≥ 19.03）。

### 3.1 设置 NVIDIA 软件源

1. 更新源，安装必备组件
   ```shell
   sudo apt-get update && sudo apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      gnupg2
   ```
2. 配置生产存储库
   ```shell
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
     && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
       sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
       sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```
3. 从存储库中更新包列表
   ```shell
   sudo apt-get update
   ```
   

### 3.2 安装 NVIDIA Container Toolkit

安装NVIDIA容器工具包软件包

```shell
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.19.1-1
  sudo apt-get install -y \
      nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

### 3.3 配置 Docker

```shell
# 重启 Docker 服务使配置生效
sudo systemctl restart docker
```

---

## 四、基本使用操作

### 4.1 拉取镜像

镜像是容器的"模板"，需先拉取镜像才能运行容器。

```shell
# 拉取指定镜像（格式：docker pull 镜像名:标签）
docker pull ultralytics/ultralytics:latest

# 拉取镜像时指定仓库（国内加速，如毫秒镜像）
docker pull docker.1ms.run/ultralytics/ultralytics:latest
```

### 4.2 管理本地镜像

```shell
# 查看本地所有镜像
docker images
```

### 4.3 运行新容器

#### GPU 容器（需调用 NVIDIA 显卡）

必须添加 `--gpus` 参数指定 GPU 资源：

```shell
# 运行 CUDA 容器并进入交互模式（使用所有 GPU，不挂载目录）
sudo docker run -it --net=host --ipc=host --gpus all docker.1ms.run/ultralytics/ultralytics

# 在容器内验证 GPU 可用（执行 nvidia-smi）
root@<容器ID>:/# nvidia-smi  # 应显示与宿主机一致的 GPU 信息

# 运行容器（挂载本地代码目录）
sudo docker run -it --net=host --ipc=host \
  -v /home/user/UltralyticsData:/ultralytics/UltralyticsData \
  --gpus all docker.1ms.run/ultralytics/ultralytics
```

### 4.4 现有容器管理

| 操作 | 命令 |
|------|------|
| 查看正在运行的容器 | `docker ps` |
| 查看所有容器（包括已停止的） | `docker ps -a` |
| 停止运行中的容器 | `docker stop <容器ID/容器名>` |
| 启动已停止的容器 | `docker start <容器ID/容器名>` |
| 进入正在运行的容器 | `docker attach <容器ID/容器名>` |
| 删除已停止的容器 | `docker rm <容器ID/容器名>` |
| 强制删除运行中的容器 | `docker rm -f <容器ID/容器名>` |

### 4.5 文件拷贝操作（宿主机 ↔ 容器）

Docker 文件拷贝通过 `docker cp` 命令实现，支持**宿主机到容器**和**容器到宿主机**双向传输，无需进入容器即可操作。容器处于运行或停止状态均可（停止状态仅支持拷贝，无法写入动态文件）。

#### 4.5.1 宿主机 → 容器

**命令格式：**

```shell
docker cp <宿主机路径> <容器ID/容器名>:<容器内目标路径>
```

> **注意**：容器内目标路径若不存在，会自动创建（目录需确保路径层级完整）；若目标路径已存在文件，会覆盖同名文件。

```shell
# 示例 1：拷贝单个文件（如本地 Python 脚本传入 GPU 容器）
docker cp /home/user/code/train.py ultralytics:/tf_code/

# 示例 2：拷贝整个目录（如本地数据集传入容器）
# 拷贝后容器内会生成 /data/cifar10 目录及所有子文件
docker cp /home/user/datasets/cifar10 a3f2d7c9e1b0:/data/
```

#### 4.5.2 容器 → 宿主机

**命令格式：**

```shell
docker cp <容器ID/容器名>:<容器内路径> <宿主机目标路径>
```

> **注意**：宿主机目标路径若不存在，Docker 会自动创建；若目标路径已存在同名文件，会覆盖本地文件（建议先备份重要文件）。

```shell
# 示例 1：导出容器内日志文件（如训练日志）
docker cp a3f2d7c9e1b0:/tf_code/train.log /home/user/logs/

# 示例 2：导出容器内整个结果目录（如模型权重文件）
# 拷贝后本地会生成 /home/user/saved_models/models 目录及所有子文件
docker cp ultralytics:/tf_code/models/ /home/user/saved_models/
```

---

## 五、参考文档

| 文档 | 链接 |
|------|------|
| Docker 官方安装指南 | https://docs.docker.com/engine/install/ |
| NVIDIA Container Toolkit 官方文档 | https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html |
| NVIDIA 显卡驱动下载 | https://www.nvidia.com/download/index.aspx |

---

## 👥 贡献者

<a href="https://github.com/yxzhc"><img src="https://avatars.githubusercontent.com/u/80094007" width="70"></a>

<br>
本项目离不开每一位提交 PR、提 Issue、优化文档的开发者，由衷致谢！

![](../images/61d7f15b093ae.png)
