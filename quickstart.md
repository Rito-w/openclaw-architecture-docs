# 快速开始

本指南帮助你在几分钟内启动 OpenClaw。

## 系统要求

- **Node.js**: 22+ (推荐使用最新 LTS 版本)
- **包管理器**: pnpm (推荐) / npm / bun
- **操作系统**: macOS / Linux / Windows (WSL2)

## 安装

### 使用 npm

```bash
npm install -g openclaw
```

### 使用 pnpm

```bash
pnpm add -g openclaw
```

### 使用 Homebrew (macOS)

```bash
brew install openclaw
```

## 初始化配置

首次运行会自动创建配置目录：

```bash
openclaw doctor
```

配置文件位置：`~/.openclaw/config.json`

## 配置 AI 模型

### Anthropic Claude (推荐)

```bash
# 设置 API Key
export ANTHROPIC_API_KEY=sk-ant-xxxxx

# 或在配置文件中设置
openclaw config set anthropicApiKey $ANTHROPIC_API_KEY
```

### OpenAI GPT

```bash
export OPENAI_API_KEY=sk-xxxxx
openclaw config set openaiApiKey $OPENAI_API_KEY
```

### Google Gemini

```bash
export GOOGLE_API_KEY=xxxxx
openclaw config set googleApiKey $GOOGLE_API_KEY
```

## 启动 Gateway

```bash
# 启动网关（本地模式）
openclaw gateway run

# 指定端口
openclaw gateway run --port 18789

# 后台运行
openclaw gateway run --detach
```

验证网关运行状态：

```bash
openclaw channels status --probe
```

## 配置消息渠道

### Telegram Bot

1. 在 Telegram 中找到 @BotFather
2. 发送 `/newbot` 创建机器人
3. 获取 Bot Token

```bash
openclaw config set telegram.token "YOUR_BOT_TOKEN"
openclaw config set telegram.allowFrom '["YOUR_USER_ID"]'
```

### Discord Bot

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 创建新应用并添加 Bot
3. 获取 Token 和 Application ID

```bash
openclaw config set discord.token "YOUR_BOT_TOKEN"
openclaw config set discord.applicationId "YOUR_APP_ID"
```

### Slack App

1. 访问 [Slack API](https://api.slack.com/apps)
2. 创建新应用
3. 配置 Socket Mode

```bash
openclaw config set slack.botToken "xoxb-..."
openclaw config set slack.appToken "xapp-..."
```

## 基本使用

### CLI 对话

```bash
# 与默认代理对话
openclaw agent run

# 指定模型
openclaw agent run --model claude-sonnet-4-20250514

# 指定工作目录
openclaw agent run --workdir ./my-project
```

### 通过消息渠道

配置好渠道后，在 Telegram/Discord/Slack 中向 Bot 发送消息即可开始对话。

## 配置代理

创建自定义代理配置：

```json
{
  "agents": {
    "list": [
      {
        "id": "assistant",
        "name": "助手",
        "default": true,
        "model": "claude-sonnet-4-20250514",
        "skills": ["general-helper"]
      },
      {
        "id": "coder",
        "name": "程序员",
        "model": "claude-opus-4-20250514",
        "skills": ["code-assistant"],
        "workspace": "~/projects"
      }
    ]
  }
}
```

## 下一步

- [系统架构](overview/architecture.md) - 了解 OpenClaw 的设计理念
- [Gateway 配置](core/gateway.md) - 深入配置网关
- [渠道接入](core/channels.md) - 接入更多消息平台
- [工具系统](tools/tools.md) - 扩展代理能力

## 常见问题

### Gateway 启动失败

检查端口是否被占用：

```bash
lsof -i :18789
```

### API Key 无效

确保 API Key 格式正确，并且有足够的配额。

### 消息渠道无法连接

1. 检查网络连接
2. 验证 Token 有效性
3. 查看 Gateway 日志：`tail -f ~/.openclaw/logs/gateway.log`