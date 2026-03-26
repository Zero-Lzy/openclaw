# src/ 其他子模块

> **路径**: `src/` 下的其余子模块  
> **核心职责**: 浏览器自动化、定时任务、媒体处理、TUI、钩子、守护进程等

---

## 模块概述

以下模块是 `src/` 中**独立职责的功能子系统**，每个虽规模较小，但在整体架构中不可或缺。

---

## 1. browser/ — 浏览器自动化 (161 文件)

基于 **Playwright** 的浏览器自动化系统。

| 关键能力 | 说明 |
|----------|------|
| Chrome 管理 | 启动、管理 Chrome/Chromium 实例 |
| CDP 代理 | Chrome DevTools Protocol 代理 |
| 页面控制 | 导航、截图、Cookie |
| Profile 管理 | 浏览器配置文件管理 |

**依赖**: Playwright  
**被依赖**: `src/gateway/server-browser.ts`、`src/commands/doctor-browser.ts`

---

## 2. cron/ — 定时任务系统 (117 文件)

定时任务的调度与执行。

| 关键能力 | 说明 |
|----------|------|
| 调度器 | Cron 表达式调度 |
| 任务执行 | Job 运行与状态管理 |
| 投递系统 | 定时消息投递 |
| 会话管理 | 定时任务的 Agent 会话 |

**依赖**: `src/config/`、`src/agents/`  
**被依赖**: `src/gateway/server-cron.ts`、`src/commands/doctor-cron.ts`

---

## 3. hooks/ — 钩子系统 (56 文件)

消息与生命周期钩子。

| 关键能力 | 说明 |
|----------|------|
| 消息钩子 | 消息到达/发送前后的拦截 |
| 生命周期钩子 | 启动、关闭、配置变更等事件 |
| Gmail 集成 | Gmail 邮件钩子 |

**依赖**: `src/config/`  
**被依赖**: `src/gateway/hooks.ts`、`src/auto-reply/`

---

## 4. media/ — 媒体处理管道 (48 文件)

音频、图片格式转换与处理。

| 关键能力 | 说明 |
|----------|------|
| 音频转码 | FFmpeg 驱动的音频格式转换 |
| 图片处理 | Sharp 驱动的图片缩放/转换 |
| 格式检测 | 自动检测媒体文件格式 |

**依赖**: FFmpeg、Sharp  
**被依赖**: `src/auto-reply/reply/`、`extensions/voice-call/`

---

## 5. media-understanding/ — 媒体理解 (56 文件)

多模态内容理解。

| 关键能力 | 说明 |
|----------|------|
| 图片理解 | 使用 AI 模型描述图片内容 |
| 音频理解 | 语音转文本 |
| 视频理解 | 视频帧分析 |
| 多模态组合 | 组合多种媒体输入 |

**依赖**: AI 模型 API  
**被依赖**: `src/agents/`

---

## 6. daemon/ — 守护进程管理 (54 文件)

跨平台后台服务管理。

| 关键能力 | 说明 |
|----------|------|
| macOS | launchd 集成 |
| Linux | systemd 集成 |
| Windows | schtasks 集成 |

**依赖**: 操作系统原生 API  
**被依赖**: `src/commands/configure.daemon.ts`、`src/cli/daemon-cli/`

---

## 7. secrets/ — 密钥管理 (53 文件)

密钥的加密存储与引用。

| 关键能力 | 说明 |
|----------|------|
| 加密存储 | AES 加密密钥值 |
| 密钥引用 | 配置中使用 `$secret:name` 引用密钥 |
| 密钥轮换 | 支持密钥更新 |

**依赖**: Node.js crypto  
**被依赖**: `src/config/io.ts`（配置解密）

---

## 8. tui/ — 终端 UI (48 文件)

基于 **Ink** 的终端交互式界面。

| 关键能力 | 说明 |
|----------|------|
| React 组件 | 使用 Ink（React for CLI）构建 TUI |
| 交互式界面 | 对话界面、设置界面 |

**依赖**: Ink (React)  
**被依赖**: `src/cli/tui-cli.ts`

---

## 9. tts/ — 语音合成 (12 文件)

文本转语音服务。

| 关键能力 | 说明 |
|----------|------|
| ElevenLabs | ElevenLabs TTS |
| OpenAI | OpenAI TTS |
| Edge TTS | Microsoft Edge TTS |

**依赖**: 外部 TTS API  
**被依赖**: `extensions/voice-call/`、`extensions/talk-voice/`

---

## 10. sessions/ — 会话管理 (15 文件)

Agent 会话的持久化与恢复。

| 关键能力 | 说明 |
|----------|------|
| 持久化 | 会话数据持久化到磁盘 |
| 恢复 | 重启后恢复会话状态 |
| 锁 | 会话并发控制锁 |

**依赖**: `src/infra/file-lock.ts`  
**被依赖**: `src/agents/`、`src/gateway/`

---

## 11. shared/ — 共享工具 (80 文件)

跨模块共享的工具函数和类型。

---

## 12. 其他小模块

| 模块 | 文件数 | 作用 |
|------|--------|------|
| `logging/` | 30 | 日志框架 |
| `process/` | 28 | 进程管理 |
| `utils/` | 30 | 工具函数 |
| `terminal/` | 19 | 终端工具 |
| `node-host/` | 16 | Node 宿主 |
| `wizard/` | 16 | 设置向导 |
| `markdown/` | 14 | Markdown 处理 |
| `types/` | 11 | 全局类型定义 |
| `test-utils/` | 35 | 测试工具 |
| `image-generation/` | 9 | 图片生成 |
| `pairing/` | 9 | 设备配对 |
| `i18n/` | — | 国际化 |
| `interactive/` | — | 交互式消息 |
| `bindings/` | — | 绑定 |
| `bootstrap/` | — | 引导 |
| `canvas-host/` | — | Canvas 宿主 |
| `compat/` | — | 兼容层 |
| `context-engine/` | — | 上下文引擎注册表（可插拔替换） |
| `docs/` | — | 内部文档 |
| `extensions/` | — | 插件扩展桥接 |
| `line/` | — | Line 通道 |
| `link-understanding/` | — | 链接理解 |
| `scripts/` | — | 内部脚本 |
| `web-search/` | — | 网页搜索 |
| `test-helpers/` | — | 测试辅助 |

---

*← [返回总览](./00-overview.md)*
