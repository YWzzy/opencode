# OpenCode 完整目录结构说明

> 本文档详细说明 OpenCode 项目的完整目录结构
> 更新时间: 2026-03-23

---

## 项目概览

OpenCode 是一个大型 monorepo 项目,使用 Bun 作为包管理器和运行时,采用 Turborepo 进行任务编排。

---

## 根目录结构

```
opencode/
├── .github/              # GitHub Actions 工作流、PR 模板、Issue 模板
├── .husky/               # Git hooks (pre-commit等)
├── .opencode/            # 项目级别的 OpenCode 配置
│   ├── agent/            # 自定义 Agent 配置
│   ├── command/          # 自定义命令
│   ├── glossary/         # 术语表配置
│   ├── themes/           # 主题配置
│   └── tool/             # 自定义工具
├── .vscode/              # VS Code 编辑器配置
├── .zed/                 # Zed 编辑器配置
├── github/               # GitHub 相关脚本和工具
│   └── script/           # GitHub 自动化脚本
├── infra/                # 基础设施配置 (IaC)
├── nix/                  # Nix 包管理器配置
│   └── scripts/          # Nix 构建脚本
├── packages/             # 核心包目录 (21个包)
├── patches/              # 依赖包补丁
├── script/               # 根级构建和开发脚本
├── sdks/                 # SDK 项目
│   └── vscode/           # VS Code 扩展
├── specs/                # 规范和设计文档
├── AGENTS.md             # AI Agent 代码规范
├── CONTRIBUTING.md       # 贡献指南
├── README.*.md           # 多语言 README
├── STATS.md              # 项目统计
├── bun.lock              # Bun 锁文件
├── bunfig.toml           # Bun 配置文件
├── flake.lock            # Nix flake 锁文件
├── flake.nix             # Nix flake 配置
├── install               # 安装脚本
├── package.json          # 根 package.json
├── sst.config.ts         # SST (Serverless Stack) 配置
├── sst-env.d.ts          # SST 类型定义
└── turbo.json            # Turborepo 配置
```

---

## Packages 目录详解 (21个包)

### 1. packages/opencode/ ⭐ 核心 CLI

**职责:** OpenCode 的核心命令行工具和 TUI 界面

