# src/routing/ — 消息路由

> **路径**: `src/routing/`  
> **文件数**: 11 个 TypeScript 文件  
> **核心职责**: 消息路由解析、会话 key 构建  
> **关键入口**: `resolve-route.ts` (~23KB)

---

## 模块概述

路由模块是一个**精巧而关键**的组件，负责将来自不同通道和账户的消息正确路由到对应的 Agent 会话。虽然文件数少，但逻辑复杂。

---

## 文件详解

| 文件 | 作用 |
|------|------|
| `resolve-route.ts` | **路由解析核心** (~23KB)。从通道/账户解析到 Agent 会话 |
| `session-key.ts` | **会话 key 构建/解析** (~8KB)。唯一标识一个会话 |
| `account-id.ts` | 账户 ID 标准化 |
| `account-lookup.ts` | 账户查找 |
| `bindings.ts` | 路由绑定（Agent 到通道的绑定关系） |
| `default-account-warnings.ts` | 默认账户警告 |

---

## 依赖关系

```
src/routing/
  ├─→ src/config/          (路由配置、Agent 绑定)
  └─→ src/channels/        (通道 ID、账户信息)
```

### 被谁依赖

- `src/gateway/` — 网关使用路由解析分派消息
- `src/agents/` — Agent 使用会话 key
- `src/auto-reply/` — 自动回复路由

---

## 关键功能描述

1. **路由解析**: 消息到达 → 解析通道+账户 → 匹配 Agent 绑定 → 生成会话 key → 分派到 Agent Session
2. **会话 key 格式**: `{channel}:{account}:{agent}:{thread?}` 的复合键结构，唯一标识每个对话

---

*← [返回总览](./00-overview.md)*
