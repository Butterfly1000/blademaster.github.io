# macOS 环境下运行老旧 Node.js 项目的完整指南

## 引言：问题的起点

在软件开发中，我们经常会遇到需要维护或运行一些“有年头”的老项目的情况。当你满怀信心地在一个全新的、配置先进的 macOS 环境下，clone 下一个老项目，并执行标志性的 `npm install` 命令时，往往会遇到一连串令人沮喪的红色错误。

我们这次的经历就是一个典型的例子：

1.  项目本身是一个基于 Hexo 的静态博客。
2.  在 `npm install` 时，一个名为 `canvas` 的依赖包编译失败。
3.  错误日志最初指向 C++ 编译错误，根本原因是系统安装的 **Node.js 版本 (v23) 太新**，与项目陈旧的依赖 (`canvas`, `nan` 等) 无法兼容。

这个场景引出了一个核心的维护哲学问题：**我们是应该花费巨大精力去升级改造老项目来适应新环境，还是应该让环境去“迁就”老项目？**

对于绝大多数维护场景，后者是成本最低、风险最小、效率最高的选择。

---

## 核心理念：改造环境，而非项目

### 1. 为什么不建议升级项目依赖？

直接升级 `package.json` 中的老旧依赖（如 `canvas`）听起来很直接，但这通常会触发一个可怕的“**依赖多米诺骨牌**”效应：

*   **升级 `canvas`** -> 可能需要新版的 `hexo` 核心。
*   **升级 `hexo`** -> 可能导致现有的主题 (`butterfly`) 和所有插件全部不兼容。
*   **升级主题和插件** -> 可能需要重写大量配置文件和文章模板。

这个过程就像一个无底洞，为了解决一个问题，引入了十个新问题。最终结果往往是，花费了大量时间，却把项目改得面目全非，甚至无法运行。

### 2. 什么是 nvm？为什么它是最佳方案？

`nvm` (Node Version Manager) 是专门为此类场景而生的“神器”。它是一个 **Node.js 的版本管理器**。

你可以把它想象成一个“Node.js 应用商店”：

*   **安装多个版本**：允许你在同一台电脑上安装任意多个不同版本的 Node.js (例如 v16, v18, v20)。
*   **自由切换**：通过一条简单的命令，就能在这些版本间“瞬间”切换。
*   **环境隔离**：为不同项目启用不同 Node.js 版本，互不干扰。

使用 `nvm`，我们就可以在不改动项目一行业务代码的情况下，为它精准地创建一个它所需要的、兼容的运行环境。

---

## 实战步骤：从零到一的完整流程

以下是我们解决本次问题的完整、详细的步骤复盘。

### 第一步：环境清理 - 卸载 Homebrew 管理的 Node.js

**目标**：避免 `nvm` 和 Homebrew 两套管理体系互相“打架”。

**原因**：如果您通过 Homebrew (`brew install node`) 安装了 Node.js，又同时使用 `nvm`，会导致系统路径混乱，您无法确定当前生效的 `node` 命令究竟来自哪里。因此，最干净的做法是，让 `nvm` 全权接管。

**命令**：
```bash
# 1. 查询当前 Node.js 版本（以便日后恢复）
node -v
# 我们的记录是：v23.4.0

# 2. 使用 brew 卸载 node
brew uninstall node
```
**说明**：请放心，此操作只会移除系统级的 `node` 和 `npm` 命令，**不会**删除您任何项目中的 `node_modules` 文件夹。

### 第二步：安装版本管理器 - nvm

**目标**：安装 `nvm` 工具本身。

**原因**：在 macOS 上，通过 Homebrew 安装 `nvm` 是最便捷、最推荐的方式。

**命令**：
```bash
brew install nvm
```

### 第三步：一次性配置 nvm

**目标**：让终端 (Shell) 知道如何加载并使用 `nvm`。

**背景**：安装 `nvm` 后，还需要告诉您的终端在每次启动时去加载它。macOS 用户主要使用 `zsh` 或 `bash` 两种终端，它们的配置文件不同。为了确保万无一失，我们对两个配置文件都进行配置。