```
packages/opencode/
├── bin/                  # CLI 入口脚本
│   └── opencode          # 可执行文件
├── migration/            # 数据库迁移文件
├── script/               # 构建和开发脚本
│   ├── build.ts          # 构建脚本
│   └── test.ts           # 测试脚本
├── src/
│   ├── account/          # 账户管理
│   ├── acp/              # Agent Client Protocol
│   ├── agent/            # Agent 系统定义
│   ├── auth/             # 认证逻辑
│   ├── bun/              # Bun 运行时相关
│   ├── bus/              # 事件总线
│   ├── cli/              # CLI 命令实现
│   │   ├── cmd/          # 各命令实现
│   │   │   ├── account.ts
│   │   │   ├── agent.ts
│   │   │   ├── debug.ts
│   │   │   ├── export.ts
│   │   │   ├── generate.ts
│   │   │   ├── github.ts
│   │   │   ├── import.ts
│   │   │   ├── mcp.ts
│   │   │   ├── models.ts
│   │   │   ├── pr.ts
│   │   │   ├── providers.ts
│   │   │   ├── run.ts          # 主要运行命令
│   │   │   ├── serve.ts
│   │   │   ├── session.ts
│   │   │   ├── stats.ts
│   │   │   ├── tui/            # TUI 相关命令
│   │   │   │   ├── attach.ts
│   │   │   │   └── thread.ts   # TUI 主入口
│   │   │   ├── uninstall.ts
│   │   │   ├── upgrade.ts
│   │   │   ├── web.ts
│   │   │   └── workspace-serve.ts
│   │   ├── effect/       # 视觉效果
│   │   ├── error.ts      # 错误处理
│   │   ├── logo.ts       # Logo 显示
│   │   ├── network.ts    # 网络工具
│   │   └── ui.ts         # UI 工具函数
│   ├── command/          # 命令抽象
│   ├── config/           # 配置管理
│   │   ├── config.ts     # 核心配置
│   │   ├── markdown.ts   # Markdown 解析
│   │   ├── paths.ts      # 路径解析
│   │   └── theme.ts      # 主题配置
│   ├── control-plane/    # 控制平面
│   ├── effect/           # Effect 集成
│   ├── env/              # 环境变量
│   ├── file/             # 文件操作
│   ├── flag/             # 功能标志
│   ├── format/           # 代码格式化
│   ├── global/           # 全局状态
│   ├── id/               # ID 生成
│   ├── index.ts          # 主入口
│   ├── installation/     # 安装信息
│   ├── lsp/              # LSP 集成
│   │   ├── client.ts
│   │   ├── error.ts
│   │   ├── index.ts
│   │   └── types.ts
│   ├── mcp/              # MCP (Model Context Protocol)
│   │   ├── auth.ts
│   │   ├── index.ts
│   │   ├── oauth-callback.ts
│   │   └── oauth-provider.ts
│   ├── patch/            # 补丁应用
│   ├── permission/       # 权限系统
│   │   ├── arity.ts
│   │   ├── next.ts
│   │   └── service.ts
│   ├── plugin/           # 插件系统
│   │   ├── codex.ts      # Codex 插件
│   │   └── copilot.ts    # Copilot 插件
│   ├── project/          # 项目管理
│   │   ├── instance.ts
│   │   ├── project.ts
│   │   ├── vcs.ts
│   │   └── vcs-watcher.ts
│   ├── provider/         # AI 提供商
│   │   ├── auth-service.ts
│   │   ├── auth.ts
│   │   ├── error.ts
│   │   ├── models.ts
│   │   ├── provider.ts
│   │   ├── schema.ts
│   │   ├── transform.ts
│   │   └── sdk/
│   │       └── copilot/  # Copilot SDK
│   ├── pty/              # 伪终端
│   ├── question/         # 用户交互
│   ├── scheduler/        # 调度器
│   ├── server/           # HTTP 服务器
│   │   ├── routes/       # API 路由
│   │   │   ├── config.ts
│   │   │   ├── file.ts
│   │   │   ├── mcp.ts
│   │   │   ├── permission.ts
│   │   │   ├── project.ts
│   │   │   ├── provider.ts
│   │   │   ├── pty.ts
│   │   │   ├── question.ts
│   │   │   ├── session.ts
│   │   │   └── tui.ts
│   │   └── server.ts
│   ├── session/          # 会话管理 ⭐
│   │   ├── compaction.ts
│   │   ├── index.ts
│   │   ├── llm.ts
│   │   ├── message-v2.ts
│   │   ├── processor.ts
│   │   ├── prompt.ts
│   │   ├── retry.ts
│   │   ├── schema.ts
│   │   ├── session.sql.ts
│   │   ├── summary.ts
│   │   └── todo.ts
│   ├── share/            # 分享功能
│   ├── shell/            # Shell 集成
│   ├── skill/            # 技能系统
│   │   ├── discovery.ts
│   │   └── skill.ts
│   ├── snapshot/         # 文件快照
│   ├── sql.d.ts          # SQL 类型
│   ├── storage/          # 数据存储
│   │   ├── data.sql.ts
│   │   ├── db.ts
│   │   ├── json-migration.ts
│   │   └── schema.ts
│   ├── tool/             # 工具实现 ⭐
│   │   ├── apply_patch.ts
│   │   ├── bash.ts
│   │   ├── batch.ts
│   │   ├── codesearch.ts
│   │   ├── edit.ts
│   │   ├── glob.ts
│   │   ├── grep.ts
│   │   ├── lsp.ts
│   │   ├── plan.ts
│   │   ├── read.ts
│   │   ├── registry.ts
│   │   ├── schema.ts
│   │   ├── skill.ts
│   │   ├── task.ts
│   │   ├── todo.ts
│   │   ├── tool.ts
│   │   ├── webfetch.ts
│   │   ├── websearch.ts
│   │   └── write.ts
│   ├── util/             # 工具函数
│   └── worktree/         # Git worktree
├── test/                 # 测试文件
├── AGENTS.md             # 包级规范
├── bunfig.toml
├── drizzle.config.ts     # Drizzle ORM 配置
└── package.json
```

---

### 2. packages/app/ ⭐ Web 应用

**职责:** OpenCode 的 Web 界面 (Vinxi + SolidStart)

