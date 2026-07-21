## 📊 图表 1：系统总览拓扑图

展示三大控制器之间的通信架构。

```mermaid
flowchart LR
    subgraph HOST_LAYER ["🖥️ 主机层"]
        RPI["🖥️ RPI4B<br/>树莓派主控"]
    end
    
    subgraph CTRL_LAYER ["🎛️ 主控制器层"]
        VMX["🎛️ VMX-PI<br/>主控制器"]
    end
    
    subgraph DRIVE_LAYER ["⚙️ 驱动层"]
        TITAN1["⚙️ TiTan-1<br/>电机驱动"]
        TITAN2["⚙️ TiTan-2<br/>电机驱动"]
    end
    
    RPI ==>|SPI 总线| VMX
    VMX ==>|CAN 总线| TITAN1
    VMX ==>|CAN 总线| TITAN2
    
    style RPI fill:#e1f5fe,stroke:#0288d1,stroke-width:4px
    style VMX fill:#fff3e0,stroke:#f57c00,stroke-width:4px
    style TITAN1 fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
    style TITAN2 fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
```

---

## 📊 图表 2：RPI4B 接口详图

```mermaid
flowchart TD
    subgraph RPI4B ["🖥️ RPI4B 树莓派4B"]
        CPU["🧠 BCM2711<br/>四核 Cortex-A72"]
        MEM["💾 内存<br/>2/4/8GB"]
        OS["🐧 操作系统<br/>Linux/ROS"]
    end
    
    subgraph RPI_PORTS ["对外接口"]
        SPI_PORT["🔌 SPI 输出<br/>连接 VMX-PI"]
        USB["🔌 USB ×4"]
        ETH["🌐 以太网"]
        HDMI["📺 HDMI ×2"]
        WIFI["📶 WiFi/蓝牙"]
    end
    
    CPU --> SPI_PORT
    CPU --> USB & ETH & HDMI & WIFI
    
    style RPI4B fill:#e1f5fe,stroke:#0288d1,stroke-width:3px
    style SPI_PORT fill:#bbdefb,stroke:#1976d2,stroke-width:3px
```

---

## 📊 图表 3：VMX-PI 通信接口详图

```mermaid
flowchart TD
    subgraph VMXPI ["🎛️ VMX-PI 控制器"]
        VMX_CPU["🧠 VMX-PI 主控核心"]
    end
    
    subgraph UPSTREAM ["⬆️ 上行通信"]
        SPI_IN["📥 SPI 输入<br/>← RPI4B"]
    end
    
    subgraph DOWNSTREAM ["⬇️ 下行通信"]
        CAN1["📤 CAN 接口 1<br/>→ TiTan-1"]
        CAN2["📤 CAN 接口 2<br/>→ TiTan-2"]
    end
    
    subgraph EXPANSION ["🔌 扩展通信"]
        I2C["🔗 I2C 总线"]
        UART["🔗 UART 串口"]
        SPI_EXT["🔗 SPI 额外接口"]
    end
    
    VMX_CPU --> SPI_IN
    VMX_CPU --> CAN1 & CAN2
    VMX_CPU --> I2C & UART & SPI_EXT
    
    style VMXPI fill:#fff3e0,stroke:#f57c00,stroke-width:3px
    style SPI_IN fill:#ffe0b2,stroke:#fb8c00,stroke-width:3px
    style CAN1 fill:#ffe0b2,stroke:#fb8c00,stroke-width:2px
    style CAN2 fill:#ffe0b2,stroke:#fb8c00,stroke-width:2px
    style EXPANSION fill:#fff9c4,stroke:#f9a825,stroke-width:2px
```

---

## 📊 图表 4：VMX-PI 传感器接口详图

```mermaid
flowchart TD
    subgraph VMXPI ["🎛️ VMX-PI 控制器"]
        VMX_CPU["🧠 VMX-PI 主控"]
    end
    
    subgraph ENCODERS ["📍 板载编码器 ×4"]
        E1["⚙️ 编码器 1<br/>正交编码"]
        E2["⚙️ 编码器 2<br/>正交编码"]
        E3["⚙️ 编码器 3<br/>正交编码"]
        E4["⚙️ 编码器 4<br/>正交编码"]
    end
    
    subgraph ANALOG ["📏 AI 模拟量输入 ×4"]
        IR1["📡 红外测距 1"]
        IR2["📡 红外测距 2"]
        IR3["📡 红外测距 3"]
        IR4["📡 红外测距 4"]
    end
    
    subgraph ULTRASONIC ["🔊 超声波传感器 ×2"]
        US1["📶 超声波 1<br/>距离检测"]
        US2["📶 超声波 2<br/>距离检测"]
    end
    
    VMX_CPU --> E1 & E2 & E3 & E4
    VMX_CPU --> IR1 & IR2 & IR3 & IR4
    VMX_CPU --> US1 & US2
    
    style VMXPI fill:#fff3e0,stroke:#f57c00,stroke-width:3px
    style ENCODERS fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style ANALOG fill:#e0f7fa,stroke:#00838f,stroke-width:2px
    style ULTRASONIC fill:#e0f7fa,stroke:#00838f,stroke-width:2px
```

