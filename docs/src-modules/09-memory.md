# src/memory/ — 记忆系统

> **路径**: `src/memory/`  
> **文件数**: 106 个 TypeScript 文件  
> **核心职责**: 向量记忆管理、语义搜索、混合搜索、多嵌入提供商  
> **关键入口**: `manager.ts` (~28KB)、`qmd-manager.ts` (~68KB)

---

## 模块概述

记忆系统为 Agent 提供**长期记忆能力**，支持：
- 文档向量嵌入与索引
- BM25 + 向量混合搜索
- MMR（最大边际相关性）多样性搜索
- 多种嵌入提供商（OpenAI、Gemini、Voyage、Ollama 等）
- 批量嵌入处理
- QMD（Query-Memory-Document）高级记忆管理

---

## 子模块拆解

### 1. 管理器

| 文件 | 作用 |
|------|------|
| `manager.ts` | **记忆索引管理器** (~28KB)。向量嵌入、同步、搜索 |
| `manager-embedding-ops.ts` | 嵌入操作 |
| `manager-search.ts` | 搜索操作 |
| `manager-sync-ops.ts` | 同步操作 |
| `manager-runtime.ts` | 运行时管理 |
| `search-manager.ts` | 搜索管理器 |

### 2. QMD 高级记忆

| 文件 | 作用 |
|------|------|
| `qmd-manager.ts` | **QMD 管理器** (~68KB)。Query-Memory-Document 高级记忆管理 |
| `qmd-process.ts` | QMD 处理 |
| `qmd-query-parser.ts` | QMD 查询解析 |
| `qmd-scope.ts` | QMD 作用域 |

### 3. 嵌入提供商

支持多种向量嵌入服务。

| 文件 | 对应提供商 |
|------|-----------|
| `embeddings.ts` | 嵌入主入口 |
| `embeddings-openai.ts` | OpenAI 嵌入 |
| `embeddings-gemini.ts` | Gemini 嵌入 |
| `embeddings-voyage.ts` | Voyage AI 嵌入 |
| `embeddings-ollama.ts` | Ollama 本地嵌入 |
| `embeddings-mistral.ts` | Mistral 嵌入 |
| `embeddings-remote-*.ts` | 远程嵌入客户端 |
| `embeddings-model-normalize.ts` | 模型归一化 |
| `embedding-vectors.ts` | 向量工具 |
| `embedding-inputs.ts` | 嵌入输入处理 |
| `embedding-chunk-limits.ts` | 分块限制 |
| `embedding-input-limits.ts` | 输入限制 |
| `embedding-model-limits.ts` | 模型限制 |

### 4. 批处理

大规模嵌入的批量处理。

| 文件 | 作用 |
|------|------|
| `batch-runner.ts` | 批处理运行器 |
| `batch-openai.ts` | OpenAI 批处理 |
| `batch-gemini.ts` | Gemini 批处理 |
| `batch-voyage.ts` | Voyage 批处理 |
| `batch-http.ts` | HTTP 批处理 |
| `batch-upload.ts` | 批量上传 |
| `batch-status.ts` | 批处理状态 |
| `batch-output.ts` | 批处理输出 |
| `batch-utils.ts` | 批处理工具 |

### 5. 搜索算法

| 文件 | 作用 |
|------|------|
| `hybrid.ts` | **混合搜索**（BM25 关键词 + 向量语义） |
| `mmr.ts` | **MMR 多样性搜索**（最大边际相关性） |
| `query-expansion.ts` | 查询扩展 |
| `temporal-decay.ts` | 时间衰减（旧记忆权重降低） |
| `prompt-section.ts` | 提示词段（将搜索结果注入上下文） |

### 6. 存储后端

| 文件 | 作用 |
|------|------|
| `sqlite-vec.ts` | **SQLite-vec** 向量存储 |
| `sqlite.ts` | SQLite 存储 |
| `index.ts` | 记忆索引 |
| `internal.ts` | 内部存储 |
| `session-files.ts` | 会话文件 |
| `fs-utils.ts` | 文件系统工具 |

### 7. 其他

| 文件 | 作用 |
|------|------|
| `backend-config.ts` | 后端配置 |
| `memory-schema.ts` | 记忆 schema |
| `multimodal.ts` | 多模态记忆（图片、音频） |
| `node-llama.ts` | 本地 llama 嵌入 |
| `post-json.ts` | JSON POST 工具 |
| `read-file.ts` | 文件读取 |
| `remote-http.ts` | 远程 HTTP |
| `secret-input.ts` | 密钥输入 |
| `status-format.ts` | 状态格式化 |
| `types.ts` | 类型定义 |

---

## 依赖关系

```
src/memory/
  ├─→ src/config/          (记忆配置)
  ├─→ src/infra/           (文件系统)
  ├─→ sqlite-vec           (向量存储)
  ├─→ lancedb              (LanceDB 向量存储 — 通过 extensions/memory-lancedb)
  └─→ 外部嵌入 API         (OpenAI Embeddings, Gemini, Voyage, Ollama)
```

### 被谁依赖

- `src/agents/memory-search.ts` — Agent 的记忆搜索工具
- `src/gateway/server-startup-memory.ts` — 网关启动时初始化记忆
- `src/commands/doctor-memory-search.ts` — 记忆诊断
- `extensions/memory-core/` — 核心记忆插件
- `extensions/memory-lancedb/` — LanceDB 记忆插件

---

## 关键功能描述

1. **记忆流**: 文档 → 嵌入向量化 → SQLite-vec/LanceDB 存储 → 查询时 BM25 + 向量混合搜索 → MMR 去重 → 注入上下文
2. **QMD 系统**: Query-Memory-Document 三层记忆管理，支持结构化记忆查询
3. **时间衰减**: 旧记忆权重自动降低，确保最新信息优先
4. **多模态**: 支持图片、音频等非文本内容的记忆

---

*← [返回总览](./00-overview.md)*
