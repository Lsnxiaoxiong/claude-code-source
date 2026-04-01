# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code CLI - an agentic coding tool that lives in your terminal, understands your codebase, and helps you code faster by executing routine tasks, explaining complex code, and handling git workflows.

## Build & Development

**Runtime:** Node.js 18+

**Package Manager:** Bun (see `bun.lock`)

**Entry Point:** `src/entrypoints/cli.tsx` - bootstrap entrypoint with fast-paths for `--version` and special flags

**Build Output:** `cli.js` (bundled) and `cli.js.map`

**Key Commands:**
- Run: `node cli.js` or `claude` (if installed globally)
- Version: `claude --version` or `claude -v`

## Architecture

### Core Structure

```
src/
├── entrypoints/      # CLI and SDK entry points
├── commands/         # Slash command implementations (/help, /init, etc.)
├── tools/            # Tool implementations (Bash, FileEdit, Agent, MCP, etc.)
├── hooks/            # React hooks for UI state and notifications
├── bridge/           # Remote session and bridge communication
├── context/          # Context management
├── state/            # Application state management
├── services/         # Background services (skill search, etc.)
├── constants/        # Prompts, settings, and constants
├── schemas/          # TypeScript type schemas
└── utils/            # Shared utilities
```

### Key Files

- `src/main.tsx` - Main application logic (800KB+ - core orchestrator)
- `src/commands.ts` - Command registry and imports
- `src/Tool.ts` - Base tool implementation
- `src/Task.ts` - Task management
- `src/QueryEngine.ts` - Code querying and search
- `sdk-tools.d.ts` - Generated TypeScript definitions for tool schemas

### Tool System

Tools are implemented in `src/tools/` with a consistent pattern:
- Each tool has its own directory (e.g., `BashTool/`, `FileEditTool/`, `AgentTool/`)
- Tools include: `Tool.tsx` (implementation), `prompt.ts` (system prompt), `UI.tsx` (rendering)

Built-in tools: Agent, Bash, FileEdit, FileRead, FileWrite, Glob, Grep, MCP, NotebookEdit, Task, WebFetch, WebSearch, AskUserQuestion, Config, Worktree management

### Command System

Slash commands live in `src/commands/` with index-based lazy loading. Each command has:
- `index.ts` - Export wrapper
- `<command>.tsx` or `<command>.ts` - Implementation

Feature flags (via `bun:bundle`) gate certain commands at build time.

### Bridge Mode

The `src/bridge/` directory handles remote session communication, JWT authentication, and session management for cloud-hosted sessions.

### Feature Flags

Build-time feature flags via `bun:bundle` control optional functionality:
- `PROACTIVE`, `KAIROS` - Proactive features and brief/assistant commands
- `BRIDGE_MODE` - Remote bridge communication
- `DAEMON` - Background daemon workers
- `VOICE_MODE` - Voice interaction
- `ABLATION_BASELINE` - Research ablation mode

## Development Notes

- **No test framework configured** - This is a shipped/bundled CLI without traditional unit tests in the repo
- **TypeScript** - Strict types with generated schemas from JSON Schema
- **React/Ink** - Terminal UI uses React via Ink (`src/ink/`)
- **Native modules** - Some features use native addons in `vendor/` directory

## External Dependencies

Minimal direct dependencies in `package.json`. Most functionality is bundled. Optional dependencies are platform-specific `sharp` binaries for image processing.

---

## Detailed Module Descriptions

### 1. entrypoints/ - 入口点模块

| 文件 | 描述 |
|------|------|
| `cli.tsx` | 主 CLI 入口，包含启动性能分析、快速路径处理（如--version）、特性标志门控 |
| `init.ts` | 初始化模块：配置系统、环境变量、遥测、OAuth、策略限制、远程托管设置 |
| `mcp.ts` | MCP (Model Context Protocol) 服务器入口 |
| `agentSdkTypes.ts` | Agent SDK 类型定义 |
| `sandboxTypes.ts` | 沙箱环境类型 |
| `sdk/` | SDK 核心 schemas 和控制 schemas |

### 2. commands/ - 命令系统 (70+ 命令)

**核心命令类别：**

