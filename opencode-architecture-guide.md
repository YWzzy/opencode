# OpenCode 架构学习指南

> 目标目录: `packages/opencode/src/`
> 生成时间: 2026-03-23

---

## 概述

OpenCode 是一个开源 AI 编程代理,采用 **客户端/服务器分离架构**,支持多种交互方式:
- **TUI (Terminal UI)**: 主要交互方式
- **Web UI**: 浏览器界面
- **桌面应用**: Tauri/Electron 打包
- **API**: HTTP REST API

---

## 核心架构分层

```
┌─────────────────────────────────────────────────────────────────┐
│  前端层 (Presentation)                                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │   CLI/TUI   │ │   Web App   │ │   Desktop   │ │   API     │ │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └─────┬─────┘ │
└─────────┼───────────────┼───────────────┼──────────────┼───────┘
          │               │               │              │
          └───────────────┴───────┬───────┴──────────────┘
                                  │
┌─────────────────────────────────▼────────────────────────────────┐
│  应用层 (Application)     - packages/opencode/src/               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Agent 系统                             │   │
│  │   - build (默认开发代理)    - plan (只读规划代理)        │   │
│  │   - general (通用子代理)    - explore (代码探索代理)       │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  Session 会话管理                         │   │
│  │   - 对话生命周期管理  - 消息持久化  - 上下文压缩          │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Tool 工具系统                          │   │
│  │   - 文件操作  - Bash执行  - Web搜索  - LSP  - MCP        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                                  │
┌─────────────────────────────────▼────────────────────────────────┐
│  基础设施层 (Infrastructure)                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│  │ Provider │ │  Storage │ │   Bus    │ │ Permission│ │ Plugin │ │
│  │  (AI模型) │ │ (SQLite) │ │(事件总线) │ │ (权限系统) │ │(插件)  │ │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 模块详情

### 1. Agent 模块 (`src/agent/`)

**文件清单:**
```
agent.ts          - Agent定义和配置
```

**核心概念:**
- Agent 是 AI 的行为配置,定义了权限、系统提示、可用工具等
- 内置 7 个 Agent:
  | Agent | 权限 | 用途 |
  |-------|------|------|
  | `build` | 完全访问 | 默认开发模式,可编辑文件 |
  | `plan` | 只读 | 分析和规划,不修改代码 |
  | `general` | 完全 | 通用子代理,复杂多步任务 |
  | `explore` | 完全 | 代码库探索 |
  | `compaction` | 系统 | 会话压缩 |
  | `title` | 系统 | 会话标题生成 |
  | `summary` | 系统 | 会话摘要生成 |

**学习路径:**
1. 从 `agent.ts` 开始,理解 `defineAgent()` 和 `allAgents` 导出
2. 看 `AgentConfig` 类型定义,了解可配置项
3. 比较不同 Agent 的 `permission` 规则集

---

### 2. Session 模块 (`src/session/`)

**文件清单:**
```
index.ts          - Session CRUD 和核心逻辑
llm.ts            - LLM 流处理和工具调用
processor.ts      - 消息处理器(核心逻辑)
prompt.ts         - 提示管理
compaction.ts     - 会话压缩
summary.ts        - 摘要生成
message-v2.ts     - 消息数据结构
schema.ts         - ID 类型定义
session.sql.ts    - 数据库表定义
todo.ts           - Todo 任务管理
```

**核心流程:**
```
用户输入
    │
    ▼
session/index.ts: createSession()
    │
    ▼
session/llm.ts: streamText()
    │ - 调用 Provider 获取 AI 响应
    │ - 处理工具调用请求
    ▼
session/processor.ts: 处理工具调用
    │ - 执行工具
    │ - 收集结果
    │ - 发送给 AI 继续
    └─> 循环直到完成
```

**学习路径:**
1. 先看 `schema.ts` 了解数据模型 (SessionID, MessageID)
2. 再看 `session.sql.ts` 了解数据库表结构
3. 重点学习 `llm.ts` 的 `streamText()` 函数
4. 深入 `processor.ts` 理解处理循环

---

### 3. Tool 模块 (`src/tool/`)

**文件清单:**
```
tool.ts           - Tool 抽象定义
registry.ts       - 工具注册表
schema.ts         - ToolID 定义

