# src/ — 核心源代码总览

> **路径**: `src/`  
> **文件数**: 5,163 个 TypeScript 文件  
> **子模块数**: 48+  
> **语言**: TypeScript (ESM)

---

## 模块索引

`src/` 按功能域划分为以下主要子模块：

| # | 模块 | 路径 | 文件数 | 职责 | 文档链接 |
|---|------|------|--------|------|----------|
| 1 | 入口层 | `src/*.ts` | ~17 | CLI 启动、库导出、全局状态 | 本文件 |
| 2 | AI Agent 引擎 | `src/agents/` | 992 | 模型交互、工具调用、会话生命周期 | [01-agents.md](./01-agents.md) |
| 3 | 网关服务器 | `src/gateway/` | 403 | WebSocket + HTTP 通信中枢 | [02-gateway.md](./02-gateway.md) |
| 4 | 配置系统 | `src/config/` | 264 | JSON5 配置、Schema 校验、密钥加密 | [03-config.md](./03-config.md) |
| 5 | CLI 命令 | `src/commands/` | 428 | 所有 CLI 命令的业务逻辑 | [04-commands.md](./04-commands.md) |
| 6 | CLI 框架 | `src/cli/` | 329 | 参数解析、命令注册、交互式提示 | [05-cli.md](./05-cli.md) |
| 7 | 插件系统 | `src/plugins/` | 241 | 插件发现、加载、注册、市场 | [06-plugins.md](./06-plugins.md) |
| 8 | 插件 SDK | `src/plugin-sdk/` | 217 | 第三方插件开发 SDK | [07-plugin-sdk.md](./07-plugin-sdk.md) |
| 9 | 安全审计 | `src/security/` | 38 | 配置安全检查、危险检测 | [08-security.md](./08-security.md) |
| 10 | 记忆系统 | `src/memory/` | 106 | 向量记忆、语义搜索 | [09-memory.md](./09-memory.md) |
| 11 | 通道系统 | `src/channels/` | 191 | 通道注册、消息分发 | [10-channels.md](./10-channels.md) |
| 12 | 消息路由 | `src/routing/` | 11 | 路由解析、会话 key 构建 | [11-routing.md](./11-routing.md) |
| 13 | ACP 协议 | `src/acp/` | 55 | Agent Client Protocol 实现 | [12-acp.md](./12-acp.md) |
| 14 | 自动回复 | `src/auto-reply/` | 355 | 消息处理、命令检测、流式传输 | [13-auto-reply.md](./13-auto-reply.md) |
| 15 | 基础设施 | `src/infra/` | 538 | 网络、文件系统、安全、进程管理 | [14-infra.md](./14-infra.md) |
| 16 | 其他子模块 | `src/*/` | ~640 | 浏览器、Cron、媒体、TUI 等 | [15-others.md](./15-others.md) |

---

## 入口层文件

入口层文件位于 `src/` 根目录，负责应用启动流程、库级 API 导出、全局状态管理。

### 核心入口

| 文件 | 作用 |
|------|------|
| `index.ts` | **主入口**。导出 `loadConfig`、`deriveSessionKey` 等库级 API；作为 main module 时启动 CLI |
| `entry.ts` | **CLI 启动流程**：respawn 检测、容器目标解析、profile env、编译缓存初始化 |
| `entry.respawn.ts` | 进程 respawn 逻辑（配置变化后自动重启） |
| `library.ts` | 库模式导出，供 npm 消费者使用 |

### 运行时与全局

| 文件 | 作用 |
|------|------|
| `runtime.ts` | 运行时全局初始化 |
| `globals.ts` | 全局常量定义 |
| `global-state.ts` | 全局状态管理 |
| `logger.ts` | 日志记录器 |
| `logging.ts` | 日志基础设施 |
| `version.ts` | 版本号管理 |
| `utils.ts` | 通用工具函数 |
| `param-key.ts` | 参数键常量 |

### 功能入口

| 文件 | 作用 |
|------|------|
| `polls.ts` | 投票/轮询功能 |
| `poll-params.ts` | 轮询参数定义 |
| `extensionAPI.ts` | 插件 API 桥接层 |
| `channel-web.ts` | Web 通道入口（WhatsApp Web） |
| `bundled-web-search-registry.ts` | 内置网页搜索提供商注册表 |

### 依赖关系

```
openclaw.mjs (CLI 二进制)
  └─→ src/entry.ts (CLI 启动)
        ├─→ src/entry.respawn.ts (进程 respawn)
        ├─→ src/runtime.ts (运行时初始化)
        ├─→ src/globals.ts + src/global-state.ts
        └─→ src/cli/program.ts (Commander 程序)
              └─→ src/commands/*.ts (各命令实现)

npm require('openclaw')
  └─→ src/index.ts (库入口)
        └─→ src/library.ts (库模式导出)
```

### 启动流程

1. `openclaw.mjs` → 调用 `src/entry.ts`
2. `entry.ts` 处理 respawn 检测、容器环境、CLI profile
3. 初始化编译缓存 + 运行时全局
4. 委托给 `src/cli/program.ts` 解析参数并执行命令

---

*本文件是 `src/` 拆解文档系列的索引文件。详细内容见各子模块独立文档。*
