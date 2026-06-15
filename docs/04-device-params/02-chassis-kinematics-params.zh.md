# 🚀 底盘运动学模型配置指南

本文档详细介绍如何根据机器人底盘类型（两轮差速、四轮差速、麦克纳姆轮、全向轮等）正确配置运动学参数，并切换对应的配置文件。

---

## 📁 配置文件概览

底盘运动学参数存储在独立 YAML 文件中，启动时由 `chassis_node` 加载。所有配置文件位于：

```bash
/home/vmx/WSR_HB_Robot/install/chassis/share/chassis/config/
```

| 底盘类型 | 配置文件 | 运动学模型 |
| :--- | :--- | :--- |
| **默认两轮差速** | [`default_config.yaml`](./res/chassis/default_config.yaml) | 两轮差速（左右轮独立） |
| **四轮差速（标准）** | [`diff_drive_4wd_config.yaml`](./res/chassis/diff_drive_4wd_config.yaml) | 四轮同向差速 |
| **四轮差速（全地形）** | [`diff_drive_4wd_at_config.yaml`](./res/chassis/diff_drive_4wd_at_config.yaml) | 四轮独立驱动，适应复杂地形 |
| **四轮麦克纳姆轮** | [`mecanum_4wd_config.yaml`](./res/chassis/mecanum_4wd_config.yaml) | 麦克纳姆轮全向移动 |
| **X 型四轮全向轮** | [`x_omni_drive_config.yaml`](./res/chassis/x_omni_drive_config.yaml) | X 形布置全向轮 |


> 📌 **📄 点击查看各底盘配置文件示例**：

> - ??? example "🔧 默认两轮差速（`default_config.yaml`）"
    ```yaml
    # titan使能服务话题
    titan_enable_topic: "/vmxpi/titan_quad0/enable"
    # VMXPI数据话题
    vmxpi_data_topic: "/vmxpi/vmxpi_datas"

    #底盘配置参数
    chassis_param:
      ctrl_rate: 50                 # 控制频率Hz
      fuse_imu_odom: True           # 里程计融合imu
      invert_imu: True              # 反转IMU角速度
    
      #里程计线速度校正系数
      correct_velx_coeff: 1.0       # x轴系数
      correct_vely_coeff: 1.0       # y轴系数
      correct_velw_coeff: 1.0       # w轴系数

      #软急停IO配置(VMXPI DI口)[di0, di1]
      enable_emg: True              # 使能硬件EMG
      emg_io: 0                     # vmx di0


    ##运动学模型参数##
    # name:轮子名称
    # radius:轮子半径(m)
    # alpha:轮子位于局部参考坐标系角度(deg)
    # gear_ratio:齿轮比
    # set_topic:轮子设置设置电机转速topic
    # feedback_topic:轮子电机反馈topic
    wheel_names: ['wheel0', 'wheel1']

    kinematics_model:
      wheel0:
        radius: 0.0675
        alpha: 90.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor0'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder0'
      wheel1:
        radius: 0.0675
        alpha: -90.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor1'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder1'

    ##动力学参数##
    # max_vel最大速度约束
    # max_acc最大加速度(注:越大响应越快,超调越来大; 越小滞后越大,速度平滑)
    # lowpass_coeff低通滤波系数(注: 越小滞后越大,加速度拐点越平滑, 到达稳态越平滑; 越大响应快,加速度拐点逼近梯形)
    dynamic_param:
      #最大线速度[x, y, w]      [m/s, m/s, rad/s]
      max_velx: 0.65
      max_vely: 0.0
      max_velw: 3.5
      #最大加速度[x, y, w]      [m/s^2, m/s^2,  rad/s^2]
      max_accx: 2.0
      max_accy: 2.0
      max_accw: 15.0
      #速度低通滤波系数 u(k) = (1 - coeff) * u(k-1) + coeff * u(k)
      lowpass_coeffx: 0.2
      lowpass_coeffy: 0.2
      lowpass_coeffw: 0.3
    ```

