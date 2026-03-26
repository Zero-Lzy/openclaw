# src/agents/ — AI Agent 引擎

> **路径**: `src/agents/`  
> **文件数**: 992 个 TypeScript 文件  
> **核心职责**: AI 模型交互、工具调用、会话生命周期管理  
> **关键入口**: `agent-command.ts` (~42KB)

---

## 模块概述

Agent 引擎是 OpenClaw 的**核心计算层**，负责：
- 与 30+ AI 模型提供商的交互（流式/非流式）
- 工具调用系统（Bash 执行、内存搜索、通道交互等）
- 上下文管理（窗口保护、缓存、压缩）
- 身份与认证（API key 轮换、多 profile）
- CLI 与网关两种运行模式

---

## 子模块拆解

### 1. 核心命令循环

Agent 的中枢控制逻辑。

| 文件 | 作用 |
|------|------|
| `agent-command.ts` | **核心 Agent 命令循环** (~42KB)。加载配置、调用模型、处理工具调用、管理会话 |
| `agent-paths.ts` | Agent 文件路径解析 |
| `agent-scope.ts` | Agent 作用域管理（多 Agent 隔离） |
| `defaults.ts` | Agent 默认配置值 |
| `lanes.ts` | Agent 执行通道 |
| `fast-mode.ts` | 快速模式（低延迟场景） |
| `failover-policy.ts` | 故障转移策略 |
| `failover-error.ts` | 故障转移错误处理 |

### 2. 模型集成

连接各 AI 提供商的适配器。

| 文件 | 作用 |
|------|------|
| `model-auth.ts` | 模型认证管理（API key、OAuth） |
| `model-catalog.ts` | 模型目录（支持的所有 AI 模型列表） |
| `model-catalog.runtime.ts` | 运行时模型目录 |
| `model-compat.ts` | 模型兼容性适配层 |
| `model-alias-lines.ts` | 模型别名映射 |
| `model-fallback.*.ts` | 模型故障转移（probe、embedded、观测） |
| `model-auth-env.ts` | 模型认证环境变量 |
| `model-auth-label.ts` | 模型认证标签显示 |
| `model-auth-markers.ts` | 认证状态标记 |

### 3. 特定提供商适配

各 AI 提供商的专用集成代码。

| 文件 | 对应提供商 |
|------|-----------|
| `google-generative-ai.ts` | Google Gemini |
| `anthropic-vertex-provider.ts` | Anthropic via Vertex AI |
| `anthropic-vertex-stream.ts` | Vertex AI 流式传输 |
| `anthropic-payload-log.ts` | Anthropic 请求负载日志 |
| `bedrock-discovery.ts` | AWS Bedrock 模型发现 |
| `byteplus-models.ts` | BytePlus（火山引擎） |
| `chutes-models.ts` / `chutes-oauth.ts` | Chutes AI |
| `deepseek-models.ts` | DeepSeek |
| `doubao-models.ts` | 豆包 |
| `huggingface-models.ts` | HuggingFace |
| `kilocode-models.ts` | Kilocode |
| `minimax-vlm.ts` | MiniMax 视觉语言模型 |
| `cloudflare-ai-gateway.ts` | Cloudflare AI Gateway |
| `github-copilot-token.ts` | GitHub Copilot |

### 4. 工具系统

Agent 可调用的各类工具。

| 文件 | 作用 |
|------|------|
| `bash-tools.ts` | Bash 命令执行工具（主入口） |
| `bash-tools.exec.ts` | 命令执行核心 |
| `bash-tools.exec-runtime.ts` | 运行时执行环境 |
| `bash-tools.exec-host-gateway.ts` | 网关宿主执行 |
| `bash-tools.exec-host-node.ts` | Node 宿主执行 |
| `bash-tools.process.ts` | 进程管理（poll、send-keys、supervisor） |
| `bash-process-registry.ts` | Bash 进程注册表 |
| `channel-tools.ts` | 通道交互工具 |
| `memory-search.ts` | 记忆搜索工具 |

### 5. 上下文管理

控制发送给 AI 模型的上下文窗口。

| 文件 | 作用 |
|------|------|
| `context.ts` | 上下文组装（系统提示、历史消息、工具定义） |
| `context-window-guard.ts` | 上下文窗口溢出保护 |
| `context-cache.ts` | 上下文缓存 |
| `context-tokens.runtime.ts` | Token 计数运行时 |
| `compaction.ts` | 上下文压缩（长对话历史自动摘要） |
| `bootstrap-budget.ts` | Bootstrap token 预算管理 |
| `bootstrap-cache.ts` | Bootstrap 缓存 |
| `bootstrap-files.ts` | Bootstrap 文件加载 |
| `bootstrap-hooks.ts` | Bootstrap 钩子 |

### 6. 身份与认证

Agent 身份管理、API key 轮换。

| 文件 | 作用 |
|------|------|
| `identity.ts` | Agent 身份管理（名称、头像、行为） |
| `identity-avatar.ts` | 头像处理 |
| `identity-file.ts` | 身份文件读写 |
| `auth-health.ts` | 认证健康检查 |
| `auth-profiles.ts` | 认证 profile 管理（多 key 轮换） |
| `api-key-rotation.ts` | API key 自动轮换 |

### 7. CLI Runner

CLI 模式下的 Agent 运行器。

| 文件 | 作用 |
|------|------|
| `cli-runner.ts` | CLI 后端运行器 |
| `cli-backends.ts` | CLI 后端注册 |
| `cli-credentials.ts` | CLI 凭证管理 |
| `cli-session.ts` | CLI 会话 |
| `claude-cli-runner.ts` | Claude CLI 运行器 |

### 8. 其他辅助

| 文件 | 作用 |
|------|------|
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

---

## 依赖关系

```
src/agents/
  ├─→ src/config/          (配置读取：模型设置、Agent 参数)
  ├─→ src/plugins/         (调用插件提供的工具和提供商)
  ├─→ src/memory/          (记忆搜索工具)
  ├─→ src/infra/           (执行审批、设备身份、网络)
  ├─→ src/security/        (工具执行安全检查)
  ├─→ src/channels/        (通道交互工具)
  ├─→ src/routing/         (会话路由解析)
  └─→ 外部 AI SDK          (anthropic, openai, @google/generative-ai, @aws-sdk/bedrock)
```

### 被谁依赖

- `src/gateway/` — 网关通过 Agent 引擎处理对话请求
- `src/auto-reply/` — 自动回复系统调用 Agent 生成回复
- `src/commands/` — CLI 命令直接运行 Agent
- `src/cli/` — CLI Runner 接入

---

## 关键功能描述

1. **命令循环** (`agent-command.ts`): Agent 的主运行循环。接收消息 → 加载配置/上下文 → 调用 AI 模型 → 处理工具调用 → 递归执行 → 返回最终回复
2. **模型故障转移**: 当主模型不可用时自动切换到备选模型，支持探针检测和嵌入式回退
3. **上下文压缩**: 长对话自动摘要历史消息以适应模型上下文窗口限制
4. **工具执行安全**: Bash 工具执行前经过安全检查、白名单校验、混淆检测

---

*← [返回总览](./00-overview.md)*
