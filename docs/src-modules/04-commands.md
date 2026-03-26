# src/commands/ — CLI 命令实现

> **路径**: `src/commands/`  
> **文件数**: 428 个 TypeScript 文件  
> **核心职责**: 所有 CLI 命令的业务逻辑实现  
> **命令入口**: 由 `src/cli/program.ts` 注册后分派

---

## 模块概述

`commands/` 包含 OpenClaw 所有 CLI 命令的**业务逻辑层**，每个命令文件对应一个可执行的 CLI 子命令。命令由 `src/cli/` 框架注册和路由到此。

---

## 子模块拆解

### 1. Agent 命令

Agent 的创建、管理、运行。

| 文件 | 作用 |
|------|------|
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

### 2. 认证命令

多种认证方式的交互式流程。

| 文件 | 作用 |
|------|------|
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

### 3. 通道命令

消息通道的管理。

| 文件 | 作用 |
|------|------|
| `channels.ts` | 通道管理命令集（add、remove、status） |
| `channel-account-context.ts` | 通道账户上下文 |
| `configure.channels.ts` | 通道配置 |

### 4. 配置命令

交互式配置管理。

| 文件 | 作用 |
|------|------|
| `configure.ts` | `openclaw configure` 命令 |
| `configure.commands.ts` | 配置子命令 |
| `configure.gateway.ts` | 网关配置 |
| `configure.gateway-auth.ts` | 网关认证配置 |
| `configure.daemon.ts` | 守护进程配置 |
| `configure.shared.ts` | 共享配置工具 |
| `configure.wizard.ts` | 配置向导 |
| `config-validation.ts` | 配置校验 |

### 5. Doctor 诊断

系统健康检查与问题诊断。

| 文件 | 作用 |
|------|------|
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

### 6. 模型命令

AI 模型的管理与选择。

| 文件 | 作用 |
|------|------|
| `models.ts` | 模型管理命令（list、set） |
| `model-picker.ts` | 交互式模型选择器 |
| `model-default.ts` | 默认模型 |
| `model-allowlist.ts` | 模型白名单 |

### 7. 其他命令

| 文件 | 作用 |
|------|------|
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
| `onboard-*.ts` | 引导流程系列（新用户设置向导） |

---

## 依赖关系

```
src/commands/
  ├─→ src/config/          (读写配置)
  ├─→ src/agents/          (Agent 运行)
  ├─→ src/gateway/         (网关操作)
  ├─→ src/plugins/         (插件管理)
  ├─→ src/channels/        (通道管理)
  ├─→ src/security/        (安全诊断)
  ├─→ src/memory/          (记忆诊断)
  ├─→ src/infra/           (设备、网络、文件)
  └─→ src/cli/             (CLI 提示、进度条)
```

### 被谁依赖

- `src/cli/program/` — CLI 框架注册并调用命令
- `src/gateway/` — 部分命令通过网关 HTTP API 调用

---

## 关键功能描述

1. **Doctor 诊断**: 全面的系统健康检查，覆盖认证、浏览器、配置、Cron、网关、安装、沙箱、安全、会话等所有子系统
2. **引导流程** (`onboard-*.ts`): 新用户的交互式设置向导，引导完成 AI 提供商选择、通道配置、首次对话
3. **配置向导**: 交互式 TUI 配置管理，支持步进式引导

---

*← [返回总览](./00-overview.md)*
