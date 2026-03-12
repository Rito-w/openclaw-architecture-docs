# OpenClaw 架构文档

OpenClaw AI 代理网关平台的架构原理与技术文档。

## 在线阅读

GitHub Pages: [在线文档](https://rito-w.github.io/openclaw-architecture-docs/)

## 本地预览

```bash
# Python
python3 -m http.server 3000

# 或 Node.js
npx serve .
```

访问 http://localhost:3000

## 文档内容

| 模块 | 说明 |
|------|------|
| 快速开始 | 安装配置与基本使用 |
| 系统架构 | 整体设计与核心组件 |
| 数据流 | 消息处理流程详解 |
| Gateway | 网关连接与路由 |
| Agents | 代理运行时与模型选择 |
| Channels | 多渠道消息接入 |
| Sessions | 会话与上下文管理 |
| Tools | 工具系统与技能扩展 |
| Security | 认证授权与权限控制 |

## 技术栈

- [Docsify](https://docsify.js.org/) - 文档框架
- [Mermaid](https://mermaid.js.org/) - 架构图渲染
- Inter + JetBrains Mono - 字体

## 相关链接

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [官方文档](https://docs.openclaw.ai)

## License

MIT