# src/acp/ — Agent Client Protocol

> **路径**: `src/acp/`  
> **文件数**: 55 个 TypeScript 文件  
> **核心职责**: ACP 协议实现，支持 stdio 和 WebSocket 传输

---

## 模块概述

ACP（Agent Client Protocol）是 OpenClaw 的**标准化通信协议**，定义了客户端与 Agent 之间的交互方式。类似 LSP（Language Server Protocol）但面向 AI Agent 场景。

---

## 子模块拆解

### 1. 子目录

| 目录 | 文件数 | 作用 |
|------|--------|------|
| `control-plane/` | 12 | 控制面板 ACP 集成 |
| `runtime/` | 13 | ACP 运行时 |

### 2. 核心文件

| 文件 | 作用 |
|------|------|
| `client.ts` | ACP 客户端 |
| `server.ts` | ACP 服务器 |
| `session.ts` | ACP 会话管理 |
| `translator.ts` | ACP 消息翻译器 |
| `event-mapper.ts` | 事件映射 |
| `session-mapper.ts` | 会话映射 |
| `commands.ts` | ACP 命令定义 |
| `conversation-id.ts` | 对话 ID |
| `meta.ts` | ACP 元数据 |
| `policy.ts` | ACP 策略 |
| `secret-file.ts` | 密钥文件 |
| `persistent-bindings.*.ts` | 持久化绑定 |
| `types.ts` | ACP 类型定义 |

---

## 依赖关系

```
src/acp/
  ├─→ src/gateway/         (通过网关暴露 ACP 端点)
  ├─→ src/agents/          (Agent 会话)
  ├─→ src/config/          (ACP 配置)
  └─→ src/infra/           (网络传输)
```

### 被谁依赖

- `src/cli/acp-cli.ts` — ACP CLI 命令
- `src/gateway/` — 网关集成 ACP
- `extensions/acpx/` — ACP 扩展协议插件

---

## 关键功能描述

1. **双传输**: 支持 stdio（进程间通信）和 WebSocket（网络通信）两种传输方式
2. **消息翻译**: `translator.ts` 将 ACP 协议消息与内部格式互转
3. **持久化绑定**: ACP 会话绑定可持久化，断连后自动恢复

---

*← [返回总览](./00-overview.md)*