---

## 📊 图表 5：VMX-PI 控制输出与人机接口详图

```mermaid
flowchart TD
    subgraph VMXPI ["🎛️ VMX-PI 控制器"]
        VMX_CPU["🧠 VMX-PI 主控"]
    end
    
    subgraph PWM_OUT ["🎮 PWM 输出 ×5 (伺服舵机)"]
        PWM1["🤖 舵机 1"]
        PWM2["🤖 舵机 2"]
        PWM3["🤖 舵机 3"]
        PWM4["🤖 舵机 4"]
        PWM5["🤖 舵机 5"]
    end
    
    subgraph DI_IN ["🛑 DI 数字输入 ×2"]
        DI_EST["⚠️ 急停按钮<br/>紧急停止"]
        DI_START["▶️ 启动按键<br/>系统启动"]
    end
    
    subgraph DO_OUT ["💡 DO 数字输出 ×3"]
        LED1["💡 LED 灯 1<br/>状态指示"]
        LED2["💡 LED 灯 2<br/>状态指示"]
        LED3["💡 LED 灯 3<br/>状态指示"]
    end
    
    VMX_CPU --> PWM1 & PWM2 & PWM3 & PWM4 & PWM5
    VMX_CPU --> DI_EST & DI_START
    VMX_CPU --> LED1 & LED2 & LED3
    
    style VMXPI fill:#fff3e0,stroke:#f57c00,stroke-width:3px
    style PWM_OUT fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    style DI_IN fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    style DO_OUT fill:#f1f8e9,stroke:#558b2f,stroke-width:2px
```

---

## 📊 图表 6：TiTan-1 完整接口详图

```mermaid
flowchart TD
    subgraph TITAN1 ["⚙️ TiTan 控制器 1"]
        T1_CPU["🧠 TiTan-1 主控"]
        T1_CAN["🔗 CAN 接口<br/>← VMX-PI"]
    end
    
    subgraph T1_MOTORS ["🔧 电机驱动组 ×4"]
        T1_M1["🔧 电机 M1"]
        T1_M2["🔧 电机 M2"]
        T1_M3["🔧 电机 M3"]
        T1_M4["🔧 电机 M4"]
    end
    
    subgraph T1_ENCODERS ["📍 编码器组 ×4 (正交)"]
        T1_E1["⚙️ 编码器 1"]
        T1_E2["⚙️ 编码器 2"]
        T1_E3["⚙️ 编码器 3"]
        T1_E4["⚙️ 编码器 4"]
    end
    
    subgraph T1_LIMITS_H ["🚧 H 限位开关 ×4"]
        T1_H1["🛑 M1-H 上限位"]
        T1_H2["🛑 M2-H 上限位"]
        T1_H3["🛑 M3-H 上限位"]
        T1_H4["🛑 M4-H 上限位"]
    end
    
    subgraph T1_LIMITS_L ["🚧 L 限位开关 ×4"]
        T1_L1["🛑 M1-L 下限位"]
        T1_L2["🛑 M2-L 下限位"]
        T1_L3["🛑 M3-L 下限位"]
        T1_L4["🛑 M4-L 下限位"]
    end
    
    T1_CAN --> T1_CPU
    T1_CPU --> T1_M1 & T1_M2 & T1_M3 & T1_M4
    T1_M1 -.附属.-> T1_E1
    T1_M2 -.附属.-> T1_E2
    T1_M3 -.附属.-> T1_E3
    T1_M4 -.附属.-> T1_E4
    T1_CPU --> T1_H1 & T1_H2 & T1_H3 & T1_H4
    T1_CPU --> T1_L1 & T1_L2 & T1_L3 & T1_L4
    
    style TITAN1 fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
    style T1_MOTORS fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style T1_ENCODERS fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style T1_LIMITS_H fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    style T1_LIMITS_L fill:#ffebee,stroke:#d32f2f,stroke-width:2px
```

---

## 📊 图表 7：TiTan-2 完整接口详图

