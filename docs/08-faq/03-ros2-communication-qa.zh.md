# ROS 2 与通信 FAQ

本页面汇总 **Topic / Service / Action 通信、DDS 配置、多机通信** 等 ROS 2 相关问题。

---

## Q1: `ros2 topic echo` 收不到数据，但节点在运行？

**症状**：节点已启动，`ros2 node list` 能看到，但 `ros2 topic echo /xxx` 无输出。

**排查步骤**：
```bash
# 1. 确认话题确实存在
ros2 topic list | grep xxx

# 2. 检查话题类型是否匹配
ros2 topic info /xxx

# 3. 检查消息类型
ros2 topic type /xxx

# 4. 检查 QoS 设置（最常见原因！）
ros2 topic echo /xxx --qos-durability transient_local
```

**常见原因与解决**：

| 原因 | 解决方案 |
|------|---------|
| QoS 不匹配（Durability/Reliability） | 发布端和订阅端使用相同的 QoS Profile |
| Domain ID 不一致 | 确认 `ROS_DOMAIN_ID` 环境变量一致 |
| 话题名称拼写/命名空间错误 | 用 `ros2 topic list` 确认完整名称 |

**QoS 速查**：
```python
# 发布者使用 transient_local 时，订阅者也必须用 transient_local
from rclpy.qos import QoSProfile, DurabilityPolicy
qos = QoSProfile(depth=10, durability=DurabilityPolicy.TRANSIENT_LOCAL)
```

---

## Q2: 多机通信不通（两台机器 ROS 2 无法互通）？

**症状**：机器人上能发布话题，但电脑端 `ros2 topic list` 看不到。

**排查清单**：
```bash
# 在两端分别检查
echo $ROS_DOMAIN_ID       # 必须相同！
echo $ROS_MASTER_URI      # ROS 2 不需要这个，确认未误设
ip addr show               # 确认 IP 地址
ping <对方IP>              # 基础网络连通性

# 检查 DDS 发现
ros2 multicast send
ros2 multicast receive    # 在另一端接收
```

**解决方案**：
1. **设置相同 Domain ID**：
   ```bash
   # 在两端 ~/.bashrc 中添加
   export ROS_DOMAIN_ID=42
   ```
2. **关闭防火墙**：
   ```bash
   sudo ufw disable
   ```
3. **检查 `FASTRTPS_DEFAULT_PROFILES_FILE`**：如果使用 FastDDS，确认配置文件正确
4. **使用 CycloneDDS**（推荐多机场景）：
   ```bash
   sudo apt install ros-humble-rmw-cyclonedds-cpp
   export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
   ```

---

## Q3: Service 调用超时无响应？

**症状**：`ros2 service call /xxx ...` 一直等待，最终超时。

**排查**：
```bash
# 1. 确认 Service 存在
ros2 service list | grep xxx
ros2 service type /xxx

# 2. 确认服务端节点存活
ros2 node info /node_name

# 3. 查看 Service 端日志
ros2 topic echo /rosout | grep -i service
```

**常见原因**：
- 服务端回调函数阻塞 → 确保回调中不做耗时操作
- 类型不匹配 → 用 `ros2 interface show <type>` 确认字段

---

## Q4: Launch 文件启动后节点立即退出？

**症状**：`ros2 launch xxx.launch.py` 启动后节点秒退。

**排查**：
```bash
# 查看详细日志
ros2 launch xxx.launch.py --debug

# 单独运行节点看报错
ros2 run <pkg> <node>
```

**常见原因**：
- 参数文件路径错误 → 用绝对路径或 `FindPackageShare`
- 依赖未安装 → `rosdep install` 检查
- Python 脚本无执行权限 → `chmod +x script.py`

---

## Q5: `ros2 bag record` 录制的数据回放没有反应？

**症状**：`ros2 bag play xxx` 执行成功，但订阅者收不到数据。

**解决方案**：
```bash
# 1. 检查 bag 内容
ros2 bag info xxx/

# 2. 确认回放的话题名称
ros2 bag play xxx/ --topics /target_topic

# 3. 如果发布端已不在，注意 QoS
ros2 bag play xxx/ --qos-profile-overrides-path qos.yaml
```

**QoS override 示例** (`qos.yaml`)：
```yaml
/topic_name:
  qos_durability: transient_local
```

---

## Q6: TF 变换报错 `Lookup would require extrapolation`？

**症状**：
```
Lookup would require extrapolation into the future/past.  
Requested time X but the latest data is at time Y.
```

**原因**：时间戳不同步，通常是因为没有使用 `use_sim_time` 或系统时间不一致。

**解决方案**：
```bash
# 确认 use_sim_time 参数
ros2 param get /node_name use_sim_time

# 如果用了仿真时间，所有节点都要设置
ros2 param set /node_name use_sim_time true

# 多机场景确保 NTP 时间同步
sudo timedatectl set-ntp true
```

---

> 📌 **更多问题？** 查看 [报错排查速查表](./06-error-troubleshooting.zh.md) | 返回 [FAQ 总览](./00-faq-overview.zh.md)