# 内置工具
read.ts           - 文件读取
glob.ts           - 文件模式匹配
grep.ts           - 代码搜索
edit.ts           - 文件编辑(搜索替换)
write.ts          - 文件写入
bash.ts           - Bash 命令执行
webfetch.ts       - 网页获取
websearch.ts      - 网页搜索
codesearch.ts     - 代码搜索(Exa)
task.ts           - 子代理任务调用
skill.ts          - 技能执行
todo.ts           - Todo管理
plan.ts           - 计划模式工具
apply_patch.ts    - 应用 Patch
lsp.ts            - LSP 工具(实验性)
batch.ts          - 批量工具调用
```

**Tool 定义模式:**
```typescript
// src/tool/tool.ts 的核心设计
export const Tool = {
  define<TInput, TOutput>(config: {
    name: string
    description: string
    input: z.ZodSchema<TInput>
    output?: z.ZodSchema<TOutput>
    execute: (input: TInput) => Promise<TOutput>
  }): ToolDefinition
}
```

**学习路径:**
1. 从 `tool.ts` 理解 Tool 抽象定义
2. 看 `registry.ts` 了解工具是如何注册和发现的
3. 选择一个具体工具(如 `read.ts`)深入理解实现模式
4. 看 `bash.ts` 学习如何处理危险操作(权限检查)

---

### 4. Provider 模块 (`src/provider/`)

**文件清单:**
```
provider.ts       - 提供商管理核心
models.ts         - 模型定义和发现
auth.ts           - 认证管理
auth-service.ts   - 认证服务
transform.ts      - 选项转换
error.ts          - 错误处理
schema.ts         - ProviderID, ModelID 定义
sdk/copilot/      - GitHub Copilot SDK
```

**支持的提供商:**
- OpenAI (GPT-4, GPT-3.5)
- Anthropic (Claude)
- Google (Gemini)
- Azure OpenAI
- Google Vertex
- Amazon Bedrock
- XAI (Grok)
- Mistral
- Groq
- OpenRouter
- DeepInfra
- Cerebras
- 本地模型 (OpenAI 兼容 API)

**学习路径:**
1. `schema.ts` - 了解 ProviderID 和 ModelID 类型
2. `provider.ts` - 核心管理逻辑,看如何创建 providers
3. `models.ts` - 模型配置和发现
4. `auth.ts` - 认证流程

---

### 5. CLI 模块 (`src/cli/`)

**文件清单:**
```
index.ts          - CLI 入口, yargs 配置
cmd/
  run.ts          - `opencode run` 命令
tui/
  thread.ts       - TUI 交互式会话
  attach.ts       - 附加到远程服务器
  account.ts      - 账户相关
  agent.ts        - Agent 命令
  providers.ts    - 提供商管理
  models.ts       - 模型管理
  upgrade.ts      - 升级
  uninstall.ts    - 卸载
  debug.ts        - 调试
  mcp.ts          - MCP 命令
  session.ts      - 会话管理
  ...
ui.ts             - UI 工具
error.ts          - 错误格式化
network.ts        - 网络功能
logo.ts           - Logo 显示
```

**学习路径:**
1. `index.ts` - 了解命令注册和中间件
2. `cmd/run.ts` - 核心运行命令
3. `cmd/tui/thread.ts` - TUI 主界面

---

### 6. Permission 模块 (`src/permission/`)

**文件清单:**
```
next.ts           - 权限系统入口
service.ts        - 权限服务
arity.ts          - Bash 元数检查
```

**权限系统工作原理:**
```
配置文件 rules:
  - 规则1: 允许/拒绝/询问特定操作
  - 规则2: 基于路径、命令、工具类型的规则

执行流程:
  调用 Tool/Command
       │
       ▼
  evaluate(rules, action)
       │
       ├─> match allow -> 执行
       ├─> match deny  -> 拒绝
       └─> match ask   -> 提示用户
```

---

### 7. MCP 模块 (`src/mcp/`)

**文件清单:**
```
index.ts          - MCP 服务器管理
auth.ts           - MCP 认证
oauth-callback.ts - OAuth 回调服务器
oauth-provider.ts - OAuth 提供商
```

**MCP (Model Context Protocol)** 是 Anthropic 推出的标准化协议,用于连接 AI 和外部工具。

---

### 8. Skill 模块 (`src/skill/`)

**文件清单:**
```
skill.ts          - 技能管理
discovery.ts      - 技能发现
```

**技能发现路径:**
1. `./.opencode/skill/` - 项目级技能
2. `./.claude/skills/` - 项目级技能(兼容)
3. `~/.opencode/skill/` - 用户级技能
4. `~/.claude/skills/` - 用户级技能(兼容)

技能通过 `SKILL.md` 文件定义,可包含自定义指令和工具。

---

### 9. Storage 模块 (`src/storage/`)

**文件清单:**
```
db.ts             - SQLite 数据库连接
schema.ts         - 数据库模式
data.sql.ts       - 通用数据表
json-migration.ts - JSON 迁移逻辑
```

**数据库:** SQLite + Drizzle ORM
**存储位置:** `~/.local/share/opencode/opencode.db`

---

### 10. Bus 模块 (`src/bus/`)

**文件清单:**
```
index.ts          - 事件总线
bus-event.ts      - 事件定义
```

**用途:** 模块间解耦通信,如会话更新、文件变更等事件。

---

### 11. Config 模块 (`src/config/`)

**文件清单:**
```
config.ts         - 配置管理
paths.ts          - 路径解析
markdown.ts       - Markdown 解析
theme.ts          - 主题配置
```

**配置文件位置:**
- `~/.opencode/config.json`
- `./.opencode/config.json` (项目级)

**配置内容:** 提供商、模型、主题、快捷键、插件、自定义工具等。

---

### 12. Server 模块 (`src/server/`)

**文件清单:**
```
server.ts         - Hono HTTP 服务器
routes/
  session.ts      - 会话 API
  file.ts         - 文件操作 API
  tui.ts          - TUI API
  config.ts       - 配置 API
  provider.ts     - 提供商 API
  project.ts      - 项目 API
  pty.ts          - 终端 API
  mcp.ts          - MCP API
  permission.ts   - 权限 API
  question.ts     - 问答 API
