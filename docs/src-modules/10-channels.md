# src/channels/ — 通道系统

> **路径**: `src/channels/`  
> **文件数**: 191 个 TypeScript 文件  
> **核心职责**: 通道注册、消息分发、会话管理、内置通道适配

---

## 模块概述

通道系统是消息平台与 Agent 之间的**桥接层**，负责：
- 内置通道（Telegram、Discord、Slack 等）的注册
- 消息入站/出站处理
- 白名单与权限控制
- 会话绑定与线程管理
- 消息分发与状态追踪

---

## 子模块拆解

### 1. 子目录

| 目录 | 文件数 | 作用 |
|------|--------|------|
| `plugins/` | 123 | 通道插件适配器 |
| `allowlists/` | — | 白名单管理 |
| `transport/` | — | 传输层 |
| `web/` | — | Web 通道 |

### 2. 注册与配置

| 文件 | 作用 |
|------|------|
| `ids.ts` | 内置通道 ID：`telegram`、`whatsapp`、`discord`、`irc`、`googlechat`、`slack`、`signal`、`imessage`、`line` |
| `registry.ts` | 通道注册表 |
| `session.ts` | 通道会话 |
| `channel-config.ts` | 通道配置 |
| `config-presence.ts` | 配置在线状态 |

### 3. 白名单与权限

| 文件 | 作用 |
|------|------|
| `allow-from.ts` | 来源白名单 |
| `allowlist-match.ts` | 白名单匹配 |
| `command-gating.ts` | 命令门控（权限控制） |
| `mention-gating.ts` | @提及门控 |

### 4. 会话与线程

| 文件 | 作用 |
|------|------|
| `thread-binding-id.ts` | 线程绑定 ID |
| `thread-bindings-messages.ts` | 线程绑定消息 |
| `thread-bindings-policy.ts` | 线程绑定策略 |
| `conversation-label.ts` | 对话标签 |
| `session-envelope.ts` | 会话信封 |
| `session-meta.ts` | 会话元数据 |

### 5. 消息处理

| 文件 | 作用 |
|------|------|
| `typing.ts` | 打字状态指示 |
| `draft-stream-controls.ts` | 草稿流控制 |
| `draft-stream-loop.ts` | 草稿流循环 |
| `run-state-machine.ts` | 运行状态机 |
| `targets.ts` | 消息目标 |
| `chat-type.ts` | 聊天类型（DM/群组/频道） |
| `location.ts` | 位置信息 |
| `ack-reactions.ts` | 确认反应 |
| `status-reactions.ts` | 状态反应 |
| `reply-prefix.ts` | 回复前缀 |
| `inbound-debounce-policy.ts` | 入站防抖（避免重复处理） |

### 6. 身份与账户

| 文件 | 作用 |
|------|------|
| `sender-identity.ts` | 发送者身份 |
| `sender-label.ts` | 发送者标签 |
| `model-overrides.ts` | 模型覆盖（按通道/用户切换模型） |
| `native-command-session-targets.ts` | 原生命令会话目标 |
| `read-only-account-inspect.ts` | 只读账户检查 |
| `account-snapshot-fields.ts` | 账户快照字段 |
| `account-summary.ts` | 账户摘要 |
| `logging.ts` | 通道日志 |

---

## 依赖关系

```
src/channels/
  ├─→ src/config/          (通道配置)
  ├─→ src/routing/         (消息路由)
  ├─→ src/plugin-sdk/      (通道插件接口)
  └─→ extensions/*/        (各通道插件实现)
```

### 被谁依赖

- `src/gateway/server-channels.ts` — 网关注册通道
- `src/agents/channel-tools.ts` — Agent 通道交互工具
- `src/auto-reply/` — 自动回复处理
- `src/commands/channels.ts` — 通道管理命令

---

## 关键功能描述

1. **通道抽象**: 将 20+ 种消息平台统一抽象为 `ChannelPlugin` 接口
2. **命令门控**: 控制哪些命令在哪些通道/群组中可用
3. **线程绑定**: 支持按话题/线程创建独立的 Agent 会话
4. **入站防抖**: 防止短时间内重复消息被多次处理

---

*← [返回总览](./00-overview.md)*
