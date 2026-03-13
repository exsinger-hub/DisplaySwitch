以下是为您整理好的完整 Markdown 源码。您可以直接将其复制并保存为 `README.md`。

```markdown
# 🖥️ DisplaySwitch: 极简双机切换指南

[![macOS](https://img.shields.io/badge/macOS-兼容-blue)](#)
[![DDC/CI](https://img.shields.io/badge/DDC/CI-支持-green)](#)
[![License](https://img.shields.io/badge/免费开源-MIT-lightgrey)](#)

这是一个专为 Mac 用户设计的超轻量级工具，只需一行命令，即可在 Mac 和 Windows/Linux 之间无缝切换显示器输入源。

---

## 📝 完整教程

### 第一步：安装必需工具

```bash
# 先安装 Homebrew（如果已有可跳过）
/bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"

# 根据芯片类型安装工具：
brew install m1ddc  # Apple Silicon (M1/M2/M3) 芯片用这个
# 或
brew install ddcctl # Intel 芯片用这个

```

### 第二步：获取显示器 ID

运行以下命令查看你的显示器信息：

```bash
m1ddc display list

```

*输出示例：*
`[1] VG2781-4K (159EAB43-F0E2-4E00-AA39-EFD114DDB050)`

> **记下括号内的 UUID**，后面配置脚本时需要用到。

### 第三步：测试通道值

手动发送指令确认显示器的输入源通道：

```bash
# 测试切换到 Type-C (通常通道值为 15)
sudo m1ddc display 你的UUID set input 15

# 测试切换到 DP (通常通道值为 16)
sudo m1ddc display 你的UUID set input 16

```

> **💡 提示**：如果 `15` 或 `16` 无效，请尝试 `17`、`18`、`27` 等值，直到找到正确的对应关系。

### 第四步：创建切换脚本

我们将创建两个核心命令：`tomac` 和 `towin`。

**1. 创建 `tomac` 命令：**

```bash
nano ~/tomac

```

粘贴以下内容（记得替换 UUID）：

```bash
#!/bin/bash
DISPLAY_ID="你的显示器UUID"
sudo m1ddc display $DISPLAY_ID set input 15
echo 15 > ~/.display_state
osascript -e 'display notification "已切换到 Mac 💻" with title "显示器切换"'
echo "✅ 已切换到 Mac (Type-C)"

```

**2. 创建 `towin` 命令：**

```bash
nano ~/towin

```

粘贴以下内容：

```bash
#!/bin/bash
DISPLAY_ID="你的显示器UUID"
sudo m1ddc display $DISPLAY_ID set input 16
echo 16 > ~/.display_state
osascript -e 'display notification "已切换到 Windows 🪟" with title "显示器切换"'
echo "✅ 已切换到 Windows (DP)"

```

### 第五步：设置权限和全局可用

让脚本在系统的任何地方都能被直接调用：

```bash
# 赋予执行权限
chmod +x ~/tomac ~/towin

# 建立软链接到系统路径
sudo ln -s ~/tomac /usr/local/bin/tomac
sudo ln -s ~/towin /usr/local/bin/towin

```

### 第六步：配置免密码执行 (关键步骤!)

为了实现“秒切”而不需要每次输入 sudo 密码：

```bash
sudo visudo

```

在文件末尾添加以下内容（**将 `username` 换成你的 Mac 用户名**）：

```text
username ALL=(ALL) NOPASSWD: /usr/local/bin/m1ddc

```

*保存并退出：按 `Ctrl+O` → `Enter` 保存，再按 `Ctrl+X` 退出。*

---

## 🎯 使用方式

现在，你可以在任何终端窗口直接输入：

* **切换到 Mac**: `tomac`
* **切换到 Windows**: `towin`

---

## 🚀 进阶技巧

### 1. 使用更短的别名

如果你觉得四个字母还是太长，可以配置两字母简写：

1. 编辑配置文件：`nano ~/.zshrc`
2. 在末尾添加：
```bash
alias tom="tomac"
alias tow="towin"

```


3. 使配置生效：`source ~/.zshrc`

**现在，只需输入 `tom` 或 `tow` 即可完成切换！**

### 2. 结合 Hammerspoon 绑定快捷键

如果你安装了 [Hammerspoon](https://www.hammerspoon.org/)，可以在 `init.lua` 中添加以下代码实现一键切换：

```lua
hs.hotkey.bind({"cmd", "alt"}, "M", function() hs.execute("tomac") end)
hs.hotkey.bind({"cmd", "alt"}, "W", function() hs.execute("towin") end)

```

---

## ⚠️ 注意事项

* **DDC/CI**: 确保显示器 OSD 菜单中已开启 DDC/CI 功能。
* **休眠限制**: 如果切换到 Windows 后，Mac 进入深度睡眠，`tomac` 命令将无法在 Mac 端执行（除非使用 Windows 端对应的 DDC 工具）。
* **线材质量**: 部分劣质 HDMI 线可能不支持 DDC 协议。

---

**Happy Switching! 🎮💻🎨**

```

如果您需要我针对特定的显示器品牌（如戴尔、LG 等）补充更详细的通道值表格，请随时告诉我！

```
