# OpenClaw 架构文档

OpenClaw AI 代理网关平台架构原理文档。

## 在线阅读

[https://openclaw-architecture-docs.vercel.app](https://openclaw-architecture-docs.vercel.app)

## 本地预览

```bash
# 使用任意静态服务器
npx serve .
# 或
python -m http.server 3000
```

然后访问 http://localhost:3000

## 文档结构

```
├── overview/          # 概览
│   ├── architecture.md    # 系统架构
│   └── data-flow.md       # 数据流
├── core/              # 核心模块
│   ├── gateway.md         # Gateway 网关
│   ├── agents.md          # Agents 代理
│   ├── channels.md        # Channels 渠道
│   ├── sessions.md        # Sessions 会话
│   ├── config.md          # Config 配置
│   └── memory.md          # Memory 记忆
├── tools/             # 工具系统
│   ├── browser.md         # Browser 浏览器
│   ├── skills.md          # Skills 技能
│   └── tools.md           # Tools 工具
├── dev/               # 扩展开发
│   ├── plugin-sdk.md      # Plugin SDK
│   └── channel-dev.md     # Channel 开发
└── security/          # 安全机制
    ├── auth.md            # 认证授权
    └── permissions.md     # 权限控制
```

## 技术栈

- [Docsify](https://docsify.js.org/) - 文档生成
- [Mermaid](https://mermaid.js.org/) - 图表渲染

## License

MIT