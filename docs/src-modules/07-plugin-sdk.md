# src/plugin-sdk/ — 插件 SDK

> **路径**: `src/plugin-sdk/`  
> **文件数**: 217 个 TypeScript 文件  
> **核心职责**: 为第三方插件开发者提供完整的开发 SDK  
> **导出子路径**: 80+

---

## 模块概述

Plugin SDK 是 OpenClaw 提供给**插件开发者的公开 API 面**，通过 `openclaw/plugin-sdk/*` 子路径导出。它封装了：
- 通道插件（Channel Plugin）开发接口
- AI 提供商插件（Provider Plugin）开发接口
- 记忆插件（Memory Plugin）开发接口
- 工具插件（Tool Plugin）开发接口
- 类型定义、辅助工具、适配器创建

---

## 核心导出

| 文件 | 作用 |
|------|------|
| `index.ts` | SDK 根入口。导出 `ChannelPlugin`、`OpenClawPluginApi`、`ProviderAuthContext` |
| `core.ts` | 核心 SDK 辅助（通道插件定义、适配器创建） |
| 其他 215+ 文件 | 各子路径实现 |

## 导出子路径示例

```
openclaw/plugin-sdk               # 根入口
openclaw/plugin-sdk/channel       # 通道插件接口
openclaw/plugin-sdk/provider      # 提供商插件接口
openclaw/plugin-sdk/memory        # 记忆插件接口
openclaw/plugin-sdk/tools         # 工具接口
openclaw/plugin-sdk/config        # 配置接口
openclaw/plugin-sdk/types         # 类型定义
openclaw/plugin-sdk/<extension>   # 各扩展的公开 API
...                               # 80+ 更多子路径
```

---

## 依赖关系

```
src/plugin-sdk/
  ├─→ src/config/types.*   (配置类型重导出)
  ├─→ src/channels/        (通道接口类型)
  ├─→ src/memory/types.ts  (记忆接口类型)
  └─→ src/agents/          (工具接口类型)
```

### 被谁依赖

- `extensions/*/` — **所有插件**通过 `openclaw/plugin-sdk/*` 导入 SDK
- `src/plugins/` — 插件系统使用 SDK 类型进行校验

---

## 导入约定

```typescript
// ✅ 正确：插件通过公开子路径导入
import { ChannelPlugin } from 'openclaw/plugin-sdk';
import { createProvider } from 'openclaw/plugin-sdk/provider';

// ❌ 错误：不要直接导入内部路径
import { something } from '../../src/plugin-sdk/internal';
```

---

## 关键功能描述

1. **公开 API 面**: SDK 是插件与核心之间唯一的公开契约，保证向后兼容
2. **子路径导出**: 通过 `package.json` 的 `exports` 字段定义 80+ 个子路径
3. **类型安全**: 完整的 TypeScript 类型定义，插件开发时获得完整的类型提示
4. **API 漂移检测**: 通过 `pnpm plugin-sdk:api:check` 检测 SDK API 是否发生意外变更

---

*← [返回总览](./00-overview.md)*
