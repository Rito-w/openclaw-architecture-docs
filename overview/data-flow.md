# 数据流

本文档描述消息从接收到响应的完整数据流程。

## 消息处理流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant Channel as 渠道适配器
    participant Gateway as 网关
    participant Session as 会话管理
    participant Agent as 代理运行时
    participant LLM as AI 模型
    participant Tools as 工具系统

    User->>Channel: 发送消息
    Channel->>Channel: 解析消息内容
    Channel->>Gateway: 转发消息

    Gateway->>Gateway: 权限检查
    Gateway->>Session: 获取/创建会话
    Session->>Session: 加载对话历史

    Session->>Agent: 启动代理处理
    Agent->>Agent: 选择模型
    Agent->>LLM: 发送请求

    LLM-->>Agent: 返回响应

    alt 需要工具调用
        Agent->>Tools: 执行工具
        Tools-->>Agent: 返回结果
        Agent->>LLM: 继续对话
        LLM-->>Agent: 返回最终响应
    end

    Agent-->>Gateway: 返回回复
    Gateway->>Channel: 发送回复
    Channel->>User: 展示消息
```

## 详细流程说明

### 1. 消息接入

```mermaid
flowchart LR
    subgraph 消息源
        TG[Telegram Bot API]
        DC[Discord Gateway]
        SL[Slack RTM]
        WA[WhatsApp Web]
    end

    subgraph 接入方式
        POLL[轮询 Polling]
        HOOK[Webhook]
        WS[WebSocket]
    end

    TG --> POLL
    DC --> WS
    SL --> WS
    WA --> POLL
```

消息通过各渠道的适配器接入系统：
- **Telegram**: 通过 Bot API 长轮询
- **Discord**: 通过 Gateway WebSocket
- **Slack**: 通过 RTM WebSocket
- **Signal**: 通过 Signald WebSocket
- **iMessage**: 通过 BlueBubbles API
- **WhatsApp**: 通过 Web API 轮询

### 2. 消息解析

```typescript
interface InboundMessage {
  // 消息标识
  messageId: string;
  timestamp: number;

  // 来源信息
  channel: ChannelId;
  chatId: string;
  senderId: string;

  // 消息内容
  text?: string;
  attachments?: Attachment[];
  replyTo?: string;

  // 元数据
  isGroup: boolean;
  mentions?: string[];
}
```

### 3. 权限检查

```mermaid
flowchart TD
    MSG[收到消息] --> CHECK{白名单检查}
    CHECK -->|在白名单| ALLOW[允许处理]
    CHECK -->|不在白名单| DENY[拒绝处理]

    ALLOW --> CMD{是命令?}
    CMD -->|是| GATE{命令门控}
    CMD -->|否| ROUTE[路由到代理]

    GATE -->|允许| ROUTE
    GATE -->|拒绝| REPLY[提示无权限]
```

权限检查包括：
1. **白名单验证**: 检查发送者是否在允许列表
2. **命令门控**: 检查是否有执行命令的权限
3. **群组策略**: 检查群组特定的访问规则

### 4. 会话管理

```mermaid
stateDiagram-v2
    [*] --> New: 新对话
    New --> Active: 首次响应
    Active --> Active: 继续对话
    Active --> Idle: 超时
    Idle --> Active: 新消息
    Idle --> Closed: 长时间无活动
    Active --> Closed: 用户结束
    Closed --> [*]
```

会话生命周期：
- **New**: 新创建的会话
- **Active**: 正在进行对话
- **Idle**: 等待用户输入
- **Closed**: 会话已结束

### 5. 代理处理

```mermaid
flowchart TD
    subgraph Agent Runtime
        INPUT[接收输入] --> CONTEXT[构建上下文]
        CONTEXT --> MODEL[选择模型]
        MODEL --> CALL[调用 LLM]
        CALL --> PARSE[解析响应]

        PARSE --> CHECK{工具调用?}
        CHECK -->|是| EXEC[执行工具]
        CHECK -->|否| OUTPUT[输出响应]

        EXEC --> CALL
    end
```

代理处理步骤：
1. **构建上下文**: 组装系统提示、对话历史、工具定义
2. **选择模型**: 根据配置和可用性选择 LLM
3. **调用 LLM**: 发送请求并处理流式响应
4. **解析响应**: 提取文本内容和工具调用
5. **执行工具**: 如有工具调用，执行并继续对话

### 6. 工具执行

```mermaid
sequenceDiagram
    participant Agent as 代理
    participant Approval as 审批系统
    participant Tool as 工具执行器
    participant System as 系统命令

    Agent->>Agent: 解析工具调用
    Agent->>Approval: 请求执行权限

    alt 需要审批
        Approval->>Approval: 发送审批请求
        Approval->>Agent: 等待用户确认
    end

    Approval-->>Agent: 允许执行
    Agent->>Tool: 执行工具
    Tool->>System: 运行命令
    System-->>Tool: 返回结果
    Tool-->>Agent: 返回输出
```

工具执行安全机制：
- **沙箱隔离**: 在受限环境中执行
- **权限审批**: 敏感操作需要用户确认
- **命令白名单**: 限制可执行的命令
- **超时保护**: 防止长时间运行的命令

### 7. 响应发送

```typescript
interface OutboundMessage {
  // 目标信息
  channel: ChannelId;
  chatId: string;
  replyTo?: string;

  // 响应内容
  text: string;
  attachments?: Attachment[];

  // 流式控制
  streaming: boolean;
  done: boolean;
}
```

响应发送策略：
- **流式输出**: 逐字符/逐段发送
- **分片处理**: 长消息自动分片
- **重试机制**: 发送失败自动重试
- **幂等保证**: 避免重复发送

## 数据存储

### 会话数据

```
~/.openclaw/sessions/
├── {sessionKey}/
│   ├── transcript.jsonl     # 对话历史
│   ├── context.json         # 上下文状态
│   └── metadata.json        # 会话元数据
```

### 配置数据

```
~/.openclaw/
├── config.json              # 主配置文件
├── credentials/             # 凭据存储
│   ├── telegram.json
│   ├── discord.json
│   └── ...
└── agents/                  # 代理配置
    └── {agentId}/
        └── ...
```

### 日志数据

```
~/.openclaw/logs/
├── gateway.log              # 网关日志
├── agent-{id}.log           # 代理日志
└── anthropic-payload.jsonl  # API 调用日志
```