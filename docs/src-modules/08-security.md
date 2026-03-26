# src/security/ — 安全审计

> **路径**: `src/security/`  
> **文件数**: 38 个 TypeScript 文件  
> **核心职责**: 配置安全审计、危险配置检测、工具策略、DM 策略、自动修复  
> **关键入口**: `audit.ts` (~56KB)

---

## 模块概述

安全审计系统负责**检测和修复**配置中的安全风险，包括：
- 危险配置标志检测
- 工具策略审计
- DM（直接消息）策略审计
- 各通道特定的安全检查
- 文件系统安全审计
- 自动修复建议与执行

---

## 文件详解

### 审计核心

| 文件 | 作用 |
|------|------|
| `audit.ts` | **安全审计核心** (~56KB)。完整的配置安全检查 |
| `audit.runtime.ts` | 审计运行时 |
| `audit.deep.runtime.ts` | 深度审计运行时 |
| `audit.nondeep.runtime.ts` | 非深度审计 |

### 审计子类

| 文件 | 作用 |
|------|------|
| `audit-channel.ts` | 通道安全审计 |
| `audit-channel.*.runtime.ts` | 各通道特定审计（Telegram、Discord、Slack 等） |
| `audit-extra.ts` | 额外审计检查 |
| `audit-fs.ts` | 文件系统审计 |
| `audit-tool-policy.ts` | 工具策略审计 |

### 修复与策略

| 文件 | 作用 |
|------|------|
| `fix.ts` | 自动修复引擎 |
| `dangerous-config-flags.ts` | 危险配置标志列表 |
| `dangerous-tools.ts` | 危险工具列表 |
| `dm-policy-shared.ts` | DM 策略共享逻辑 |

### 检测工具

| 文件 | 作用 |
|------|------|
| `channel-metadata.ts` | 通道元数据 |
| `config-regex.ts` | 配置正则匹配 |
| `external-content.ts` | 外部内容安全检查 |
| `mutable-allowlist-detectors.ts` | 可变白名单检测 |
| `safe-regex.ts` | 安全正则（防 ReDoS） |
| `scan-paths.ts` | 扫描路径 |
| `secret-equal.ts` | 密钥比较（时序安全，防侧信道攻击） |
| `skill-scanner.ts` | 技能扫描器 |
| `temp-path-guard.ts` | 临时路径保护 |
| `windows-acl.ts` | Windows ACL 检查 |

---

## 依赖关系

```
src/security/
  ├─→ src/config/          (读取配置进行审计)
  └─→ src/infra/           (文件系统操作)
```

### 被谁依赖

- `src/gateway/` — 启动时运行安全审计
- `src/commands/doctor-security.ts` — `openclaw doctor` 安全诊断
- `src/plugins/install-security-scan.ts` — 插件安装安全扫描

---

## 关键功能描述

1. **深度/非深度审计**: 深度审计检查更多细节（如文件权限、环境变量泄露），非深度审计用于快速检查
2. **时序安全比较**: `secret-equal.ts` 使用恒定时间比较防止侧信道攻击
3. **ReDoS 防护**: `safe-regex.ts` 检测可能导致正则表达式拒绝服务的模式
4. **自动修复**: `fix.ts` 可自动修复检测到的部分安全问题

---

*← [返回总览](./00-overview.md)*
