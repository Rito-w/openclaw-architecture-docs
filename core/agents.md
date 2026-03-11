# Agents 代理

Agents 模块是 AI 代理的运行时环境，负责模型选择、工具调用和对话管理。

## 架构概览

```mermaid
graph TB
    subgraph Agent Runtime
        BOOT[启动引导]
        CONFIG[配置解析]
        AUTH[认证管理]
        MODEL[模型选择]
        TOOLS[工具系统]
        MEMORY[记忆系统]
    end

    subgraph 认证配置
        PROFILES[Auth Profiles]
        OAUTH[OAuth Tokens]
        APIKEY[API Keys]
    end

    subgraph 模型提供商
        CLAUDE[Claude]
        OPENAI[OpenAI]
        GEMINI[Gemini]
        LOCAL[本地模型]
    end

    subgraph 工具
        BASH[Bash 执行]
        BROWSER[浏览器]
        SKILLS[Skills]
        CUSTOM[自定义工具]
    end

    BOOT --> CONFIG
    CONFIG --> AUTH
    AUTH --> PROFILES
    PROFILES --> OAUTH
    PROFILES --> APIKEY

    AUTH --> MODEL
    MODEL --> CLAUDE
    MODEL --> OPENAI
    MODEL --> GEMINI
    MODEL --> LOCAL

    MODEL --> TOOLS
    TOOLS --> BASH
    TOOLS --> BROWSER
    TOOLS --> SKILLS
    TOOLS --> CUSTOM

    TOOLS --> MEMORY
```

## 代理配置

### 配置结构

```typescript
interface AgentEntry {
  // 基本信息
  id: string;
  name?: string;
  default?: boolean;

  // 模型配置
  model?: string | ModelConfig;

  // 工作空间
  workspace?: string;
  agentDir?: string;

  // 能力配置
  skills?: string[];
  tools?: ToolsConfig;
  memorySearch?: MemorySearchConfig;

  // 身份配置
  identity?: AgentIdentity;

  // 子代理
  subagents?: SubagentConfig;

  // 沙箱配置
  sandbox?: SandboxConfig;

  // 行为配置
  humanDelay?: HumanDelayConfig;
  heartbeat?: HeartbeatConfig;
  groupChat?: GroupChatConfig;
}
```

### 模型配置

```typescript
interface ModelConfig {
  primary: string;           // 主模型
  fallbacks?: string[];      // 回退模型列表
}

// 配置示例
{
  "model": {
    "primary": "claude-sonnet-4-20250514",
    "fallbacks": ["gpt-4o", "gemini-2.0-flash"]
  }
}
```

## 模型选择

### 选择流程

```mermaid
flowchart TD
    START[开始] --> PRIMARY{主模型可用?}
    PRIMARY -->|是| USE_PRIMARY[使用主模型]
    PRIMARY -->|否| FALLBACK{有回退模型?}

    FALLBACK -->|是| TRY_NEXT[尝试下一个回退]
    TRY_NEXT --> CHECK{模型可用?}
    CHECK -->|是| USE_FALLBACK[使用回退模型]
    CHECK -->|否| FALLBACK

    FALLBACK -->|否| ERROR[报错]

    USE_PRIMARY --> CALL[调用模型]
    USE_FALLBACK --> CALL

    CALL --> RESULT{调用成功?}
    RESULT -->|是| DONE[完成]
    RESULT -->|否| COOLDOWN{需要冷却?}

    COOLDOWN -->|是| MARK[标记冷却]
    COOLDOWN -->|否| ERROR

    MARK --> FALLBACK
```

### 冷却机制

当模型调用失败时，会进入冷却状态：

```typescript
interface CooldownState {
  reason: CooldownReason;
  expiresAt: number;
  failureCount: number;
}

type CooldownReason =
  | 'rate_limit'
  | 'billing_error'
  | 'context_overflow'
  | 'unknown';
```

### Auth Profiles

认证配置管理不同 API 提供商的凭据：

```typescript
interface AuthProfile {
  provider: string;         // anthropic | openai | google | ...
  label?: string;           // 显示标签

  // 认证方式
  apiKey?: string;          // API Key
  oauth?: OAuthState;       // OAuth 状态

  // 状态
  lastUsed?: number;
  lastGood?: number;
  cooldown?: CooldownState;
}
```

## 工具系统

### 工具类型