> - ??? example "🔧 四轮差速（标准）（`diff_drive_4wd_config.yaml`）"
    ```yaml
    # titan使能服务话题
    titan_enable_topic: "/vmxpi/titan_quad0/enable"
    # VMXPI数据话题
    vmxpi_data_topic: "/vmxpi/vmxpi_datas"

    #底盘配置参数
    chassis_param:
      ctrl_rate: 25                 # 控制频率Hz
      fuse_imu_odom: True           # 里程计融合imu
      invert_imu: True              # 反转IMU角速度
    
      #里程计线速度校正系数
      correct_velx_coeff: 1.0       # x轴系数
      correct_vely_coeff: 1.0       # y轴系数
      correct_velw_coeff: 1.0       # w轴系数

      #软急停IO配置(VMXPI DI口)[di0, di1]
      enable_emg: True              # 使能硬件EMG
      emg_io: 0                     # vmx di0


    ##运动学模型参数##
    # name:轮子名称
    # radius:轮子半径(m)
    # alpha:轮子位于局部参考坐标系角度(deg)
    # gear_ratio:齿轮比
    # set_topic:轮子设置设置电机转速topic
    # feedback_topic:轮子电机反馈topic
    wheel_names: ['wheel0', 'wheel1', 'wheel2', 'wheel3']

    kinematics_model:
      wheel0:
        radius: 0.06085
        alpha: 90.0
        wheel_base: 0.175
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor0'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder0'
      wheel1:
        radius: 0.06085
        alpha: -90.0
        wheel_base: 0.175
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor1'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder1'
      wheel2:
        radius: 0.06085
        alpha: -90.0
        wheel_base: 0.175
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor2'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder2'
      wheel3:
        radius: 0.06085
        alpha: 90.0
        wheel_base: 0.175
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor3'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder3'

    ##动力学参数##
    # max_vel最大速度约束
    # max_acc最大加速度(注:越大响应越快,超调越来大; 越小滞后越大,速度平滑)
    # lowpass_coeff低通滤波系数(注: 越小滞后越大,加速度拐点越平滑, 到达稳态越平滑; 越大响应快,加速度拐点逼近梯形)
    dynamic_param:
      #最大线速度[x, y, w]      [m/s, m/s, rad/s]
      max_velx: 0.65
      max_vely: 0.0
      max_velw: 3.5
      #最大加速度[x, y, w]      [m/s^2, m/s^2,  rad/s^2]
      max_accx: 2.0
      max_accy: 2.0
      max_accw: 15.0
      #速度低通滤波系数 u(k) = (1 - coeff) * u(k-1) + coeff * u(k)
      lowpass_coeffx: 0.2
      lowpass_coeffy: 0.2
      lowpass_coeffw: 0.3

    ```