| 类别 | 命令示例 |
|------|----------|
| 会话管理 | `/clear`, `/resume`, `/rename`, `/export`, `/share` |
| 配置 | `/config`, `/theme`, `/model`, `/keybindings`, `/settings` |
| Git/协作 | `/commit`, `/branch`, `/review`, `/autofix-pr`, `/pr_comments` |
| 上下文 | `/context`, `/context`, `/diff`, `/files` |
| 调试 | `/doctor`, `/status`, `/stats`, `/ant-trace`, `/perf-issue` |
| 扩展 | `/mcp`, `/plugins`, `/skills`, `/hooks`, `/memory` |
| 认证 | `/login`, `/logout`, `/desktop` |
| 特殊 | `/help`, `/version`, `/exit`, `/feedback`, `/bug` |

**命令注册流程：** `commands.ts` → 动态导入 → 特性标志过滤 → 注册到命令系统

### 3. tools/ - 工具系统

**核心工具分类：**

| 工具类别 | 工具名称 | 描述 |
|----------|----------|------|
| 文件操作 | Bash, FileRead, FileEdit, FileWrite, Glob, Grep | 文件系统交互 |
| Agent | AgentTool, TaskOutput, TaskStop | 子代理管理 |
| 任务管理 | TaskCreate, TaskGet, TaskUpdate, TaskList, TodoWrite | 任务跟踪 |
| 网络 | WebFetch, WebSearch, WebBrowser | 网络请求 |
| MCP | MCP, ListMcpResources, ReadMcpResource | MCP 服务器集成 |
| 模式 | EnterPlanMode, ExitPlanMode, EnterWorktree, ExitWorktree | 模式切换 |
| 用户交互 | AskUserQuestion | 向用户提问 |
| 配置 | Config | 设置管理 |
| 技能 | SkillTool | 技能系统 |
| LSP | LSPTool | 语言服务器协议 |

**工具权限系统：** 每个工具有权限检查、破坏性命令警告、沙箱验证

### 4. state/ - 状态管理

| 文件 | 描述 |
|------|------|
| `AppState.ts` | 应用状态定义和获取器（git 状态、工作目录等） |
| `AppState.tsx` | React 状态管理和提供器 |
| `AppStateStore.ts` | Zustand 风格的 store 实现 |
| `store.ts` | 基础 store 创建 |
| `onChangeAppState.ts` | 状态变更监听 |

### 5. services/ - 后台服务

**主要服务类别：**

| 服务 | 描述 |
|------|------|
| **Analytics** | 遥测、GrowthBook 特性标志、Datadog 日志、第一方事件日志 |
| **API** | Claude API 客户端、文件 API、使用量跟踪、错误处理 |
| **MCP** | MCP 连接管理、OAuth 认证、资源配置、官方注册表 |
| **LSP** | 语言服务器管理、诊断注册表、被动反馈 |
| **Compact** | 自动压缩、会话内存压缩、微压缩 |
| **Plugins** | 插件安装、CLI 命令、操作管理 |
| **OAuth** | OAuth 流程、客户端、加密 |
| **PolicyLimits** | 使用限制、策略执行 |
| **RemoteManagedSettings** | 远程托管设置同步 |

### 6. constants/ - 常量定义

| 文件 | 描述 |
|------|------|
| `prompts.ts` | 系统提示生成（含特性标志门控） |
| `tools.ts` | 工具名称常量和代理工具限制 |
| `systemPromptSections.ts` | 系统提示分段模板 |
| `outputStyles.ts` | 输出样式配置 |
| `messages.ts` | 用户消息模板 |
| `apiLimits.ts` | API 限制常量 |
| `betas.ts` | Beta 功能列表 |
| `github-app.ts` | GitHub 应用配置 |

### 7. components/ - UI 组件 (Ink/React)

**组件类别：**

| 类别 | 组件示例 |
|------|----------|
| 应用框架 | App, FullscreenLayout, ExitFlow |
| 消息渲染 | BashModeProgress, AgentProgressLine, FileEditToolDiff |
| 对话框 | AutoModeOptInDialog, ExportDialog, MCPServerDialog* |
| 帮助系统 | HelpV2 (Commands, General) |
| Logo/品牌 | LogoV2 系列（Clawd, Feed, Welcome） |
| 反馈 | Feedback, FeedbackSurvey |
| 设置 | InvalidSettingsDialog, KeybindingWarnings |

### 8. hooks/ - React Hooks

| 类别 | Hook 示例 |
|------|-----------|
| 工具权限 | useCanUseTool, PermissionContext |
| 设置 | useSettings, useDynamicConfig |
| 命令 | useCommandQueue, useCommandKeybindings |
| 会话 | useRemoteSession, useReplBridge |
| 通知 | 各类状态指示器（IDE, MCP, AutoMode, FastMode） |

### 9. bridge/ - 桥接通信