```mermaid
graph LR
    subgraph 内置工具
        BASH[Bash 命令]
        EDIT[文件编辑]
        READ[文件读取]
        WRITE[文件写入]
    end

    subgraph 浏览器工具
        NAV[导航]
        CLICK[点击]
        TYPE[输入]
        SCREENSHOT[截图]
    end

    subgraph Skills
        CUSTOM[自定义技能]
    end

    BASH --> EXEC[执行器]
    EDIT --> EXEC
    READ --> EXEC
    WRITE --> EXEC
    NAV --> EXEC
    CLICK --> EXEC
    TYPE --> EXEC
    SCREENSHOT --> EXEC
    CUSTOM --> EXEC
```

### Bash 工具

Bash 工具执行 shell 命令：

```typescript
interface BashToolParams {
  command: string;          // 命令
  timeout?: number;         // 超时时间
  background?: boolean;     // 后台执行
  workdir?: string;         // 工作目录
  env?: Record<string, string>;  // 环境变量
}
```

安全机制：
- 命令白名单
- 执行审批
- 沙箱隔离
- 超时保护

### 浏览器工具

浏览器自动化工具集：

| 工具 | 描述 |
|------|------|
| `browser_navigate` | 导航到 URL |
| `browser_click` | 点击元素 |
| `browser_type` | 输入文本 |
| `browser_screenshot` | 截取屏幕 |
| `browser_scroll` | 滚动页面 |
| `browser_wait` | 等待条件 |

## Skills 技能系统

### Skill 结构

```
~/.openclaw/skills/{skillName}/
├── skill.md          # 提示词
├── tools.json        # 工具定义
├── config.json       # 配置
└── files/            # 附加文件
```

### Skill 加载

```mermaid
flowchart LR
    SCAN[扫描目录] --> PARSE[解析 Skill]
    PARSE --> VALIDATE[验证配置]
    VALIDATE --> REGISTER[注册工具]
    REGISTER --> READY[就绪]
```

### Skill 配置

```typescript
interface SkillConfig {
  name: string;
  description: string;
  tools?: ToolDefinition[];
  prompt?: string;
  files?: string[];
}
```

## 记忆系统

### 向量搜索

```mermaid
sequenceDiagram
    participant Agent as 代理
    participant Memory as 记忆系统
    participant Embedding as 嵌入服务
    participant Index as 向量索引

    Agent->>Memory: 查询相关记忆
    Memory->>Embedding: 生成查询向量
    Embedding-->>Memory: 返回向量
    Memory->>Index: 搜索相似向量
    Index-->>Memory: 返回结果
    Memory-->>Agent: 相关记忆
```

### 配置

```typescript
interface MemorySearchConfig {
  enabled: boolean;
  embeddingModel?: string;
  topK?: number;
  threshold?: number;
}
```

## 子代理系统

### 子代理调用

```mermaid
sequenceDiagram
    participant Main as 主代理
    participant Router as 路由器
    participant Sub as 子代理

    Main->>Router: 需要调用子代理
    Router->>Router: 选择子代理
    Router->>Sub: 启动子代理
    Sub->>Sub: 执行任务
    Sub-->>Router: 返回结果
    Router-->>Main: 汇总结果
```

### 子代理配置

```typescript
interface SubagentConfig {
  [name: string]: {
    model?: string;
    skills?: string[];
    tools?: string[];
    systemPrompt?: string;
  };
}
```

## 身份配置

### Agent Identity

```typescript
interface AgentIdentity {
  name?: string;              // 名称
  bio?: string;               // 简介
  personality?: string;       // 性格
  expertise?: string[];       // 专业领域
  communication_style?: string;  // 沟通风格
}
```

### 提示词模板

系统提示词由以下部分组成：

1. **身份提示**: 基于 identity 配置
2. **技能提示**: 加载的 Skills
3. **工具描述**: 可用工具列表
4. **上下文**: 对话历史和环境信息

## 沙箱配置

### 沙箱选项

```typescript
interface SandboxConfig {
  enabled: boolean;
  workdir?: string;           // 工作目录
  allowCommands?: string[];   // 允许的命令
  denyCommands?: string[];    // 禁止的命令
  env?: Record<string, string>;
  timeout?: number;
}
```

### 权限控制

- 文件系统访问限制
- 网络访问控制
- 命令执行白名单
- 资源使用限制