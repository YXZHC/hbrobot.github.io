# IMU 数据读取与应用

本教程将带你学习如何使用 ROS2 RCLPY 订阅 IMU（惯性测量单元）传感器发布的原始数据，并将四元数姿态转换为更直观的欧拉角进行显示。

---

## 1. 课前准备

### 1.1 知识预备

在开始本教程之前，建议你先了解以下内容：

- ROS2 话题（Topic）通信的基本概念
- RCLPY 节点（Node）的创建与运行
- IMU 传感器的基本工作原理（加速度计、陀螺仪）

### 1.2 环境要求

| 项目 | 要求 |
|------|------|
| ROS2 发行版 | Humble 及以上 |
| Python 版本 | 3.10+ |
| 依赖包 | `tf_transformations` |

!!! warning "关于 tf_transformations"

    在 ROS2 Humble 及以上版本中，`tf_transformations` 库提供了 `euler_from_quaternion` 函数用于四元数到欧拉角的转换。该库并非默认安装，需要手动安装：

    ```bash
    sudo apt install ros-humble-tf-transformations
    ```

### 1.3 启动 IMU 数据源

在运行本教程的代码之前，请确保你的机器人或仿真环境正在发布 IMU 数据话题。例如，使用以下命令检查话题是否存在：

```bash
ros2 topic list | grep imu
```

典型的话题名称为 `/vmxpi/imu` 或 `/imu`。

---

## 2. 代码详解

### 2.1 完整代码

以下是完整的 IMU 数据订阅节点代码：

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Imu
from tf_transformations import euler_from_quaternion

class IMUSubscriber(Node):
    def __init__(self):
        super().__init__('imu_subscriber')
        self.subscription = self.create_subscription(
            Imu,
            '/vmxpi/imu',
            self.imu_callback,
            10
        )
        self.subscription 
        self.get_logger().info('IMU订阅节点已启动,等待数据...')
    
    def imu_callback(self, msg: Imu):
        # 获取四元数
        orientation_q = msg.orientation
        quaternion = [
            orientation_q.x,
            orientation_q.y,
            orientation_q.z,
            orientation_q.w
        ]
        
        # 转换为欧拉角 (roll, pitch, yaw)
        euler = euler_from_quaternion(quaternion)
        roll, pitch, yaw = euler
        
        # 角速度 (rad/s)
        angular_velocity = msg.angular_velocity
        angular_velocity_x = angular_velocity.x
        angular_velocity_y = angular_velocity.y
        angular_velocity_z = angular_velocity.z
        
        # 线加速度 (m/s²)
        linear_acceleration = msg.linear_acceleration
        linear_acceleration_x = linear_acceleration.x
        linear_acceleration_y = linear_acceleration.y
        linear_acceleration_z = linear_acceleration.z
        
        # 打印信息（节流输出）
        time_acc = 0.5
        self.get_logger().info('\n' + '='*50, throttle_duration_sec=time_acc)
        self.get_logger().info('IMU数据:', throttle_duration_sec=time_acc)

        import math
        roll_deg = math.degrees(roll)
        pitch_deg = math.degrees(pitch)
        yaw_deg = math.degrees(yaw)
        self.get_logger().info(
            f'欧拉角: 滚转={roll_deg:.3f}°, 俯仰={pitch_deg:.3f}°, 偏航={yaw_deg:.3f}°',
            throttle_duration_sec=time_acc
        )

        self.get_logger().info(
            f'欧拉角: 滚转={roll:.3f} rad, 俯仰={pitch:.3f} rad, 偏航={yaw:.3f} rad',
            throttle_duration_sec=time_acc
        )
        self.get_logger().info(
            f'角速度: X={angular_velocity_x:.3f}, Y={angular_velocity_y:.3f}, Z={angular_velocity_z:.3f} rad/s',
            throttle_duration_sec=time_acc
        )
        self.get_logger().info(
            f'线加速度: X={linear_acceleration_x:.3f}, Y={linear_acceleration_y:.3f}, Z={linear_acceleration_z:.3f} m/s²',
            throttle_duration_sec=time_acc
        )
        self.get_logger().info('-'*50, throttle_duration_sec=time_acc)

