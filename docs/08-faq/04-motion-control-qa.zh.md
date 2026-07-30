# 运动控制 FAQ

本页面汇总 **底盘运动、舵机控制、电机参数、标定** 等运动控制相关问题。

---

## Q1: 底盘不运动，但节点无报错？

**症状**：发布速度指令后，轮子不转，终端无错误输出。

**排查流程**：
```bash
# 1. 确认指令话题有数据
ros2 topic echo /cmd_vel

# 2. 确认底盘节点在运行
ros2 node list | grep chassis

# 3. 检查底盘是否使能（E-Stop / 急停）
ros2 topic echo /chassis/status

# 4. 检查 CAN / 串口通信
ip link show can0          # CAN 总线状态
candump can0               # 查看 CAN 报文
```

**常见原因与解决**：

| 原因 | 解决方案 |
|------|---------|
| 急停未释放 | 旋出急停按钮，确认 `/chassis/status` 中 `e_stop=false` |
| CAN 总线未启动 | `sudo ip link set can0 up type can bitrate 500000` |
| 使能信号未发送 | 先发布使能指令 `ros2 topic pub /chassis/enable ...` |
| 通信超时 | 检查线缆连接，确认波特率/比特率匹配 |

---

## Q2: 底盘运动方向反了？

**症状**：发送前进指令，底盘后退；发送左转，底盘右转。

**原因**：电机接线极性与 URDF/参数文件中的方向定义不一致。

**解决方案**（二选一）：
1. **硬件端**：交换电机两相接线（A/B 相互换）
2. **软件端**：在参数文件中取反速度方向
   ```yaml
   # chassis_params.yaml
   left_wheel_multiplier: -1.0
   right_wheel_multiplier: -1.0
   ```

> ⚠️ 修改后需重新标定运动学参数，参见 [底盘运动学参数](../04-device-params/02-chassis-kinematics-params.zh.md)

---

## Q3: 舵机抖动 / 发出异响？

**症状**：舵机到达目标位置后持续抖动，或空载时发出"滋滋"声。

**原因与解决**：

| 原因 | 解决方案 |
|------|---------|
| 供电不足 | 单独给舵机供电（5V/2A 以上） |
| PID 参数过激 | 增大 D 或减小 P，增加死区 |
| 目标角度超出范围 | 检查角度限位参数，添加软件限位 |
| 信号干扰 | 加屏蔽线，远离电机动力线 |

**调试命令**：
```bash
# 查看当前舵机角度反馈
ros2 topic echo /servo/state

# 发送测试角度
ros2 topic pub /servo/cmd std_msgs/msg/Float32 "{data: 90.0}"
```

---

## Q4: OMS 电机报过流 / 过温？

**症状**：电机运行一段时间后自动停转，日志显示 `OverCurrent` 或 `OverTemperature`。

**解决方案**：
1. **降低负载**：检查机械结构是否卡滞
2. **降低加速度**：增大 `acc_lim` 参数，平滑启停
3. **改善散热**：加装散热片或风扇
4. **检查电流限制参数**：
   ```yaml
   # oms_motor_params.yaml
   max_current: 5.0        # 适当降低
   acceleration_limit: 2.0 # 增大（单位：rad/s²）
   ```

---

## Q5: 底盘走直线时偏航严重？

**症状**：发送 `linear.x` 速度，底盘明显偏离直线轨迹。

**排查与解决**：
1. **检查轮子机械对称性**：四个轮子是否都正常着地
2. **标定轮径和轴距**：重新测量并填入参数文件
   ```yaml
   wheel_radius: 0.065      # 实测值
   wheel_separation: 0.280  # 实测值（左右轮距）
   wheel_base: 0.320        # 实测值（前后轴距）
   ```
3. **启用 IMU 闭环校正**：在底盘控制器中融合 IMU 偏航角
4. **检查轮胎磨损**：单侧磨损会导致速度差

---

## Q6: 举升机构到位后不停（冲过头）？

**症状**：发送举升到目标高度，到达后继续运动一小段才停。

**原因**：减速距离（deceleration distance）参数过大或编码器分辨率不足。

**解决方案**：
```yaml
# lift_params.yaml
lift_max_speed: 0.1        # 降低最大速度
lift_acceleration: 0.2      # 降低加速度
lift_decel_distance: 0.01   # 减小减速距离（单位：m）
lift_position_tolerance: 0.002  # 位置容差（单位：m）
```

---

> 📌 **更多问题？** 查看 [报错排查速查表](./06-error-troubleshooting.zh.md) | 返回 [FAQ 总览](./00-faq-overview.zh.md)
