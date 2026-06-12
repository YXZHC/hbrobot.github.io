# Python 依赖库与 pip 使用指南
---
`pip` 是 Python 官方包管理工具，负责 Python 第三方包的**查找、下载、安装、卸载**。目前从 Python 官网下载的新版安装包，均已默认内置 `pip`。

pip 官网：https://pypi.org/project/pip/

---

## 一、pip 检查与安装
### 1. 检查是否已安装
根据 Python 版本执行对应命令，查看 pip 版本：
```shell
# Python 2.x 版本
pip --version

# Python 3.x 版本（主流推荐）
pip3 --version
```

### 2. 手动安装 pip（Debian/Ubuntu 系统）
若系统未预装 pip，可通过系统包管理器安装：
```shell
sudo apt install python3-pip
```

---

## 二、Python 虚拟环境（venv）
使用虚拟环境可**隔离不同项目的依赖包**，避免版本冲突，以下为 Linux 环境常用操作命令。

### 1. 安装 venv 模块
部分精简系统未自带虚拟环境工具，需先安装（示例为 Python3.10）：
```shell
apt install python3.10-venv
```

### 2. 创建虚拟环境
在当前目录创建名为 `my_venv` 的虚拟环境：
```shell
python3 -m venv my_venv
```

### 3. 激活虚拟环境
Linux / macOS 系统激活命令：
```shell
source ./my_venv/bin/activate
```
激活成功后，终端前缀会显示虚拟环境名称，此后使用 `pip` 安装的包仅作用于当前环境。

---

## 三、第三方依赖库安装
> [!TIP]
> 国内网络访问官方源速度较慢，可使用**国内镜像站**加速下载。使用方式：在原安装命令后追加 `-i 镜像地址` 即可。

### 3.1 OpenCV-Python
OpenCV-Python 是 OpenCV 的 Python 接口，跨平台支持图像、视频处理，底层基于 C++ 实现，性能优异。

#### 常规安装
```shell
sudo pip install opencv-python==4.5.5
```

#### 中科大镜像加速安装
```shell
sudo pip install opencv-python==4.5.5 -i https://mirrors.ustc.edu.cn/pypi/simple/
```

### 3.2 NumPy
NumPy 是 Python 高性能数值计算库，核心用于多维数组运算、矩阵计算，是数据分析、人工智能项目的基础依赖。

#### 常规安装
```shell
sudo pip install numpy==1.26.4
```

#### 中科大镜像加速安装
```shell
sudo pip install numpy==1.26.4 -i https://mirrors.ustc.edu.cn/pypi/simple/
```

### 3.3 MNN
MNN 是阿里开源的全平台轻量级高性能深度学习推理引擎，广泛应用于计算机视觉、语音识别、自然语言处理等 AI 场景。

#### 常规安装
```shell
sudo pip install mnn
```

#### 中科大镜像加速安装
```shell
sudo pip install mnn -i https://mirrors.ustc.edu.cn/pypi/simple/
```

### 3.4 其他依赖库（通用模板）
将下方命令中 `xxx` 替换为实际包名/指定版本即可：
#### 常规安装
```shell
sudo pip install xxx
```

#### 中科大镜像加速安装
```shell
sudo pip install xxx -i https://mirrors.ustc.edu.cn/pypi/simple/
```

---

## 四、国内镜像源加速（通用）
受网络影响，国内使用 pip 安装包时建议添加国内镜像源加速，格式如下：
```shell
pip3 install [包名]==[版本号] -i [镜像地址]
```
常用国内 PyPI 镜像源：

| 镜像源名称 | 镜像地址 |
|---|---|
| 中国科学技术大学 | https://mirrors.ustc.edu.cn/pypi/simple/ |
| 清华大学 | https://pypi.tuna.tsinghua.edu.cn/simple |
| 阿里云 | http://mirrors.aliyun.com/pypi/simple/ |
| 华为云 | https://repo.huaweicloud.com/repository/pypi/simple/ |
| 腾讯云 | https://mirrors.cloud.tencent.com/pypi/simple/ |
| 上海交通大学 | https://mirror.sjtu.edu.cn/pypi/web/simple/ |


## 补充：全局配置镜像（永久生效）
若想每次 `pip` 都默认使用镜像，无需每次追加参数，可配置全局镜像（以清华源为例）：
```shell
# Linux/Mac 临时配置（当前用户）
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

> [!TIP]
> 1. 所有命令中 `pip` 可替换为 `pip3`（推荐，避免与 Python2 冲突）；
> 2. 安装时若提示权限不足，可保留 `sudo` 或添加 `--user` 参数（仅当前用户安装）；
> 3. 虚拟环境中安装时，无需添加 `sudo`。

---

## 👥 贡献者

<a href="https://github.com/yxzhc"><img src="https://avatars.githubusercontent.com/u/80094007" width="70"></a>

<br>
本项目离不开每一位提交 PR、提 Issue、优化文档的开发者，由衷致谢！

![](../images/61d7f15b093ae.png)
