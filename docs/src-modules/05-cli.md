# src/cli/ — CLI 框架

> **路径**: `src/cli/`  
> **文件数**: 329 个 TypeScript 文件  
> **核心职责**: CLI 应用框架、参数解析、命令注册、交互式提示  
> **关键入口**: `program.ts`

---

## 模块概述

`cli/` 是 OpenClaw 的**CLI 应用框架层**，基于 Commander.js 构建，负责：
- 命令树的注册与路由
- 参数解析与校验
- 交互式提示与进度条
- CLI Profile 管理
- 各功能域的 CLI 子命令模块

---

## 子模块拆解

### 1. 核心框架

| 文件 | 作用 |
|------|------|
| `program.ts` | CLI 主程序入口 |
| `run-main.ts` | CLI 主运行入口 |
| `argv.ts` | 参数解析 |
| `banner.ts` | 启动横幅（ASCII art） |
| `prompt.ts` | 交互式提示 |
| `progress.ts` | 进度条 |
| `profile.ts` | CLI profile 管理 |

### 2. 子命令程序目录 (`program/`)

Commander.js 命令程序定义，**57 个文件**。

| 文件 | 作用 |
|------|------|
| `program/*.ts` | 各子命令的 Commander 程序定义（参数、选项、帮助文本） |

### 3. 功能域 CLI 模块

每个功能域有独立的 CLI 模块文件。

| 文件 | 功能域 |
|------|--------|
| `config-cli.ts` | 配置管理 |
| `channels-cli.ts` | 通道管理 |
| `models-cli.ts` | 模型管理 |
| `plugins-cli.ts` | 插件管理 |
| `memory-cli.ts` | 记忆管理 |
| `secrets-cli.ts` | 密钥管理 |
| `security-cli.ts` | 安全管理 |
| `skills-cli.ts` | 技能管理 |
| `mcp-cli.ts` | MCP 管理 |
| `hooks-cli.ts` | 钩子管理 |
| `cron-cli.ts` | 定时任务管理 |
| `acp-cli.ts` | ACP 管理 |
| `pairing-cli.ts` | 设备配对 |
| `devices-cli.ts` | 设备管理 |
| `logs-cli.ts` | 日志管理 |
| `webhooks-cli.ts` | Webhook 管理 |
| `sandbox-cli.ts` | 沙箱管理 |
| `system-cli.ts` | 系统管理 |
| `tui-cli.ts` | TUI 管理 |
| `completion-cli.ts` | Shell 自动补全 |

### 4. 专用 CLI 子模块

| 目录 | 文件数 | 作用 |
|------|--------|------|
| `daemon-cli/` | 28 | 守护进程 CLI 子命令 |
| `gateway-cli/` | 10 | 网关 CLI 子命令 |
| `nodes-cli/` | 17 | 节点 CLI 子命令 |
| `update-cli/` | 10 | 更新 CLI 子命令 |

### 5. 浏览器管理 CLI

| 文件 | 作用 |
|------|------|
| `browser-cli.ts` | 浏览器管理 CLI 主入口 |
| `browser-cli-*.ts` | 浏览器管理子命令系列 |

---

## 依赖关系

```
src/cli/
  ├─→ commander            (CLI 框架)
  ├─→ src/commands/        (业务逻辑实现)
  ├─→ src/config/          (配置读取)
  └─→ src/infra/           (CLI 根选项、环境)
```

### 被谁依赖

- `src/entry.ts` — 启动 CLI 主程序
- `src/index.ts` — 作为 main module 时启动

---

## 关键功能描述

1. **命令分层**: `cli/` 负责命令注册和参数解析，`commands/` 负责业务逻辑，实现了清晰的关注点分离
2. **Shell 自动补全**: `completion-cli.ts` 提供 Bash/Zsh/Fish 的命令自动补全
3. **Profile**: 支持多套 CLI 配置 profile 切换（如 dev/staging/prod）

---

*← [返回总览](./00-overview.md)*