```
packages/app/
├── e2e/                  # Playwright E2E 测试
│   ├── app/
│   ├── commands/
│   ├── files/
│   ├── models/
│   ├── projects/
│   ├── prompt/
│   ├── session/
│   ├── settings/
│   ├── sidebar/
│   ├── status/
│   └── terminal/
├── public/               # 静态资源
├── script/
├── src/
│   ├── addons/           # 扩展组件
│   ├── components/       # 组件
│   │   ├── prompt-input/ # 输入组件
│   │   ├── server/       # 服务端组件
│   │   └── session/      # 会话组件
│   ├── context/          # 上下文
│   │   ├── file/
│   │   └── global-sync/
│   ├── hooks/            # 自定义 Hooks
│   ├── i18n/             # 国际化
│   ├── pages/            # 页面路由
│   │   ├── layout/
│   │   └── session/
│   │       └── composer/
│   ├── testing/          # 测试工具
│   └── utils/            # 工具函数
└── package.json
```

---

### 3. packages/ui/ ⭐ UI 组件库

**职责:** OpenCode 的 UI 组件库 (SolidJS + TailwindCSS)

```
packages/ui/
├── script/               # 构建脚本
├── src/
│   ├── assets/           # 静态资源
│   │   ├── audio/
│   │   ├── favicon/
│   │   ├── fonts/
│   │   ├── icons/
│   │   │   ├── app/
│   │   │   ├── file-types/
│   │   │   └── provider/
│   │   └── images/
│   ├── components/       # 组件
│   │   ├── app-icons/    # 应用图标
│   │   ├── file-icons/   # 文件类型图标
│   │   └── provider-icons/ # 提供商图标
│   ├── context/          # 上下文
│   ├── hooks/            # Hooks
│   ├── i18n/             # 国际化
│   ├── pierre/           # Pierre 设计系统组件
│   ├── storybook/        # Storybook 配置
│   ├── styles/           # 样式
│   │   └── tailwind/
│   └── theme/            # 主题系统
│       └── themes/
└── package.json
```

---

### 4. packages/desktop/ ⭐ Tauri 桌面应用

**职责:** OpenCode 的 Tauri 桌面应用 (推荐)

```
packages/desktop/
├── script/               # 构建脚本
├── src/
└── package.json
```

---

### 5. packages/desktop-electron/ ⭐ Electron 桌面应用

**职责:** OpenCode 的 Electron 桌面应用 (旧版)

```
packages/desktop-electron/
├── icons/                # 应用图标
│   ├── beta/
│   ├── dev/
│   └── prod/
├── resources/            # 资源文件
├── scripts/              # 构建脚本
├── src/
│   ├── main/             # 主进程
│   ├── preload/          # 预加载脚本
│   └── renderer/         # 渲染进程
│       └── i18n/         # 国际化
└── package.json
```

---

### 6. packages/console/ ⭐ 控制台服务

**职责:** OpenCode 的云服务后端

```
packages/console/
├── app/                  # Hono 应用
├── core/                 # 核心逻辑
├── function/             # Cloud Functions
├── mail/                 # 邮件模板
└── resource/             # 基础设施资源
```

---

### 7. packages/web/ ⭐ 官方网站

**职责:** OpenCode 官网 (Astro)

```
packages/web/
├── public/               # 静态资源
├── src/
│   ├── content/          # 内容 (MDX)
│   │   └── docs/         # 文档 (多语言)
│   │       ├── da/
│   │       ├── it/
│   │       ├── ja/
│   │       ├── pl/
│   │       └── ru/
│   └── types/            # 类型定义
└── package.json
```

---

### 8. packages/docs/ ⭐ 文档站点

**职责:** OpenCode 文档

```
packages/docs/
└── ...
```

---

### 9. packages/storybook/ ⭐ 组件展示

**职责:** UI 组件的 Storybook 文档

```
packages/storybook/
├── .storybook/           # Storybook 配置
│   └── mocks/            # 模拟数据
│       ├── app/
│       └── hooks/
└── package.json
```

---

### 10. packages/sdk/

**职责:** OpenCode SDK

```
packages/sdk/
├── js/                   # JavaScript/TypeScript SDK
│   ├── script/           # 构建脚本
│   └── src/
└── openapi.json          # OpenAPI 规范
```

---

### 11. packages/enterprise/

**职责:** 企业版功能

```
packages/enterprise/
├── public/
├── script/
├── src/
│   ├── core/             # 核心逻辑
│   └── routes/           # 路由
│       ├── api/
│       └── share/
└── test/
    └── core/
```

---

### 12. packages/slack/

**职责:** Slack 集成

