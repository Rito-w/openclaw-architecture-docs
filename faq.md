# 常见问题

本页面收集了 OpenClaw 使用过程中的常见问题和解决方案。

## 安装与配置

### 如何选择 Node.js 版本？

OpenClaw 需要 Node.js 22 或更高版本。推荐使用 LTS 版本以获得更好的稳定性。

```bash
# 检查 Node.js 版本
node --version

# 使用 nvm 安装最新 LTS
nvm install --lts
nvm use --lts
```

### pnpm 和 npm 有什么区别？

pnpm 更快、更节省磁盘空间，并且有更好的依赖管理。OpenClaw 项目推荐使用 pnpm。

```bash
# 安装 pnpm
npm install -g pnpm

# 使用 pnpm 安装 OpenClaw
pnpm add -g openclaw
```

### 配置文件在哪里？

OpenClaw 的配置文件位于：

```
~/.openclaw/
├── config.json          # 主配置文件
├── credentials/         # 凭据存储
├── agents/             # 代理配置
├── sessions/           # 会话数据
└── logs/               # 日志文件
```

### 如何重置配置？

```bash
# 备份当前配置
cp -r ~/.openclaw ~/.openclaw.backup

# 重置配置
rm -rf ~/.openclaw
openclaw doctor
```

## Gateway 相关

### Gateway 无法启动

**端口被占用**：

```bash
# 检查端口占用
lsof -i :18789

# 结束占用进程
kill -9 <PID>
```

**权限问题**：

```bash
# 检查配置目录权限
ls -la ~/.openclaw

# 修复权限
chmod -R 755 ~/.openclaw
```

### 如何远程访问 Gateway？

推荐使用 SSH 隧道或 Tailscale：

**SSH 隧道**：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
```

**Tailscale**：

```bash
# 安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 启动
tailscale up

# 使用 Funnel 暴露服务
tailscale funnel 18789
```

### Gateway 日志在哪里？

```bash
# 查看日志
tail -f ~/.openclaw/logs/gateway.log

# 查看 API 调用日志
tail -f ~/.openclaw/logs/anthropic-payload.jsonl
```

## 模型相关

### 如何切换默认模型？

```bash
# 设置主模型
openclaw config set model.primary claude-sonnet-4-20250514

# 设置回退模型
openclaw config set model.fallbacks '["gpt-4o", "gemini-2.0-flash"]'
```

### 模型调用失败怎么办？

1. **检查 API Key**：确保 Key 有效且有配额
2. **检查网络**：确保能访问 API 端点
3. **查看日志**：检查具体错误信息

```bash
# 查看模型状态
openclaw config get model

# 查看冷却状态
openclaw agents status
```

### 如何使用本地模型？

OpenClaw 支持通过 Ollama 或本地 API 接入本地模型：

```json
{
  "model": {
    "primary": "local:llama3",
    "localEndpoint": "http://localhost:11434"
  }
}
```

## 渠道相关

### Telegram Bot 无响应

1. **检查 Token**：确保 Bot Token 正确
2. **检查白名单**：确保你的用户 ID 在 allowFrom 列表中
3. **检查网络**：确保能访问 Telegram API

```bash
# 获取你的用户 ID
# 向 @userinfobot 发送消息

# 更新白名单
openclaw config set telegram.allowFrom '["YOUR_USER_ID"]'
```

### Discord Bot 无法连接

1. **检查 Token 和 Application ID**
2. **确保 Bot 有正确的权限**
3. **检查 WebSocket 连接**

```bash
# 检查 Discord 配置
openclaw config get discord

# 测试连接
openclaw channels status --probe discord
```

### 群组消息不响应

检查群组权限和白名单配置：

```json
{
  "telegram": {
    "allowFrom": {
      "users": ["user1"],
      "groups": ["-1001234567890"]
    }
  }
}
```

## 工具相关

### Bash 命令执行失败

1. **检查命令白名单**：某些命令可能被禁止
2. **检查工作目录**：确保路径正确
3. **检查超时设置**：长时间运行的命令可能超时

```json
{
  "tools": {
    "bash": {
      "timeout": 60000,
      "allowCommands": ["git", "npm", "node"]
    }
  }
}
```

### 浏览器工具无法使用

确保已安装浏览器驱动：

```bash
# 安装 Playwright 浏览器
npx playwright install chromium
```

### 文件操作权限错误

检查沙箱配置和路径权限：

```json
{
  "sandbox": {
    "workdir": "/workspace",
    "allowPaths": ["/workspace/**", "/tmp/**"]
  }
}
```

## 性能优化

### 如何提高响应速度？

1. **选择更快的模型**：如 claude-sonnet-4-20250514 或 gpt-4o
2. **减少工具调用**：禁用不必要的工具
3. **优化提示词**：更简洁的提示词响应更快

### 内存占用过高

```bash
# 清理会话历史
openclaw sessions compact --older-than 7d

# 清理日志
rm -rf ~/.openclaw/logs/*.log.old
```

### Token 消耗过快

1. **使用更短的上下文**
2. **启用记忆压缩**
3. **选择更经济的模型**

```json
{
  "sessions": {
    "maxContextTokens": 50000,
    "compressionEnabled": true
  }
}
```

## 安全相关

### 如何保护 API Key？

1. 使用环境变量存储敏感信息
2. 不要在配置文件中硬编码
3. 定期轮换密钥

```bash
# 使用环境变量
export ANTHROPIC_API_KEY=sk-ant-xxxxx

# 在配置中引用
openclaw config set anthropicApiKey $ANTHROPIC_API_KEY
```

### 如何限制用户访问？

使用白名单和角色控制：

```json
{
  "security": {
    "allowFrom": {
      "telegram": ["user1", "user2"]
    },
    "roles": {
      "admin": {
        "users": ["user1"],
        "permissions": ["*"]
      }
    }
  }
}
```

### 如何启用 TLS？

```json
{
  "gateway": {
    "bind": "all",
    "port": 18789,
    "tls": {
      "enabled": true,
      "cert": "/path/to/cert.pem",
      "key": "/path/to/key.pem"
    }
  }
}
```

## 故障排除

### 启用调试模式

```bash
# 设置日志级别
export OPENCLAW_LOG_LEVEL=debug

# 启动 Gateway
openclaw gateway run
```

### 运行诊断

```bash
# 运行系统诊断
openclaw doctor

# 检查渠道状态
openclaw channels status --probe

# 检查代理状态
openclaw agents status
```

### 报告问题

如果遇到无法解决的问题，请：

1. 收集诊断信息：`openclaw doctor --output diag.json`
2. 查看相关日志
3. 在 [GitHub Issues](https://github.com/openclaw/openclaw/issues) 提交问题

---

没有找到你的问题？[提交 Issue](https://github.com/openclaw/openclaw/issues/new) 或加入 [Discord 社区](https://discord.gg/openclaw) 寻求帮助。