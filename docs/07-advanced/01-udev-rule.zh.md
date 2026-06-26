# UDEV 规则配置指南 📝
在自主移动机器人（AMR）开发与部署过程中，USB设备（如激光雷达、串口传感器、无线模块等）的挂载路径常因插拔顺序、系统重启等因素发生变化（例如 `/dev/ttyUSB0` 变为 `/dev/ttyUSB1`），导致程序无法稳定访问设备。UDEV 作为 Linux 系统的设备管理框架，可通过自定义规则为指定设备绑定**固定符号链接**，彻底解决路径漂移问题。

本文以世界技能大赛自主移动机器人项目中激光雷达的 UDEV 配置为例，详细讲解 UDEV 规则的设计、编写与验证全流程。

## 🎯 核心目标

- 为机器人的关键 USB 设备（如激光雷达、串口舵机、GPS 模块）绑定固定符号链接（如 `/dev/lidar0`、`/dev/imu0`）；
- 确保设备插拔、系统重启后，符号链接与物理设备的映射关系稳定不变；
- 统一设备权限配置，避免程序因权限不足无法访问设备。

## 📋 前置准备
### 1. 环境要求

- 操作系统：Ubuntu 20.04/22.04（兼容树莓派 Raspberry Pi OS、VMXPi 系统）；
- 权限：拥有 `sudo` 管理员权限；
- 工具：`udevadm`（Linux 内置，无需额外安装）。

### 2. 设备连接
将需要配置的 USB 设备（如激光雷达）接入机器人的工控机/树莓派，确保设备被系统识别（可通过 `dmesg` 或 `lsusb` 验证）。

## 🔍 第一步：识别设备关键属性
UDEV 规则通过**设备唯一属性**匹配目标设备，核心属性包括：

- `idVendor`：设备厂商 ID（由 USB-IF 分配）；
- `idProduct`：设备产品 ID（厂商自定义）；
- `devpath`：设备挂载的物理 USB 端口路径；
- `serial`：设备序列号（部分设备独有，优先级最高）。

### 操作步骤
1. **查看已挂载的 USB 串口设备**  
   终端执行以下命令，列出系统中已识别的 USB 串口设备：
   ```bash
   ls /dev/ttyUSB*  # 常见串口设备命名，也可能是 ttyACM*
   ```
   正常输出示例（如连接2个激光雷达）：
   ```bash
   /dev/ttyUSB0  /dev/ttyUSB1
   ```

2. **查询设备详细属性**  
   对每个串口设备执行 `udevadm info` 命令，筛选关键属性：
   ```bash
   # 以 /dev/ttyUSB0 为例，替换为实际设备路径
   udevadm info --attribute-walk --name=/dev/ttyUSB0 | grep -E "serial|devpath|idVendor|idProduct"
   ```
   输出示例及说明：
   ```bash
   Udevadm info starts with the device specified by the devpath and then
       SUBSYSTEMS=="usb-serial"
       ATTRS{devpath}=="1.1"        # 物理 USB 端口路径（核心匹配项）
       ATTRS{idProduct}=="ea60"     # 产品 ID（CP2102 串口芯片）
       ATTRS{idVendor}=="10c4"      # 厂商 ID（Silicon Labs）
       ATTRS{serial}=="0001"        # 设备序列号（唯一标识，如果有，则优先级最高）
       ATTRS{devpath}=="1"
       ATTRS{idProduct}=="3431"
       ATTRS{idVendor}=="2109"
       ATTRS{devpath}=="0"
       ATTRS{idProduct}=="0002"
       ATTRS{idVendor}=="1d6b"
       ATTRS{serial}=="0000:01:00.0"
   ```
   ✨ **关键提示**：  
   - 优先选择 `serial`（序列号）作为匹配项（唯一性最强）；  
   - 若无序列号，使用 `idVendor + idProduct + devpath` 组合（确保物理端口固定）；  
   - `devpath` 需选择与设备直接关联的路径（如示例中的 `1.1`，而非 `1`/`0`）。

## ✍️ 第二步：编写 UDEV 规则文件
UDEV 规则文件存放于 `/etc/udev/rules.d/` 目录，命名规范为 `[优先级]-[名称].rules`（如 `99-lidar-usb.rules`，优先级数值越大，执行优先级越高）。

### 1. 创建规则文件
```bash
sudo nano /etc/udev/rules.d/99-amr-udev.rules
```

### 2. 规则语法说明
UDEV 规则核心格式：
```bash
# 注释内容
SUBSYSTEM=="匹配子系统", ATTRS{属性名}=="属性值", SYMLINK+="自定义符号链接", MODE="设备权限"
```

