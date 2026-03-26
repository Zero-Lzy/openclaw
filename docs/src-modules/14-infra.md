# src/infra/ — 基础设施

> **路径**: `src/infra/`  
> **文件数**: 538 个 TypeScript 文件  
> **核心职责**: 底层基础设施 — 执行审批、设备管理、网络发现、文件系统、心跳、安全

---

## 模块概述

`infra/` 是 OpenClaw 的**底层支撑层**，提供所有上层模块所需的基础能力：
- 命令执行审批系统（安全沙箱）
- 设备身份与配对
- 网络发现（Bonjour/mDNS）
- 文件系统操作（安全读写、锁、归档）
- 心跳系统
- 宿主环境安全

---

## 子模块拆解

### 1. 执行审批系统

控制 Agent 执行命令的安全审批流程。

| 文件 | 作用 |
|------|------|
| `exec-approvals.ts` | 执行审批核心 |
| `exec-approvals-allowlist.ts` | 审批白名单 |
| `exec-approvals-analysis.ts` | 命令分析 |
| `exec-approval-*.ts` | 审批系列（forwarder、reply、surface、session-target 等） |
| `exec-safety.ts` | 执行安全检查 |
| `exec-safe-bin-policy.ts` | 安全二进制策略 |
| `exec-obfuscation-detect.ts` | 命令混淆检测 |
| `exec-command-resolution.ts` | 命令解析 |
| `exec-wrapper-*.ts` | 执行包装器 |
| `exec-inline-eval.ts` | 内联求值检测 |
| `exec-host.ts` | 执行宿主 |
| `executable-path.ts` | 可执行路径 |

### 2. 设备与认证

| 文件 | 作用 |
|------|------|
| `device-identity.ts` | 设备身份 |
| `device-auth-store.ts` | 设备认证存储 |
| `device-bootstrap.ts` | 设备引导 |
| `device-pairing.ts` | 设备配对 |

### 3. 网络与发现

| 文件 | 作用 |
|------|------|
| `fetch.ts` | HTTP 抓取 |
| `bonjour.ts` | Bonjour/mDNS 服务发现 |
| `bonjour-discovery.ts` | 发现实现 |
| `bonjour-ciao.ts` | Ciao 实现 |
| `gateway-discovery-targets.ts` | 网关发现目标 |
| `gateway-lock.ts` | 网关锁 |
| `gateway-processes.ts` | 网关进程 |
| `gateway-process-argv.ts` | 网关进程参数 |

### 4. 文件与存储

| 文件 | 作用 |
|------|------|
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

### 5. 心跳系统

监控 Agent 和通道的活跃状态。

| 文件 | 作用 |
|------|------|
| `heartbeat-runner.ts` | 心跳运行器 |
| `heartbeat-events.ts` | 心跳事件 |
| `heartbeat-events-filter.ts` | 事件过滤 |
| `heartbeat-active-hours.ts` | 活跃时段 |
| `heartbeat-reason.ts` | 心跳原因 |
| `heartbeat-summary.ts` | 心跳摘要 |
| `heartbeat-visibility.ts` | 心跳可见性 |
| `heartbeat-wake.ts` | 心跳唤醒 |

### 6. 安全

| 文件 | 作用 |
|------|------|
| `host-env-security.ts` | 宿主环境安全 |
| `host-env-security-policy.json` | 安全策略配置 |

### 7. 其他基础设施

| 文件 | 作用 |
|------|------|
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

---

## 依赖关系

```
src/infra/
  ├─→ src/config/          (配置读取)
  └─→ Node.js 标准库       (fs, net, child_process, crypto)
```

### 被谁依赖

- **几乎所有模块都依赖 infra/**
- `src/agents/` — 命令执行、设备身份
- `src/gateway/` — 网络、服务发现
- `src/commands/` — 安装、备份
- `src/plugins/` — 文件系统、npm 操作
- `src/security/` — 文件系统审计
- `src/memory/` — 文件存储

---

## 关键功能描述

1. **执行审批**: Agent 执行 Bash 命令前必须经过审批系统，检查白名单、混淆检测、安全二进制策略
2. **Bonjour 发现**: 本地网络自动发现 OpenClaw 网关实例
3. **文件安全**: 硬链接保护、边界路径检查、文件锁，防止路径穿越攻击
4. **心跳系统**: 监控活跃状态，支持活跃时段配置和自动唤醒

---

*← [返回总览](./00-overview.md)*
