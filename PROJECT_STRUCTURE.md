# OpenClaw 项目代码结构全面分析

> **版本**: 2026.3.24  
> **仓库**: https://github.com/openclaw/openclaw  
> **协议**: MIT  
> **描述**: Multi-channel AI gateway with extensible messaging integrations

---

## 目录

- [1. 项目概述](#1-项目概述)
- [2. 技术栈](#2-技术栈)
- [3. 根目录文件](#3-根目录文件)
- [4. src/ — 核心源代码](#4-src--核心源代码)
- [5. extensions/ — 插件系统](#5-extensions--插件系统)
- [6. apps/ — 原生客户端](#6-apps--原生客户端)
- [7. ui/ — Web 前端](#7-ui--web-前端)
- [8. Swabble/ — 语音唤醒守护进程](#8-swabble--语音唤醒守护进程)
- [9. scripts/ — 构建与运维脚本](#9-scripts--构建与运维脚本)
- [10. docs/ — 项目文档](#10-docs--项目文档)
- [11. vendor/ — 第三方代码](#11-vendor--第三方代码)
- [12. packages/ — 工作区子包](#12-packages--工作区子包)
- [13. skills/ — AI 技能定义](#13-skills--ai-技能定义)
- [14. test/ — 测试基础设施](#14-test--测试基础设施)
- [15. 其他支撑目录](#15-其他支撑目录)
- [16. 架构关系图](#16-架构关系图)

---

## 1. 项目概述

OpenClaw 是一个**多通道 AI 网关平台**，它将 AI Agent 通过统一的网关服务器连接到 20+ 种消息平台（Telegram、Discord、Slack、WhatsApp、微信等）。核心架构包括：

- **Gateway Server**: WebSocket + HTTP 网关服务器，负责消息路由、会话管理和通道适配
- **Agent Engine**: AI Agent 命令循环，支持工具调用、上下文管理、多模型切换
- **Plugin System**: 可扩展的插件架构，包含 80+ 个内置插件
- **Native Clients**: iOS / macOS / Android 原生客户端 + Web 前端
- **Plugin SDK**: 为第三方开发者提供的完整插件 SDK

**项目规模统计**:
| 目录 | 文件数 | 主要语言 |
|------|--------|---------|
| `src/` | 5,163 | TypeScript |
| `extensions/` | 2,506 | TypeScript |
| `apps/` | 835 | Swift / Kotlin |
| `ui/` | 243 | TypeScript / CSS |
| `scripts/` | 244 | Shell / MJS / TypeScript / Go |
| `docs/` | 740 | Markdown |
| `vendor/` | 158 | TypeScript |
| `test/` | 115 | TypeScript |
| `skills/` | 69 | Markdown / Python |

---

## 2. 技术栈

| 层面 | 技术选型 |
|------|---------|
| **语言** | TypeScript (ESM)、Swift 6.2、Kotlin (Jetpack Compose) |
| **运行时** | Node 22+、Bun (开发/测试) |
| **构建** | tsdown (打包)、pnpm workspace (monorepo) |
| **测试** | Vitest + V8 覆盖率 (70% 门槛) |
| **Lint/格式化** | Oxlint + Oxfmt |
| **Web 框架** | Hono (HTTP)、WebSocket (网关) |
| **Web 前端** | Lit (Web Components)、Vite |
| **浏览器自动化** | Playwright |
| **数据库** | SQLite + sqlite-vec (向量搜索)、LanceDB |
| **AI SDK** | Anthropic SDK、OpenAI SDK、Google Generative AI、AWS Bedrock |
| **包管理** | pnpm (主)、npm (插件安装)、Bun (兼容) |
| **iOS/macOS** | SwiftUI、Swift Package Manager |
| **Android** | Kotlin、Jetpack Compose、Material 3 |
| **CI/CD** | GitHub Actions |
| **容器化** | Docker (沙箱执行)、Podman (可选) |
| **部署** | Fly.io、Render、Docker Compose |

---

## 3. 根目录文件

### 3.1 核心配置文件

| 文件 | 作用 |
|------|------|
| `package.json` | 根包配置。定义 `openclaw` CLI 入口、80+ 个 `plugin-sdk` 子路径导出、所有 npm scripts |
| `pnpm-workspace.yaml` | pnpm monorepo 工作区配置，包含 `extensions/*`、`packages/*`、`vendor/*` 等 |
| `pnpm-lock.yaml` | pnpm 锁定文件 |
| `tsconfig.json` | TypeScript 配置：ES2023 目标、NodeNext 模块、严格模式、`openclaw/plugin-sdk/*` 路径别名 |
| `tsconfig.plugin-sdk.dts.json` | Plugin SDK 类型声明专用 TS 配置 |
| `tsdown.config.ts` | tsdown 打包配置，定义产物输出到 `dist/` |
| `knip.config.ts` | Knip 无用代码检测配置 |
| `pyproject.toml` | Python 工具配置（用于部分脚本） |

### 3.2 入口与启动文件

| 文件 | 作用 |
|------|------|
| `openclaw.mjs` | CLI 二进制入口脚本。`package.json` 中 `bin.openclaw` 指向此文件 |
| `src/index.ts` | 主入口。导出库级 API（`loadConfig`、`deriveSessionKey` 等），作为 main module 时启动 CLI |
| `src/entry.ts` | CLI 启动入口。处理进程 respawn、容器目标解析、CLI profile 环境、Windows argv 规范化、编译缓存 |

### 3.3 Docker 与部署

| 文件 | 作用 |
|------|------|
| `Dockerfile` | 主 Docker 镜像构建文件 |
| `Dockerfile.sandbox` | 沙箱执行环境 Docker 镜像 |
| `Dockerfile.sandbox-browser` | 带浏览器支持的沙箱镜像 |
| `Dockerfile.sandbox-common` | 沙箱通用基础镜像 |
| `docker-compose.yml` | Docker Compose 编排配置 |
| `docker-setup.sh` | Docker 环境初始化脚本 |
| `fly.toml` | Fly.io 部署配置 |
| `fly.private.toml` | Fly.io 私有部署配置 |
| `render.yaml` | Render 平台部署配置 |
| `setup-podman.sh` | Podman 容器运行时设置脚本 |
| `openclaw.podman.env` | Podman 环境变量配置 |

### 3.4 测试配置

| 文件 | 作用 |
|------|------|
| `vitest.config.ts` | Vitest 主配置 |
| `vitest.unit.config.ts` | 单元测试配置 |
| `vitest.e2e.config.ts` | 端到端测试配置 |
| `vitest.extensions.config.ts` | 插件测试配置 |
| `vitest.channels.config.ts` | 通道测试配置 |
| `vitest.gateway.config.ts` | 网关测试配置 |
| `vitest.live.config.ts` | 实时测试配置（需要真实 API key） |
| `vitest.performance-config.ts` | 性能测试配置 |
| `vitest.scoped-config.ts` | 范围测试配置 |
| `vitest.channel-paths.mjs` | 通道测试路径映射 |
| `vitest.unit-paths.mjs` | 单元测试路径映射 |
| `vitest.pattern-file.ts` | 测试模式匹配文件 |

### 3.5 文档与社区

| 文件 | 作用 |
|------|------|
| `README.md` | 项目介绍、快速开始指南 |
| `CHANGELOG.md` | 版本变更记录 |
| `CONTRIBUTING.md` | 贡献者指南 |
| `SECURITY.md` | 安全策略 |
| `LICENSE` | MIT 许可证 |
| `VISION.md` | 项目愿景文档 |
| `AGENTS.md` | AI Agent 编码助手指南（代码规范、项目结构约定） |
| `CLAUDE.md` | Claude AI 专用的项目提示文件 |
| `docs.acp.md` | Agent Client Protocol (ACP) 协议文档 |
| `appcast.xml` | macOS 自动更新 appcast（Sparkle 框架） |
| `zizmor.yml` | GitHub Actions 安全审计工具配置 |

---

## 4. src/ — 核心源代码

`src/` 目录包含 **5,163 个 TypeScript 文件**，组织为 **48+ 个子模块**。以下按功能域分组详细说明。

### 4.1 入口层

| 文件 | 作用 |
|------|------|
| `index.ts` | 主入口。导出 `loadConfig`、`deriveSessionKey` 等库级 API；作为 main 时启动 CLI |
| `entry.ts` | CLI 启动流程：respawn 检测、容器目标解析、profile env、编译缓存初始化 |
| `entry.respawn.ts` | 进程 respawn 逻辑（配置变化后自动重启） |
| `library.ts` | 库模式导出，供 npm 消费者使用 |
| `runtime.ts` | 运行时全局初始化 |
| `globals.ts` | 全局常量定义 |
| `global-state.ts` | 全局状态管理 |
| `logger.ts` | 日志记录器 |
| `logging.ts` | 日志基础设施 |
| `version.ts` | 版本号管理 |
| `utils.ts` | 通用工具函数 |
| `param-key.ts` | 参数键常量 |
| `polls.ts` | 投票/轮询功能 |
| `poll-params.ts` | 轮询参数定义 |
| `extensionAPI.ts` | 插件 API 桥接层 |
| `channel-web.ts` | Web 通道入口（WhatsApp Web） |
| `bundled-web-search-registry.ts` | 内置网页搜索提供商注册表 |

### 4.2 agents/ — AI Agent 引擎 (992 文件)

Agent 引擎是系统核心，负责 AI 模型交互、工具调用、会话生命周期管理。

| 文件/子模块 | 作用 |
|------------|------|
| `agent-command.ts` | **核心 Agent 命令循环**（~42KB）。加载配置、调用模型、处理工具调用、管理会话 |
| `agent-paths.ts` | Agent 文件路径解析 |
| `agent-scope.ts` | Agent 作用域管理（多 Agent 隔离） |
| `defaults.ts` | Agent 默认配置值 |
| **模型集成** | |
| `model-auth.ts` | 模型认证管理（API key、OAuth） |
| `model-catalog.ts` | 模型目录（支持的所有 AI 模型列表） |
| `model-catalog.runtime.ts` | 运行时模型目录 |
| `model-compat.ts` | 模型兼容性适配层 |
| `model-alias-lines.ts` | 模型别名映射 |
| `model-fallback.*.ts` | 模型故障转移（probe、embedded、观测） |
| `model-auth-env.ts` | 模型认证环境变量 |
| `model-auth-label.ts` | 模型认证标签显示 |
| `model-auth-markers.ts` | 认证状态标记 |
| **特定提供商** | |
| `google-generative-ai.ts` | Google Gemini 集成 |
| `anthropic-vertex-provider.ts` | Anthropic Vertex AI 集成 |
| `anthropic-vertex-stream.ts` | Vertex AI 流式传输 |
| `anthropic-payload-log.ts` | Anthropic 请求负载日志 |
| `bedrock-discovery.ts` | AWS Bedrock 模型发现 |
| `byteplus-models.ts` | BytePlus（火山引擎）模型 |
| `chutes-models.ts` / `chutes-oauth.ts` | Chutes AI 模型与 OAuth |
| `deepseek-models.ts` | DeepSeek 模型定义 |
| `doubao-models.ts` | 豆包模型定义 |
| `huggingface-models.ts` | HuggingFace 模型定义 |
| `kilocode-models.ts` | Kilocode 模型定义 |
| `minimax-vlm.ts` | MiniMax 视觉语言模型 |
| `cloudflare-ai-gateway.ts` | Cloudflare AI Gateway 适配 |
| `github-copilot-token.ts` | GitHub Copilot token 管理 |
| **工具系统** | |
| `bash-tools.ts` | Bash 命令执行工具 |
| `bash-tools.exec.ts` | 命令执行核心 |
| `bash-tools.exec-runtime.ts` | 运行时执行环境 |
| `bash-tools.exec-host-gateway.ts` | 网关宿主执行 |
| `bash-tools.exec-host-node.ts` | Node 宿主执行 |
| `bash-tools.process.ts` | 进程管理（poll、send-keys、supervisor） |
| `bash-process-registry.ts` | Bash 进程注册表 |
| `channel-tools.ts` | 通道交互工具 |
| `memory-search.ts` | 记忆搜索工具 |
| **上下文管理** | |
| `context.ts` | 上下文组装（系统提示、历史消息、工具定义） |
| `context-window-guard.ts` | 上下文窗口溢出保护 |
| `context-cache.ts` | 上下文缓存 |
| `context-tokens.runtime.ts` | Token 计数运行时 |
| `compaction.ts` | 上下文压缩（长对话历史自动摘要） |
| `bootstrap-budget.ts` | Bootstrap token 预算管理 |
| `bootstrap-cache.ts` | Bootstrap 缓存 |
| `bootstrap-files.ts` | Bootstrap 文件加载 |
| `bootstrap-hooks.ts` | Bootstrap 钩子 |
| **身份与认证** | |
| `identity.ts` | Agent 身份管理（名称、头像、行为） |
| `identity-avatar.ts` | 头像处理 |
| `identity-file.ts` | 身份文件读写 |
| `auth-health.ts` | 认证健康检查 |
| `auth-profiles.ts` | 认证 profile 管理（多 key 轮换） |
| `api-key-rotation.ts` | API key 自动轮换 |
| **CLI Runner** | |
| `cli-runner.ts` | CLI 后端运行器 |
| `cli-backends.ts` | CLI 后端注册 |
| `cli-credentials.ts` | CLI 凭证管理 |
| `cli-session.ts` | CLI 会话 |
| `claude-cli-runner.ts` | Claude CLI 运行器 |
| **其他** | |
| `lanes.ts` | Agent 执行通道 |
| `fast-mode.ts` | 快速模式 |
| `failover-policy.ts` | 故障转移策略 |
| `failover-error.ts` | 故障转移错误处理 |
| `cache-trace.ts` | 缓存追踪 |
| `content-blocks.ts` | 内容块处理 |
| `current-time.ts` | 当前时间注入 |
| `date-time.ts` | 日期时间工具 |
| `announce-idempotency.ts` | 公告幂等性 |
| `image-sanitization.ts` | 图片安全处理 |
| `internal-events.ts` | 内部事件系统 |
| `mcp-stdio.ts` | MCP stdio 传输 |
| `embedded-pi-lsp.ts` | 嵌入式 Pi LSP |
| `embedded-pi-mcp.ts` | 嵌入式 Pi MCP |
| `apply-patch.ts` | 补丁应用工具 |
| `glob-pattern.ts` | Glob 模式匹配 |
| `console-sanitize.ts` | 控制台输出清理 |
| `docs-path.ts` | 文档路径解析 |
| `btw.ts` | "By the way" 提示功能 |

### 4.3 gateway/ — 网关服务器 (403 文件)

WebSocket + HTTP 网关服务器，是整个系统的通信中枢。

| 文件/子模块 | 作用 |
|------------|------|
| **服务器核心** | |
| `server-http.ts` | HTTP 服务器（基于 Hono） |
| `server-methods.ts` | WebSocket RPC 方法注册 |
| `server-methods-list.ts` | 方法列表 |
| `server-ws-runtime.ts` | WebSocket 运行时 |
| `server-shared.ts` | 服务器共享工具 |
| `server-utils.ts` | 服务器工具函数 |
| `server-constants.ts` | 服务器常量 |
| `server-runtime-state.ts` | 运行时状态管理 |
| `server-runtime-config.ts` | 运行时配置 |
| **启动与生命周期** | |
| `boot.ts` | 网关启动流程 |
| `server-startup.ts` | 服务器启动序列 |
| `server-startup-log.ts` | 启动日志 |
| `server-startup-memory.ts` | 启动时内存初始化 |
| `server-startup-matrix-migration.ts` | Matrix 迁移 |
| `server-close.ts` | 服务器关闭流程 |
| `server-restart-sentinel.ts` | 重启哨兵 |
| `server-maintenance.ts` | 维护模式 |
| **通道管理** | |
| `server-channels.ts` | 通道注册与管理 |
| `server-plugins.ts` | 插件集成 |
| `server-discovery.ts` | 服务发现 |
| `server-discovery-runtime.ts` | 服务发现运行时 |
| **会话与对话** | |
| `server-chat.ts` | 对话处理 |
| `chat-abort.ts` | 对话中止 |
| `chat-sanitize.ts` | 对话内容清理 |
| `chat-attachments.ts` | 附件处理 |
| `server-session-key.ts` | 会话 key 管理 |
| `server-wizard-sessions.ts` | 向导会话 |
| `agent-prompt.ts` | Agent 提示词组装 |
| `agent-list.ts` | Agent 列表 |
| `agent-event-assistant-text.ts` | Agent 事件文本处理 |
| **认证与安全** | |
| `auth.ts` | 网关认证核心 |
| `auth-config-utils.ts` | 认证配置工具 |
| `auth-mode-policy.ts` | 认证模式策略 |
| `auth-rate-limit.ts` | 认证速率限制 |
| `auth-install-policy.ts` | 安装认证策略 |
| `connection-auth.ts` | 连接认证 |
| `device-auth.ts` | 设备认证 |
| `credentials.ts` | 凭证管理 |
| `credential-planner.ts` | 凭证规划 |
| `role-policy.ts` | 角色策略 |
| `method-scopes.ts` | 方法权限范围 |
| `origin-check.ts` | 源检查 |
| `security-path.ts` | 安全路径 |
| `probe-auth.ts` | 探针认证 |
| **HTTP API** | |
| `http-common.ts` | HTTP 通用处理 |
| `http-auth-helpers.ts` | HTTP 认证辅助 |
| `http-endpoint-helpers.ts` | 端点辅助 |
| `http-utils.ts` | HTTP 工具 |
| `openai-http.ts` | OpenAI 兼容 HTTP API |
| `openresponses-http.ts` | Open Responses HTTP API |
| `openresponses-prompt.ts` | Open Responses 提示 |
| `embeddings-http.ts` | Embeddings HTTP 端点 |
| `models-http.ts` | 模型列表 HTTP 端点 |
| `model-pricing-cache.ts` | 模型定价缓存 |
| **Control UI** | |
| `control-ui.ts` | 控制面板 UI 服务 |
| `control-ui-routing.ts` | 控制面板路由 |
| `control-ui-csp.ts` | CSP 安全策略 |
| `control-ui-shared.ts` | 控制面板共享 |
| `control-ui-http-utils.ts` | HTTP 工具 |
| `control-ui-contract.ts` | UI 契约 |
| `control-plane-audit.ts` | 控制面板审计 |
| `control-plane-rate-limit.ts` | 速率限制 |
| **网络与节点** | |
| `net.ts` | 网络工具 |
| `server-broadcast.ts` | 广播 |
| `node-registry.ts` | 节点注册表 |
| `node-pending-work.ts` | 待处理工作 |
| `server-node-events.ts` | 节点事件 |
| `server-node-subscriptions.ts` | 节点订阅 |
| `server-mobile-nodes.ts` | 移动节点 |
| `server-lanes.ts` | 执行通道 |
| `server-tailscale.ts` | Tailscale 集成 |
| **其他** | |
| `config-reload.ts` | 配置热重载 |
| `hooks.ts` / `hooks-mapping.ts` | 钩子系统 |
| `call.ts` | 通话功能 |
| `canvas-capability.ts` | Canvas 能力 |
| `probe.ts` | 健康探针 |
| `input-allowlist.ts` | 输入白名单 |
| `exec-approval-manager.ts` | 执行审批管理器 |
| `server-cron.ts` | 定时任务 |
| `server-browser.ts` | 浏览器集成 |
| `server-model-catalog.ts` | 模型目录 |

### 4.4 config/ — 配置系统 (264 文件)

完整的配置管理系统，支持 JSON5 格式、环境变量替换、密钥加密、配置校验。

| 文件/子模块 | 作用 |
|------------|------|
| **核心 I/O** | |
| `io.ts` | **配置 I/O 核心**（~67KB）。读写 JSON5 配置、密钥加密、环境变量替换 |
| `config.ts` | 配置实例管理 |
| `defaults.ts` | 默认配置值 |
| `paths.ts` | 配置文件路径 |
| `config-paths.ts` | 配置路径常量 |
| `config-env-vars.ts` | 配置环境变量 |
| **Schema 与校验** | |
| `schema.ts` | Zod schema 定义（完整配置 schema） |
| `schema-base.ts` | 基础 schema |
| `schema.base.generated.ts` | 自动生成的基础 schema |
| `schema.help.ts` | Schema 帮助文本 |
| `schema.hints.ts` | Schema 提示 |
| `schema.shared.ts` | 共享 schema |
| `schema.labels.ts` | Schema 标签 |
| `schema.tags.ts` | Schema 标签系统 |
| `schema.irc.ts` | IRC schema |
| **类型定义** (`types.*.ts`) | |
| `types.ts` | 主类型导出 |
| `types.base.ts` | 基础类型 |
| `types.agents.ts` | Agent 配置类型 |
| `types.channels.ts` | 通道配置类型 |
| `types.gateway.ts` | 网关配置类型 |
| `types.models.ts` | 模型配置类型 |
| `types.plugins.ts` | 插件配置类型 |
| `types.tools.ts` | 工具配置类型 |
| `types.hooks.ts` | 钩子配置类型 |
| `types.memory.ts` | 记忆配置类型 |
| `types.sandbox.ts` | 沙箱配置类型 |
| `types.secrets.ts` | 密钥配置类型 |
| `types.mcp.ts` | MCP 配置类型 |
| `types.cron.ts` | Cron 配置类型 |
| `types.auth.ts` | 认证配置类型 |
| `types.browser.ts` | 浏览器配置类型 |
| `types.cli.ts` | CLI 配置类型 |
| `types.acp.ts` | ACP 配置类型 |
| `types.openclaw.ts` | OpenClaw 特有类型 |
| （以及 `types.discord.ts`、`types.telegram.ts`、`types.slack.ts` 等各通道类型） | |
| **环境变量与合并** | |
| `env-substitution.ts` | 环境变量替换引擎 |
| `env-preserve.ts` | 环境变量保留 |
| `env-vars.ts` | 环境变量定义 |
| `merge-config.ts` | 配置合并 |
| `merge-patch.ts` | JSON Merge Patch |
| `includes.ts` | 配置文件 include 机制 |
| `includes-scan.ts` | include 扫描 |
| **迁移与兼容** | |
| `legacy.ts` | 旧版配置兼容 |
| `legacy-migrate.ts` | 旧版迁移 |
| `legacy.migrations.ts` | 迁移规则集 |
| `legacy.migrations.part-1/2/3.ts` | 分段迁移规则 |
| `legacy.rules.ts` | 迁移规则引擎 |
| `legacy.shared.ts` | 共享迁移工具 |
| `legacy-web-search.ts` | 旧版搜索迁移 |
| **其他** | |
| `agent-dirs.ts` | Agent 目录管理 |
| `agent-limits.ts` | Agent 限制 |
| `allowed-values.ts` | 允许值校验 |
| `bindings.ts` | 绑定配置 |
| `channel-capabilities.ts` | 通道能力定义 |
| `channel-configured.ts` | 通道配置检查 |
| `commands.ts` | 命令配置 |
| `sessions.ts` | 会话配置 |
| `logging.ts` | 日志配置 |
| `talk-defaults.ts` / `talk.ts` | Talk 模式配置 |
| `telegram-custom-commands.ts` | Telegram 自定义命令配置 |
| `discord-preview-streaming.ts` | Discord 预览流式配置 |
| `group-policy.ts` | 群组策略 |
| `runtime-group-policy.ts` | 运行时群组策略 |
| `runtime-overrides.ts` | 运行时配置覆盖 |
| `dangerous-name-matching.ts` | 危险名称匹配 |
| `gateway-control-ui-origins.ts` | 控制面板 CORS 源 |
| `mcp-config.ts` | MCP 配置 |
| `port-defaults.ts` | 默认端口 |
| `normalize-paths.ts` | 路径规范化 |
| `normalize-exec-safe-bin.ts` | 安全二进制规范化 |
| `plugins-allowlist.ts` | 插件白名单 |
| `plugin-auto-enable.ts` | 插件自动启用 |
| `redact-snapshot.ts` | 快照脱敏 |
| `doc-baseline.ts` | 文档基线 |
| `state-dir-dotenv.ts` | 状态目录 dotenv |
| `backup-rotation.ts` | 备份轮换 |
| `byte-size.ts` | 字节大小工具 |
| `cache-utils.ts` | 缓存工具 |
| `issue-format.ts` | Issue 格式化 |
| `markdown-tables.ts` | Markdown 表格生成 |
| `model-input.ts` | 模型输入 |
| `prototype-keys.ts` | 原型键保护 |

### 4.5 commands/ — CLI 命令实现 (428 文件)

所有 CLI 命令的业务逻辑实现。

| 文件/子模块 | 作用 |
|------------|------|
| **Agent 命令** | |
| `agent.ts` | `openclaw agent` 命令主入口 |
| `agent-via-gateway.ts` | 通过网关运行 Agent |
| `agents.ts` | Agent 管理命令集 |
| `agents.commands.add.ts` | 添加 Agent |
| `agents.commands.delete.ts` | 删除 Agent |
| `agents.commands.list.ts` | 列出 Agent |
| `agents.commands.bind.ts` | 绑定 Agent 到通道 |
| `agents.commands.identity.ts` | Agent 身份管理 |
| `agents.bindings.ts` | Agent 绑定 |
| `agents.config.ts` | Agent 配置 |
| `agents.providers.ts` | Agent 提供商 |
| **认证命令** | |
| `auth-choice.ts` | 认证选择流程 |
| `auth-choice.apply.ts` | 应用认证选择 |
| `auth-choice.api-key.ts` | API Key 认证 |
| `auth-choice-options.ts` | 认证选项 |
| `auth-choice-prompt.ts` | 认证提示 |
| `auth-token.ts` | Token 认证 |
| `chutes-oauth.ts` | Chutes OAuth 流程 |
| `oauth-flow.ts` | OAuth 通用流程 |
| `oauth-env.ts` | OAuth 环境 |
| `oauth-tls-preflight.ts` | OAuth TLS 预检 |
| **通道命令** | |
| `channels.ts` | 通道管理命令集（add、remove、status） |
| `channel-account-context.ts` | 通道账户上下文 |
| `configure.channels.ts` | 通道配置 |
| **配置命令** | |
| `configure.ts` | `openclaw configure` 命令 |
| `configure.commands.ts` | 配置子命令 |
| `configure.gateway.ts` | 网关配置 |
| `configure.gateway-auth.ts` | 网关认证配置 |
| `configure.daemon.ts` | 守护进程配置 |
| `configure.shared.ts` | 共享配置工具 |
| `configure.wizard.ts` | 配置向导 |
| `config-validation.ts` | 配置校验 |
| **Doctor 诊断** | |
| `doctor.ts` | `openclaw doctor` 主诊断命令 |
| `doctor-auth.ts` | 认证诊断 |
| `doctor-browser.ts` | 浏览器诊断 |
| `doctor-config-flow.ts` | 配置流诊断 |
| `doctor-config-analysis.ts` | 配置分析 |
| `doctor-cron.ts` | Cron 诊断 |
| `doctor-gateway-*.ts` | 网关诊断系列 |
| `doctor-install.ts` | 安装诊断 |
| `doctor-legacy-config.ts` | 旧配置诊断 |
| `doctor-memory-search.ts` | 记忆搜索诊断 |
| `doctor-platform-notes.ts` | 平台说明 |
| `doctor-sandbox.ts` | 沙箱诊断 |
| `doctor-security.ts` | 安全诊断 |
| `doctor-session-locks.ts` | 会话锁诊断 |
| `doctor-state-integrity.ts` | 状态完整性诊断 |
| `doctor-state-migrations.ts` | 状态迁移诊断 |
| `doctor-update.ts` | 更新诊断 |
| `doctor-workspace*.ts` | 工作区诊断 |
| **模型命令** | |
| `models.ts` | 模型管理命令（list、set） |
| `model-picker.ts` | 模型选择器 |
| `model-default.ts` | 默认模型 |
| `model-allowlist.ts` | 模型白名单 |
| **其他命令** | |
| `backup.ts` / `backup-verify.ts` | 备份命令 |
| `cleanup-plan.ts` / `cleanup-utils.ts` | 清理工具 |
| `daemon-install-helpers.ts` | 守护进程安装 |
| `daemon-runtime.ts` | 守护进程运行时 |
| `dashboard.ts` | 仪表盘 |
| `docs.ts` | 文档命令 |
| `gateway-status.ts` | 网关状态 |
| `gateway-install-token.ts` | 网关安装 token |
| `gateway-presence.ts` | 网关在线状态 |
| `health.ts` | 健康检查命令 |
| `message.ts` | 消息命令 |
| `ollama-setup.ts` | Ollama 设置 |
| `onboard-*.ts` | 引导流程系列 |

### 4.6 cli/ — CLI 框架 (329 文件)

CLI 应用框架、参数解析、命令注册、交互式提示。

| 文件/子模块 | 作用 |
|------------|------|
| `program/` (57 文件) | Commander.js 命令程序定义 |
| `daemon-cli/` (28 文件) | 守护进程 CLI 子命令 |
| `gateway-cli/` (10 文件) | 网关 CLI 子命令 |
| `nodes-cli/` (17 文件) | 节点 CLI 子命令 |
| `update-cli/` (10 文件) | 更新 CLI 子命令 |
| `program.ts` | CLI 主程序入口 |
| `run-main.ts` | CLI 主运行入口 |
| `argv.ts` | 参数解析 |
| `banner.ts` | 启动横幅 |
| `browser-cli.ts` / `browser-cli-*.ts` | 浏览器管理 CLI |
| `config-cli.ts` | 配置 CLI |
| `channels-cli.ts` | 通道 CLI |
| `memory-cli.ts` | 记忆 CLI |
| `models-cli.ts` | 模型 CLI |
| `plugins-cli.ts` | 插件 CLI |
| `secrets-cli.ts` | 密钥 CLI |
| `security-cli.ts` | 安全 CLI |
| `skills-cli.ts` | 技能 CLI |
| `mcp-cli.ts` | MCP CLI |
| `hooks-cli.ts` | 钩子 CLI |
| `cron-cli.ts` | 定时任务 CLI |
| `daemon-cli.ts` | 守护进程 CLI |
| `gateway-cli.ts` | 网关 CLI |
| `nodes-cli.ts` | 节点 CLI |
| `acp-cli.ts` | ACP CLI |
| `completion-cli.ts` | Shell 自动补全 |
| `pairing-cli.ts` | 设备配对 CLI |
| `devices-cli.ts` | 设备管理 CLI |
| `logs-cli.ts` | 日志 CLI |
| `webhooks-cli.ts` | Webhook CLI |
| `sandbox-cli.ts` | 沙箱 CLI |
| `system-cli.ts` | 系统 CLI |
| `tui-cli.ts` | TUI CLI |
| `prompt.ts` | 交互式提示 |
| `progress.ts` | 进度条 |
| `profile.ts` | CLI profile 管理 |

### 4.7 plugins/ — 插件系统 (241 文件)

插件发现、加载、注册、市场、安装管理。

| 文件/子模块 | 作用 |
|------------|------|
| `contracts/` (14 文件) | 插件契约/接口定义 |
| `runtime/` (46 文件) | 插件运行时 |
| **发现与加载** | |
| `discovery.ts` | **插件发现引擎**（~26KB）。文件系统扫描、内置插件、npm 包 |
| `loader.ts` | **插件加载器**（~41KB）。Manifest 解析、入口验证、安全检查 |
| `registry.ts` | 插件注册表 |
| `manifest-registry.ts` | Manifest 注册表 |
| `manifest.ts` | Manifest 类型 |
| `schema-validator.ts` | Schema 校验器 |
| `bundled-dir.ts` | 内置插件目录 |
| `bundled-sources.ts` | 内置插件源 |
| `bundled-compat.ts` | 内置插件兼容性 |
| `bundled-plugin-metadata.ts` | 内置插件元数据 |
| `bundled-plugin-metadata.generated.ts` | 自动生成的元数据 |
| **安装与更新** | |
| `install.ts` | 插件安装 |
| `install.runtime.ts` | 安装运行时 |
| `install-security-scan.ts` | 安装安全扫描 |
| `uninstall.ts` | 卸载 |
| `update.ts` | 更新 |
| `installs.ts` | 安装管理 |
| `clawhub.ts` | ClawHub 市场 |
| `marketplace.ts` | 市场操作 |
| **提供商系统** | |
| `providers.ts` | 提供商管理 |
| `providers.runtime.ts` | 提供商运行时 |
| `provider-catalog.ts` | 提供商目录 |
| `provider-discovery.ts` | 提供商发现 |
| `provider-runtime.ts` | 提供商运行时核心 |
| `provider-auth-*.ts` | 提供商认证系列 (choice、key、mode、oauth、storage、token 等) |
| `provider-model-*.ts` | 提供商模型系列 (allowlist、defaults、definitions、helpers 等) |
| `provider-wizard.ts` | 提供商设置向导 |
| `provider-validation.ts` | 提供商校验 |
| **命令与钩子** | |
| `commands.ts` | 插件命令 |
| `command-registration.ts` | 命令注册 |
| `bundle-commands.ts` | 内置命令 |
| `hooks.ts` | 插件钩子 |
| `wired-hooks-*.ts` | 已连线钩子系列 |
| **通道与搜索** | |
| `channel-plugin-ids.ts` | 通道插件 ID |
| `web-search-providers.ts` | 网页搜索提供商 |
| `bundled-web-search.ts` | 内置网页搜索 |
| `bundled-web-search-ids.ts` | 内置搜索 ID |
| **其他** | |
| `cli.ts` | 插件 CLI 集成 |
| `config-state.ts` | 配置状态 |
| `conversation-binding.ts` | 对话绑定 |
| `enable.ts` | 插件启用 |
| `http-registry.ts` | HTTP 注册表 |
| `interactive.ts` | 交互式操作 |
| `logger.ts` | 插件日志 |
| `roots.ts` | 插件根目录 |
| `sdk-alias.ts` | SDK 别名 |
| `services.ts` | 插件服务 |
| `slots.ts` | 插件插槽 |
| `status.ts` | 插件状态 |
| `tools.ts` | 插件工具 |
| `types.ts` | 插件类型 |
| `setup-binary.ts` | 二进制设置 |
| `setup-browser.ts` | 浏览器设置 |
| `signal-cli-install.ts` | Signal CLI 安装 |

### 4.8 plugin-sdk/ — 插件 SDK (217 文件)

为插件开发者提供的 SDK，包含 80+ 个子路径导出。

| 文件 | 作用 |
|------|------|
| `index.ts` | SDK 根入口。导出 `ChannelPlugin`、`OpenClawPluginApi`、`ProviderAuthContext` |
| `core.ts` | 核心 SDK 辅助（通道插件定义、适配器创建） |
| 其他 216 个文件 | 各子路径实现（provider、channel、memory、tools 等模块的公开 API 面） |

### 4.9 security/ — 安全审计 (38 文件)

配置安全审计系统，检测危险配置、工具策略、DM 策略等。

| 文件 | 作用 |
|------|------|
| `audit.ts` | **安全审计核心**（~56KB）。完整的配置安全检查 |
| `audit.runtime.ts` | 审计运行时 |
| `audit.deep.runtime.ts` | 深度审计运行时 |
| `audit.nondeep.runtime.ts` | 非深度审计 |
| `audit-channel.ts` | 通道安全审计 |
| `audit-channel.*.runtime.ts` | 各通道特定审计 |
| `audit-extra.ts` | 额外审计检查 |
| `audit-fs.ts` | 文件系统审计 |
| `audit-tool-policy.ts` | 工具策略审计 |
| `fix.ts` | 自动修复 |
| `dangerous-config-flags.ts` | 危险配置标志 |
| `dangerous-tools.ts` | 危险工具列表 |
| `dm-policy-shared.ts` | DM 策略共享 |
| `channel-metadata.ts` | 通道元数据 |
| `config-regex.ts` | 配置正则 |
| `external-content.ts` | 外部内容安全 |
| `mutable-allowlist-detectors.ts` | 可变白名单检测 |
| `safe-regex.ts` | 安全正则 |
| `scan-paths.ts` | 扫描路径 |
| `secret-equal.ts` | 密钥比较（时序安全） |
| `skill-scanner.ts` | 技能扫描器 |
| `temp-path-guard.ts` | 临时路径保护 |
| `windows-acl.ts` | Windows ACL |

### 4.10 memory/ — 记忆系统 (106 文件)

向量记忆系统，支持语义搜索、混合搜索、多种嵌入提供商。

| 文件 | 作用 |
|------|------|
| **管理器** | |
| `manager.ts` | **记忆索引管理器**（~28KB）。向量嵌入、同步、搜索 |
| `manager-embedding-ops.ts` | 嵌入操作 |
| `manager-search.ts` | 搜索操作 |
| `manager-sync-ops.ts` | 同步操作 |
| `manager-runtime.ts` | 运行时管理 |
| `search-manager.ts` | 搜索管理器 |
| **QMD 高级记忆** | |
| `qmd-manager.ts` | **QMD 管理器**（~68KB）。Query-Memory-Document 高级记忆管理 |
| `qmd-process.ts` | QMD 处理 |
| `qmd-query-parser.ts` | QMD 查询解析 |
| `qmd-scope.ts` | QMD 作用域 |
| **嵌入提供商** | |
| `embeddings.ts` | 嵌入主入口 |
| `embeddings-openai.ts` | OpenAI 嵌入 |
| `embeddings-gemini.ts` | Gemini 嵌入 |
| `embeddings-voyage.ts` | Voyage AI 嵌入 |
| `embeddings-ollama.ts` | Ollama 本地嵌入 |
| `embeddings-mistral.ts` | Mistral 嵌入 |
| `embeddings-remote-*.ts` | 远程嵌入客户端 |
| `embeddings-model-normalize.ts` | 模型归一化 |
| `embedding-vectors.ts` | 向量工具 |
| `embedding-inputs.ts` | 嵌入输入处理 |
| `embedding-chunk-limits.ts` | 分块限制 |
| `embedding-input-limits.ts` | 输入限制 |
| `embedding-model-limits.ts` | 模型限制 |
| **批处理** | |
| `batch-runner.ts` | 批处理运行器 |
| `batch-openai.ts` / `batch-gemini.ts` / `batch-voyage.ts` | 各提供商批处理 |
| `batch-http.ts` | HTTP 批处理 |
| `batch-upload.ts` | 批量上传 |
| `batch-status.ts` | 批处理状态 |
| `batch-output.ts` | 批处理输出 |
| `batch-utils.ts` | 批处理工具 |
| **搜索** | |
| `hybrid.ts` | 混合搜索（BM25 + 向量） |
| `mmr.ts` | MMR 多样性搜索 |
| `query-expansion.ts` | 查询扩展 |
| `temporal-decay.ts` | 时间衰减 |
| `prompt-section.ts` | 提示词段 |
| **存储** | |
| `sqlite-vec.ts` | SQLite-vec 向量存储 |
| `sqlite.ts` | SQLite 存储 |
| `index.ts` | 记忆索引 |
| `internal.ts` | 内部存储 |
| `session-files.ts` | 会话文件 |
| `fs-utils.ts` | 文件系统工具 |
| **其他** | |
| `backend-config.ts` | 后端配置 |
| `memory-schema.ts` | 记忆 schema |
| `multimodal.ts` | 多模态记忆 |
| `node-llama.ts` | 本地 llama 嵌入 |
| `post-json.ts` | JSON POST 工具 |
| `read-file.ts` | 文件读取 |
| `remote-http.ts` | 远程 HTTP |
| `secret-input.ts` | 密钥输入 |
| `status-format.ts` | 状态格式化 |
| `types.ts` | 类型定义 |

### 4.11 channels/ — 通道系统 (191 文件)

通道注册、消息分发、会话管理、内置通道适配。

| 文件/子模块 | 作用 |
|------------|------|
| `plugins/` (123 文件) | 通道插件适配器 |
| `allowlists/` | 白名单管理 |
| `transport/` | 传输层 |
| `web/` | Web 通道 |
| `ids.ts` | 内置通道 ID（telegram、whatsapp、discord、irc、googlechat、slack、signal、imessage、line） |
| `registry.ts` | 通道注册表 |
| `session.ts` | 通道会话 |
| `channel-config.ts` | 通道配置 |
| `allow-from.ts` | 来源白名单 |
| `allowlist-match.ts` | 白名单匹配 |
| `command-gating.ts` | 命令门控 |
| `mention-gating.ts` | @提及门控 |
| `typing.ts` | 打字状态 |
| `draft-stream-controls.ts` | 草稿流控制 |
| `draft-stream-loop.ts` | 草稿流循环 |
| `run-state-machine.ts` | 运行状态机 |
| `thread-binding-id.ts` | 线程绑定 |
| `thread-bindings-messages.ts` | 线程绑定消息 |
| `thread-bindings-policy.ts` | 线程绑定策略 |
| `conversation-label.ts` | 对话标签 |
| `model-overrides.ts` | 模型覆盖 |
| `sender-identity.ts` | 发送者身份 |
| `sender-label.ts` | 发送者标签 |
| `session-envelope.ts` | 会话信封 |
| `session-meta.ts` | 会话元数据 |
| `targets.ts` | 消息目标 |
| `chat-type.ts` | 聊天类型 |
| `config-presence.ts` | 配置在线状态 |
| `location.ts` | 位置信息 |
| `logging.ts` | 通道日志 |
| `ack-reactions.ts` | 确认反应 |
| `status-reactions.ts` | 状态反应 |
| `reply-prefix.ts` | 回复前缀 |
| `inbound-debounce-policy.ts` | 入站防抖 |
| `native-command-session-targets.ts` | 原生命令会话目标 |
| `read-only-account-inspect.ts` | 只读账户检查 |
| `account-snapshot-fields.ts` | 账户快照字段 |
| `account-summary.ts` | 账户摘要 |

### 4.12 routing/ — 消息路由 (11 文件)

消息路由核心逻辑。

| 文件 | 作用 |
|------|------|
| `resolve-route.ts` | **路由解析核心**（~23KB）。从通道/账户解析到 Agent 会话 |
| `session-key.ts` | 会话 key 构建/解析（~8KB） |
| `account-id.ts` | 账户 ID 标准化 |
| `account-lookup.ts` | 账户查找 |
| `bindings.ts` | 路由绑定 |
| `default-account-warnings.ts` | 默认账户警告 |

### 4.13 acp/ — Agent Client Protocol (55 文件)

ACP 协议实现层，支持 stdio 和 WebSocket 传输。

| 文件/子模块 | 作用 |
|------------|------|
| `control-plane/` (12 文件) | 控制面板 ACP |
| `runtime/` (13 文件) | ACP 运行时 |
| `client.ts` | ACP 客户端 |
| `server.ts` | ACP 服务器 |
| `session.ts` | ACP 会话 |
| `translator.ts` | ACP 消息翻译器 |
| `event-mapper.ts` | 事件映射 |
| `session-mapper.ts` | 会话映射 |
| `commands.ts` | ACP 命令 |
| `conversation-id.ts` | 对话 ID |
| `meta.ts` | ACP 元数据 |
| `policy.ts` | ACP 策略 |
| `secret-file.ts` | 密钥文件 |
| `persistent-bindings.*.ts` | 持久化绑定 |
| `types.ts` | ACP 类型 |

### 4.14 auto-reply/ — 自动回复引擎 (355 文件)

消息处理、命令检测、回复生成、流式传输。

| 文件/子模块 | 作用 |
|------------|------|
| `reply/` (283 文件) | 回复生成核心（模板、流式、指令、沙箱媒体等） |
| `reply.ts` | 回复主入口 |
| `status.ts` | **回复状态机**（~36KB）。AI 响应生成状态管理 |
| `dispatch.ts` | 消息分发 |
| `envelope.ts` | 消息信封 |
| `chunk.ts` | 消息分块（按平台大小分割） |
| `command-detection.ts` | 命令检测 |
| `command-auth.ts` | 命令认证 |
| `command-control.ts` | 命令控制 |
| `commands-registry.ts` | 命令注册表 |
| `commands-args.ts` | 命令参数 |
| `model.ts` | 模型运行时 |
| `model-runtime.ts` | 模型运行时配置 |
| `tokens.ts` | Token 计数 |
| `thinking.ts` | 思考过程 |
| `templating.ts` | 模板系统 |
| `send-policy.ts` | 发送策略 |
| `skill-commands.ts` | 技能命令 |
| `group-activation.ts` | 群组激活 |
| `heartbeat.ts` | 心跳回复 |
| `inbound-debounce.ts` | 入站防抖 |
| `media-note.ts` | 媒体说明 |
| `fallback-state.ts` | 回退状态 |
| `tool-meta.ts` | 工具元数据 |
| `types.ts` | 类型定义 |

### 4.15 infra/ — 基础设施 (538 文件)

底层基础设施：网络、文件系统、安全、进程管理等。

| 文件/子模块 | 作用 |
|------------|------|
| **执行审批系统** | |
| `exec-approvals.ts` | 执行审批核心 |
| `exec-approvals-allowlist.ts` | 审批白名单 |
| `exec-approvals-analysis.ts` | 命令分析 |
| `exec-approval-*.ts` | 审批系列（forwarder、reply、surface、session-target 等） |
| `exec-safety.ts` | 执行安全检查 |
| `exec-safe-bin-policy.ts` | 安全二进制策略 |
| `exec-obfuscation-detect.ts` | 混淆检测 |
| `exec-command-resolution.ts` | 命令解析 |
| `exec-wrapper-*.ts` | 执行包装器 |
| `exec-inline-eval.ts` | 内联求值检测 |
| `exec-host.ts` | 执行宿主 |
| `executable-path.ts` | 可执行路径 |
| **设备与认证** | |
| `device-identity.ts` | 设备身份 |
| `device-auth-store.ts` | 设备认证存储 |
| `device-bootstrap.ts` | 设备引导 |
| `device-pairing.ts` | 设备配对 |
| **网络与发现** | |
| `fetch.ts` | HTTP 抓取 |
| `bonjour.ts` | Bonjour/mDNS 服务发现 |
| `bonjour-discovery.ts` | 发现实现 |
| `bonjour-ciao.ts` | Ciao 实现 |
| `gateway-discovery-targets.ts` | 网关发现目标 |
| `gateway-lock.ts` | 网关锁 |
| `gateway-processes.ts` | 网关进程 |
| `gateway-process-argv.ts` | 网关进程参数 |
| **文件与存储** | |
| `archive.ts` | 归档工具 |
| `archive-staging.ts` | 归档暂存 |
| `archive-path.ts` | 归档路径 |
| `backup-create.ts` | 备份创建 |
| `file-identity.ts` | 文件身份 |
| `file-lock.ts` | 文件锁 |
| `fs-safe.ts` | 安全文件操作 |
| `fs-pinned-write-helper.ts` | 固定写入 |
| `hardlink-guards.ts` | 硬链接保护 |
| `boundary-file-read.ts` | 边界文件读取 |
| `boundary-path.ts` | 边界路径 |
| `home-dir.ts` | Home 目录 |
| **心跳系统** | |
| `heartbeat-runner.ts` | 心跳运行器 |
| `heartbeat-events.ts` | 心跳事件 |
| `heartbeat-events-filter.ts` | 事件过滤 |
| `heartbeat-active-hours.ts` | 活跃时段 |
| `heartbeat-reason.ts` | 心跳原因 |
| `heartbeat-summary.ts` | 心跳摘要 |
| `heartbeat-visibility.ts` | 心跳可见性 |
| `heartbeat-wake.ts` | 心跳唤醒 |
| **安全** | |
| `host-env-security.ts` | 宿主环境安全 |
| `host-env-security-policy.json` | 安全策略配置 |
| **其他** | |
| `agent-events.ts` | Agent 事件 |
| `backoff.ts` | 退避策略 |
| `binaries.ts` | 二进制管理 |
| `brew.ts` | Homebrew 集成 |
| `canvas-host-url.ts` | Canvas URL |
| `channel-activity.ts` | 通道活动 |
| `channel-summary.ts` | 通道摘要 |
| `channels-status-issues.ts` | 通道状态问题 |
| `clawhub.ts` | ClawHub 客户端 |
| `cli-root-options.ts` | CLI 根选项 |
| `clipboard.ts` | 剪贴板 |
| `control-ui-assets.ts` | 控制面板资源 |
| `dedupe.ts` | 去重 |
| `detect-binary.ts` | 二进制检测 |
| `detect-package-manager.ts` | 包管理器检测 |
| `diagnostic-events.ts` | 诊断事件 |
| `diagnostic-flags.ts` | 诊断标志 |
| `dotenv.ts` | dotenv 加载 |
| `env.ts` | 环境工具 |
| `errors.ts` | 错误处理 |
| `fixed-window-rate-limit.ts` | 固定窗口限流 |
| `gaxios-fetch-compat.ts` | Gaxios 兼容 |
| `gemini-auth.ts` | Gemini 认证 |
| `git-commit.ts` | Git 提交 |
| `git-root.ts` | Git 根目录 |
| `google-api-base-url.ts` | Google API URL |
| `http-body.ts` | HTTP body |
| `install-*.ts` | 安装工具系列 |
| `abort-signal.ts` | 中止信号 |

### 4.16 其他 src/ 子模块

| 模块 | 文件数 | 作用 |
|------|--------|------|
| `browser/` | 161 | Playwright 浏览器自动化。Chrome 管理、CDP 代理、页面控制、截图、Cookie、Profile 管理 |
| `cron/` | 117 | 定时任务系统。调度、执行、投递、会话管理 |
| `shared/` | 80 | 跨模块共享工具函数 |
| `tui/` | 48 | 终端 UI（TUI）。Ink 组件、交互式界面 |
| `media/` | 48 | 媒体处理管道。音频转码（FFmpeg）、图片处理（Sharp）、格式转换 |
| `media-understanding/` | 56 | 媒体理解。图片/音频/视频内容理解、多模态处理 |
| `hooks/` | 56 | 钩子系统。消息钩子、生命周期钩子、Gmail 集成 |
| `daemon/` | 54 | 守护进程管理。macOS launchd、Linux systemd、Windows schtasks |
| `secrets/` | 53 | 密钥管理。加密存储、密钥引用 |
| `sessions/` | 15 | 会话管理。持久化、恢复、锁 |
| `tts/` | 12 | TTS 语音合成。ElevenLabs、OpenAI、Edge TTS |
| `terminal/` | 19 | 终端工具 |
| `logging/` | 30 | 日志框架 |
| `markdown/` | 14 | Markdown 处理 |
| `utils/` | 30 | 工具函数 |
| `process/` | 28 | 进程管理 |
| `pairing/` | 9 | 设备配对 |
| `node-host/` | 16 | Node 宿主 |
| `image-generation/` | 9 | 图片生成 |
| `wizard/` | 16 | 设置向导 |
| `types/` | 11 | 全局类型定义 |
| `test-utils/` | 35 | 测试工具 |
| `context-engine/` | — | 上下文引擎注册表（可插拔替换） |
| `i18n/` | — | 国际化 |
| `interactive/` | — | 交互式消息 |
| `bindings/` | — | 绑定 |
| `bootstrap/` | — | 引导 |
| `canvas-host/` | — | Canvas 宿主 |
| `compat/` | — | 兼容层 |
| `docs/` | — | 文档 |
| `extensions/` | — | 插件扩展桥接 |
| `line/` | — | Line 通道 |
| `link-understanding/` | — | 链接理解 |
| `scripts/` | — | 内部脚本 |
| `web-search/` | — | 网页搜索 |
| `test-helpers/` | — | 测试辅助 |

---

## 5. extensions/ — 插件系统

`extensions/` 目录包含 **82 个插件**（2,506 个 TypeScript 文件），每个插件都有独立的 `openclaw.plugin.json` manifest 和 `package.json`。以下按功能分类。

### 5.1 LLM 提供商插件 (36 个)

连接各种 AI 模型提供商的适配器。

| 插件 | 作用 |
|------|------|
| `openai/` (15 文件) | OpenAI (GPT 系列) 提供商 |
| `anthropic/` | Anthropic (Claude 系列) 提供商 |
| `anthropic-vertex/` | Anthropic via Google Vertex AI |
| `google/` (22 文件) | Google Gemini 提供商 |
| `amazon-bedrock/` | AWS Bedrock 多模型网关 |
| `microsoft/` | Azure OpenAI 提供商 |
| `deepseek/` | DeepSeek 提供商 |
| `mistral/` | Mistral AI 提供商 |
| `groq/` | Groq 高速推理 |
| `together/` | Together AI 提供商 |
| `ollama/` | Ollama 本地模型 |
| `xai/` (13 文件) | xAI (Grok) 提供商 |
| `openrouter/` | OpenRouter 多模型路由 |
| `perplexity/` | Perplexity AI 提供商 |
| `huggingface/` | HuggingFace Inference API |
| `nvidia/` | NVIDIA NIM 提供商 |
| `minimax/` (11 文件) | MiniMax 提供商 |
| `moonshot/` | Moonshot (Kimi) 提供商 |
| `qianfan/` | 百度千帆提供商 |
| `modelstudio/` | 阿里通义 ModelStudio |
| `volcengine/` | 火山引擎（豆包） |
| `byteplus/` | BytePlus（海外火山引擎） |
| `venice/` | Venice AI 提供商 |
| `chutes/` | Chutes AI 提供商 |
| `sglang/` | SGLang 提供商 |
| `vllm/` | vLLM 提供商 |
| `kilocode/` | Kilocode 提供商 |
| `kimi-coding/` | Kimi Coding 提供商 |
| `github-copilot/` (12 文件) | GitHub Copilot 提供商 |
| `copilot-proxy/` | Copilot 代理 |
| `zai/` (9 文件) | Z.AI 提供商 |
| `fal/` | Fal.ai (图像生成) |
| `lobster/` (11 文件) | Lobster 提供商 |
| `cloudflare-ai-gateway/` | Cloudflare AI Gateway 中间层 |
| `vercel-ai-gateway/` | Vercel AI Gateway 中间层 |
| `xiaomi/` | 小米大模型提供商 |

### 5.2 消息通道插件 (22 个)

各种消息平台的完整集成。

| 插件 | 文件数 | 作用 |
|------|--------|------|
| `telegram/` | 193 | **Telegram** 完整集成：Bot 管理、内联键盘、媒体、原生命令、Webhook |
| `discord/` | 245 | **Discord** 完整集成：Guild/Channel 管理、组件、PluralKit、Webhook 活动 |
| `slack/` | 160 | **Slack** 完整集成：Channels、Blocks、Modals、交互式消息、线程 |
| `whatsapp/` | 125 | **WhatsApp** Web 集成：消息收发、媒体、群组 |
| `feishu/` | 135 | **飞书/Lark** 集成：Bot、Bitable、DocX、Drive、卡片交互 |
| `msteams/` | 118 | **Microsoft Teams** 集成：消息、卡片、会话 |
| `matrix/` | 218 | **Matrix** 协议集成：房间、加密、桥接 |
| `signal/` | 57 | **Signal** 集成：消息、群组、媒体 |
| `imessage/` | 53 | **iMessage** 集成（macOS）：消息、群组 |
| `line/` | 72 | **LINE** 集成：消息、Flex Message |
| `irc/` | 39 | **IRC** 集成：通道、消息 |
| `googlechat/` | 41 | **Google Chat** 集成：Space、消息、卡片 |
| `mattermost/` | 76 | **Mattermost** 集成：通道、消息 |
| `nostr/` | 39 | **Nostr** 去中心化社交协议 |
| `nextcloud-talk/` | 46 | **Nextcloud Talk** 集成 |
| `synology-chat/` | 29 | **Synology Chat** 集成 |
| `tlon/` | 55 | **Tlon (Urbit)** 集成 |
| `twitch/` | 38 | **Twitch** 直播聊天集成 |
| `zalo/` | 46 | **Zalo OA** (越南) 集成 |
| `zalouser/` | 52 | **Zalo User** 个人号集成 |
| `bluebubbles/` | 63 | **BlueBubbles** (iMessage 替代方案) |
| `voice-call/` | 85 | **电话系统**：Twilio / Telnyx / Plivo、TTS、STT、ngrok/Tailscale 隧道 |

### 5.3 网页搜索插件 (8 个)

| 插件 | 作用 |
|------|------|
| `brave/` | Brave Search API |
| `duckduckgo/` | DuckDuckGo 搜索 |
| `exa/` | Exa AI 搜索 |
| `tavily/` (11 文件) | Tavily AI 搜索 |
| `firecrawl/` (10 文件) | Firecrawl 网页抓取 |
| `perplexity/` | Perplexity 搜索（复用 LLM 插件） |

### 5.4 语音与音频插件 (5 个)

| 插件 | 作用 |
|------|------|
| `elevenlabs/` | ElevenLabs TTS/STT |
| `deepgram/` | Deepgram 语音识别 |
| `talk-voice/` | Talk 模式语音处理 |
| `voice-call/` | 电话通话（见消息通道） |

### 5.5 运行时与沙箱插件 (2 个)

| 插件 | 作用 |
|------|------|
| `openshell/` (14 文件) | OpenShell 远程执行环境 |
| `opencode-go/` | OpenCode Go 运行时 |

### 5.6 记忆插件 (2 个)

| 插件 | 作用 |
|------|------|
| `memory-core/` | 核心记忆后端 |
| `memory-lancedb/` | LanceDB 向量记忆后端 |

### 5.7 工具与技能插件 (6 个)

| 插件 | 作用 |
|------|------|
| `diffs/` (30 文件) | Diff 工具（代码差异展示） |
| `llm-task/` | LLM 子任务工具 |
| `phone-control/` | 手机控制工具 |
| `thread-ownership/` | 线程所有权管理 |
| `device-pair/` | 设备配对工具 |
| `acpx/` (27 文件) | ACP 扩展协议 |

### 5.8 基础设施插件 (4 个)

| 插件 | 作用 |
|------|------|
| `diagnostics-otel/` | OpenTelemetry 诊断 |
| `qwen-portal-auth/` (9 文件) | 通义千问门户认证 |
| `synthetic/` | 合成数据/测试 |
| `opencode/` | OpenCode 集成 |

### 5.9 共享模块

| 目录 | 作用 |
|------|------|
| `shared/` | 跨插件共享工具和类型 |
| `open-prose/` (92 文件) | Open Prose 文档格式（64 个 `.prose` 文件 + 23 个 `.md`） |

---

## 6. apps/ — 原生客户端

`apps/` 包含 **835 个文件**，覆盖 iOS、macOS、Android 三个平台和跨平台共享代码。

### 6.1 apps/ios/ — iOS 客户端 (186 文件)

| 目录/文件 | 作用 |
|-----------|------|
| `OpenClaw/` | 主 App 工程（SwiftUI） |
| `OpenClawApp.swift` | App 入口 |
| `Views/` | UI 视图（对话、设置、Agent 列表等） |
| `ViewModels/` | MVVM ViewModel 层 |
| `Services/` | 网络服务、WebSocket 通信 |
| `Models/` | 数据模型 |
| `Extensions/` | Swift 扩展 |
| `Assets.xcassets/` | 图片和颜色资源 |

### 6.2 apps/macos/ — macOS 客户端 (369 文件)

| 目录/文件 | 作用 |
|-----------|------|
| `OpenClawMac/` | 主 App 工程（Swift Package） |
| `Sources/` | 源代码 |
| `AppDelegate.swift` | 应用生命周期 |
| `MenuBar/` | 菜单栏集成 |
| `SystemExtension/` | 系统扩展 |
| `Preferences/` | 偏好设置 |
| `Package.swift` | SPM 包定义 |

### 6.3 apps/android/ — Android 客户端 (161 文件)

| 目录/文件 | 作用 |
|-----------|------|
| `app/src/main/` | 主源代码 |
| `kotlin/` | Kotlin 代码（Jetpack Compose + Material 3） |
| `ui/` | Compose UI 组件 |
| `viewmodel/` | ViewModel 层 |
| `service/` | 后台服务、WebSocket |
| `data/` | 数据层 |
| `res/` | Android 资源 |
| `build.gradle.kts` | Gradle 构建配置 |

### 6.4 apps/shared/ — 跨平台共享 (119 文件)

| 目录/文件 | 作用 |
|-----------|------|
| `Models/` | 跨平台数据模型（Swift） |
| `Services/` | 共享服务逻辑 |
| `WebSocket/` | WebSocket 客户端库 |
| `Networking/` | 网络通信 |
| `Config/` | 配置共享 |

---

## 7. ui/ — Web 前端

`ui/` 包含 **243 个文件**，基于 **Lit (Web Components)** + **Vite** 构建。

| 目录/文件 | 作用 |
|-----------|------|
| `src/main.ts` | 前端入口 |
| `src/styles.css` | 全局样式 |
| `src/local-storage.ts` | 本地存储管理 |
| `src/css.d.ts` | CSS 模块类型声明 |
| `src/ui/` (205 文件) | Web Components 组件库 |
| `src/styles/` (12 文件) | CSS 样式文件 |
| `src/i18n/` (12 文件) | 国际化（多语言） |
| `src/types/` | 类型定义 |

---

## 8. Swabble/ — 语音唤醒守护进程

`Swabble/` 是一个 **Swift 6.2** 编写的 macOS 守护进程，使用 macOS 26 `Speech.framework` 实现本地离线语音唤醒词检测。

| 文件 | 作用 |
|------|------|
| `Package.swift` | SPM 包定义 |
| `Sources/Swabble/` | 主源代码（25 个 Swift 文件） |
| `SwabbleApp.swift` | 应用入口 |
| `SpeechRecognizer.swift` | 语音识别核心 |
| `WakeWordDetector.swift` | 唤醒词检测 |
| `AudioEngine.swift` | 音频引擎 |
| `README.md` | 使用文档 |

---

## 9. scripts/ — 构建与运维脚本

`scripts/` 包含 **244 个文件**（Shell、MJS、TypeScript、Go），覆盖构建、测试、发布、运维全流程。

| 子目录/文件 | 语言 | 作用 |
|------------|------|------|
| `build-*.sh` | Shell | 各平台构建脚本 |
| `package-mac-app.sh` | Shell | macOS App 打包 |
| `release-*.mjs` | MJS | 发布流程自动化 |
| `docs-i18n/` | TS | 文档国际化翻译管道 |
| `gen-*.ts` | TS | 代码/文档生成器 |
| `test-*.sh` | Shell | 测试辅助脚本 |
| `docker-*.sh` | Shell | Docker 构建/推送 |
| `benchmark/` | Go/TS | 性能基准测试 |
| `migration-*.ts` | TS | 数据迁移脚本 |
| `ci-*.sh` | Shell | CI/CD 流程脚本 |

---

## 10. docs/ — 项目文档

`docs/` 包含 **740 个文件**（685 个 Markdown + 图片/SVG），基于 **Mintlify** 文档平台。

| 子目录 | 作用 |
|--------|------|
| `channels/` | 各消息通道配置指南 |
| `configuration/` | 配置参考文档 |
| `plugins/` | 插件开发指南 |
| `providers/` | AI 提供商配置 |
| `quickstart/` | 快速开始 |
| `help/` | 常见问题/故障排查 |
| `zh-CN/` | 中文翻译（自动生成） |
| `.i18n/` | 国际化配置与术语表 |
| `.generated/` | 自动生成的基线工件 |
| `images/` | 文档图片 |
| `mint.json` | Mintlify 配置 |

---

## 11. vendor/ — 第三方代码

`vendor/` 包含 **158 个文件**，是内部维护或修改的第三方库。

| 子目录 | 作用 |
|--------|------|
| 各第三方库目录 | 内部 fork/修改的依赖，以 workspace 包形式管理 |

---

## 12. packages/ — 工作区子包

`packages/` 包含独立的工作区子包。

| 子包 | 作用 |
|------|------|
| 各子包目录 | 可独立发布的库/工具包 |

---

## 13. skills/ — AI 技能定义

`skills/` 包含 **69 个文件**（56 个 Markdown + 7 个 Python + 4 个 Shell），定义 AI Agent 可使用的技能。

| 子目录 | 作用 |
|--------|------|
| 各技能目录 | 每个目录一个技能定义（`SKILL.md` 描述 + 可选脚本） |

---

## 14. test/ — 测试基础设施

`test/` 包含 **115 个文件**。

| 子目录/文件 | 作用 |
|------------|------|
| `fixtures/` | 测试固定数据 |
| `helpers/` | 测试辅助工具 |
| `e2e/` | 端到端测试 |
| `*.tar` / `*.tgz` / `*.zip` | 测试用的归档文件 |

---

## 15. 其他支撑目录

| 目录 | 作用 |
|------|------|
| `assets/` | 项目资源文件（Logo PNG/SVG 等） |
| `git-hooks/` | Git 钩子脚本（pre-commit 等） |
| `patches/` | pnpm patch 补丁（第三方依赖修复） |
| `test-fixtures/` | 额外测试固定数据 |

---

## 16. 架构关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                        Native Clients                           │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│   │ iOS App  │  │macOS App │  │Android   │  │ Web UI   │      │
│   │ (Swift)  │  │ (Swift)  │  │(Kotlin)  │  │  (Lit)   │      │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
│        └──────────────┴──────────────┴──────────────┘           │
│                          WebSocket / HTTP                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                     Gateway Server                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │ HTTP API │  │WebSocket │  │ OpenAI   │  │ Control UI   │   │
│  │ (Hono)   │  │  Server  │  │ Compat   │  │   Panel      │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │  Auth    │  │ Routing  │  │ Session  │  │  Channels    │   │
│  │ System   │  │ Engine   │  │ Manager  │  │  Registry    │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│  Agent Engine │   │  Plugin       │   │  Channel      │
│               │   │  System       │   │  Adapters     │
│ ┌───────────┐ │   │ ┌───────────┐ │   │               │
│ │ Model     │ │   │ │ Discovery │ │   │  82 Plugins:  │
│ │ Catalog   │ │   │ │ & Loader  │ │   │  ┌─────────┐  │
│ ├───────────┤ │   │ ├───────────┤ │   │  │Telegram │  │
│ │ Tool      │ │   │ │ Provider  │ │   │  │Discord  │  │
│ │ System    │ │   │ │ Runtime   │ │   │  │Slack    │  │
│ ├───────────┤ │   │ ├───────────┤ │   │  │WhatsApp │  │
│ │ Context   │ │   │ │ SDK       │ │   │  │Feishu   │  │
│ │ Engine    │ │   │ │ (80+      │ │   │  │Matrix   │  │
│ ├───────────┤ │   │ │ exports)  │ │   │  │Teams    │  │
│ │ Compaction│ │   │ └───────────┘ │   │  │...20+   │  │
│ └───────────┘ │   └───────────────┘   │  └─────────┘  │
└───────┬───────┘                       └───────────────┘
        │
┌───────▼───────────────────────────────────────────────┐
│                  Core Infrastructure                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │ Config   │  │ Memory   │  │ Security │            │
│  │ System   │  │ System   │  │ Audit    │            │
│  ├──────────┤  ├──────────┤  ├──────────┤            │
│  │ JSON5    │  │ SQLite-  │  │ Config   │            │
│  │ Schema   │  │ vec      │  │ Scanner  │            │
│  │ Env Subs │  │ LanceDB  │  │ Auto-fix │            │
│  │ Encrypt  │  │ BM25+Vec │  │ DM Policy│            │
│  └──────────┘  └──────────┘  └──────────┘            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │ Cron     │  │ Browser  │  │ Hooks    │            │
│  │ Scheduler│  │ (PW)     │  │ System   │            │
│  ├──────────┤  ├──────────┤  ├──────────┤            │
│  │ Jobs     │  │ CDP/     │  │ Lifecycle│            │
│  │ Delivery │  │ Chrome   │  │ Message  │            │
│  │ Sessions │  │ Profiles │  │ Gmail    │            │
│  └──────────┘  └──────────┘  └──────────┘            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │ ACP      │  │ Secrets  │  │ Daemon   │            │
│  │ Protocol │  │ Manager  │  │ Service  │            │
│  ├──────────┤  ├──────────┤  ├──────────┤            │
│  │ stdio    │  │ Encrypt  │  │ launchd  │            │
│  │ WebSocket│  │ Storage  │  │ systemd  │            │
│  │ Sessions │  │ Refs     │  │ schtasks │            │
│  └──────────┘  └──────────┘  └──────────┘            │
└───────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────┐
│                 External Providers                     │
│  OpenAI │ Anthropic │ Google │ Bedrock │ DeepSeek     │
│  Groq   │ Ollama    │ Mistral│ xAI     │ 30+ more    │
└───────────────────────────────────────────────────────┘
```

### 数据流说明

1. **用户消息流**: 用户 → 消息平台 → Channel Plugin → Gateway → Routing → Agent Session → AI Model → 回复 → Channel Plugin → 消息平台 → 用户
2. **CLI 命令流**: CLI 入口 (`entry.ts`) → Commander 程序 (`cli/program.ts`) → 命令实现 (`commands/*.ts`)
3. **配置流**: JSON5 文件 → `config/io.ts` 加载 → Schema 校验 → 环境变量替换 → 密钥解密 → 运行时配置
4. **插件流**: `plugins/discovery.ts` 扫描 → `plugins/loader.ts` 加载 → manifest 校验 → 注册到 `plugins/registry.ts`
5. **记忆流**: 文档 → 嵌入向量化 → SQLite-vec/LanceDB 存储 → 查询时 BM25 + 向量混合搜索 → MMR 去重 → 注入上下文

---

*本文档由自动分析生成，涵盖 OpenClaw 项目 10,000+ 源文件的完整结构。最后更新: 2026-03-25*
