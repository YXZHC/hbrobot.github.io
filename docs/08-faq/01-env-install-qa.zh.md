# 环境与安装 FAQ

本页面汇总 **Ubuntu、ROS 2、驱动、Python 依赖** 等环境搭建过程中的常见问题。

---

## Q1: Ubuntu 22.04 安装 ROS 2 Humble 时 GPG 密钥报错？

**症状**：
```
Err:1 http://packages.ros.org/ros2/ubuntu jammy InRelease
  The following signatures couldn't be verified because the public key is not available
```

**原因**：ROS 2 官方 GPG 密钥未正确导入或已过期。

**解决方案**：
```bash
# 1. 重新下载并安装密钥
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# 2. 确认密钥文件存在
ls -la /usr/share/keyrings/ros-archive-keyring.gpg

# 3. 重新添加源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 4. 更新
sudo apt update
```

**验证**：`apt update` 不再出现 GPG 报错。

---

## Q2: `pip install` 时报 SSL 证书错误？

**症状**：
```
WARNING: Retrying (Retry(total=4, ...)) after connection broken by 'SSLError'
```

**原因**：系统时间不正确或企业代理拦截了 HTTPS 流量。

**解决方案**：
```bash
# 方法一：校准系统时间
sudo timedatectl set-ntp true
sudo timedatectl status

# 方法二：使用国内镜像源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <package-name>

# 方法三（临时绕过，不推荐生产环境）
pip install --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host files.pythonhosted.org <package-name>
```

---

## Q3: NVIDIA 驱动安装后 `nvidia-smi` 无输出？

**症状**：
```
$ nvidia-smi
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

**原因**：Secure Boot 阻止了未签名的内核模块加载。

**解决方案**：
1. 进入 BIOS → 关闭 **Secure Boot**
2. 重启后重新安装驱动：
   ```bash
   sudo apt purge nvidia-*
   sudo apt autoremove
   sudo reboot
   # 重启后
   sudo apt install nvidia-driver-535  # 或推荐版本
   sudo reboot
   ```
3. 验证：`nvidia-smi` 应显示 GPU 信息

---

## Q4: Docker 容器内无法访问 NVIDIA GPU？

**症状**：
```
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```

**原因**：`nvidia-container-toolkit` 未安装或未配置。

**解决方案**：
```bash
# 1. 安装 NVIDIA Container Toolkit
sudo apt install -y nvidia-container-toolkit

# 2. 配置 Docker
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# 3. 验证
docker run --rm --gpus all nvidia/cuda:12.0-base nvidia-smi
```

---

## Q5: ROS 2 工作空间 `colcon build` 报依赖缺失？

**症状**：
```
CMake Error: The following variables are used in this project, but they are set to NOTFOUND.
```

**原因**：`rosdep` 未执行或源未更新。

**解决方案**：
```bash
# 1. 确保 rosdep 已初始化
sudo rosdep init
rosdep update

# 2. 在工作空间根目录安装依赖
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y

# 3. 重新编译
colcon build --symlink-install
```

---

## Q6: VS Code 远程 SSH 连接机器人频繁断开？

**症状**：SSH 连接几分钟后自动断开，VS Code 提示 "Connection lost"。

**原因**：SSH 默认空闲超时 + 网络不稳定。

**解决方案**：
```bash
# 在本地 ~/.ssh/config 中添加：
Host robot
    HostName 192.168.x.x
    User your_username
    ServerAliveInterval 30
    ServerAliveCountMax 5
    TCPKeepAlive yes
```

---

> 📌 **更多问题？** 查看 [报错排查速查表](./06-error-troubleshooting.zh.md)
