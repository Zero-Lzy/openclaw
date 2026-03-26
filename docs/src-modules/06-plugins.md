# src/plugins/ — 插件系统

> **路径**: `src/plugins/`  
> **文件数**: 241 个 TypeScript 文件  
> **核心职责**: 插件发现、加载、注册、市场、安装管理  
> **关键入口**: `discovery.ts` (~26KB)、`loader.ts` (~41KB)

---

## 模块概述

插件系统是 OpenClaw 的**可扩展性基础**，负责：
- 文件系统扫描发现插件
- Manifest 解析与安全校验
- 插件加载与生命周期管理
- ClawHub 市场集成
- 提供商（Provider）系统管理
- 插件命令与钩子注册

---

## 子模块拆解

### 1. 契约与运行时

| 目录 | 文件数 | 作用 |
|------|--------|------|
| `contracts/` | 14 | 插件契约/接口定义 |
| `runtime/` | 46 | 插件运行时（沙箱、进程管理） |

### 2. 发现与加载

| 文件 | 作用 |
|------|------|
| `discovery.ts` | **插件发现引擎** (~26KB)。文件系统扫描、内置插件、npm 包 |
| `loader.ts` | **插件加载器** (~41KB)。Manifest 解析、入口验证、安全检查 |
| `registry.ts` | 插件注册表 |
| `manifest-registry.ts` | Manifest 注册表 |
| `manifest.ts` | Manifest 类型定义 |
| `schema-validator.ts` | Schema 校验器 |
| `bundled-dir.ts` | 内置插件目录 |
| `bundled-sources.ts` | 内置插件源 |
| `bundled-compat.ts` | 内置插件兼容性 |
| `bundled-plugin-metadata.ts` | 内置插件元数据 |
| `bundled-plugin-metadata.generated.ts` | 自动生成的元数据 |

### 3. 安装与更新

| 文件 | 作用 |
|------|------|
| `install.ts` | 插件安装 |
| `install.runtime.ts` | 安装运行时 |
| `install-security-scan.ts` | 安装安全扫描 |
| `uninstall.ts` | 卸载 |
| `update.ts` | 更新 |
| `installs.ts` | 安装管理 |
| `clawhub.ts` | ClawHub 市场客户端 |
| `marketplace.ts` | 市场操作 |

### 4. 提供商系统

AI 模型提供商的注册与管理。

| 文件 | 作用 |
|------|------|
| `providers.ts` | 提供商管理 |
| `providers.runtime.ts` | 提供商运行时 |
| `provider-catalog.ts` | 提供商目录 |
| `provider-discovery.ts` | 提供商发现 |
| `provider-runtime.ts` | 提供商运行时核心 |
| `provider-auth-*.ts` | 提供商认证系列（choice、key、mode、oauth、storage、token 等） |
| `provider-model-*.ts` | 提供商模型系列（allowlist、defaults、definitions、helpers 等） |
| `provider-wizard.ts` | 提供商设置向导 |
| `provider-validation.ts` | 提供商校验 |

### 5. 命令与钩子

| 文件 | 作用 |
|------|------|
| `commands.ts` | 插件命令 |
| `command-registration.ts` | 命令注册 |
| `bundle-commands.ts` | 内置命令 |
| `hooks.ts` | 插件钩子 |
| `wired-hooks-*.ts` | 已连线钩子系列 |

### 6. 通道与搜索

| 文件 | 作用 |
|------|------|
| `channel-plugin-ids.ts` | 通道插件 ID 映射 |
| `web-search-providers.ts` | 网页搜索提供商 |
| `bundled-web-search.ts` | 内置网页搜索 |
| `bundled-web-search-ids.ts` | 内置搜索 ID |

### 7. 其他

| 文件 | 作用 |
|------|------|
| `cli.ts` | 插件 CLI 集成 |
| `config-state.ts` | 配置状态 |
| `conversation-binding.ts` | 对话绑定 |
| `enable.ts` | 插件启用 |
| `http-registry.ts` | HTTP 注册表 |
| `interactive.ts` | 交互式操作 |
| `logger.ts` | 插件日志 |
| `roots.ts` | 插件根目录 |
| `sdk-alias.ts` | SDK 别名 |
| `services.ts` | 插件服务 |
| `slots.ts` | 插件插槽 |
| `status.ts` | 插件状态 |
| `tools.ts` | 插件工具 |
| `types.ts` | 插件类型 |
| `setup-binary.ts` | 二进制设置 |
| `setup-browser.ts` | 浏览器设置 |
| `signal-cli-install.ts` | Signal CLI 安装 |

---

## 依赖关系

```
src/plugins/
  ├─→ src/config/          (插件配置、白名单)
  ├─→ src/security/        (安装安全扫描)
  ├─→ src/infra/           (文件系统、网络、npm)
  ├─→ src/plugin-sdk/      (SDK 类型)
  └─→ extensions/*/        (各插件 manifest + 代码)
```

### 被谁依赖

- `src/gateway/` — 启动时加载和注册插件
- `src/agents/` — 使用插件提供的工具和提供商
- `src/auto-reply/` — 使用插件命令和搜索
- `src/commands/` — 插件管理 CLI 命令
- `src/cli/` — 插件 CLI 子命令

---

## 关键功能描述

1. **插件发现流程**: `discovery.ts` 扫描 → 内置插件 (`extensions/`) + npm 安装的插件 + 用户本地插件
2. **安全扫描**: 安装第三方插件时自动进行安全扫描
3. **Provider 系统**: 通过插件注册 AI 提供商，支持认证、模型发现、向导配置
4. **ClawHub 市场**: 连接在线市场发现、安装、更新插件

---

*← [返回总览](./00-overview.md)*
