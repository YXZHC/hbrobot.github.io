# 安装与配置 Git
---
## 一、Git 简介
Git 是目前最主流的**分布式版本控制系统（DVCS）**，用于：

- 管理源代码版本
- 多人协作开发
- 追踪代码变更历史
- 支持分支、合并、回滚等操作

Git 具有以下特点：

- ✅ 完全开源、免费
- ✅ 本地仓库速度快、效率高
- ✅ 支持离线工作
- ✅ 广泛应用于企业、高校与开源社区


---

## 二、安装 Git

### 2.1 安装方式（Ubuntu）
在 Ubuntu 环境下，推荐使用 `apt` 进行安装：

```bash
sudo apt update
sudo apt install git
```

### 2.2 验证安装结果
安装完成后，执行以下命令检查 Git 是否可用：

```bash
git --version
```

✅ 预期输出示例：
```
git version 2.x.x
```

---

## 三、Git 基础配置（首次必做）

> ⚠️ 新环境首次使用 Git **必须完成以下配置**，否则无法正常提交代码。

### 3.1 配置用户名和邮箱
Git 每次提交都会记录提交者的身份信息：

```bash
git config --global user.name "你的姓名"
git config --global user.email "你的邮箱地址"
```

📌 示例：
```bash
git config --global user.name "Zhang San"
git config --global user.email "zhangsan@example.com"
```

### 3.2 查看当前配置
```bash
git config --list
```

---

## 四、Git 常用基础命令速查

| 功能 | 命令 |
|----|----|
| 查看帮助 | `git --help` |
| 初始化仓库 | `git init` |
| 克隆远程仓库 | `git clone <仓库地址>` |
| 查看状态 | `git status` |
| 添加文件到暂存区 | `git add <文件名>` |
| 提交更改 | `git commit -m "提交说明"` |
| 推送至远程仓库 | `git push` |
| 拉取远程更新 | `git pull` |
| 查看提交历史 | `git log` |

---

## 五、学习建议（教育用途）

### 5.1 官方文档（推荐）
- [Git 官网](https://git-scm.com/)：[https://git-scm.com/](https://git-scm.com/)
- [Pro Git 中文电子书](https://git-scm.com/book/zh/v2)：[https://git-scm.com/book/zh/v2](https://git-scm.com/book/zh/v2)

### 5.2 入门学习路径（建议顺序）
1. Git 是什么 & 为什么要用 Git
2. 工作区 / 暂存区 / 版本库概念
3. 基本提交流程（`add → commit → push`）
4. 分支管理与合并
5. 解决冲突的基本方法

---

## 六、常见问题（SOP 提示）

- ❓ **提交时报错未设置用户名/邮箱**  
  → 执行第 3.1 节的用户名与邮箱配置

- ❓ **clone 速度慢或失败**  
  → 检查网络连接或更换仓库镜像源

- ❓ **权限拒绝（Permission denied）**  
  → 确认 SSH key 已正确配置并添加到 Git 平台

---

✅ **本章目标达成标准**：
- 能成功安装 Git
- 能正确配置用户信息
- 能使用 Git 完成一次完整的 `clone → modify → commit → push` 流程

---

## 👥 贡献者

<a href="https://github.com/yxzhc"><img src="https://avatars.githubusercontent.com/u/80094007" width="70"></a>

<br>
本项目离不开每一位提交 PR、提 Issue、优化文档的开发者，由衷致谢！

![](../images/61d7f15b093ae.png)
