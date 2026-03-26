# src/gateway/ — 网关服务器

> **路径**: `src/gateway/`  
> **文件数**: 403 个 TypeScript 文件  
> **核心职责**: WebSocket + HTTP 网关服务器，系统通信中枢  
> **Web 框架**: Hono (HTTP)

---

## 模块概述

Gateway 是 OpenClaw 的**通信中枢**，负责：
- HTTP REST API 服务（基于 Hono 框架）
- WebSocket 双向通信（实时消息推送）
- OpenAI 兼容 API（可作为 OpenAI API 替代）
- 控制面板 UI（Web 管理界面）
- 通道管理与注册
- 会话与认证管理
- 节点集群与服务发现

---

## 子模块拆解

### 1. 服务器核心

HTTP 与 WebSocket 服务器实现。

| 文件 | 作用 |
|------|------|
| `server-http.ts` | HTTP 服务器（基于 Hono） |
| `server-methods.ts` | WebSocket RPC 方法注册 |
| `server-methods-list.ts` | 方法列表 |
| `server-ws-runtime.ts` | WebSocket 运行时 |
| `server-shared.ts` | 服务器共享工具 |
| `server-utils.ts` | 服务器工具函数 |
| `server-constants.ts` | 服务器常量 |
| `server-runtime-state.ts` | 运行时状态管理 |
| `server-runtime-config.ts` | 运行时配置 |

### 2. 启动与生命周期

服务器启动、关闭、重启流程。

| 文件 | 作用 |
|------|------|
| `boot.ts` | 网关启动流程 |
| `server-startup.ts` | 服务器启动序列 |
| `server-startup-log.ts` | 启动日志 |
| `server-startup-memory.ts` | 启动时内存初始化 |
| `server-startup-matrix-migration.ts` | Matrix 迁移 |
| `server-close.ts` | 服务器关闭流程 |
| `server-restart-sentinel.ts` | 重启哨兵 |
| `server-maintenance.ts` | 维护模式 |

### 3. 通道管理

通道注册与插件集成。

| 文件 | 作用 |
|------|------|
| `server-channels.ts` | 通道注册与管理 |
| `server-plugins.ts` | 插件集成 |
| `server-discovery.ts` | 服务发现 |
| `server-discovery-runtime.ts` | 服务发现运行时 |

### 4. 会话与对话

对话处理与会话管理。

| 文件 | 作用 |
|------|------|
| `server-chat.ts` | 对话处理 |
| `chat-abort.ts` | 对话中止 |
| `chat-sanitize.ts` | 对话内容清理 |
| `chat-attachments.ts` | 附件处理 |
| `server-session-key.ts` | 会话 key 管理 |
| `server-wizard-sessions.ts` | 向导会话 |
| `agent-prompt.ts` | Agent 提示词组装 |
| `agent-list.ts` | Agent 列表 |
| `agent-event-assistant-text.ts` | Agent 事件文本处理 |

### 5. 认证与安全

多层认证与访问控制。

| 文件 | 作用 |
|------|------|
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

### 6. HTTP API

对外暴露的 HTTP 端点。

| 文件 | 作用 |
|------|------|
| `http-common.ts` | HTTP 通用处理 |
| `http-auth-helpers.ts` | HTTP 认证辅助 |
| `http-endpoint-helpers.ts` | 端点辅助 |
| `http-utils.ts` | HTTP 工具 |
| `openai-http.ts` | **OpenAI 兼容 HTTP API** |
| `openresponses-http.ts` | Open Responses HTTP API |
| `openresponses-prompt.ts` | Open Responses 提示 |
| `embeddings-http.ts` | Embeddings HTTP 端点 |
| `models-http.ts` | 模型列表 HTTP 端点 |
| `model-pricing-cache.ts` | 模型定价缓存 |

### 7. Control UI（控制面板）

Web 管理界面服务。

| 文件 | 作用 |
|------|------|
| `control-ui.ts` | 控制面板 UI 服务 |
| `control-ui-routing.ts` | 控制面板路由 |
| `control-ui-csp.ts` | CSP 安全策略 |
| `control-ui-shared.ts` | 控制面板共享 |
| `control-ui-http-utils.ts` | HTTP 工具 |
| `control-ui-contract.ts` | UI 契约 |
| `control-plane-audit.ts` | 控制面板审计 |
| `control-plane-rate-limit.ts` | 速率限制 |

### 8. 网络与节点

集群、广播、移动节点管理。

| 文件 | 作用 |
|------|------|
| `net.ts` | 网络工具 |
| `server-broadcast.ts` | 广播 |
| `node-registry.ts` | 节点注册表 |
| `node-pending-work.ts` | 待处理工作 |
| `server-node-events.ts` | 节点事件 |
| `server-node-subscriptions.ts` | 节点订阅 |
| `server-mobile-nodes.ts` | 移动节点 |
| `server-lanes.ts` | 执行通道 |
| `server-tailscale.ts` | Tailscale 集成 |

### 9. 其他

| 文件 | 作用 |
|------|------|
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

---

## 依赖关系

```
src/gateway/
  ├─→ src/agents/          (调用 Agent 引擎处理对话)
  ├─→ src/config/          (加载网关配置)
  ├─→ src/channels/        (注册和管理消息通道)
  ├─→ src/plugins/         (加载和管理插件)
  ├─→ src/routing/         (消息路由解析)
  ├─→ src/auto-reply/      (自动回复处理)
  ├─→ src/security/        (安全审计)
  ├─→ src/memory/          (记忆系统启动)
  ├─→ src/infra/           (网络、设备、文件系统)
  ├─→ src/hooks/           (钩子系统)
  ├─→ src/cron/            (定时任务)
  └─→ hono                 (HTTP 框架)
```

### 被谁依赖

- `src/cli/gateway-cli/` — CLI 启动网关的入口
- `src/commands/` — 部分命令通过网关 API 操作
- `extensions/*/` — 插件通过网关注册通道和路由

---

## 关键功能描述

1. **OpenAI 兼容 API** (`openai-http.ts`): 提供与 OpenAI API 完全兼容的 HTTP 接口，可直接替换 OpenAI 端点
2. **WebSocket RPC**: 客户端（iOS/macOS/Android/Web）通过 WebSocket 进行实时双向通信
3. **多层认证**: 支持 API key、设备认证、OAuth、角色策略等多种认证方式
4. **配置热重载**: 支持运行时修改配置不重启服务
5. **Control UI**: 内置 Web 管理面板，提供可视化管理界面

---

*← [返回总览](./00-overview.md)*