```mermaid
flowchart TD
    subgraph TITAN2 ["⚙️ TiTan 控制器 2"]
        T2_CPU["🧠 TiTan-2 主控"]
        T2_CAN["🔗 CAN 接口<br/>← VMX-PI"]
    end
    
    subgraph T2_MOTORS ["🔧 电机驱动组 ×4"]
        T2_M1["🔧 电机 M1"]
        T2_M2["🔧 电机 M2"]
        T2_M3["🔧 电机 M3"]
        T2_M4["🔧 电机 M4"]
    end
    
    subgraph T2_ENCODERS ["📍 编码器组 ×4 (正交)"]
        T2_E1["⚙️ 编码器 1"]
        T2_E2["⚙️ 编码器 2"]
        T2_E3["⚙️ 编码器 3"]
        T2_E4["⚙️ 编码器 4"]
    end
    
    subgraph T2_LIMITS_H ["🚧 H 限位开关 ×4"]
        T2_H1["🛑 M1-H 上限位"]
        T2_H2["🛑 M2-H 上限位"]
        T2_H3["🛑 M3-H 上限位"]
        T2_H4["🛑 M4-H 上限位"]
    end
    
    subgraph T2_LIMITS_L ["🚧 L 限位开关 ×4"]
        T2_L1["🛑 M1-L 下限位"]
        T2_L2["🛑 M2-L 下限位"]
        T2_L3["🛑 M3-L 下限位"]
        T2_L4["🛑 M4-L 下限位"]
    end
    
    T2_CAN --> T2_CPU
    T2_CPU --> T2_M1 & T2_M2 & T2_M3 & T2_M4
    T2_M1 -.附属.-> T2_E1
    T2_M2 -.附属.-> T2_E2
    T2_M3 -.附属.-> T2_E3
    T2_M4 -.附属.-> T2_E4
    T2_CPU --> T2_H1 & T2_H2 & T2_H3 & T2_H4
    T2_CPU --> T2_L1 & T2_L2 & T2_L3 & T2_L4
    
    style TITAN2 fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
    style T2_MOTORS fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style T2_ENCODERS fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style T2_LIMITS_H fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    style T2_LIMITS_L fill:#ffebee,stroke:#d32f2f,stroke-width:2px
```

---

## 📊 图表 8：通信数据流向图

```mermaid
flowchart LR
    subgraph HOST ["🖥️ 主机"]
        RPI["RPI4B<br/>上层控制"]
    end
    
    subgraph SPI_BUS ["SPI 总线"]
        SPI[("🔗 SPI<br/>高速同步")]
    end
    
    subgraph MAIN_CTRL ["🎛️ 主控制器"]
        VMX["VMX-PI<br/>中间协调"]
    end
    
    subgraph CAN_BUS ["CAN 总线"]
        CAN[("🔗 CAN<br/>工业现场")]
    end
    
    subgraph DRIVERS ["⚙️ 驱动器"]
        T1["TiTan-1<br/>电机0-3"]
        T2["TiTan-2<br/>电机4-7"]
    end
    
    RPI <==>|控制指令<br/>状态反馈| SPI
    SPI <==>|命令下发<br/>数据上报| VMX
    VMX <==>|运动指令<br/>状态回传| CAN
    CAN <==>|驱动命令<br/>编码器数据| T1
    CAN <==>|驱动命令<br/>编码器数据| T2
    
    style RPI fill:#e1f5fe,stroke:#0288d1,stroke-width:3px
    style VMX fill:#fff3e0,stroke:#f57c00,stroke-width:3px
    style T1 fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
    style T2 fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
    style SPI_BUS fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style CAN_BUS fill:#fff9c4,stroke:#f9a825,stroke-width:2px
```

---

## 📋 完整接口汇总表

| 控制器 | 接口类型 | 数量 | 用途 |
|--------|----------|------|------|
| **RPI4B** | SPI 输出 | 1 | 连接 VMX-PI |
| **VMX-PI** | SPI 输入 | 1 | 连接 RPI4B |
| | CAN 输出 | 2 | 连接 TiTan-1/2 |
| | I2C / UART / SPI | 各1 | 扩展通信 |
| | 板载编码器 | 4 | 正交编码 |
| | AI 红外测距 | 4 | 模拟量输入 |
| | 超声波传感器 | 2 | 距离检测 |
| | PWM 舵机 | 5 | 伺服控制 |
| | DI (急停+启动) | 2 | 数字输入 |
| | DO (LED) | 3 | 数字输出 |
| **TiTan×2** | CAN 接口 | 各1 | 连接 VMX-PI |
| | 电机驱动 | 各4 | 伺服电机 |
| | 编码器 | 各4 | 正交(附属电机) |
| | 限位开关 | 各8 | H限位+L限位 |

---

