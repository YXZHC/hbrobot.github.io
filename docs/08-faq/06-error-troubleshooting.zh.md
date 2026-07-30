# 报错排查速查表

本页面提供 **常见报错信息 → 可能原因 → 快速解决** 的速查参考。按模块分类，方便 Ctrl+F 搜索。

---

## 🔴 ROS 2 相关

| 报错信息（关键词） | 可能原因 | 快速解决 |
|-------------------|---------|---------|
| `Could not communicate with any discovery routes` | DDS 发现失败 | 检查 `ROS_DOMAIN_ID`，关闭防火墙 |
| `No executable found` | 包未编译或环境未 source | `colcon build` + `source install/setup.bash` |
| `Package 'xxx' not found` | 依赖未安装 | `rosdep install --from-paths src -r -y` |
| `Failed to load shared library` | 动态库路径未设置 | `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/lib` |
| `QoS policy mismatch` | 发布/订阅 QoS 不一致 | 统一 Durability / Reliability 设置 |
| `TF_OLD_DATA ignoring data` | 时间戳过期 | 检查 `use_sim_time` 设置 |
| `Lookup would require extrapolation` | TF 时间不同步 | 确认时钟源一致，启用 NTP |
| `ros2: command not found` | ROS 2 未安装或未 source | `source /opt/ros/humble/setup.bash` |

---

## 🟠 硬件与通信

| 报错信息（关键词） | 可能原因 | 快速解决 |
|-------------------|---------|---------|
| `Permission denied: /dev/ttyUSB0` | 用户无串口权限 | `sudo usermod -aG dialout $USER` + 重启 |
| `Device or resource busy` | 设备被占用 | `lsof /dev/ttyUSB0` 找进程并 kill |
| `Cannot open /dev/can0` | CAN 接口未启动 | `sudo ip link set can0 up type can bitrate 500000` |
| `Write to CAN bus failed` | 波特率不匹配 | 确认 CAN 比特率与硬件一致 |
| `I2C read failed` | I2C 地址冲突/接线错误 | `i2cdetect -y 1` 检查设备地址 |
| `USB disconnect` 反复出现 | 供电不足/接触不良 | 换 USB 口，加独立供电 |

---

## 🟡 Python 相关

| 报错信息（关键词） | 可能原因 | 快速解决 |
|-------------------|---------|---------|
| `ModuleNotFoundError: No module named 'xxx'` | 包未安装或环境错误 | `pip install xxx` 或确认在正确 venv 中 |
| `ImportError: libxxx.so: cannot open` | C++ 依赖缺失 | `sudo apt install libxxx-dev` |
| `cv2.error: (-215:Assertion failed)` | 图像为空/路径错误 | 检查文件路径，确认 `frame is not None` |
| `numpy.ndarray size changed` | NumPy 版本不兼容 | `pip install numpy==1.24.*` |
| `CUDA not available` | PyTorch 装了 CPU 版 | 重装 GPU 版 `pip install torch --index-url ...` |
| `Segmentation fault (core dumped)` | C 扩展崩溃/内存越界 | 用 `gdb python` 定位，检查数组越界 |

---

## 🟢 运动控制

| 报错信息（关键词） | 可能原因 | 快速解决 |
|-------------------|---------|---------|
| `Wheel odometry not updating` | 编码器无信号 | 检查编码器接线和供电 |
| `Motor overcurrent shutdown` | 堵转/负载过大 | 降低加速度，检查机械卡滞 |
| `CAN frame lost` | CAN 总线负载过高 | 降低发送频率，检查终端电阻 |
| `Chassis not responding to cmd_vel` | 急停未释放/未使能 | 确认 E-Stop 复位，发送使能指令 |
| `Servo angle out of range` | 目标角度超限 | 检查限位参数，添加 clamp |

---

## 🔵 视觉与 AI

| 报错信息（关键词） | 可能原因 | 快速解决 |
|-------------------|---------|---------|
| `Cannot open camera` | 设备节点不存在/被占用 | `ls /dev/video*` + `lsof /dev/video0` |
| `FourCC codec not found` | 编解码器缺失 | `sudo apt install ffmpeg` |
| `YOLO CUDA out of memory` | 显存不足 | 减小 batch_size 或 imgsz |
| `ONNX Runtime Error` | ONNX 版本不匹配 | 重新导出 ONNX，指定 `opset=12` |
| `MNN inference result all zeros` | 输入未归一化 | 检查预处理均值/标准差 |
| `cv2.VideoCapture.read() returns None` | 摄像头断开/超时 | 加重连逻辑，检查 USB 供电 |

---

## 🟣 系统与网络

| 报错信息（关键词） | 可能原因 | 快速解决 |
|-------------------|---------|---------|
| `Network is unreachable` | IP/网关配置错误 | `ip route show` 检查默认路由 |
| `Connection refused` | 目标端口未开放 | `ss -tlnp` 检查监听端口 |
| `ssh: connect to host timeout` | IP 错误/网络不通 | `ping` + `nmap -p 22` 排查 |
| `Disk full` | 磁盘空间不足 | `df -h` 清理日志/旧 bag 文件 |
| `Out of memory: Kill process` | 内存不足被 OOM Killer 杀掉 | `dmesg | grep "Killed"` 确认，加 Swap |

---

## 🔧 通用排查命令速查

```bash
# 系统状态
top / htop              # CPU & 内存
df -h                   # 磁盘空间
dmesg | tail -50        # 内核日志
journalctl -xe          # 系统日志

# 网络
ping <ip>               # 连通性
ss -tlnp                # 监听端口
tcpdump -i eth0         # 抓包
nmap -sn 192.168.1.0/24 # 扫描局域网

# ROS 2
ros2 node list          # 节点列表
ros2 topic list         # 话题列表
ros2 topic hz /xxx      # 话题频率
ros2 service list       # 服务列表
ros2 param list         # 参数列表
rqt_graph               # 计算图可视化

# 硬件
lsusb                   # USB 设备
ls /dev/tty*            # 串口设备
i2cdetect -y 1          # I2C 设备
ip link show can0       # CAN 状态
v4l2-ctl --list-devices # 视频设备

# GPU
nvidia-smi              # GPU 状态
nvidia-smi dmon         # GPU 实时监控
```

---

> 📌 如果以上未覆盖你的问题，请提交 Issue 并附上完整报错日志。返回 [FAQ 总览](./00-faq-overview.zh.md)
