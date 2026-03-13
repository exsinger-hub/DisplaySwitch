# 🖥️ DisplaySwitch

[![macOS](https://img.shields.io/badge/macOS-兼容-blue)](#)
[![DDC/CI](https://img.shields.io/badge/DDC/CI-支持-green)](#)
[![License](https://img.shields.io/badge/免费开源-MIT_License-lightgrey)](#)

**DisplaySwitch** 是一个简单高效的命令行工具，让你只需一行命令即可在 Mac 和 Windows/Linux 之间无缝切换显示器输入源。告别繁琐的显示器物理按键，拥抱丝滑的工作流！

## ✨ 特性

* **🚀 一键切换**：输入 `tomac` 切到 Mac，输入 `towin` 切到 Windows。
* **🔔 即时反馈**：提供桌面通知和终端输出的双重确认。
* **🔧 零成本**：完全免费，无需购买额外的 KVM 硬件切换器。
* **📱 状态同步**：自动记录并管理当前的输入源状态。
* **💻 跨版本支持**：全面兼容 Apple Silicon (M1/M2/M3) 和 Intel Mac。
* **🎯 即装即用**：一次简单配置，永久有效。

## 📋 兼容性

| 组件 | 要求 |
| :--- | :--- |
| **操作系统** | macOS 10.15+ |
| **显示器** | 必须支持 **DDC/CI 协议**（绝大多数现代显示器均支持） |
| **主机 1** | Mac (Type-C / Thunderbolt) |
| **主机 2** | Windows / Linux (HDMI / DP) |

> **⚠️ 注意**：使用前请务必确认显示器 OSD 菜单设置中已开启 **DDC/CI** 选项。

---

## 🚀 快速开始

### 1. 安装依赖

```bash
# 安装 Homebrew (如未安装)
/bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"

# 根据芯片类型安装工具：
# 👉 对于 Apple Silicon (M1/M2/M3):
brew install m1ddc

# 👉 对于 Intel Mac:
brew install ddcctl
```

### 2. 获取显示器 ID

```bash
m1ddc display list
```
*输出示例：*
`[1] VG2781-4K (159EAB43-F0E2-4E00-AA39-EFD114DDB050)`

请记下括号内的显示器 UUID（如：`159EAB43-F0E2-4E00-AA39-EFD114DDB050`）。

### 3. 测试通道值

```bash
# 测试切换到 Mac (假设为 Type-C，通常通道值为 15)
sudo m1ddc display YOUR_UUID set input 15

# 测试切换到 Windows (假设为 DP，通常通道值为 16) 
sudo m1ddc display YOUR_UUID set input 16
```
> **💡 提示**：如果 `15` 或 `16` 无效，请尝试 `17`、`18`、`27` 等值，或使用 Windows 端的 `ControlMyMonitor` 工具查看具体通道对应的值。

### 4. 部署与配置

```bash
# 克隆仓库
git clone [https://github.com/yourusername/DisplaySwitch.git](https://github.com/yourusername/DisplaySwitch.git)
cd DisplaySwitch

# 编辑配置文件
nano config.sh
```

**`config.sh` 配置示例：**
```bash
#!/bin/bash
# 显示器配置
DISPLAY_ID="YOUR_DISPLAY_UUID"   # 替换为你的显示器 UUID
TYPE_C_PORT=15                   # 切换到 Mac 的接口通道值
DP_PORT=16                       # 切换到 Windows 的接口通道值
NOTIFICATIONS=true               # 是否开启桌面通知
```

### 5. 执行安装

```bash
# 赋予执行权限
chmod +x install.sh tomac towin

# 运行安装脚本
sudo ./install.sh
```
*安装脚本将自动为你配置 sudo 免密执行、创建系统环境变量符号链接，并验证安装结果。*

---

## 🎯 使用方法

### 基本命令

打开终端，直接输入：
```bash
tomac  # 切换显示器到 Mac
towin  # 切换显示器到 Windows
```

*成功输出示例：*
```text
✅ 已切换到 Mac (Type-C)
📢 桌面通知：已切换到 Mac 💻
```

### 状态管理

工具会在本地维护当前显示器的状态记录。
```bash
# 查看当前状态
cat ~/.display_state

# 手动修正状态（如果你使用了物理按键切屏，导致状态不同步）
echo 15 > ~/.display_state  # 强制标记为 Mac
echo 16 > ~/.display_state  # 强制标记为 Windows
```

---

## ⚙️ 配置说明

所有核心配置均在 `config.sh` 中管理：

| 参数 | 说明 | 默认值 |
| :--- | :--- | :--- |
| `DISPLAY_ID` | 显示器 UUID | **必需填写** |
| `TYPE_C_PORT` | 连接 Mac 的接口通道值 | `15` |
| `DP_PORT` | 连接 Win/Linux 的接口通道值 | `16` |
| `NOTIFICATIONS` | 是否启用 macOS 桌面通知 | `true` |

---

## 🔧 高级功能与玩法

### 快捷键集成 (推荐)
结合 Hammerspoon，你可以实现全局快捷键秒切屏幕，彻底解放终端！

```bash
# 1. 安装 Hammerspoon
brew install --cask hammerspoon

# 2. 写入快捷键配置 (示例：Cmd+Alt+M 切 Mac，Cmd+Alt+W 切 Win)
echo 'hs.hotkey.bind({"cmd", "alt"}, "M", function() hs.execute("tomac") end)' >> ~/.hammerspoon/init.lua
echo 'hs.hotkey.bind({"cmd", "alt"}, "W", function() hs.execute("towin") end)' >> ~/.hammerspoon/init.lua
```

### 与 Barrier / Synergy 键鼠共享集成
在两台电脑安装 [Barrier](https://github.com/debauchee/barrier)（Mac 作为服务端，Windows 作为客户端）。结合 DisplaySwitch，你可以实现真正的**一套键鼠无缝穿梭控制两台主机**的终极体验！

---

## 🐛 故障排除

| 问题现象 | 解决方案 |
| :--- | :--- |
| **命令不存在 (Command not found)** | 请回到项目目录，重新运行 `sudo ./install.sh`。 |
| **切换无反应** | 1. 检查显示器设置中的 DDC/CI 是否开启。<br>2. 确认目标电脑处于唤醒状态。<br>3. 检查通道值是否正确。 |
| **执行时仍需要输入密码** | 脚本可能未能成功修改 sudoers，请检查 `/etc/sudoers` 配置。 |
| **切换后黑屏** | 目标电脑可能已休眠，请先敲击键盘或移动鼠标唤醒目标电脑。 |

**常用诊断命令：**
```bash
# 测试 DDC/CI 通信是否正常
sudo m1ddc display YOUR_UUID get input

# 检查可执行文件是否已正确链接
ls -la /usr/local/bin/tomac
ls -la /usr/local/bin/towin

# 检查 sudo 免密权限
sudo visudo -c
```

---

## 📁 项目结构

```text
DisplaySwitch/
├── README.md          # 说明文档
├── install.sh         # 安装脚本
├── config.sh          # 配置文件
├── tomac              # 切换到 Mac 核心脚本
├── towin              # 切换到 Windows 核心脚本
├── uninstall.sh       # 卸载脚本
└── examples/          # 进阶玩法示例
    ├── hammerspoon/   # 快捷键配置脚本
    └── barrier/       # 键鼠共享搭配指南
```

---

## 🔄 更新与卸载

**更新到最新版：**
```bash
git pull origin main
sudo ./install.sh
```

**完全卸载：**
```bash
sudo ./uninstall.sh
```

---

## 🤝 贡献指南

我们非常欢迎提交 Issue 和 Pull Request 来完善这个项目！
1. Fork 本仓库
2. 创建你的功能分支：`git checkout -b feature/AmazingFeature`
3. 提交你的更改：`git commit -m 'Add some AmazingFeature'`
4. 推送到分支：`git push origin feature/AmazingFeature`
5. 开启一个 Pull Request

---

## 📄 许可证

本项目基于 [MIT 许可证](LICENSE) 开源。

## 🙏 致谢

* [m1ddc](https://github.com/waydabber/m1ddc) - Apple Silicon DDC/CI 控制核心
* [ddcctl](https://github.com/kfix/ddcctl) - Intel Mac DDC/CI 控制核心
* 感谢所有参与测试和提供反馈的社区贡献者！

> 如果你觉得这个项目对你有帮助，请给个 **⭐ Star** 支持一下！有任何问题，欢迎在 Issues 留言交流。
> 
> **Happy Switching! 🎮💻🎨**

---

### 📝 更新日志

**v1.0.0 (2024-03-20)**
* 🚀 初始版本发布
* 💻 支持 Apple Silicon 和 Intel Mac
* 🔄 引入自动状态管理机制
* 🔔 增加 macOS 桌面通知集成功能
