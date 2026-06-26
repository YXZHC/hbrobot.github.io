# 📷 图像采集

> 🤖 在世界技能大赛（WorldSkills）自主移动机器人项目中，视觉系统是机器人感知周围环境、识别目标物体和完成精准抓取的核心“眼睛”。本项目采用了 Orbbec 深度相机，结合 ROS 2 框架提供了稳定高效的图像采集接口。

---

## 📌 概述

视觉感知是自主移动机器人实现环境理解、目标识别与导航避障的核心能力。在 ROS 2 架构下，我们通过 **Orbbec 深度相机** 获取彩色图像与深度图像，为上层视觉算法提供原始数据。

本文档将详细介绍 Python 接口下相机模块的两种图像采集方式：

- **📸 拍照服务（Service）**：通过 ROS 2 服务请求单帧图像，支持同步/异步模式。
- **📹 视频流订阅（Topic）**：通过 ROS 2 话题持续订阅图像流，适用于实时处理场景。

我们将基于 `wsc_robotic.vision.camera` 模块的 `Camera` 类，结合示例程序，解析各接口的使用方法与注意事项。

---

## 🧩 模块导入与初始化

在使用相机功能前，需要先实例化机器人对象，再创建相机实例：

```python
from wsc_robotic.robotics import Robotics
from wsc_robotic.vision.camera import Camera, SnapMethod

# 实例化机器人（包含 ROS 2 节点）
robot = Robotics()
# 创建相机对象，传入机器人节点
camera = Camera(robot)
```

> 💡 `Robotics` 类封装了 ROS 2 节点的初始化与生命周期管理，所有设备模块均依赖该对象。

---

## 📸 方式一：拍照服务（Snap）

拍照服务基于 ROS 2 **Service（服务）** 机制，客户端向 `/camera/req_image` 服务发送请求，服务端返回一帧或多帧图像。适用于需要精确控制采集时刻的场景，如：

- 识别特定目标时拍照
- 记录关键帧用于建图
- 人工触发图像保存

### 🔧 API 说明

```python
def snap(self, method=SnapMethod.ALL, sync=False):
    """
    请求单帧图像（同步/异步）
    
    Args:
        method (SnapMethod): 图像类型枚举
            - SnapMethod.ALL   : 同时获取彩色图 + 深度图
            - SnapMethod.COLOR : 仅获取彩色图
            - SnapMethod.DEPTH : 仅获取深度图
        sync (bool): 同步模式开关
            - True  : 同步模式，等待相机曝光最新帧，响应时间约 100~200ms
            - False : 异步模式，直接返回缓存中的最新帧，响应极快（通常 < 10ms）
    
    Returns:
        success (bool): 是否成功
        color_image (np.ndarray): 彩色图（BGR 格式，8UC3），若未请求则为 None
        depth_image (np.ndarray): 深度图（16UC1，单位 mm），若未请求则为 None
        info (dict): 相机内参与畸变系数
            {
                'intrinsic': {'fx': , 'fy': , 'cx': , 'cy': },
                'distortion': {'k1': , 'k2': , 'k3': , 'k4': , 'k5': }
            }
    """
```

### 📝 示例代码解析

```python
import time
import cv2
from wsc_robotic.robotics import Robotics
from wsc_robotic.vision.camera import Camera, SnapMethod

def main():
    robot = Robotics()
    camera = Camera(robot)

    # 请求彩色 + 深度图，异步模式（快速返回缓存）
    start = time.perf_counter()
    success, color, depth, info = camera.snap(method=SnapMethod.ALL, sync=False)
    end = time.perf_counter()

    if success:
        print(f"拍照完成，耗时: {end - start:.6f} 秒")
        # 显示图像（深度图乘以 50 增强可视性）
        cv2.imshow('color', color)
        cv2.imshow('depth', depth * 50)
        cv2.waitKey(0)
        cv2.destroyAllWindows()
        # 保存图像（时间戳命名）
        timestamp = time.time_ns()
        cv2.imwrite(f"{timestamp}_color.png", color)
        cv2.imwrite(f"{timestamp}_depth.png", depth)

    robot.stop()

if __name__ == '__main__':
    main()
```

> ⚙️ **同步 vs 异步选择建议**
>
> - **异步模式（`sync=False`）**：适用于需要高频采集或实时处理，且允许使用上一帧图像的场景（如连续跟踪）。
> - **同步模式（`sync=True`）**：适用于需要“此刻”精确曝光的场景（如抓拍特定动作），但会引入额外延迟。