```

**用途:** 客户端/服务器架构的核心,允许远程控制 OpenCode。

---

## 数据流示例

### 用户输入处理流程

```
[用户] 输入: "帮我修改这个函数"
    │
    ▼
[CLI] cmd/tui/thread.ts 接收输入
    │
    ▼
[Session] llm.ts: streamText()
    - 构建消息历史
    - 添加系统提示
    - 调用 Provider
    │
    ▼
[Provider] provider.ts: 调用 AI API
    │
    ▼
[AI 响应] 返回: 工具调用请求
    │
    ▼
[Tool] tool/*.ts (如 edit.ts)
    - 执行文件编辑
    - 权限检查
    - 结果返回
    │
    ▼
[Session] processor.ts: 处理结果
    - 存储到数据库
    - 发送给 AI 继续
    │
    ▼
[AI 响应] 返回: 完成确认
    │
    ▼
[UI] 显示最终结果给用户
```

---

## 文件读取顺序建议

### Phase 1: 整体理解 (30 分钟)
1. `src/index.ts` - 入口和命令注册
2. `src/agent/agent.ts` - Agent 系统
3. `src/session/schema.ts` - 数据模型
4. `src/tool/tool.ts` - 工具抽象

### Phase 2: 核心逻辑 (1-2 小时)
1. `src/session/llm.ts` - LLM 处理
2. `src/session/processor.ts` - 消息处理核心
3. `src/tool/read.ts` / `edit.ts` / `bash.ts` - 典型工具实现
4. `src/provider/provider.ts` - 提供商管理

### Phase 3: 扩展层 (1 小时)
1. `src/permission/next.ts` - 权限系统
2. `src/skill/skill.ts` - 技能系统
3. `src/mcp/index.ts` - MCP 集成
4. `src/server/server.ts` - HTTP API

### Phase 4: 深入细节
- 选择一个感兴趣的模块深入
- 阅读测试文件辅助理解
- 跟踪一个完整的功能调用链

---

## 关键类型定义

### Session 相关
```typescript
// src/session/schema.ts
type SessionID = string & { __brand: "SessionID" }
type MessageID = string & { __brand: "MessageID" }
type PartID = string & { __brand: "PartID" }
```

### Tool 相关
```typescript
// src/tool/tool.ts
interface ToolDefinition<TInput, TOutput> {
  name: string
  description: string
  input: z.ZodSchema<TInput>
  output?: z.ZodSchema<TOutput>
  execute: (input: TInput) => Promise<TOutput>
}
```

### Agent 相关
```typescript
// src/agent/agent.ts
interface AgentConfig {
  name: string
  mode: "primary" | "subagent" | "all"
  system: string[]  // 系统提示
  permission: PermissionConfig
  // ...
}
```

---

## 调试技巧

1. **日志**: 使用 `--print-logs` 查看详细日志
2. **调试命令**: `opencode debug` 系列命令
3. **数据库**: 直接查看 `~/.local/share/opencode/opencode.db`
4. **TypeCheck**: 在 `packages/opencode` 目录运行 `bun typecheck`

---

## 扩展指南

### 添加新工具
1. 在 `src/tool/` 创建 `my-tool.ts`
2. 使用 `Tool.define()` 定义工具
3. 在 `src/tool/registry.ts` 注册

### 添加新 Provider
1. `src/provider/provider.ts` 中添加配置
2. 使用 `ai` 库的提供商 SDK

### 添加自定义技能
1. 在 `.opencode/skill/` 创建目录
2. 编写 `SKILL.md` 定义技能

---

## 相关文档

- [AGENTS.md](/Users/temptrip/Desktop/wsp/opencode/AGENTS.md) - 代码规范
- [CONTRIBUTING.md](/Users/temptrip/Desktop/wsp/opencode/CONTRIBUTING.md) - 贡献指南
- [README.md](/Users/temptrip/Desktop/wsp/opencode/README.md) - 项目介绍

---

*文档结束。祝你学习愉快!*