> - ??? example "🔧 四轮差速（全地形）（`diff_drive_4wd_at_config.yaml`）"
    ```yaml
    # titan使能服务话题
    titan_enable_topic: "/vmxpi/titan_quad0/enable"
    # VMXPI数据话题
    vmxpi_data_topic: "/vmxpi/vmxpi_datas"

    #底盘配置参数
    chassis_param:
      ctrl_rate: 25                 # 控制频率Hz
      fuse_imu_odom: True           # 里程计融合imu
      invert_imu: True              # 反转IMU角速度
    
      #里程计线速度校正系数
      correct_velx_coeff: 1.0       # x轴系数
      correct_vely_coeff: 1.0       # y轴系数
      correct_velw_coeff: 1.0       # w轴系数

      #软急停IO配置(VMXPI DI口)[di0, di1]
      enable_emg: True              # 使能硬件EMG
      emg_io: 0                     # vmx di0


    ##运动学模型参数##
    # name:轮子名称
    # radius:轮子半径(m)
    # alpha:轮子位于局部参考坐标系角度(deg)
    # gear_ratio:齿轮比
    # set_topic:轮子设置设置电机转速topic
    # feedback_topic:轮子电机反馈topic
    wheel_names: ['wheel0', 'wheel1', 'wheel2', 'wheel3']

    kinematics_model:
      wheel0:
        radius: 0.0675
        alpha: 90.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor0'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder0'
      wheel1:
        radius: 0.0675
        alpha: -90.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor1'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder1'
      wheel2:
        radius: 0.0675
        alpha: 90.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor2'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder2'
      wheel3:
        radius: 0.0675
        alpha: -90.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor3'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder3'

    ##动力学参数##
    # max_vel最大速度约束
    # max_acc最大加速度(注:越大响应越快,超调越来大; 越小滞后越大,速度平滑)
    # lowpass_coeff低通滤波系数(注: 越小滞后越大,加速度拐点越平滑, 到达稳态越平滑; 越大响应快,加速度拐点逼近梯形)
    dynamic_param:
      #最大线速度[x, y, w]      [m/s, m/s, rad/s]
      max_velx: 0.65
      max_vely: 0.0
      max_velw: 3.5
      #最大加速度[x, y, w]      [m/s^2, m/s^2,  rad/s^2]
      max_accx: 2.0
      max_accy: 2.0
      max_accw: 15.0
      #速度低通滤波系数 u(k) = (1 - coeff) * u(k-1) + coeff * u(k)
      lowpass_coeffx: 0.2
      lowpass_coeffy: 0.2
      lowpass_coeffw: 0.3
    ```

> - ??? example "🔧 四轮麦克纳姆轮（`mecanum_4wd_config.yaml`）"
    ```yaml
    # titan使能服务话题
    titan_enable_topic: "/vmxpi/titan_quad0/enable"
    # VMXPI数据话题
    vmxpi_data_topic: "/vmxpi/vmxpi_datas"

    #底盘配置参数
    chassis_param:
      ctrl_rate: 25                 # 控制频率Hz
      fuse_imu_odom: True           # 里程计融合imu
      invert_imu: True              # 反转IMU角速度
    
      #里程计线速度校正系数
      correct_velx_coeff: 1.0       # x轴系数
      correct_vely_coeff: 0.91434   # y轴系数
      correct_velw_coeff: 1.0       # w轴系数

      #软急停IO配置(VMXPI DI口)[di0, di1]
      enable_emg: True              # 使能硬件EMG
      emg_io: 0                     # vmx di0


    ##运动学模型参数##
    # name:轮子名称
    # radius:轮子半径(m)
    # alpha:轮子位于局部参考坐标系角度(deg)
    # gear_ratio:齿轮比
    # set_topic:轮子设置设置电机转速topic
    # feedback_topic:轮子电机反馈topic
    wheel_names: ['wheel0', 'wheel1', 'wheel2', 'wheel3']

    kinematics_model:
      wheel0:
        radius: 0.03626775
        alpha: 135.0
        wheel_base: 0.17
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor0'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder0'
      wheel1:
        radius: 0.03626775
        alpha: -135.0
        wheel_base: 0.17
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor1'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder1'
      wheel2:
        radius: 0.03626775
        alpha: -45.0
        wheel_base: 0.17
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor2'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder2'
      wheel3:
        radius: 0.03626775
        alpha: 45.0
        wheel_base: 0.17
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor3'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder3'
        
    ##动力学参数##
    # max_vel最大速度约束
    # max_acc最大加速度(注:越大响应越快,超调越来大; 越小滞后越大,速度平滑)
    # lowpass_coeff低通滤波系数(注: 越小滞后越大,加速度拐点越平滑, 到达稳态越平滑; 越大响应快,加速度拐点逼近梯形)
    dynamic_param:
      #最大线速度[x, y, w]      [m/s, m/s, rad/s]
      max_velx: 0.65
      max_vely: 0.65
      max_velw: 3.5
      #最大加速度[x, y, w]      [m/s^2, m/s^2,  rad/s^2]
      max_accx: 2.0
      max_accy: 2.0
      max_accw: 15.0
      #速度低通滤波系数 u(k) = (1 - coeff) * u(k-1) + coeff * u(k)
      lowpass_coeffx: 0.2
      lowpass_coeffy: 0.2
      lowpass_coeffw: 0.3

    ```

