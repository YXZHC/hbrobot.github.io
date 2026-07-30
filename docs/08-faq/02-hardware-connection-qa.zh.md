# 硬件与连接 FAQ

本页面汇总 **机器人上电、网络连接、串口通信、传感器外设** 等硬件相关问题。

---

## Q1: 机器人上电后无任何反应？

**症状**：按下电源按钮后，指示灯不亮，风扇不转。

**排查步骤**：
1. **检查电池**：确认电池电量 > 20%，电压正常（用万用表测量输出端）
2. **检查保险丝**：查看底盘电源板上保险丝是否熔断
3. **检查急停开关**：确认急停按钮已旋出复位
4. **检查电源接线**：DC-DC 模块输入端是否有电压

**解决方案**：

- 电池欠压 → 充电后再试
- 保险丝熔断 → 更换同规格保险丝（切勿用铜丝替代）
- 急停未复位 → 顺时针旋转急停按钮

---

## Q2: 无法通过 Wi-Fi 连接到机器人？

**症状**：机器人已上电，但 `ping` 不通，SSH 连不上。

**原因与解决**：

| 可能原因 | 排查方法 | 解决方案 |
|---------|---------|---------|
| 机器人未开启热点 | 用手机搜索 Wi-Fi 列表 | 确认机器人 Wi-Fi 模块已启动 |
| IP 地址不对 | `arp -a` 查看局域网设备 | 确认机器人默认 IP（如 `192.168.1.100`） |
| 电脑防火墙拦截 | 临时关闭防火墙测试 | 添加防火墙规则放行 SSH 端口 |
| 多网卡路由冲突 | `route -n` 查看路由表 | 禁用多余网卡或指定路由 |

**快速诊断命令**：
```bash
# 扫描局域网内机器人
nmap -sn 192.168.1.0/24

# 测试端口连通性
nc -zv 192.168.1.100 22
```

---

## Q3: 串口设备权限不足 `/dev/ttyUSB0: Permission denied`？

**症状**：
```
serial.serialutil.SerialException: [Errno 13] could not open port /dev/ttyUSB0: [Errno 13] Permission denied
```

**原因**：当前用户不在 `dialout` 组中。

**解决方案**：
```bash
# 1. 将用户加入 dialout 组
sudo usermod -aG dialout $USER

# 2. 重新登录或重启生效
sudo reboot

# 3. 验证
groups $USER  # 应包含 dialout
```

> 如果仍有问题，检查 UDEV 规则是否正确配置（参见 [UDEV 规则配置](../07-advanced/system/01-udev-rule.zh.md)）。

---

## Q4: 激光雷达不发布数据 `/scan` 话题为空？

**症状**：`ros2 topic echo /scan` 无输出或报错。

**排查流程**：
```bash
# 1. 检查设备是否被识别
ls /dev/ | grep -i ttyUSB
ls /dev/ | grep -i ttyACM

# 2. 检查驱动节点是否运行
ros2 node list | grep lidar

# 3. 检查话题列表
ros2 topic list | grep scan

# 4. 查看驱动日志
ros2 topic echo /rosout | grep -i lidar
```

**常见原因**：
- 串口波特率不匹配 → 检查 launch 文件中的 `baud_rate` 参数
- UDEV 规则未生效 → 重新插拔后检查设备名
- 供电不足 → 确认雷达独立供电正常

---

## Q5: IMU 数据跳动大 / 数值异常？

**症状**：静止时 IMU 的加速度或角速度数据漂移严重。

**解决方案**：
1. **重新标定 IMU**：参照 [IMU 水平标定](../04-device-params/05-imu-calibration.zh.md)
2. **检查安装方向**：确认 IMU 安装平面水平，螺丝拧紧
3. **排除电磁干扰**：IMU 远离电机等强磁场源
4. **软件滤波**：在节点中添加低通滤波器

```bash
# 快速查看 IMU 数据
ros2 topic echo /imu/data --field linear_acceleration
```

---

## Q6: 多个 USB 设备插拔后设备名变化？

**症状**：`/dev/ttyUSB0` 和 `/dev/ttyUSB1` 顺序随机变化，导致启动文件找不到设备。

**原因**：Linux 按插入顺序分配设备名，不稳定。

**解决方案**：使用 UDEV 规则按 **USB 端口位置** 或 **串口芯片 ID** 固定设备名。

```bash
# 查看设备的串口芯片 ID
udevadm info --name=/dev/ttyUSB0 --attribute-walk | grep "idVendor\|idProduct\|serial"

# 创建规则文件 /etc/udev/rules.d/99-robot-serial.rules
# SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", SYMLINK+="lidar"
# SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", SYMLINK+="motor"
```

详细步骤见 [UDEV 规则配置](../07-advanced/system/01-udev-rule.zh.md)。

---

> 📌 **更多问题？** 查看 [报错排查速查表](./06-error-troubleshooting.zh.md)
