---
title: MCP安装指南
date: 2025-07-04 16:02:09
tags:
- MCP
- Cursor
- 环境配置
- Node.js
- Python
categories: 技术
---

# MCP 安装与故障排查指南

本指南旨在记录在 macOS 环境下安装和配置模型上下文协议 (MCP) 服务器时遇到的常见问题及其解决方案。核心原则是：**为保证稳定性，在 `mcp.json` 配置文件中应始终使用可执行文件的绝对路径**。

## 1. 前置环境准备

MCP 服务器主要依赖于本地的 `Node.js` 和 `Python` 环境。

### Node.js 环境

建议安装 `22.0.0` 或更高版本。

```bash
# 查看版本
node -v

# 使用 Homebrew 安装/升级
brew install node
```

#### 常见问题：Homebrew 提示 `node not installed`

- **问题描述**: 执行 `brew upgrade node` 时，系统报错 `Error: node not installed`，但 `node -v` 却能正常显示版本号。
- **原因**: 这通常意味着 Node.js 是通过其他方式（例如，官方安装包）安装的，而不是 Homebrew。Homebrew 无法管理非其安装的软件。
- **解决方案**: 卸载已有的 Node.js，然后通过 Homebrew重新安装，以实现统一管理。

  ```bash
  # 1. 查找现有 node 的安装路径
  which node 
  # 示例输出: /usr/local/bin/node

  # 2. 手动删除相关文件
  sudo rm -rf /usr/local/bin/node
  sudo rm -rf /usr/local/lib/node_modules
  sudo rm -rf /usr/local/share/man/man1/node.1

  # 3. 使用 Homebrew 重新安装
  brew install node

  # 4. 验证安装
  node -v
  ```

### Python 环境

建议安装 `3.10` 或更高版本。

```bash
# 查看版本
python3 -V
# 示例输出: Python 3.13.1

# 使用 Homebrew 安装/升级
brew install python
```

---

## 2. `sequential-thinking` 服务配置

此服务用于提供深度思考能力。

### 初始配置 (npx)

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking@latest"
      ]
    }
  }
}
```

### 问题：服务状态灯为橙色或红色

- **问题描述**: 配置后，Cursor 中的服务状态灯不显示为绿色，表示服务未成功启动或未通过健康检查。
- **原因**: `npx` 在执行带有 `@latest` 标签的包时，会强制进行网络检查以获取最新版本。这个过程可能引入数百毫秒甚至数秒的延迟，导致 Cursor 的健康检查超时。
- **解决方案**: 绕过 `npx` 和网络检查，直接使用 `node` 运行服务的脚本文件，并提供完整的绝对路径。

#### 解决步骤

1.  **找到 `node` 的绝对路径**:

    ```bash
    which node
    # 示例输出: /opt/homebrew/bin/node
    ```

2.  **找到服务脚本的绝对路径**:
    `mcp-server-sequential-thinking` 命令只是一个符号链接（快捷方式）。我们需要找到它指向的真实脚本。

    ```bash
    # 首先，全局安装包以获得该命令
    npm install -g @modelcontextprotocol/server-sequential-thinking

    # 然后，查看符号链接指向
    ls -l $(which mcp-server-sequential-thinking)
    # 示例输出: ... -> ../lib/node_modules/@modelcontextprotocol/server-sequential-thinking/dist/index.js
    ```
    真实的脚本路径为 `/opt/homebrew/lib/node_modules/@modelcontextprotocol/server-sequential-thinking/dist/index.js`。

3.  **更新 `mcp.json` 配置**:

    使用 `node` 和脚本的绝对路径来更新配置，这是最稳妥的方案。

    ```json
    "sequential-thinking": {
      "command": "/opt/homebrew/bin/node",
      "args": [
        "/opt/homebrew/lib/node_modules/@modelcontextprotocol/server-sequential-thinking/dist/index.js"
      ]
    }
    ```

---

## 3. `fetch` 服务配置

此服务用于从网络上获取信息。

### 问题 1: `externally-managed-environment` 错误

- **问题描述**: 执行 `pip install mcp-server-fetch` 时，出现此错误。
- **原因**: 这是 Python (PEP 668) 和 macOS 的一种保护机制，防止用户使用 `pip` 直接修改由系统包管理器 (Homebrew) 安装的 Python 环境，以免破坏系统稳定性。
- **解决方案**: 使用 `--break-system-packages` 和 `--user` 标志。这会告诉 `pip` 在确认风险的情况下，将包安全地安装到当前用户的个人目录下，而不是系统目录。

  ```bash
  # 推荐的安装命令
  python3 -m pip install --break-system-packages --user mcp-server-fetch

  # 或者从 git 仓库安装
  python3 -m pip install --break-system-packages --user git+https://github.com/MaartenSmeets/mcp-server-fetch.git
  ```

### 问题 2: `pip` 网络超时

- **问题描述**: `pip` 下载包时卡住，最终出现 `TimeoutError` 或 `Read timed out`。
- **原因**: 连接 Python 官方包仓库 (`files.pythonhosted.org`) 的网络不稳定。
- **解决方案**: 将 `pip` 的下载源更换为国内的镜像源。

  ```bash
  # 设置 pip 全局源为清华大学镜像
  pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
  ```

### 问题 3: `No module named mcp_server_fetch`

- **问题描述**: 即使 `pip` 显示安装成功，运行服务时依然报错找不到模块。
- **原因**: 系统中存在多个 Python 环境。我们用 `pip3` 安装模块的环境，与 `mcp.json` 中配置的 `python` 命令指向的环境不是同一个。
- **解决方案**: 找出正确的 Python 路径，并更新 `mcp.json`。

#### 解决步骤

1.  **确认 `pip3` 对应的 Python 环境**:

    ```bash
    which pip3
    # 示例输出: /opt/homebrew/bin/pip3 
    # 这意味着模块被安装到了 Homebrew 的 Python 环境中。
    ```
    对应的 Python 可执行文件路径是 `/opt/homebrew/bin/python3`。

2.  **更新 `mcp.json` 配置**:
    将 `command` 的值从 `"python"` 修改为我们找到的绝对路径。

    ```json
    "fetch": {
      "command": "/opt/homebrew/bin/python3",
      "args": ["-m", "mcp_server_fetch"]
    }
    ```

---

## 4. `git` 服务配置

此服务用于与本地的 Git 仓库进行交互。

### 基础配置

```json
"git": {
  "command": "python",
  "args": ["-m", "mcp_server_git", "--repository", "你的项目绝地路径"]
}
```

> **注意**: 同样建议将 `command` 设置为 `python` 的绝对路径，例如 `"/opt/homebrew/bin/python3"`。

### 动态项目路径配置

当您需要操作多个 Git 项目时，为每个项目手动修改 `--repository` 路径会非常繁琐。可以利用 Cursor 的 `rules` 机制实现动态路径切换。

1.  **创建规则文件**: 在 `.cursor` 目录下创建一个 `git-project-locate.md` 文件。

2.  **编写规则**: 在文件中定义项目关键字和本地路径的映射关系。

    ```markdown
    # .cursor/rules/git-project-locate.md

    # 这边存放 git 项目和本地的目录之前的匹配关系:
    {
        "kid项目": "/Users/huangyanyu/Php/kid",
        "博客项目": "/Users/huangyanyu/Php/blog"
    }

    **Must**:如果匹配到对应项目的 key，那么就要将 git 这个的 mcp server中的 `--repository`改成对应的 value的值
    ```

通过这种方式，当您在对话中提到 "kid项目" 或 "博客项目" 时，AI 会自动将 `git` 服务的操作目录切换到对应的路径。