| 字段          | 说明                                                                 |
|---------------|----------------------------------------------------------------------|
| `SUBSYSTEM`   | 匹配设备子系统，串口设备填 `tty`，USB 设备可填 `usb`                 |
| `ATTRS{key}`  | 设备属性匹配（如 `ATTRS{idVendor}=="10c4"`），可多个组合             |
| `SYMLINK+`    | 自定义固定符号链接（如 `lidar0`，最终路径为 `/dev/lidar0`）          |
| `MODE`        | 设备权限（如 `0666` 表示所有用户可读写，避免程序权限不足）           |

### 3. 示例规则（激光雷达配置）

#### 场景1：按物理端口匹配（无序列号）
若设备无序列号，通过 `devpath` 绑定物理端口：
```bash
# 自主移动机器人 - 激光雷达1（物理端口 devpath=1.1）
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{devpath}=="1.1", SYMLINK+="lidar0", MODE="0666"
# 自主移动机器人 - 激光雷达2（物理端口 devpath=1.2）
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{devpath}=="1.2", SYMLINK+="lidar1", MODE="0666"
```

#### 场景2：按序列号匹配（有序列号时推荐）
若设备有唯一序列号，规则如下：
```bash
# 自主移动机器人 - 激光雷达1（序列号 0001）
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="0001", SYMLINK+="lidar0", MODE="0666"
# 自主移动机器人 - 激光雷达2（序列号 0002）
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="0002", SYMLINK+="lidar1", MODE="0666"
```

⚠️ **重要警告**：  
`devpath` 和序列号需根据实际设备属性修改，严禁直接复制示例值！物理端口插拔顺序变更后，需重新查询 `devpath` 并更新规则。

### 4. 保存规则文件
在 `nano` 编辑器中：

- 按 `Ctrl+O` 保存文件；
- 按 `Ctrl+X` 退出编辑器。

## 🔄 第三步：重载 UDEV 规则并验证
编写完规则后，需重载规则并触发设备重新识别，使规则生效。

### 1. 重载 UDEV 规则
```bash
# 重新加载所有 UDEV 规则
sudo udevadm control --reload-rules
# 触发 UDEV 重新识别串口设备
sudo udevadm trigger
```

### 2. 验证规则生效结果
```bash
# 查看自定义符号链接是否创建成功
ls -l /dev/lidar*  # 替换为自定义的符号链接名称
```
正常输出示例（符号链接指向实际串口设备）：
```bash
lrwxrwxrwx 1 root root 7 Jan  5 10:08 /dev/lidar0 -> ttyUSB0
lrwxrwxrwx 1 root root 7 Jan  5 10:08 /dev/lidar1 -> ttyUSB1
```

### 3. 进阶验证（可选）
- 拔插设备后重新执行 `ls -l /dev/lidar*`，确认链接未漂移；
- 重启系统后验证链接仍存在；
- 测试程序访问 ` /dev/lidar0`/`/dev/lidar1`，确认设备可正常通信。

## 📌 常见问题与解决方案 ❗
???+ warning "常见问题：符号链接未创建"
    - 检查规则文件语法（如括号、等号、逗号是否正确，无多余空格）；
    - 确认 `idVendor`/`idProduct`/`devpath` 与设备实际属性一致；
    - 执行 `sudo udevadm test /sys/class/tty/ttyUSB0`（替换为实际设备），查看规则执行日志。

???+ warning "常见问题：权限不足，程序无法访问设备"
    - 规则中添加 `MODE="0666"`（所有用户可读写）；
    - 或将当前用户加入 `dialout` 组：`sudo usermod -aG dialout $USER`（需重启终端生效）。

???+ warning "常见问题：设备插拔后链接漂移"
    - 优先使用 `devpath` 序列号匹配（而非 `serial`）；
    - 确认物理端口未变更（树莓派/工控机的 USB 端口编号固定，避免插错端口）。

## 🎨 扩展场景：其他设备的 UDEV 配置
除激光雷达外，自主移动机器人的其他 USB 设备也可按相同逻辑配置：

### 示例1：IMU 模块（ttyACM 类型）
```bash
# 自主移动机器人 - IMU 模块（序列号 A1B2C3）
SUBSYSTEM=="tty", ATTRS{idVendor}=="2886", ATTRS{idProduct}=="0018", ATTRS{serial}=="A1B2C3", SYMLINK+="imu0", MODE="0666"
```

### 示例2：USB 转串口舵机控制器
```bash
# 自主移动机器人 - 舵机控制器（devpath=2.1）
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{devpath}=="2.1", SYMLINK+="servo_ctrl", MODE="0666"
```

## 📖 总结
UDEV 规则是保障自主移动机器人 USB 设备访问稳定性的核心手段，关键步骤可总结为：  
`识别设备属性 → 编写规则文件 → 重载规则 → 验证生效`  

通过合理选择 `serial`/`devpath`/`idVendor`/`idProduct` 等匹配项，可实现设备与固定符号链接的精准绑定，避免因路径漂移导致的程序异常，为机器人的稳定运行提供基础保障。

## 📚 参考资料
- 树莓派 USB 设备管理：https://www.raspberrypi.com/documentation/computers/configuration.html