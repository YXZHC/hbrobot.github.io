# 视觉与 AI FAQ

本页面汇总 **摄像头采集、OpenCV 处理、YOLO 训练与部署** 等视觉相关问题。

---

## Q1: 摄像头无法打开 `/dev/video0 not found`？

**症状**：
```
cv2.error: OpenCV(4.x) ... VIDEOIO(CV_IMAGES): raised OpenCV exception:
Can't open camera by index 0
```

**排查步骤**：
```bash
# 1. 检查设备节点
ls /dev/video*

# 2. 检查是否被占用
lsof /dev/video0

# 3. 检查 USB 连接
lsusb | grep -i camera

# 4. 测试 v4l2 能否读取
v4l2-ctl --device=/dev/video0 --list-formats
```

**解决方案**：

| 原因 | 解决方案 |
|------|---------|
| USB 接触不良 | 重新插拔，换 USB 口（优先 USB 3.0） |
| 权限不足 | `sudo usermod -aG video $USER`，重新登录 |
| 多摄像头索引错乱 | 用 `v4l2-ctl --list-devices` 确认对应关系 |
| 驱动未加载 | 安装 `sudo apt install cheese` 测试是否能预览 |

---

## Q2: OpenCV 读取的画面很卡 / 帧率低？

**症状**：`cv2.VideoCapture.read()` 帧率远低于摄像头标称值。

**优化方案**：
```python
import cv2

cap = cv2.VideoCapture(0)
# 1. 设置缓冲区大小为 1（减少延迟）
cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)
# 2. 设置分辨率匹配摄像头原生分辨率
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
# 3. 设置帧率
cap.set(cv2.CAP_PROP_FPS, 30)

# 4. 使用多线程读取（生产者-消费者模式）
from threading import Thread
class VideoStream:
    def __init__(self, src=0):
        self.cap = cv2.VideoCapture(src)
        self.cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)
        self.frame = None
        self.stopped = False
        self.thread = Thread(target=self._update, daemon=True)
        self.thread.start()

    def _update(self):
        while not self.stopped:
            self.frame = self.cap.read()[1]

    def read(self):
        return self.frame

    def stop(self):
        self.stopped = True
        self.thread.join()
```

---

## Q3: YOLO 训练时显存不足 `CUDA out of memory`？

**症状**：训练刚开始就报 OOM 错误。

**解决方案**：
```bash
# 1. 减小 batch size
yolo train data=xxx.yaml model=yolov8n.pt batch=4  # 从 16 降到 4 或 2

# 2. 使用更小的模型
# yolov8n → yolov8s → yolov8m → yolov8l → yolov8x
yolo train data=xxx.yaml model=yolov8n.pt

# 3. 降低输入分辨率
yolo train data=xxx.yaml model=yolov8n.pt imgsz=416  # 默认 640

# 4. 启用梯度累积（模拟大 batch）
yolo train data=xxx.yaml model=yolov8n.pt batch=2 accumulation=8

# 5. 清理显存
nvidia-smi --gpu-reset  # 慎用，会杀掉所有 GPU 进程
```

---

## Q4: YOLO 推理精度差 / 漏检严重？

**排查与优化**：

1. **检查数据集质量**：
   - 标注框是否紧贴目标？
   - 正负样本比例是否均衡？
   - 是否覆盖了不同光照、角度、尺度？

2. **数据增强**：
   ```yaml
   # data.yaml
   augmentation:
     hsv_h: 0.015
     hsv_s: 0.7
     hsv_v: 0.4
     flipud: 0.5
     fliplr: 0.5
     mosaic: 1.0
     mixup: 0.1
   ```

3. **调整置信度阈值**：
   ```python
   results = model(frame, conf=0.25)  # 降低 conf 看是否能检出
   ```

4. **换更大模型**：n → s → m，精度递增但速度递减

---

## Q5: MNN 模型部署后推理结果和 PyTorch 不一致？

**症状**：同一张图片，PyTorch 推理正确，MNN 部署后结果偏差大。

**排查清单**：
1. **确认导出流程正确**：
   ```
   PyTorch → ONNX → MNN
   ```
2. **检查预处理一致性**：
   - 均值/标准差是否一致
   - 归一化方式（0-1 还是 0-255）
   - Resize 插值方式（bilinear / nearest）
3. **检查输入尺寸**：MNN 输入 shape 是否和训练时一致
4. **FP16 精度损失**：尝试用 FP32 推理对比

```python
# MNN 预处理参考（必须和训练时完全一致）
import cv2
import numpy as np

def preprocess(img, input_size=640):
    img = cv2.resize(img, (input_size, input_size), interpolation=cv2.INTER_LINEAR)
    img = img.astype(np.float32) / 255.0
    img = (img - np.array([0.485, 0.456, 0.406])) / np.array([0.229, 0.224, 0.225])
    img = img.transpose(2, 0, 1)  # HWC → CHW
    img = np.expand_dims(img, axis=0)
    return img
```

---

## Q6: 摄像头画面和 YOLO 检测结果不同步？

**症状**：检测框明显滞后于目标运动。

**原因与解决**：
- 缓冲区积压 → 设置 `CAP_PROP_BUFFERSIZE=1`
- 推理耗时 > 帧间隔 → 降低分辨率或换更快的模型
- 多线程竞争 → 使用独立的采集线程 + 推理线程，加锁保护

---

> 📌 **更多问题？** 查看 [报错排查速查表](./06-error-troubleshooting.zh.md) | 返回 [FAQ 总览](./00-faq-overview.zh.md)
