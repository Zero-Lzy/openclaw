# OpenClaw 项目整体运转流程

> **版本**: 2026.3.26  
> **描述**: 本文档详细描述 OpenClaw 系统从启动到消息处理的完整运转流程，涵盖各核心模块的协作关系与数据流向。

---

## 目录

- [1. 系统架构总览](#1-系统架构总览)
- [2. 启动流程](#2-启动流程)
- [3. 消息处理主流程](#3-消息处理主流程)
- [4. Agent 命令循环](#4-agent-命令循环)
- [5. 插件系统运转](#5-插件系统运转)
- [6. 配置系统运转](#6-配置系统运转)
- [7. 记忆系统运转](#7-记忆系统运转)
- [8. 安全审计流程](#8-安全审计流程)
- [9. CLI 命令执行流程](#9-cli-命令执行流程)
- [10. 客户端连接流程](#10-客户端连接流程)
- [11. 定时任务与钩子](#11-定时任务与钩子)
- [12. 完整数据流总览](#12-完整数据流总览)

---

## 1. 系统架构总览

OpenClaw 是一个**多通道 AI 网关平台**，核心理念是将 AI Agent 通过统一的网关服务器连接到 20+ 种消息平台。整体架构分为 **五个层次**：

```
┌──────────────────────────────────────────────────────────────────────┐
│                       ① 客户端层 (Client Layer)                      │
│  iOS App  |  macOS App  |  Android App  |  Web UI  |  CLI Terminal  │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ WebSocket / HTTP / CLI
┌──────────────────────────▼───────────────────────────────────────────┐
│                      ② 网关层 (Gateway Layer)                        │
│  HTTP API (Hono)  |  WebSocket Server  |  OpenAI Compat  |  Auth    │
│  Control UI       |  Routing Engine    |  Session Manager            │
└──────────────────────────┬───────────────────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────────────────┐
│                     ③ 业务逻辑层 (Business Layer)                     │
│  Agent Engine  |  Auto-Reply  |  Channel Adapters  |  Plugin System  │
│  Command Detection  |  Message Chunking  |  Reply State Machine      │
└──────────────────────────┬───────────────────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────────────────┐
│                    ④ 基础设施层 (Infrastructure Layer)                 │
│  Config System  |  Memory System  |  Security Audit  |  Secrets      │
│  Hooks  |  Cron  |  Browser Automation  |  Media Pipeline            │
└──────────────────────────┬───────────────────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────────────────┐
│                    ⑤ 外部服务层 (External Services)                    │
│  AI Providers (OpenAI, Anthropic, Google, DeepSeek, Ollama, 30+)    │
│  Messaging Platforms (Telegram, Discord, Slack, WhatsApp, 20+)      │
│  Vector DB (SQLite-vec, LanceDB)  |  Embedding Providers             │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. 启动流程

OpenClaw 有两种主要启动模式：**CLI 模式** 和 **Gateway 模式**。

### 2.1 CLI 启动流程

```
openclaw.mjs (CLI 二进制入口)
  │
  ▼
src/entry.ts (CLI 启动入口)
  ├── 1. 进程 Respawn 检测 (entry.respawn.ts)
  │     └── 配置变化后自动重启进程
  ├── 2. 容器目标解析
  │     └── 检测 Docker/沙箱环境
  ├── 3. CLI Profile 环境设置
  │     └── 加载用户 profile 配置
  ├── 4. Windows argv 规范化
  ├── 5. 编译缓存初始化
  │
  ▼
src/runtime.ts (运行时全局初始化)
  ├── 加载全局常量 (globals.ts)
  ├── 初始化全局状态 (global-state.ts)
  └── 设置日志记录器 (logger.ts)
  │
  ▼
src/cli/program.ts (Commander.js 命令程序)
  ├── 解析命令行参数 (argv.ts)
  ├── 注册所有子命令 (program/*.ts)
  ├── 显示启动横幅 (banner.ts)
  │
  ▼
src/commands/*.ts (执行具体命令)
```

### 2.2 Gateway 启动流程

当执行 `openclaw gateway run` 时：

```
src/cli/gateway-cli.ts
  │
  ▼
src/gateway/boot.ts (网关引导)
  │
  ├── 1. 加载配置
  │     └── src/config/io.ts → 读取 JSON5 → Schema 校验 → 环境变量替换 → 密钥解密
  │
  ├── 2. 安全审计
  │     └── src/security/audit.ts → 检查危险配置 → 自动修复/告警
  │
  ├── 3. 初始化记忆系统
  │     └── src/memory/manager.ts → 向量索引 → 嵌入模型加载
  │
  ├── 4. 插件发现与加载
  │     ├── src/plugins/discovery.ts → 扫描文件系统 + 内置插件
  │     ├── src/plugins/loader.ts → Manifest 解析 + 安全检查
  │     └── src/plugins/registry.ts → 注册到全局注册表
  │
  ├── 5. 通道注册
  │     ├── src/channels/registry.ts → 注册内置通道
  │     └── extensions/*/ → 注册插件通道 (Telegram, Discord, Slack...)
  │
  ├── 6. 启动 HTTP 服务器
  │     └── src/gateway/server-http.ts → Hono HTTP 框架
  │         ├── REST API 端点
  │         ├── OpenAI 兼容 API (openai-http.ts)
  │         ├── Control UI 面板 (control-ui.ts)
  │         └── 健康探针 (probe.ts)
  │
  ├── 7. 启动 WebSocket 服务器
  │     └── src/gateway/server-ws-runtime.ts
  │         ├── RPC 方法注册 (server-methods.ts)
  │         └── 实时双向通信
  │
  ├── 8. 启动定时任务
  │     └── src/cron/ → 调度器 + 任务投递
  │
  ├── 9. 启动钩子系统
  │     └── src/hooks/ → 生命周期钩子 + 消息钩子
  │
  └── 10. 服务发现与广播
        ├── src/infra/bonjour.ts → mDNS 局域网发现
        └── src/gateway/server-broadcast.ts → 节点广播
```

### 2.3 启动时序图

```
时间 ──────────────────────────────────────────────────────────────────►

[entry.ts]  ──► [runtime.ts] ──► [config/io.ts] ──► [security/audit.ts]
                                       │
                                       ▼
                              [plugins/discovery.ts]
                                       │
                                       ▼
                              [plugins/loader.ts] ──► [plugins/registry.ts]
                                       │
                                       ▼
                              [channels/registry.ts] ──► [各通道插件启动]
                                       │
                                       ▼
                              [gateway/server-http.ts] + [server-ws-runtime.ts]
                                       │
                                       ▼
                              [memory/manager.ts] + [cron/] + [hooks/]
                                       │
                                       ▼
                                 ✅ 网关就绪，开始接收消息
```

---

## 3. 消息处理主流程

这是 OpenClaw 最核心的流程——一条用户消息从到达到回复的完整旅程。

### 3.1 消息入站流程

```
用户在 Telegram/Discord/Slack/WhatsApp... 发送一条消息
  │
  ▼
① 消息平台 Webhook / 长轮询
  │  (各平台 SDK 接收原始消息事件)
  │
  ▼
② Channel Plugin 适配 (extensions/telegram/、extensions/discord/ ...)
  │  ├── 解析平台原始消息格式
  │  ├── 提取文本、媒体附件、发送者身份
  │  ├── 转换为 OpenClaw 统一消息格式 (session-envelope)
  │  └── 处理平台特有功能 (内联键盘、Blocks、卡片等)
  │
  ▼
③ Gateway 消息接收 (src/gateway/server-chat.ts)
  │  ├── 连接认证 (auth.ts → connection-auth.ts)
  │  ├── 来源白名单检查 (channels/allow-from.ts)
  │  └── 消息清理 (chat-sanitize.ts)
  │
  ▼
④ 消息路由解析 (src/routing/resolve-route.ts)
  │  ├── 从通道 + 账户解析目标 Agent
  │  ├── 构建会话 Key (session-key.ts)
  │  ├── 账户 ID 标准化 (account-id.ts)
  │  └── 路由绑定查找 (bindings.ts)
  │
  ▼
⑤ 自动回复引擎入口 (src/auto-reply/)
  │  ├── 入站防抖 (inbound-debounce.ts)
  │  │     └── 短时间内多条消息合并处理
  │  ├── 群组激活检查 (group-activation.ts)
  │  │     └── 群聊中需要 @提及 或关键词才激活
  │  └── 命令检测 (command-detection.ts)
  │        ├── 检测 /command 风格的命令
  │        ├── 命令认证 (command-auth.ts)
  │        └── 非命令消息 → 进入 Agent 回复流程
  │
  ▼
⑥ 回复生成 (src/auto-reply/reply.ts + status.ts)
     └── 详见下方「Agent 命令循环」
```

### 3.2 回复出站流程

```
Agent 生成回复文本
  │
  ▼
① 回复状态机 (src/auto-reply/status.ts)
  │  状态流转: 等待 → 生成中 → [工具调用 → 工具执行 →] 完成 / 错误
  │
  ▼
② 消息分块 (src/auto-reply/chunk.ts)
  │  ├── Telegram: 最大 4096 字符
  │  ├── Discord: 最大 2000 字符
  │  ├── Slack: 最大 40000 字符
  │  └── 其他平台各自限制
  │
  ▼
③ 流式传输 (src/auto-reply/reply/)
  │  ├── 流式 SSE 传输 (边生成边发送)
  │  ├── 草稿流控制 (channels/draft-stream-controls.ts)
  │  └── 发送策略 (send-policy.ts)
  │
  ▼
④ Channel Plugin 发送 (extensions/*/）
  │  ├── 转换为平台原生格式
  │  ├── 处理媒体附件上传
  │  ├── 添加 ACK 反应 (channels/ack-reactions.ts)
  │  └── 回复前缀处理 (channels/reply-prefix.ts)
  │
  ▼
⑤ 消息平台
  │
  ▼
用户收到 AI 回复 ✅
```

### 3.3 消息处理完整数据流图

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Telegram │    │ Discord  │    │  Slack   │    │ WhatsApp │  ...20+ 平台
└────┬─────┘    └────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │               │
     └───────────────┴───────┬───────┴───────────────┘
                             │ Webhook / 长轮询
                    ┌────────▼────────┐
                    │  Channel Plugin │  (extensions/*/)
                    │  消息适配 & 转换  │
                    └────────┬────────┘
                             │ 统一消息格式
                    ┌────────▼────────┐
                    │    Gateway      │  (src/gateway/)
                    │  认证 + 路由     │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Routing Engine │  (src/routing/)
                    │  会话 Key 解析   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Auto-Reply     │  (src/auto-reply/)
                    │  命令检测 + 分发  │
                    └────┬───────┬────┘
                         │       │
              ┌──────────▼─┐  ┌──▼──────────┐
              │  命令执行    │  │  Agent 引擎  │  (src/agents/)
              │  (内置命令)  │  │  AI 模型调用  │
              └──────────┬─┘  └──┬──────────┘
                         │       │
                         │  ┌────▼────────────────────┐
                         │  │  AI Provider (External)  │
                         │  │  OpenAI / Anthropic /    │
                         │  │  Google / DeepSeek / ... │
                         │  └────┬────────────────────┘
                         │       │ AI 回复
                    ┌────▼───────▼────┐
                    │  Reply Engine   │
                    │  分块 + 流式传输  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Channel Plugin │
                    │  格式转换 + 发送  │
                    └────────┬────────┘
                             │
     ┌───────────────┬───────┴───────┬───────────────┐
     │               │               │               │
┌────▼─────┐  ┌──────▼────┐  ┌──────▼────┐  ┌──────▼────┐
│ Telegram │  │  Discord  │  │   Slack   │  │ WhatsApp  │
└──────────┘  └───────────┘  └───────────┘  └───────────┘
```

---

## 4. Agent 命令循环

Agent 命令循环是整个系统的**计算核心**，位于 `src/agents/agent-command.ts`。

### 4.1 单次 Agent 调用流程

```
接收用户消息 (来自 auto-reply 或 CLI)
  │
  ▼
① 加载 Agent 配置
  │  ├── Agent 身份 (identity.ts): 名称、头像、行为指令
  │  ├── Agent 作用域 (agent-scope.ts): 多 Agent 隔离
  │  └── Agent 默认值 (defaults.ts)
  │
  ▼
② 组装上下文 (context.ts)
  │  ├── 系统提示 (System Prompt)
  │  │     ├── Agent 身份描述
  │  │     ├── 当前时间注入 (current-time.ts)
  │  │     └── 工具说明
  │  ├── 对话历史 (History Messages)
  │  │     ├── 上下文窗口保护 (context-window-guard.ts)
  │  │     └── 历史压缩 (compaction.ts → 自动摘要)
  │  ├── 记忆检索结果 (memory-search.ts)
  │  │     └── 向量相似度 + BM25 混合搜索
  │  ├── Bootstrap 文件 (bootstrap-files.ts)
  │  │     └── 预加载的上下文文件
  │  └── 工具定义列表
  │
  ▼
③ 选择 AI 模型
  │  ├── 模型目录 (model-catalog.ts)
  │  ├── 模型别名解析 (model-alias-lines.ts)
  │  ├── 认证检查 (model-auth.ts)
  │  │     ├── API Key 认证
  │  │     ├── OAuth 认证
  │  │     └── Key 自动轮换 (api-key-rotation.ts)
  │  └── 故障转移策略 (failover-policy.ts)
  │        └── 主模型不可用 → 探针检测 → 切换备选模型
  │
  ▼
④ 调用 AI 模型
  │  ├── 流式 / 非流式调用
  │  ├── 提供商适配
  │  │     ├── OpenAI SDK → GPT 系列
  │  │     ├── Anthropic SDK → Claude 系列
  │  │     ├── Google AI SDK → Gemini 系列
  │  │     ├── AWS Bedrock → 多模型
  │  │     └── ... 30+ 提供商
  │  └── Token 预算管理 (bootstrap-budget.ts)
  │
  ▼
⑤ 处理模型响应
  │  ├─── 纯文本回复 → 直接返回
  │  └─── 工具调用请求 → 进入工具执行循环 ⑥
  │
  ▼
⑥ 工具执行 (可递归)
  │  ├── Bash 工具 (bash-tools.ts)
  │  │     ├── 安全检查 (src/infra/exec-safety.ts)
  │  │     ├── 混淆检测 (exec-obfuscation-detect.ts)
  │  │     ├── 白名单校验 (exec-approvals-allowlist.ts)
  │  │     ├── 执行审批 (exec-approvals.ts)
  │  │     └── 实际执行 (bash-tools.exec.ts)
  │  ├── 记忆搜索工具 (memory-search.ts)
  │  ├── 通道交互工具 (channel-tools.ts)
  │  ├── 补丁应用工具 (apply-patch.ts)
  │  └── 插件提供的工具 (plugins/tools.ts)
  │
  │  工具执行结果 → 追加到上下文 → 回到 ④ 再次调用模型
  │  (递归直到模型返回纯文本回复)
  │
  ▼
⑦ 返回最终回复
  │  ├── 内容块处理 (content-blocks.ts)
  │  ├── 控制台输出清理 (console-sanitize.ts)
  │  └── 缓存追踪 (cache-trace.ts)
  │
  ▼
回复发送给调用方 (auto-reply / CLI)
```

### 4.2 Agent 工具调用递归示意

```
用户: "帮我查看服务器上 /var/log/syslog 的最后 10 行"

Round 1:
  ├── 模型输出: tool_call(bash, "tail -n 10 /var/log/syslog")
  ├── 安全检查 → 通过
  ├── 执行命令 → 获得输出
  └── 工具结果追加到上下文

Round 2:
  ├── 模型收到命令输出
  ├── 模型输出: "以下是 syslog 的最后 10 行：..."
  └── 纯文本回复 → 结束循环

总结: 用户消息 → [模型调用 → 工具执行]×N → 最终回复
```

---

## 5. 插件系统运转

### 5.1 插件生命周期

```
① 发现 (discovery.ts)
  │  ├── 扫描 extensions/ 目录
  │  ├── 扫描 npm 全局安装的插件
  │  ├── 扫描用户自定义插件目录
  │  └── 收集内置插件 (bundled-dir.ts)
  │
  ▼
② 加载 (loader.ts)
  │  ├── 读取 openclaw.plugin.json (Manifest)
  │  ├── Schema 校验 (schema-validator.ts)
  │  ├── 入口文件验证
  │  ├── 安全检查 (install-security-scan.ts)
  │  └── 依赖解析
  │
  ▼
③ 注册 (registry.ts)
  │  ├── 注册到全局插件注册表
  │  ├── 注册通道适配器 (channel-plugin-ids.ts)
  │  ├── 注册提供商 (providers.ts)
  │  ├── 注册命令 (command-registration.ts)
  │  ├── 注册钩子 (hooks.ts)
  │  └── 注册工具 (tools.ts)
  │
  ▼
④ 运行时 (runtime/)
  │  ├── 提供商运行时 (provider-runtime.ts)
  │  ├── 通道消息收发
  │  └── 工具调用响应
  │
  ▼
⑤ 更新/卸载
     ├── update.ts → 检查更新 + 安装新版
     └── uninstall.ts → 清理 + 移除
```

### 5.2 插件类型与职责

```
┌─────────────────────────────────────────────────────┐
│                    Plugin System                     │
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │
│  │ LLM Provider│  │   Channel   │  │   Tool     │  │
│  │   Plugin    │  │   Plugin    │  │  Plugin    │  │
│  │             │  │             │  │            │  │
│  │ OpenAI     │  │ Telegram   │  │ Diff       │  │
│  │ Anthropic  │  │ Discord    │  │ LLM Task   │  │
│  │ Google     │  │ Slack      │  │ Phone Ctrl │  │
│  │ DeepSeek   │  │ WhatsApp   │  │ ...        │  │
│  │ ...36 个   │  │ ...22 个   │  │            │  │
│  └─────────────┘  └─────────────┘  └────────────┘  │
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │
│  │ Web Search  │  │   Memory    │  │   Voice    │  │
│  │   Plugin    │  │   Plugin    │  │  Plugin    │  │
│  │             │  │             │  │            │  │
│  │ Brave      │  │ Core       │  │ ElevenLabs │  │
│  │ DuckDuckGo │  │ LanceDB    │  │ Deepgram   │  │
│  │ Tavily     │  │            │  │ Talk Voice │  │
│  │ ...8 个    │  │            │  │            │  │
│  └─────────────┘  └─────────────┘  └────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 6. 配置系统运转

### 6.1 配置加载流程

```
① 读取配置文件
  │  └── config/io.ts → 读取 ~/.openclaw/config.json5
  │
  ▼
② Include 解析
  │  └── config/includes.ts → 处理 include 引用的外部配置片段
  │
  ▼
③ Schema 校验
  │  └── config/schema.ts → Zod Schema 验证配置格式
  │
  ▼
④ 环境变量替换
  │  └── config/env-substitution.ts → ${ENV_VAR} 替换为实际值
  │
  ▼
⑤ 密钥解密
  │  └── 加密字段解密 (如 API key、token)
  │
  ▼
⑥ 旧版兼容迁移
  │  └── config/legacy.ts → 旧格式自动迁移到新格式
  │
  ▼
⑦ 配置合并
  │  └── config/merge-config.ts → 默认值 + 用户配置 + 运行时覆盖
  │
  ▼
⑧ 生成运行时配置对象
     └── 供各模块使用
```

### 6.2 配置热重载

```
配置文件变更 (文件系统监听)
  │
  ▼
gateway/config-reload.ts
  ├── 重新加载配置 (不重启服务)
  ├── 通知各模块配置已更新
  └── 触发 respawn (如需要) → entry.respawn.ts
```

---

## 7. 记忆系统运转

### 7.1 记忆写入流程

```
文档/对话内容
  │
  ▼
① 文本分块
  │  └── 按语义切分为合适大小的块
  │
  ▼
② 向量嵌入
  │  ├── OpenAI Embeddings (embeddings-openai.ts)
  │  ├── Gemini Embeddings (embeddings-gemini.ts)
  │  ├── Voyage AI (embeddings-voyage.ts)
  │  ├── Ollama 本地嵌入 (embeddings-ollama.ts)
  │  └── Mistral (embeddings-mistral.ts)
  │
  ▼
③ 存储到向量数据库
     ├── SQLite-vec (sqlite-vec.ts) — 默认
     └── LanceDB (extensions/memory-lancedb/) — 可选
```

### 7.2 记忆检索流程

```
Agent 需要记忆上下文
  │
  ▼
① 查询扩展 (query-expansion.ts)
  │  └── 扩展查询词以提高召回率
  │
  ▼
② 混合搜索 (hybrid.ts)
  │  ├── 向量相似度搜索 (语义匹配)
  │  └── BM25 关键词搜索 (精确匹配)
  │
  ▼
③ 结果排序
  │  ├── MMR 多样性重排 (mmr.ts)
  │  └── 时间衰减 (temporal-decay.ts)
  │
  ▼
④ 注入到 Agent 上下文
     └── 作为提示词的一部分发送给 AI 模型
```

---

## 8. 安全审计流程

### 8.1 配置安全审计

```
配置加载完成后
  │
  ▼
security/audit.ts (安全审计核心)
  │
  ├── 危险配置检测 (dangerous-config-flags.ts)
  │     └── 检查是否启用了危险选项
  │
  ├── 工具策略审计 (audit-tool-policy.ts)
  │     └── 检查工具执行权限是否过大
  │
  ├── 通道安全审计 (audit-channel.ts)
  │     └── 检查各通道的安全配置
  │
  ├── DM 策略检查 (dm-policy-shared.ts)
  │     └── 检查私信策略是否安全
  │
  ├── 文件系统审计 (audit-fs.ts)
  │     └── 检查文件权限
  │
  └── 外部内容安全 (external-content.ts)
        └── 检查外部内容加载策略
  │
  ▼
发现问题 → 自动修复 (fix.ts) 或 告警
```

### 8.2 工具执行安全

```
Agent 请求执行工具
  │
  ▼
infra/exec-approvals.ts
  ├── 命令分析 (exec-approvals-analysis.ts)
  ├── 白名单检查 (exec-approvals-allowlist.ts)
  ├── 混淆检测 (exec-obfuscation-detect.ts)
  ├── 安全二进制策略 (exec-safe-bin-policy.ts)
  └── 内联求值检测 (exec-inline-eval.ts)
  │
  ▼
通过 → 执行  |  拒绝 → 提示需要审批
```

---

## 9. CLI 命令执行流程

### 9.1 主要 CLI 命令流

```
openclaw <command> [options]
  │
  ▼
src/cli/program.ts → Commander.js 路由
  │
  ├── openclaw agent          → commands/agent.ts → Agent 命令循环
  ├── openclaw gateway run    → cli/gateway-cli.ts → 启动网关
  ├── openclaw channels add   → commands/channels.ts → 添加通道
  ├── openclaw configure      → commands/configure.ts → 配置向导
  ├── openclaw doctor         → commands/doctor.ts → 系统诊断
  ├── openclaw models list    → commands/models.ts → 模型管理
  ├── openclaw plugins        → cli/plugins-cli.ts → 插件管理
  ├── openclaw onboard        → commands/onboard-*.ts → 引导流程
  ├── openclaw message send   → commands/message.ts → 发送消息
  └── openclaw backup         → commands/backup.ts → 备份
```

### 9.2 `openclaw onboard` 引导流程

```
openclaw onboard --install-daemon
  │
  ▼
① 模型选择与认证 (auth-choice.ts)
  │  └── 选择 AI 提供商 → 配置 API Key / OAuth
  │
  ▼
② 工作区配置 (configure.wizard.ts)
  │  └── 设置配置文件路径、Agent 参数
  │
  ▼
③ 通道配置 (configure.channels.ts)
  │  └── 添加 Telegram / Discord / Slack 等
  │
  ▼
④ 安全审计 (doctor-security.ts)
  │  └── 检查配置安全性
  │
  ▼
⑤ 守护进程安装 (daemon-install-helpers.ts)
  │  ├── macOS → launchd
  │  ├── Linux → systemd
  │  └── Windows → schtasks
  │
  ▼
⑥ 启动网关 → 完成 ✅
```

---

## 10. 客户端连接流程

### 10.1 原生客户端连接

```
iOS / macOS / Android App
  │
  ▼
① 设备发现
  │  ├── Bonjour/mDNS 局域网发现 (infra/bonjour.ts)
  │  ├── Tailscale 远程连接 (gateway/server-tailscale.ts)
  │  └── 手动输入网关地址
  │
  ▼
② 设备配对 (infra/device-pairing.ts)
  │  ├── 生成配对码
  │  ├── 扫描二维码 / 输入配对码
  │  └── 交换设备身份 (device-identity.ts)
  │
  ▼
③ WebSocket 连接
  │  ├── 建立 WebSocket 连接
  │  ├── 设备认证 (gateway/device-auth.ts)
  │  └── 注册为移动节点 (server-mobile-nodes.ts)
  │
  ▼
④ 实时通信
     ├── 发送消息 → WebSocket → Gateway → Agent
     ├── 接收回复 → Gateway → WebSocket → App
     ├── 执行审批 → Gateway → WebSocket → App → 用户确认
     └── 状态同步 (typing 状态、在线状态等)
```

### 10.2 Web UI 连接

```
浏览器访问 http://localhost:18789
  │
  ▼
gateway/control-ui.ts
  ├── 加载 Web Components (ui/src/)
  ├── CSP 安全策略 (control-ui-csp.ts)
  │
  ▼
WebSocket 连接建立
  ├── 对话界面
  ├── Agent 管理
  ├── 通道状态
  ├── 配置管理
  └── 日志查看
```

---

## 11. 定时任务与钩子

### 11.1 定时任务 (Cron)

```
src/cron/
  │
  ├── 调度器 → 按 cron 表达式计算下次执行时间
  ├── 到达执行时间
  │     ├── 创建会话
  │     ├── 调用 Agent 执行任务
  │     └── 投递结果到指定通道
  └── 任务管理 (添加/删除/暂停/恢复)
```

### 11.2 钩子系统 (Hooks)

```
src/hooks/
  │
  ├── 生命周期钩子
  │     ├── 网关启动 (on_gateway_start)
  │     ├── 网关关闭 (on_gateway_stop)
  │     └── 配置重载 (on_config_reload)
  │
  ├── 消息钩子
  │     ├── 消息入站前 (before_message)
  │     ├── 消息入站后 (after_message)
  │     ├── 回复前 (before_reply)
  │     └── 回复后 (after_reply)
  │
  └── 特殊钩子
        └── Gmail 集成 (hooks/gmail)
```

---

## 12. 完整数据流总览

### 12.1 请求-响应全链路

```
                                ┌─────────────────┐
                                │   用户 (User)    │
                                └────────┬────────┘
                                         │ 发送消息
                              ┌──────────▼──────────┐
                              │   消息平台           │
                              │ Telegram/Discord/... │
                              └──────────┬──────────┘
                                         │ Webhook
                              ┌──────────▼──────────┐
                              │   Channel Plugin     │ ← extensions/*/
                              │   消息适配 & 格式转换  │
                              └──────────┬──────────┘
                                         │
                              ┌──────────▼──────────┐
                              │      Gateway         │ ← src/gateway/
                              │   认证 → 路由 → 分发  │
                              └──────────┬──────────┘
                                         │
                   ┌─────────────────────┼─────────────────────┐
                   │                     │                     │
          ┌────────▼────────┐  ┌─────────▼────────┐  ┌────────▼────────┐
          │  命令检测 & 执行  │  │   Auto-Reply     │  │   钩子触发      │
          │ (内置/插件命令)  │  │   消息流水线       │  │  (Hooks)       │
          └────────┬────────┘  └─────────┬────────┘  └────────┬────────┘
                   │                     │                     │
                   │           ┌─────────▼────────┐            │
                   │           │   Agent Engine    │ ← src/agents/
                   │           │   命令循环 (Core)  │
                   │           └───┬────┬────┬────┘
                   │               │    │    │
                   │    ┌──────────┘    │    └──────────┐
                   │    │               │               │
              ┌────▼────▼──┐   ┌───────▼──────┐  ┌────▼────────┐
              │ 工具执行    │   │  AI Provider  │  │ 记忆检索     │
              │ (Bash/MCP) │   │  模型 API 调用 │  │ (向量搜索)  │
              │ exec-*.ts  │   │  30+ 提供商    │  │ memory/     │
              └────────────┘   └───────┬──────┘  └─────────────┘
                                       │ AI 回复
                              ┌────────▼────────┐
                              │   Reply Engine   │ ← src/auto-reply/
                              │   状态机 → 分块    │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  Channel Plugin  │
                              │  格式转换 → 发送   │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │   消息平台        │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │   用户收到回复 ✅  │
                              └─────────────────┘
```

### 12.2 模块间依赖总览

```
                    ┌───────────────┐
                    │   entry.ts    │
                    │  (CLI 入口)   │
                    └───────┬───────┘
                            │
                ┌───────────▼───────────┐
                │     cli/program.ts    │
                │   (Commander 框架)    │
                └───────────┬───────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
    ┌─────▼─────┐    ┌─────▼─────┐    ┌──────▼─────┐
    │ commands/ │    │ gateway/  │    │   agent   │
    │ (命令)    │    │ (网关)    │    │  (Agent)  │
    └─────┬─────┘    └─────┬─────┘    └──────┬─────┘
          │                │                  │
          │          ┌─────┼─────┐            │
          │          │     │     │            │
    ┌─────▼──┐  ┌───▼──┐ ┌▼────┐ ┌▼─────┐  ┌▼───────┐
    │config/ │  │route/│ │chan/ │ │plug/ │  │memory/ │
    │(配置)  │  │(路由)│ │(通道)│ │(插件)│  │(记忆)  │
    └────────┘  └──────┘ └─────┘ └──────┘  └────────┘
          │          │       │       │          │
          └──────────┴───────┴───────┴──────────┘
                             │
                    ┌────────▼────────┐
                    │     infra/      │
                    │   (基础设施)     │
                    │ 执行/设备/网络/  │
                    │ 文件/安全/心跳   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   security/     │
                    │   (安全审计)     │
                    └─────────────────┘
```

### 12.3 运行模式对比

| 维度 | CLI 模式 | Gateway 模式 |
|------|---------|-------------|
| **启动命令** | `openclaw agent` | `openclaw gateway run` |
| **运行方式** | 前台交互，单次会话 | 后台守护进程，持续运行 |
| **消息来源** | 终端输入 / 管道 | 20+ 消息平台 Webhook |
| **Agent 调用** | 直接调用 `agent-command.ts` | 通过 `auto-reply` → `agent-command.ts` |
| **消息输出** | 终端 stdout | 各平台通道回发 |
| **HTTP API** | ❌ 无 | ✅ REST + OpenAI 兼容 |
| **WebSocket** | ❌ 无 | ✅ 实时双向通信 |
| **Control UI** | ❌ 无 | ✅ Web 管理面板 |
| **多通道** | ❌ 单通道 | ✅ 20+ 通道并行 |
| **定时任务** | ❌ 无 | ✅ Cron 调度 |
| **守护进程** | ❌ 手动 | ✅ launchd/systemd/schtasks |

---

*本文档描述了 OpenClaw 项目的整体运转流程。更多模块细节请参阅 [src 模块拆解文档](./00-overview.md)。*

*最后更新: 2026-03-26*
