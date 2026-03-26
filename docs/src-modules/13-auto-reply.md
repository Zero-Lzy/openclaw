# src/auto-reply/ — 自动回复引擎

> **路径**: `src/auto-reply/`  
> **文件数**: 355 个 TypeScript 文件  
> **核心职责**: 消息处理、命令检测、回复生成、流式传输  
> **关键入口**: `status.ts` (~36KB)

---

## 模块概述

自动回复引擎是**消息处理的核心流水线**，负责：
- 接收来自通道的消息
- 检测并执行命令
- 调用 Agent 生成回复
- 流式传输回复到通道
- 管理回复状态机

---

## 子模块拆解

### 1. 回复核心 (`reply/`)

| 目录 | 文件数 | 作用 |
|------|--------|------|
| `reply/` | 283 | 回复生成核心（模板、流式、指令、沙箱媒体等） |

### 2. 消息处理

| 文件 | 作用 |
|------|------|
| `reply.ts` | 回复主入口 |
| `status.ts` | **回复状态机** (~36KB)。AI 响应生成状态管理 |
| `dispatch.ts` | 消息分发 |
| `envelope.ts` | 消息信封 |
| `chunk.ts` | 消息分块（按平台消息大小限制分割） |

### 3. 命令系统

| 文件 | 作用 |
|------|------|
| `command-detection.ts` | 命令检测（`/command` 风格） |
| `command-auth.ts` | 命令认证 |
| `command-control.ts` | 命令控制 |
| `commands-registry.ts` | 命令注册表 |
| `commands-args.ts` | 命令参数解析 |

### 4. 模型与 Token

| 文件 | 作用 |
|------|------|
| `model.ts` | 模型运行时 |
| `model-runtime.ts` | 模型运行时配置 |
| `tokens.ts` | Token 计数 |
| `thinking.ts` | 思考过程（Chain-of-Thought 显示） |

### 5. 其他

| 文件 | 作用 |
|------|------|
| `templating.ts` | 模板系统 |
| `send-policy.ts` | 发送策略 |
| `skill-commands.ts` | 技能命令 |
| `group-activation.ts` | 群组激活（在群组中被 @提及时激活） |
| `heartbeat.ts` | 心跳回复 |
| `inbound-debounce.ts` | 入站防抖 |
| `media-note.ts` | 媒体说明 |
| `fallback-state.ts` | 回退状态 |
| `tool-meta.ts` | 工具元数据 |
| `types.ts` | 类型定义 |

---

## 依赖关系

```
src/auto-reply/
  ├─→ src/agents/          (调用 Agent 引擎生成回复)
  ├─→ src/channels/        (通道消息收发)
  ├─→ src/routing/         (路由解析)
  ├─→ src/config/          (回复配置)
  ├─→ src/plugins/         (插件命令)
  └─→ src/infra/           (基础设施)
```

### 被谁依赖

- `src/gateway/server-chat.ts` — 网关对话处理调用自动回复
- `src/channels/` — 通道消息触发自动回复

---

## 关键功能描述

1. **消息流水线**: 消息入站 → 命令检测 → 认证检查 → Agent 调用 → 回复生成 → 分块 → 流式发送
2. **回复状态机**: `status.ts` 管理整个回复过程的状态流转（等待、生成中、工具调用、完成、错误）
3. **消息分块**: 根据各平台的消息长度限制自动分割长回复
4. **群组激活**: 在群组聊天中通过 @提及或关键词触发 Agent 回复

---

*← [返回总览](./00-overview.md)*