def main(args=None):
    rclpy.init(args=args)
    imu_subscriber = IMUSubscriber()
    
    try:
        rclpy.spin(imu_subscriber)
    except KeyboardInterrupt:
        imu_subscriber.get_logger().info('节点被用户关闭')
    finally:
        imu_subscriber.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 2.2 逐段解析

#### 2.2.1 导入依赖

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Imu
from tf_transformations import euler_from_quaternion
```

| 导入模块 | 用途 |
|----------|------|
| `rclpy` | ROS2 的 Python 客户端库 |
| `rclpy.node.Node` | 节点基类 |
| `sensor_msgs.msg.Imu` | IMU 数据消息类型 |
| `tf_transformations.euler_from_quaternion` | 四元数 → 欧拉角转换函数 |

??? tip "关于 sensor_msgs/Imu 消息结构"

    `sensor_msgs/msg/Imu` 是 ROS2 中用于传输 IMU 数据的标准消息类型，包含以下主要字段：

    ```yaml
    std_msgs/Header header              # 时间戳和坐标系
    geometry_msgs/Quaternion orientation # 姿态四元数 (x, y, z, w)
    float64[9] orientation_covariance    # 姿态协方差矩阵
    geometry_msgs/Vector3 angular_velocity     # 角速度 (rad/s)
    float64[9] angular_velocity_covariance     # 角速度协方差
    geometry_msgs/Vector3 linear_acceleration  # 线加速度 (m/s²)
    float64[9] linear_acceleration_covariance  # 加速度协方差
    ```

    需要注意的是，加速度的单位是 **m/s²**（而非重力加速度 g），角速度的单位是 **rad/s**。

#### 2.2.2 创建订阅

```python
self.subscription = self.create_subscription(
    Imu,
    '/vmxpi/imu',
    self.imu_callback,
    10
)
```

`create_subscription` 方法的参数说明：

| 参数 | 说明 |
|------|------|
| `Imu` | 消息类型 |
| `/vmxpi/imu` | 话题名称（请根据实际情况修改） |
| `self.imu_callback` | 回调函数 |
| `10` | 队列大小（Quality of Service） |

!!! warning "话题名称可能不同"

    不同机器人平台发布的 IMU 话题名称可能不同，常见的有：

    - `/vmxpi/imu`（VMXPi 平台）
    - `/imu`（通用）
    - `/imu/data`（融合后的数据）
    
    请使用 `ros2 topic list` 确认实际的话题名称。

#### 2.2.3 四元数转欧拉角

IMU 消息中的姿态数据以**四元数（Quaternion）** 形式存储，这是一种不受万向节锁限制的旋转表示方法。但在实际应用中，**欧拉角（Euler Angle）** 更加直观易懂。

```python
orientation_q = msg.orientation
quaternion = [
    orientation_q.x,
    orientation_q.y,
    orientation_q.z,
    orientation_q.w
]
euler = euler_from_quaternion(quaternion)
roll, pitch, yaw = euler
```

!!! tip "四元数的顺序"

    `euler_from_quaternion` 函数接收一个包含 `[x, y, z, w]` 的列表，返回 `(roll, pitch, yaw)` 元组，单位为**弧度（rad）**。

??? example "四元数 vs 欧拉角"

    | 表示方法 | 优点 | 缺点 |
    |----------|------|------|
    | **四元数** | 无万向节锁、计算高效、适合插值 | 不直观 |
    | **欧拉角** | 直观易懂（滚转/俯仰/偏航） | 可能存在万向节锁 |

    在机器人导航中，通常将四元数转换为欧拉角以便于理解和调试。

#### 2.2.4 提取角速度与线加速度

```python
angular_velocity = msg.angular_velocity
angular_velocity_x = angular_velocity.x
# ... 同理提取 y, z