---

## 📹 方式二：视频流订阅（Video Stream）

视频流基于 ROS 2 **Topic（话题）** 机制，相机节点持续发布图像消息，客户端订阅后通过回调接收。适用于：

- 实时目标检测与跟踪
- 视觉里程计
- 导航避障

`Camera` 类提供了 `start_video_stream()` / `stop_video_stream()` / `read_video_stream()` 三个方法，封装了订阅与缓存读取。

### 🔧 API 说明

```python
def start_video_stream(self, compressed=False):
    """
    开始订阅彩色图像话题
    
    Args:
        compressed (bool): 是否使用压缩图像
            - True  : 订阅 `/camera/color/image_compressed`（JPEG 压缩），带宽小，但图像有损
            - False : 订阅 `/camera/color/image_raw`（原始图像），质量高，但带宽大
    """

def stop_video_stream(self):
    """停止订阅，释放资源"""

def read_video_stream(self):
    """
    从缓存读取一帧彩色图像（非阻塞）
    
    Returns:
        ret (bool): 是否成功读取到新帧
        img (np.ndarray): BGR 彩色图像，若 ret=False 则为 None
    """
```

### 🔄 工作流程

1. 调用 `start_video_stream()` 创建订阅，图像数据持续存入内部缓存。
2. 在主循环中调用 `read_video_stream()`，若缓存中有新帧则返回 `True` 和图像，否则返回 `False`。
3. 使用完毕后调用 `stop_video_stream()` 停止订阅。

### 📝 示例代码解析

```python
import cv2
from wsc_robotic.robotics import Robotics
from wsc_robotic.vision.camera import Camera

def main():
    robot = Robotics()
    camera = Camera(robot)

    # 开启压缩视频流（节约网络带宽）
    camera.start_video_stream(compressed=True)

    print("正在订阅相机流，按 ESC 退出...")
    while True:
        ret, frame = camera.read_video_stream()
        if ret:
            cv2.imshow('color viewer', frame)
            if cv2.waitKey(20) & 0xFF == 27:  # ESC 键退出
                break

    camera.stop_video_stream()
    robot.stop()
    cv2.destroyAllWindows()

if __name__ == '__main__':
    main()
```

> 🚀 **性能提示**
>
> - 使用 `compressed=True` 可显著降低网络负载，尤其适用于无线通信场景，但会牺牲少许图像细节。
> - 回调组采用 `ReentrantCallbackGroup`，允许多线程并发，需注意互斥锁保护共享缓存（已由 `Camera` 内部实现）。

---

## 🧠 进阶：深度图与内参的使用

- **编码格式**：`mono16`（16 位单通道）
- **单位**：**毫米（mm）**
- **有效范围**：通常为 0.2m ~ 5m（视型号而定）
- **显示技巧**：直接显示时像素值较小（如 1000mm），需乘以系数（如 50）以增强可视性。
- **无效值**：深度为 0 或 65535 表示无效（超出范围或遮挡）。

### 深度图解析
通过 `SnapMethod.ALL` 或 `SnapMethod.DEPTH` 获取的 `depth_image` 是一个 16 位无符号整型单通道图像（`mono16` / `16UC1`）。其像素值直接代表了该点距离相机光心的物理距离（单位通常为毫米 mm）。

在 OpenCV 中直接 `imshow` 深度图会显示为全黑，需要进行归一化或乘以放大系数（如示例中的 `* 50`）才能肉眼可见。

### 像素坐标转 3D 空间坐标
`camera.snap()` 返回的 `info` 字典包含了相机的内参。结合深度图，可将图像中的像素点 $(u, v)$ 转换为机器人坐标系下的 3D 坐标 $(X, Y, Z)$，这是抓取任务的核心数学基础：

$$
Z = depth\_image[v, u]
$$

$$
X = (u - c_x) \times \frac{Z}{f_x}
$$

$$
Y = (v - c_y) \times \frac{Z}{f_y}
$$

其中 $f_x, f_y, c_x, c_y$ 均可从 `info['intrinsic']` 中直接读取。

---


## 📋 采集方式对比