> - ??? example "🔧 X 型四轮全向轮（`x_omni_drive_config.yaml`）"
    ```yaml
    # titan使能服务话题
    titan_enable_topic: "/vmxpi/titan_quad0/enable"
    # VMXPI数据话题
    vmxpi_data_topic: "/vmxpi/vmxpi_datas"

    #底盘配置参数
    chassis_param:
      ctrl_rate: 25                 # 控制频率Hz
      fuse_imu_odom: True           # 里程计融合imu
      invert_imu: True              # 反转IMU角速度
    
      #里程计线速度校正系数
      correct_velx_coeff: 1.0       # x轴系数
      correct_vely_coeff: 0.983     # y轴系数
      correct_velw_coeff: 1.0       # w轴系数

      #软急停IO配置(VMXPI DI口)[di0, di1]
      enable_emg: True              # 使能硬件EMG
      emg_io: 0                     # vmx di0


    ##运动学模型参数##
    # name:轮子名称
    # radius:轮子半径(m)
    # alpha:轮子位于局部参考坐标系角度(deg)
    # gear_ratio:齿轮比
    # set_topic:轮子设置设置电机转速topic
    # feedback_topic:轮子电机反馈topic
    wheel_names: ['wheel0', 'wheel1', 'wheel2', 'wheel3']

    kinematics_model:
      wheel0:
        radius: 0.049
        alpha: 135.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor0'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder0'
      wheel1:
        radius: 0.049
        alpha: -135.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor1'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder1'
      wheel2:
        radius: 0.049
        alpha: -45.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor2'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder2'
      wheel3:
        radius: 0.049
        alpha: 45.0
        wheel_base: 0.16
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor3'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder3'

    ##动力学参数##
    # max_vel最大速度约束
    # max_acc最大加速度(注:越大响应越快,超调越来大; 越小滞后越大,速度平滑)
    # lowpass_coeff低通滤波系数(注: 越小滞后越大,加速度拐点越平滑, 到达稳态越平滑; 越大响应快,加速度拐点逼近梯形)
    dynamic_param:
      #最大线速度[x, y, w]      [m/s, m/s, rad/s]
      max_velx: 0.65
      max_vely: 0.65
      max_velw: 3.5
      #最大加速度[x, y, w]      [m/s^2, m/s^2,  rad/s^2]
      max_accx: 2.0
      max_accy: 2.0
      max_accw: 15.0
      #速度低通滤波系数 u(k) = (1 - coeff) * u(k-1) + coeff * u(k)
      lowpass_coeffx: 0.2
      lowpass_coeffy: 0.2
      lowpass_coeffw: 0.3

    ```

---

## 🧩 核心参数详解

无论哪种底盘类型，配置文件均包含以下几大模块。下面以 `mecanum_4wd_config.yaml` 为例进行说明（其他配置文件结构相同，仅数值和轮系定义不同）。

### 1️⃣ 话题与通信参数

```yaml
# Titan 驱动器使能服务话题
titan_enable_topic: "/vmxpi/titan_quad0/enable"

# VMXPi 综合数据话题（包含编码器、IMU 等）
vmxpi_data_topic: "/vmxpi/vmxpi_datas"
```

- **`titan_enable_topic`**：控制 Titan 电机驱动器的使能/失能。
- **`vmxpi_data_topic`**：订阅 VMXPi 发布的底层传感器数据。

### 2️⃣ 底盘全局配置 (`chassis_param`)

