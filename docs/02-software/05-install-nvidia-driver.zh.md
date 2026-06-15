# Ubuntu NVIDIA 显卡驱动环境配置
---
## 显卡要求
仅支持 **NVIDIA 系列显卡**，非 NVIDIA 显卡无法使用本流程，支持的显卡包括但不限于：

- RTX 40 系列  
  - RTX 4090 D  
  - RTX 4080 系列  
  - RTX 4070 系列  
  - RTX 4060 系列  
- RTX 30 系列  
- RTX 20 系列  
- GTX 16 系列  

---

## 一、NVIDIA 显卡驱动安装

### 1.1 安装前准备（禁用 nouveau）

Ubuntu 自带开源显卡驱动 `nouveau` 会与 NVIDIA 官方驱动冲突，需在安装前禁用。

#### 1.1.1 检查 nouveau 状态
```bash
lsmod | grep nouveau
```

- **有输出**：nouveau 正在运行，需禁用  
- **无输出**：nouveau 已禁用，可跳过本节

#### 1.1.2 禁用 nouveau
编辑黑名单配置文件：
```bash
sudo gedit /etc/modprobe.d/blacklist.conf
```

在文件末尾追加以下内容并保存：
```text
blacklist nouveau
options nouveau modeset=0
```

#### 1.1.3 应用配置并重启
```bash
sudo update-initramfs -u
sudo reboot
```

---

### 1.2 安装 NVIDIA 驱动

#### 1.2.1 添加驱动仓库并更新
```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

#### 1.2.2 安装基础编译依赖
```bash
sudo apt-get install gcc g++ make
```

#### 1.2.3 安装驱动管理工具
```bash
sudo apt install ubuntu-drivers-common
```

#### 1.2.4（可选）查看 GPU 硬件信息
```bash
lspci | grep -i nvidia
lshw -numeric -C display
```

#### 1.2.5 查看系统推荐驱动
```bash
ubuntu-drivers devices
```

> 📌 **选择建议**  
> - 消费级显卡：**不建议**选择名称中带 `open` 的驱动  
> - 桌面/图形化环境：**不要**选择带 `server` 的驱动  
> - 优先选择带有 **`recommended`** 标签的版本

#### 1.2.6 安装推荐驱动
将 `<version>` 替换为实际推荐版本号（示例为 `570`）：
```bash
sudo apt install nvidia-driver-<推荐的驱动版本号>
# 示例
sudo apt install nvidia-driver-570
```

#### 1.2.7 重启并验证安装
```bash
sudo reboot
nvidia-smi
```

✅ **验证标准**：  
若终端输出 NVIDIA GPU 信息表，说明驱动安装成功。

---

### 1.3 驱动卸载（异常或重装时使用）

如需清理已安装的 NVIDIA 驱动，可执行以下任一命令：
```bash
sudo apt-get remove --purge nvidia*
```
或
```bash
sudo apt purge nvidia-* -y
```

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