**命令**：
```bash
# 1. 创建 nvm 用来存放 Node.js 版本的目录
mkdir ~/.nvm

# 2. 将 nvm 的加载脚本写入 zsh 的配置文件 (~/.zshrc)
echo '
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"
' >> ~/.zshrc

# 3. 将同样的脚本也写入 bash 的配置文件 (~/.bash_profile)
echo '
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"
' >> ~/.bash_profile

# 4. 立即“刷新”当前终端的配置，使其马上生效
# (根据您当前正在使用的终端类型，执行其中一个)
source ~/.zshrc
# 或者
source ~/.bash_profile
```

### 第四步：安装并切换 Node.js 版本

**目标**：安装一个与老项目兼容的 Node.js 版本。

**原因**：我们的目标是解决 `canvas` 的编译问题，Node.js v16 是一个非常稳定且兼容性好的长期支持版 (LTS)。

**命令**：
```bash
# 1. 安装 Node.js v16 的最新版
nvm install 16

# 2. 切换到该版本 (通常在安装后会自动切换，此为手动确认)
nvm use 16
```
执行后，您会看到 `Now using node v16.x.x (npm v8.x.x)` 的提示。

### 第五步：解决深层编译依赖

**背景**：在解决了最主要的 Node.js 版本问题后，我们有时会遇到更深层次的、与 C/C++ 编译环境相关的依赖问题。我们的经历就遇到了两个典型。

#### 5.1 Python 环境问题：`ModuleNotFoundError: No module named 'distutils'`

*   **原因**：`node-gyp` (Node.js 的编译工具) 需要调用 Python。但从 Python 3.12 开始，`distutils` 这个标准库被移除了。而您系统中的 Python 版本 (v3.13) 过新，导致 `node-gyp` 无法工作。
*   **解决方案**：为新版 Python 手动装回 `distutils` 的功能。这可以通过安装 `setuptools` 包来实现。
*   **命令**：
    ```bash
    # 使用 --break-system-packages 旗标来绕过 brew 的环境隔离保护机制
    pip3 install setuptools --break-system-packages
    ```

#### 5.2 系统库依赖问题：`requires 'libffi >= 3.0.0' but version of libffi is 2.1`

*   **原因**：`canvas` 的编译不仅依赖 Node 和 Python，还依赖一系列系统底层的图形库。错误日志明确指出，`libffi` 这个库的版本太旧了。
*   **解决方案**：使用 Homebrew 将 `canvas` 所需的所有系统库一次性更新到最新版本。
*   **命令**：
    ```bash
    brew install pkg-config cairo pango libpng jpeg giflib librsvg libffi
    ```

### 第六步：最终安装与运行

**目标**：在万事俱备的环境下，成功安装项目依赖并运行项目。

**命令**：
```bash
# 1. 回到项目根目录，执行最终的安装命令
npm install

# 2. 启动 Hexo 本地开发服务器
hexo s
```
执行 `hexo s` 后，终端会提示 `Hexo is running at http://localhost:4000/`。此时，您就可以在浏览器中打开这个地址，预览您久违的博客了！

---

## 总结与日常使用

通过这一系列操作，我们不仅成功运行了项目，更重要的是建立了一套强大而灵活的开发环境。

**常用 `nvm` 命令**：

*   `nvm ls`：列出所有已安装的 Node.js 版本。
*   `nvm use <版本号>`：切换到指定版本。
*   `nvm install <版本号>`：安装新的版本。
*   `nvm alias default <版本号>`：设置默认的 Node.js 版本。

**如何恢复到最新的 Node.js 环境？**

当您完成了对老项目的维护，想切换回最新的 Node.js 环境时，只需要：
```bash
# 安装我们最初记录的版本
nvm install 23.4.0

# 切换并使用
nvm use 23.4.0
```
这就是 `nvm` 的魅力所在。希望这份指南对您有所帮助！