| 参数 | 类型 | 说明 |
| :--- | :---: | :--- |
| `ctrl_rate` | int | 底盘控制频率，影响响应平滑度。 |
| `fuse_imu_odom` | bool | 是否融合 IMU 与里程计数据（提高位姿估计精度）。 |
| `invert_imu` | bool | 反转 IMU 角速度方向（适配不同安装朝向）。 |
| `correct_velx_coeff` | float | X 轴线速度校正系数（里程计刻度校准）。 |
| `correct_vely_coeff` | float | Y 轴线速度校正系数（全向轮/麦克纳姆轮需要）。 |
| `correct_velw_coeff` | float | 角速度校正系数。 |
| `enable_emg` | bool | 使能硬件急停检测。 |
| `emg_io` | int | 急停信号对应的 VMXPi 数字输入引脚编号。 |

示例配置：

```yaml
chassis_param:
  ctrl_rate: 25
  fuse_imu_odom: True
  invert_imu: True
  correct_velx_coeff: 1.0
  correct_vely_coeff: 0.91434
  correct_velw_coeff: 1.0
  enable_emg: True
  emg_io: 0
```

### 3️⃣ 运动学模型 (`kinematics_model`)

该模块定义每个车轮的几何参数。

| 参数 | 单位 | 说明 |
| :--- | :---: | :--- |
| `radius` | m | 车轮半径。 |
| `alpha` | deg | 车轮在机器人局部坐标系中的安装角度（0° 为 X 轴正向）。 |
| `wheel_base` | m | 轮距（或轴距，具体取决于模型）。 |
| `gear_ratio` | - | 电机到车轮的减速比（通常为 1.0）。 |
| `set_topic` | string | 该轮转速指令的发布话题。 |
| `feedback_topic` | string | 该轮编码器反馈数据的订阅话题。 |

!!! tip "麦克纳姆轮角度示例"
    对于四轮麦克纳姆轮底盘，其安装角度 α 通常分别为 135°, -135°, -45°, 45°：

    ```yaml
    kinematics_model:
      wheel0:
        radius: 0.03626775
        alpha: 135.0
        wheel_base: 0.17
        gear_ratio: 1.0
        set_topic: '/vmxpi/titan_quad0/set_motor0'
        feedback_topic: '/vmxpi/titan_quad0/vmxpi/encoder0'
      wheel1:  # 省略相似内容...
    ```

### 4️⃣ 动力学约束 (`dynamic_param`)

限制机器人的最大速度和加速度，并提供速度平滑滤波。

| 参数 | 单位 | 说明 |
| :--- | :---: | :--- |
| `max_velx`, `max_vely`, `max_velw` | m/s, m/s, rad/s | 最大线速度（X/Y）和角速度。 |
| `max_accx`, `max_accy`, `max_accw` | m/s², m/s², rad/s² | 最大加速度。 |
| `lowpass_coeffx`, `coeffy`, `coeffw` | - | 一阶低通滤波系数（0~1）。值越小，速度变化越平滑但滞后越大。 |

```yaml
dynamic_param:
  max_velx: 0.65
  max_vely: 0.65
  max_velw: 3.5
  max_accx: 2.0
  max_accy: 2.0
  max_accw: 15.0
  lowpass_coeffx: 0.2
  lowpass_coeffy: 0.2
  lowpass_coeffw: 0.3
```

---

## 🔄 切换底盘模型（修改启动配置）

底盘节点启动文件位于：

```bash
/home/vmx/WSR_HB_Robot/install/chassis/share/chassis/launch/chassis.launch.py
```

打开该文件，找到 `chassis_config` 变量定义部分：

```python
chassis_config = os.path.join(
    get_package_share_directory('chassis'),
    'config',
    'default_config.yaml',          # 👈 在此处切换配置文件
    # 'diff_drive_4wd_at_config.yaml',
    # 'diff_drive_4wd_config.yaml',
    # 'mecanum_4wd_config.yaml',
    # 'x_omni_drive_config.yaml',
)
```

**操作步骤**：

