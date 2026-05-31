## Claude Code + DeepSeek API 完整配置指南（Windows）

### 原理

```
你的问题 → Claude Code (CLI/VS Code扩展) → DeepSeek /anthropic端点 → 回复
```

DeepSeek 提供原生 Anthropic 兼容端点，Claude Code 直接对接，**零中间代理**（这里就不用去下载littel LLM）。

------

### 前提准备

#### 1. 安装 Node.js

前往 [nodejs.org](https://nodejs.org)，下载 LTS 版本安装。

安装完成后验证：

cmd输入：

```cmd
node --version
npm --version
```

#### 2. 获取 DeepSeek API Key

前往 [platform.deepseek.com](https://platform.deepseek.com) 注册账号，进入控制台创建 API Key，复制保存好。

------

### 第一步：解决 PowerShell 执行策略问题

Windows 默认 PowerShell 禁止运行脚本，**全程使用 CMD（命令提示符）**，不要用 PowerShell。

打开方式：按 `Win+R`，输入 `cmd`，回车。

------

### 第二步：安装 Claude Code

在 CMD 里运行：

cmd输入：

```cmd
npm install -g @anthropic-ai/claude-code
```

验证安装：

cmd输入：

```cmd
claude --version
```

输出类似 `2.1.158 (Claude Code)` 说明安装成功。

------

### 第三步：清除所有可能的干扰环境变量（根据情况做选择）

如果你之前尝试过任何配置，必须先清除残留变量，否则会产生 `Auth conflict` 冲突报错：

cmd输入：

```cmd
setx ANTHROPIC_API_KEY ""
setx ANTHROPIC_AUTH_TOKEN ""
setx ANTHROPIC_BASE_URL ""
setx OPENAI_API_KEY ""
setx OPENAI_BASE_URL ""
```

**关闭当前 CMD，重新打开一个新的 CMD 窗口**，让清除生效。

------

### 第四步：删除旧的登录状态（根据情况做选择）

如果之前登录过 Claude Code，旧的 token 会干扰新配置：

cmd输入：

```cmd
del %USERPROFILE%\.claude.json
```

> 文件不存在的话会报错，忽略即可，继续下一步。

------

### 第五步：写入 DeepSeek 配置

创建 `.claude` 目录（如果不存在）：

cmd输入：

```cmd
mkdir %USERPROFILE%\.claude
```

打开配置文件：

cmd输入：

```cmd
notepad %USERPROFILE%\.claude\settings.json
```

> 如果提示文件不存在，选**是**新建。

粘贴以下内容，**把 `你的DeepSeek_API_Key` 替换成你的真实 Key**：

json

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的DeepSeek_API_Key",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },
  "permissions": {
    "allow": [],
    "deny": []
  },
  "theme": "dark"
}
```

保存（`Ctrl+S`），关闭记事本。

> ⚠️ **只写 `ANTHROPIC_AUTH_TOKEN`，绝对不要同时写 `ANTHROPIC_API_KEY`**，两个共存会导致冲突。

------

### 第六步：验证命令行是否成功

在 CMD 里运行：

cmd输入：

```cmd
claude "你好，用中文回复我"
```

看到 DeepSeek 用中文回复（比如"你好！有什么我可以帮你的吗？"），说明配置完全成功。

------

### 第七步：创建一键启动脚本（以后免去重复操作，可省略）

这是关键步骤——以后每次用，双击脚本即可，**不需要打开 CMD，不需要任何手动配置**。

(当然也可以先打开cmd,然后输入`claude`,直接进入)

在桌面新建一个文件，命名为 `启动Claude.bat`：

cmd输入:

```cmd
notepad %USERPROFILE%\Desktop\启动Claude.bat
```

粘贴以下内容：

```bat
@echo off
title Claude Code (DeepSeek)
cd /d %USERPROFILE%
echo 正在启动 Claude Code (DeepSeek 后端)...
echo.
claude
```

保存关闭。以后**双击这个 bat 文件**，直接进入 Claude Code 对话界面，全程调用 DeepSeek API。

------

### 第八步(拓展)：接入 VS Code 扩展

#### 安装扩展

在 VS Code 扩展市场（`Ctrl+Shift+X`）搜索 **Claude Code**，安装 Anthropic 官方出品的插件。

#### 让 VS Code 读取配置

VS Code 需要从继承了正确环境的终端启动，才能读到 settings.json 的配置。

**方法一（推荐）：从 CMD 启动 VS Code**

新建一个文件 `用DeepSeek打开VSCode.bat`：

(cmd输入)

```cmd
notepad %USERPROFILE%\Desktop\用DeepSeek打开VSCode.bat
```

```bat
@echo off
title 启动 VS Code + Claude Code (DeepSeek)
echo 正在启动 VS Code，Claude Code 将使用 DeepSeek 后端...
echo.
code
```

保存。以后**双击这个脚本启动 VS Code**，再打开 Claude Code 侧边栏，选 **Anthropic Console**，自动读取 DeepSeek 配置，无需任何额外操作。

**方法二：在 VS Code 内置终端里用命令行**

VS Code 内置终端默认是 PowerShell，需要切换为 CMD：

1. 点终端右上角 `+` 旁边的下拉箭头
2. 选 **Command Prompt**
3. 输入 `claude` 回车，即可在终端内使用

------

### 全流程总结

```
第一次配置（只做一次）：
  安装 Node.js → 安装 Claude Code → 清除旧变量 → 写入 settings.json → 创建启动脚本

日常使用：
  双击"启动Claude.bat" → 直接对话（调用DeepSeek）
  或
  双击"用DeepSeek打开VSCode.bat" → VS Code里用Claude Code侧边栏
```

------

### 费用参考

| 模型              | 缓存命中输入   | 普通输入      | 输出          |
| ----------------- | -------------- | ------------- | ------------- |
| DeepSeek V4 Flash | $0.003/M token | $0.14/M token | $0.28/M token |
| DeepSeek V4 Pro   | $0.004/M token | $0.44/M token | $0.88/M token |

相比 Claude Sonnet 原价 $3/$15 per M token，**便宜约 10–50 倍**。

------

### 常见报错速查

| 报错                      | 原因                                                   | 解决                                        |
| ------------------------- | ------------------------------------------------------ | ------------------------------------------- |
| `Auth conflict`           | `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 同时存在 | settings.json 只保留 `ANTHROPIC_AUTH_TOKEN` |
| `Invalid API key`         | API Key 填错，或两个 Key 字段冲突                      | 检查 settings.json，确认只有一个 Key 字段   |
| `无法加载文件 claude.ps1` | 在 PowerShell 里运行了命令                             | 换用 CMD                                    |
| `claude` 不是内部命令     | Node.js 未安装或 npm 全局路径未加入 PATH               | 重新安装 Node.js，重启 CMD                  |