```
packages/slack/
└── ...
```

---

### 13. packages/identity/

**职责:** 身份验证服务

```
packages/identity/
└── ...
```

---

### 14. packages/function/

**职责:** Cloudflare Workers / Lambda Functions

```
packages/function/
└── ...
```

---

### 15. packages/containers/

**职责:** 容器化支持 (Docker)

```
packages/containers/
└── ...
```

---

### 16. packages/extensions/

**职责:** VS Code 扩展和其他编辑器扩展

```
packages/extensions/
└── ...
```

---

### 17-21. 其他包

```
packages/
├── plugin/               # 插件系统核心
├── script/               # 共享脚本工具
├── util/                 # 共享工具函数
```

---

## sdks/ 目录

### sdks/vscode/ ⭐ VS Code 扩展

**职责:** OpenCode VS Code 扩展

```
sdks/vscode/
├── images/               # 扩展图标
├── script/               # 构建脚本
├── src/                  # 扩展源码
└── ...
```

---

## infra/ 和 github/ 目录

### infra/ 基础设施

基础设施即代码配置

### github/ GitHub 自动化

GitHub Actions 和工作流脚本

---

## 配置文件详解

| 文件 | 用途 |
|------|------|
| `package.json` | 根包配置,工作区定义 |
| `turbo.json` | Turborepo 任务编排 |
| `bunfig.toml` | Bun 配置 |
| `flake.nix` | Nix 包定义 |
| `sst.config.ts` | SST (AWS 部署) 配置 |
| `drizzle.config.ts` | ORM 配置 (各包内) |
| `.editorconfig` | 编辑器配置 |
| `.prettierignore` | Prettier 忽略规则 |

---

## 阅读顺序建议

### 第1步: 理解项目结构 (10分钟)
1. 阅读本文档了解整体结构
2. 查看 `package.json` 了解工作区结构
3. 查看 `turbo.json` 了解任务依赖

### 第2步: 核心包 (2-3小时)
1. `packages/opencode/` - 核心 CLI
2. `packages/app/` - Web 应用
3. `packages/ui/` - 组件库

### 第3步: 扩展包 (1-2小时)
1. `packages/desktop/` - 桌面应用
2. `packages/console/` - 云服务
3. `packages/sdk/` - SDK

### 第4步: 基础设施 (30分钟)
1. `infra/` - 基础设施
2. `.github/` - CI/CD

---

## 关键文件入口

### 开发入口

| 命令 | 入口文件 |
|------|----------|
| `bun dev` | `packages/opencode/src/index.ts` |
| `bun dev:web` | `packages/app/dev` |
| `bun dev:desktop` | `packages/desktop/tauri dev` |

### 核心源码入口

| 模块 | 入口文件 |
|------|----------|
| CLI | `packages/opencode/src/index.ts` |
| Agent | `packages/opencode/src/agent/agent.ts` |
| Session | `packages/opencode/src/session/index.ts` |
| Tool | `packages/opencode/src/tool/tool.ts` |
| Provider | `packages/opencode/src/provider/provider.ts` |

---

## 依赖关系图

```
┌────────────────────────────────────────────────────────────────┐
│                    应用层 (Applications)                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐ ┌─────────────┐   │
│  │  opencode │ │   app    │ │   desktop    │ │ web (site)  │   │
│  │   (CLI)   │ │  (Web)   │ │ (Tauri/Electron)│  (官网)    │   │
│  └─────┬────┘ └─────┬────┘ └──────┬───────┘ └──────┬──────┘   │
└────────┼────────────┼─────────────┼────────────────┼──────────┘
         │            │             │                │
         └────────────┴──────┬──────┴────────────────┘
                             │
┌────────────────────────────▼───────────────────────────────────┐
│                    共享层 (Shared)                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │    ui    │ │   util   │ │  plugin  │ │      sdk         │  │
│  │ (组件库) │ │ (工具)   │ │ (插件)   │ │  (JS/TS SDK)     │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │
└────────────────────────────────────────────────────────────────┘
                             │
┌────────────────────────────▼───────────────────────────────────┐
│                    服务层 (Services)                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ console  │ │ identity │ │  enterprise  │ │    slack     │  │
│  │ (云服务) │ │ (认证)   │ │  (企业版)    │ │  (Slack)     │  │
│  └──────────┘ └──────────┘ └──────────────┘ └──────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

---

*文档结束*