| 文件 | 描述 |
|------|------|
| `bridgeApi.ts` | 桥接 API 封装 |
| `bridgeConfig.ts` | 桥接配置 |
| `replBridge.ts` | REPL 桥接传输 |
| `sessionIdCompat.ts` | 会话 ID 兼容性 |
| `sessionRunner.ts` | 会话运行器 |
| `jwtUtils.ts` | JWT 工具 |
| `workSecret.ts` | 工作密钥 |

### 10. context/ - 上下文管理

| 文件 | 描述 |
|------|------|
| `context.ts` | 用户上下文和系统上下文生成 |
| `notifications.ts` | 通知系统 |
| `stats.ts` | 统计信息 |

### 11. ink/ - 终端渲染引擎

完整的 React 终端渲染实现：
- **DOM 模拟** - 节点树、布局引擎（Yoga）
- **事件系统** - 键盘、鼠标、焦点事件
- **终端协议** - ANSI/CSI/OSC 解析
- **组件** - Box, Text, Link, ScrollBox 等
- **Hooks** - use-app, use-input, use-terminal-focus

### 12. utils/ - 工具函数 (200+ 文件)

**主要工具类别：**

| 类别 | 示例 |
|------|------|
| 认证 | auth.ts, authPortable.ts, aws.ts |
| 文件系统 | fsOperations.ts, cachePaths.ts, claudemd.ts |
| Git | git.ts, gitignore.ts, githubRepoPathMapping.ts |
| 模型 | model.ts, modelCapabilities.ts, deprecation.ts |
| 权限 | permissions.ts, PermissionMode.ts, autoModeState.ts |
| 设置 | settings.ts, settingsCache.ts, validation.ts |
| 性能 | startupProfiler.ts, fpsTracker.ts |
| 调试 | debug.ts, log.ts, diagLogs.ts |

### 13. types/ - 类型定义

| 文件 | 描述 |
|------|------|
| `ids.ts` | ID 类型（AgentId, SessionId） |
| `permissions.ts` | 权限模式、规则 |
| `command.ts` | 命令接口 |
| `hooks.ts` | Hook 类型 |
| `logs.ts` | 日志类型 |
| `generated/` | 生成的事件类型 |

### 14. skills/ - 技能系统

| 文件 | 描述 |
|------|------|
| `loadSkillsDir.ts` | 从目录加载技能 |
| `bundledSkills.ts` | 内置技能 |
| `mcpSkillBuilders.ts` | MCP 技能构建器 |

### 15. plugins/ - 插件系统

| 文件 | 描述 |
|------|------|
| `bundled/` | 内置插件 |
| `builtinPlugins.ts` | 内置插件定义 |

### 16. memdir/ - 内存目录

长期记忆存储和管理。

### 17. migrations/ - 数据迁移

设置迁移、模型迁移、权限迁移等。

### 18. coordinator/ - 协调器模式

多代理协调和任务分配。

### 19. remote/ - 远程会话

远程会话管理和配置。

### 20. server/ - 服务器组件

直接连接会话创建等。

### 21. assistant/ - 助手模式

Kairos 助手功能实现。

### 22. buddy/ - 伙伴系统

桌面伙伴功能和动画。

### 23. query/ - 查询系统

代码查询引擎和链式查询跟踪。

### 24. screens/ - 屏幕管理

终端屏幕状态和管理。

### 25. outputStyles/ - 输出样式

不同输出风格的定义和渲染。

### 26. vendor/ - 第三方库

| 目录 | 描述 |
|------|------|
| `ripgrep/` | 快速文件搜索 |
| `audio-capture/` | 音频捕获 |
| `modifiers-napi/` | 键盘修饰符 NAPI |
| `url-handler/` | URL 处理 |

---

## 数据流

1. **启动流程：** `cli.tsx` → `init.ts` → `main.tsx` → REPL/App
2. **命令执行：** 用户输入 → 命令解析 → 命令处理器 → 结果输出
3. **工具调用：** 模型请求 → 工具验证 → 权限检查 → 工具执行 → 结果返回
4. **状态更新：** 工具结果 → AppState 更新 → UI 重新渲染

## 权限系统

- **模式：** default, auto, bypassPermissions
- **规则来源：** 用户设置、项目设置、策略设置
- **工具限制：** 基于代理类型的工具白名单/黑名单

## 特性标志系统

使用 `bun:bundle` 的 `feature()` 函数进行编译时死代码消除：
- 内部版本（USER_TYPE='ant'）解锁额外功能
- 条件导入避免未使用代码打包
