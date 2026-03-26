# src/config/ — 配置系统

> **路径**: `src/config/`  
> **文件数**: 264 个 TypeScript 文件  
> **核心职责**: JSON5 配置管理、Schema 校验、环境变量替换、密钥加密  
> **关键入口**: `io.ts` (~67KB)

---

## 模块概述

配置系统是 OpenClaw 的**数据驱动核心**，负责：
- JSON5 格式配置文件的读写
- 基于 Zod 的完整 Schema 校验
- 环境变量替换 (`${ENV_VAR}` 语法)
- 密钥字段的透明加密/解密
- 配置合并与 include 机制
- 旧版配置的自动迁移
- 完整的 TypeScript 类型定义

---

## 子模块拆解

### 1. 核心 I/O

配置文件的读写与处理核心。

| 文件 | 作用 |
|------|------|
| `io.ts` | **配置 I/O 核心** (~67KB)。读写 JSON5 配置、密钥加密、环境变量替换 |
| `config.ts` | 配置实例管理 |
| `defaults.ts` | 默认配置值 |
| `paths.ts` | 配置文件路径 |
| `config-paths.ts` | 配置路径常量 |
| `config-env-vars.ts` | 配置环境变量 |

### 2. Schema 与校验

基于 Zod 的配置 Schema 定义。

| 文件 | 作用 |
|------|------|
| `schema.ts` | Zod schema 定义（完整配置 schema） |
| `schema-base.ts` | 基础 schema |
| `schema.base.generated.ts` | 自动生成的基础 schema |
| `schema.help.ts` | Schema 帮助文本 |
| `schema.hints.ts` | Schema 提示 |
| `schema.shared.ts` | 共享 schema |
| `schema.labels.ts` | Schema 标签 |
| `schema.tags.ts` | Schema 标签系统 |
| `schema.irc.ts` | IRC 专用 schema |

### 3. 类型定义 (`types.*.ts`)

每个功能域都有独立的 TypeScript 类型文件。

| 文件 | 类型域 |
|------|--------|
| `types.ts` | 主类型导出 |
| `types.base.ts` | 基础类型 |
| `types.agents.ts` | Agent 配置 |
| `types.channels.ts` | 通道配置 |
| `types.gateway.ts` | 网关配置 |
| `types.models.ts` | 模型配置 |
| `types.plugins.ts` | 插件配置 |
| `types.tools.ts` | 工具配置 |
| `types.hooks.ts` | 钩子配置 |
| `types.memory.ts` | 记忆配置 |
| `types.sandbox.ts` | 沙箱配置 |
| `types.secrets.ts` | 密钥配置 |
| `types.mcp.ts` | MCP 配置 |
| `types.cron.ts` | Cron 配置 |
| `types.auth.ts` | 认证配置 |
| `types.browser.ts` | 浏览器配置 |
| `types.cli.ts` | CLI 配置 |
| `types.acp.ts` | ACP 配置 |
| `types.openclaw.ts` | OpenClaw 特有类型 |
| `types.discord.ts` / `types.telegram.ts` / `types.slack.ts` 等 | 各通道特定类型 |

### 4. 环境变量与合并

| 文件 | 作用 |
|------|------|
| `env-substitution.ts` | 环境变量替换引擎 |
| `env-preserve.ts` | 环境变量保留 |
| `env-vars.ts` | 环境变量定义 |
| `merge-config.ts` | 配置合并 |
| `merge-patch.ts` | JSON Merge Patch |
| `includes.ts` | 配置文件 include 机制 |
| `includes-scan.ts` | include 扫描 |

### 5. 迁移与兼容

旧版配置的自动迁移系统。

| 文件 | 作用 |
|------|------|
| `legacy.ts` | 旧版配置兼容 |
| `legacy-migrate.ts` | 旧版迁移入口 |
| `legacy.migrations.ts` | 迁移规则集 |
| `legacy.migrations.part-1/2/3.ts` | 分段迁移规则 |
| `legacy.rules.ts` | 迁移规则引擎 |
| `legacy.shared.ts` | 共享迁移工具 |
| `legacy-web-search.ts` | 旧版搜索迁移 |

### 6. 其他辅助

| 文件 | 作用 |
|------|------|
| `agent-dirs.ts` | Agent 目录管理 |
| `agent-limits.ts` | Agent 限制 |
| `allowed-values.ts` | 允许值校验 |
| `bindings.ts` | 绑定配置 |
| `channel-capabilities.ts` | 通道能力定义 |
| `channel-configured.ts` | 通道配置检查 |
| `commands.ts` | 命令配置 |
| `sessions.ts` | 会话配置 |
| `logging.ts` | 日志配置 |
| `talk-defaults.ts` / `talk.ts` | Talk 模式配置 |
| `telegram-custom-commands.ts` | Telegram 自定义命令配置 |
| `discord-preview-streaming.ts` | Discord 预览流式配置 |
| `group-policy.ts` | 群组策略 |
| `runtime-group-policy.ts` | 运行时群组策略 |
| `runtime-overrides.ts` | 运行时配置覆盖 |
| `dangerous-name-matching.ts` | 危险名称匹配 |
| `gateway-control-ui-origins.ts` | 控制面板 CORS 源 |
| `mcp-config.ts` | MCP 配置 |
| `port-defaults.ts` | 默认端口 |
| `normalize-paths.ts` | 路径规范化 |
| `normalize-exec-safe-bin.ts` | 安全二进制规范化 |
| `plugins-allowlist.ts` | 插件白名单 |
| `plugin-auto-enable.ts` | 插件自动启用 |
| `redact-snapshot.ts` | 快照脱敏 |
| `doc-baseline.ts` | 文档基线 |
| `state-dir-dotenv.ts` | 状态目录 dotenv |
| `backup-rotation.ts` | 备份轮换 |
| `byte-size.ts` | 字节大小工具 |
| `cache-utils.ts` | 缓存工具 |
| `issue-format.ts` | Issue 格式化 |
| `markdown-tables.ts` | Markdown 表格生成 |
| `model-input.ts` | 模型输入 |
| `prototype-keys.ts` | 原型键保护 |

---

## 依赖关系

```
src/config/
  ├─→ zod                  (Schema 校验)
  ├─→ json5                (JSON5 解析)
  ├─→ src/secrets/         (密钥加密/解密)
  └─→ src/infra/           (文件系统、环境变量)
```

### 被谁依赖

- **几乎所有模块都依赖 config/**
- `src/agents/` — 读取 Agent、模型配置
- `src/gateway/` — 网关启动配置
- `src/commands/` — CLI 命令配置操作
- `src/plugins/` — 插件配置
- `src/security/` — 安全审计检查配置
- `src/channels/` — 通道配置
- `src/memory/` — 记忆系统配置

---

## 关键功能描述

1. **配置流**: JSON5 文件 → `io.ts` 加载 → Zod Schema 校验 → 环境变量替换 → 密钥解密 → 运行时配置对象
2. **Include 机制**: 配置文件可通过 `include` 字段引用其他配置文件，支持嵌套
3. **自动迁移**: 旧版配置自动检测并迁移到最新格式，保持向后兼容
4. **脱敏快照**: `redact-snapshot.ts` 可生成脱敏的配置快照用于调试/支持

---

*← [返回总览](./00-overview.md)*