1. 注释当前使用的配置文件行。
2. 取消注释与您底盘匹配的配置文件行。
3. 保存文件。

!!! warning "注意"
    修改启动文件后，需要重启 ROS 节点或整个机器人，新配置才会生效。

    ```bash
    # 重启机器人主节点（推荐）
    sudo /home/vmx/WSR_HB_Robot/robot_manager.py restart
    ```
---

## 🧪 各底盘典型配置差异速览

| 参数 | 两轮差速 (default) | 四轮差速 (4wd) | 麦克纳姆轮 (mecanum) | X 全向轮 (x_omni) |
| :--- | :--- | :--- | :--- | :--- |
| **车轮数量** | 2 | 4 | 4 | 4 |
| **`max_vely`** | 0.0（无法横向移动） | 0.0 | 0.65（可横行） | 0.65 |
| **车轮角度 α** | 90°, -90° | 90°, -90°（四轮同向） | 135°, -135°, -45°, 45° | 135°, -135°, -45°, 45° |
| **`correct_vely_coeff`** | 1.0 | 1.0 | ≈0.914 | 0.983 |
| **`ctrl_rate`** | 50 Hz | 25 Hz | 25 Hz | 25 Hz |

!!! info "提示"
    更多详细对比，请直接参考 `config` 目录下各配置文件中的具体数值。

---

## 🛠️ 调参建议

!!! tip "里程计刻度校准（确保轮子半径参数与实际尺寸一致）"
    1. 让机器人直线行驶 1 米，测量实际行驶距离。
    2. 调整 `correct_velx_coeff` = 实际距离 / 理论距离。

!!! tip "角速度校准"
    1. 控制机器人原地旋转 360°，测量实际旋转角度。
    2. 调整 `correct_velw_coeff` = 360° / 实际角度。

!!! tip "加速度与滤波参数"
    - **启停抖动明显** → 降低 `max_accx` 或增大 `lowpass_coeffx`。
    - **响应太慢** → 增大 `max_accx` 或减小 `lowpass_coeffx`。

---

## ❓ 常见问题

| 现象 | 可能原因 | 解决方法 |
| :--- | :--- | :--- |
| **机器人不响应控制指令** | Titan 未使能 | 检查 `titan_enable_topic` 是否正确发布，或手动调用使能服务。 |
| **移动方向错误（前后颠倒）** | 电机方向配置反了 | 修改 YAML 中对应的电机朝向（参考 `kinematics_model.wheel0.alpha`）。 |
| **全向移动时轨迹偏移** | 里程计融合参数不准 | 重新校准 `correct_velx/vely/w` 系数，并确保 IMU 正确融合。 |
| **急停触发后无法恢复** | 急停 IO 配置错误或未复位 | 确认 `enable_emg` 为 `True`，`emg_io` 引脚正确，且急停按钮已松开。 |

---

## ✅ 本章小结

- 根据底盘硬件选择对应的配置文件，并通过 `chassis.launch.py` 切换。
- 核心参数包括运动学模型（车轮半径、角度）、动力学约束（速度/加速度）以及里程计校正系数。
- 合理调参可显著提升移动精度和平顺性。

!!! success "下一步"
    完成底盘配置后，可以尝试控制底盘运动，并使用 `ros2 topic echo /chassis/odom/user` 观察里程计数据是否正常：

---

**📚 相关文档**：
- [VMX_Pi / Titan 设备参数配置](01-vmxpi-titan-params.zh.md)


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
            <a href="https://github.com/其他用户" style="text-decoration: none;">HBRobot</a>
        </div>
    </div>
</div>
---
🤝 **欢迎参与共建：**

[:fontawesome-brands-github: 提交 Issue](https://github.com/hbrobot/hbrobot.github.io/issues/new/choose){: .md-button }
[:octicons-git-pull-request-24: 提交 PR](https://github.com/hbrobot/hbrobot.github.io/compare){: .md-button .md-button--primary }