linear_acceleration = msg.linear_acceleration
linear_acceleration_x = linear_acceleration.x
# ... 同理提取 y, z
```

`geometry_msgs/Vector3` 类型包含 `x`、`y`、`z` 三个分量，分别对应三个轴向的数据。

#### 2.2.5 日志节流输出

```python
time_acc = 0.5
self.get_logger().info('IMU数据:', throttle_duration_sec=time_acc)
```

`throttle_duration_sec` 参数用于**日志节流（Logging Throttled）** —— 即在一定时间间隔内只输出一次日志，避免高频数据淹没控制台。

??? tip "为什么要使用日志节流？"

    IMU 数据的发布频率通常很高（50Hz~200Hz），如果不加节流，终端会被海量日志刷屏，既影响阅读也消耗性能。设置 `throttle_duration_sec=0.5` 表示每 0.5 秒最多输出一次日志。

#### 2.2.6 弧度转角度

```python
import math
roll_deg = math.degrees(roll)
pitch_deg = math.degrees(pitch)
yaw_deg = math.degrees(yaw)
```

`math.degrees()` 将弧度转换为角度，便于人类阅读。

---

## 3. 运行与测试

### 3.1 运行节点

```bash
python3 02-读取IMU原始数据.py
```

### 3.2 预期输出

```
==================================================
IMU数据:
欧拉角: 滚转=0.523°, 俯仰=-0.012°, 偏航=45.678°
欧拉角: 滚转=0.009 rad, 俯仰=-0.000 rad, 偏航=0.797 rad
角速度: X=0.001, Y=-0.002, Z=0.015 rad/s
线加速度: X=0.012, Y=-0.008, Z=9.801 m/s²
--------------------------------------------------
```

### 3.3 验证数据

你也可以使用命令行工具直接查看 IMU 话题的数据：

```bash
# 查看话题数据
ros2 topic echo /vmxpi/imu

# 查看话题发布频率
ros2 topic hz /vmxpi/imu
```

---

## 4. 常见问题

??? warning "ImportError: No module named 'tf_transformations'"

    如果遇到此错误，说明 `tf_transformations` 库未安装：
    
    ```bash
    sudo apt install ros-humble-tf-transformations
    ```
    
    请将 `humble` 替换为你使用的 ROS2 发行版名称。

??? warning "无法接收到 IMU 数据"

    1. 确认 IMU 话题名称是否正确：`ros2 topic list | grep imu`
    2. 确认 IMU 驱动节点是否正在运行
    3. 检查话题是否有数据发布：`ros2 topic echo /你的话题名称`

??? question "欧拉角的顺序是什么？"

    `euler_from_quaternion` 返回的欧拉角顺序为 **roll（滚转）→ pitch（俯仰）→ yaw（偏航）**，对应绕 **X → Y → Z** 轴的旋转。

??? question "为什么偏航角（Yaw）会有跳变？"

    欧拉角的偏航角在 ±π（±180°）边界处会发生跳变，这是欧拉角表示法的固有问题。如果需要连续的角度输出，建议直接使用四元数或进行角度解包裹（unwrap）处理。

---

## 5. 扩展思考

### 5.1 协方差矩阵的使用

`sensor_msgs/Imu` 消息还包含了三个协方差矩阵：

- `orientation_covariance`：姿态协方差
- `angular_velocity_covariance`：角速度协方差
- `linear_acceleration_covariance`：线加速度协方差

!!! tip "协方差矩阵的约定"

    如果某个数据项无法提供（例如 IMU 不输出姿态估计），应将对应协方差矩阵的第一个元素设置为 **-1**。协方差矩阵为全零则表示"协方差未知"。

在传感器融合（如卡尔曼滤波）中，协方差矩阵是非常重要的输入。

### 5.2 数据记录与回放

你可以使用 ROS2 Bag 工具记录 IMU 数据以便离线分析：

```bash
# 录制 IMU 数据
ros2 bag record /vmxpi/imu -o imu_recording

# 回放数据
ros2 bag play imu_recording
```

### 5.3 下一步学习

掌握 IMU 原始数据的读取后，你可以继续学习：

1. **IMU 数据滤波**：使用互补滤波或卡尔曼滤波融合加速度计和陀螺仪数据
2. **IMU 与里程计融合**：通过 `robot_localization` 包实现多传感器融合
3. **IMU 在 SLAM 中的应用**：为激光 SLAM 或视觉 SLAM 提供姿态估计

---

## 6. 参考资料

- [sensor_msgs/Imu 消息定义](https://docs.ros2.org/galactic/api/sensor_msgs/msg/Imu.html)
- [tf_transformations.euler_from_quaternion 用法](https://robotics.stackexchange.com/questions/96357)
- [ROS2 日志系统文档](https://docs.ros.org/en/ros2_documentation/jazzy/Tutorials/Demos/Logging-and-logger-configuration.html)

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