| 特性               | 拍照服务（Snap）                | 视频流订阅（Stream）           |
|--------------------|--------------------------------|-------------------------------|
| **通信机制**       | Service（请求-响应）            | Topic（发布-订阅）             |
| **图像时效性**     | 可选同步（最新曝光）或异步（缓存）| 实时流（每帧最新）             |
| **数据内容**       | 可单独请求彩色/深度/全部        | 仅彩色（深度需另行订阅）       |
| **典型延迟**       | 同步 ~150ms，异步 <10ms         | 取决于发布频率（通常 30fps）   |
| **适用场景**       | 单次抓拍、关键帧记录            | 连续实时处理、视频显示         |
| **资源占用**       | 按需请求，省带宽               | 持续传输，带宽占用高           |

---

## ⚠️ 常见问题与建议

???+ tip "Q1：调用 `snap()` 后一直等待服务超时怎么办？"
    - **排查服务状态**：首先确认 Orbbec 相机 ROS 2 节点是否已正常启动。在终端执行 `ros2 node list`，检查是否存在类似 `orbbec_camera_node` 或 `/camera` 相关节点。
    - **检查网络通信**：确保 ROS 2 环境变量（如 `ROS_DOMAIN_ID`）在主机与机器人之间一致，且网络互通。可执行 `ros2 service list` 确认 `/camera/req_image` 服务是否已注册。
    - **常见原因**：相机节点未启动、USB 连接松动、或服务名称拼写错误。建议重启相机节点后再试。

???+ warning "Q2：视频流读取不到图像（`ret` 始终为 `False`）？"
    - **确认订阅已开启**：检查代码中是否已正确调用 `start_video_stream(compressed=...)`。订阅未开启时，缓存中无数据，`read_video_stream()` 始终返回 `False`。
    - **核对话题名称**：原始图像话题为 `/camera/color/image_raw`，压缩图像话题为 `/camera/color/image_compressed`。若相机驱动发布的话题名不同（例如带命名空间），需调整订阅参数。
    - **检查相机发布频率**：执行 `ros2 topic echo /camera/color/image_raw --once` 或 `ros2 topic hz /camera/color/image_raw` 确认是否有数据流。若话题无数据，检查相机硬件连接与驱动状态。
    - **注意互斥锁**：`Camera` 类内部已使用互斥锁保护共享缓存，但若在多线程环境下调用，请确保遵循线程安全规范。

???+ tip "Q3：深度图显示全黑或全白，无法正常观察？"
    - **全黑（图像几乎为零）**：深度图以毫米为单位，有效距离通常为 0.2~5m。若拍摄物体太近或太远，深度值可能为 0（无效）。显示时建议乘以系数（如 `depth * 50`）增强对比度；也可使用 `cv2.normalize` 进行归一化显示。
    - **全白（像素值异常）**：可能是将 `mono16` 图像误当作 `mono8` 或 `bgr8` 解码，导致数据溢出。请确认使用 `cv_bridge` 时指定正确的编码 `'mono16'`。
    - **噪点较多**：深度图受光照和物体材质影响，透明或反光表面可能出现空洞。可尝试调整相机曝光参数，或使用中值滤波进行预处理。

???+ tip "Q4：同步模式拍照响应缓慢（>200ms）？"
    - **原理解释**：同步模式（`sync=True`）会触发相机重新曝光并等待一帧新的完整图像，响应时间取决于当前曝光时间（Exposure Time）和传感器读出速度，通常为 100~200ms，属于正常现象。
    - **优化建议**：若对实时性要求高（如高速跟随），建议使用异步模式（`sync=False`），直接返回缓存中的最新帧（通常 <10ms），但需要注意该帧可能略早于请求时刻。
    - **降低曝光时间**：在光照充足的环境下，可适当降低相机曝光时间（通过动态参数配置），以缩短同步模式的等待时间。

???+ tip "进阶扩展：如何连续采集多帧用于标定或训练？"
    - **循环调用 `snap()`**：在循环中重复调用异步模式，可以快速采集大量图像对，用于相机标定或数据集构建。注意每次调用后需等待服务响应，建议加入适当的延时控制采集频率。
    - **利用视频流缓存**：若需连续获取每一帧，建议开启视频流订阅，并配合队列（如 `collections.deque`）缓存最近 N 帧，便于回溯处理。
    - **同步时间戳**：彩色图与深度图时间戳可能略有偏差，如需严格对齐，可查阅 `Image` 消息中的 `header.stamp` 字段，在应用层进行软同步。

---

## 📚 参考资料

- [Orbbec Camera ROS 2 驱动文档](https://github.com/orbbec/OrbbecSDK_ROS2)
- [OpenCV-Python 图像处理手册](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)

---


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