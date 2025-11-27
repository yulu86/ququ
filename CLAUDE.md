# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

蛐蛐 (QuQu) 是一个开源免费的智能语音工作流应用，基于 Electron + React 构建，使用 FunASR 进行本地中文语音识别，支持可配置的 AI 模型进行文本优化。

## 核心架构

### 技术栈
- **桌面端**: Electron 36.5.0
- **前端**: React 19, Vite 6, Tailwind CSS, shadcn/ui
- **语音技术**: FunASR (阿里巴巴开源中文语音识别)
- **AI集成**: 支持兼容 OpenAI API 的各种模型
- **数据存储**: better-sqlite3

### 核心模块架构

#### Electron 主进程 (main.js)
- **环境管理**: `src/helpers/environment.js` - 管理不同平台下的 Python 环境和依赖
- **窗口管理**: `src/helpers/windowManager.js` - 管理主窗口和设置窗口
- **数据库**: `src/helpers/database.js` - SQLite 数据库操作
- **剪贴板**: `src/helpers/clipboard.js` - 跨平台剪贴板操作
- **FunASR管理**: `src/helpers/funasrManager.js` - 启动和管理 FunASR Python 服务
- **系统托盘**: `src/helpers/tray.js` - 系统托盘集成
- **快捷键**: `src/helpers/hotkeyManager.js` - 全局快捷键管理 (F2 唤醒)
- **IPC通信**: `src/helpers/ipcHandlers.js` - 主进程与渲染进程通信

#### Python 服务 (funasr_server.py)
- FunASR 模型服务和音频处理
- 通过 stdin/stdout 与 Electron 主进程通信
- 支持语音识别、VAD、语音活动检测

#### React 渲染进程
- **主应用**: `src/App.jsx` - 核心录音界面和状态管理
- **设置页面**: `src/settings.jsx` - AI 模型配置和应用设置
- **历史记录**: `src/history.jsx` - 语音识别历史管理
- **自定义 Hooks**:
  - `src/hooks/useRecording.js` - 录音状态和音频处理
  - `src/hooks/useTextProcessing.js` - AI 文本优化处理
  - `src/hooks/useModelStatus.js` - FunASR 模型状态监控
  - `src/hooks/useHotkey.js` - 快捷键处理
  - `src/hooks/usePermissions.js` - 系统权限管理

## 开发环境设置

### 环境要求
- Node.js 18+
- Python 3.8+ (推荐使用 uv 管理环境)
- pnpm

### 常用开发命令

#### 开发环境启动
```bash
# 安装依赖
pnpm install

# 使用 uv 管理 Python 环境 (推荐)
uv sync
uv run python download_models.py

# 启动开发服务器 (主进程 + 渲染进程)
pnpm run dev
```

#### 构建和打包
```bash
# 仅构建前端
pnpm run build:renderer

# 完整构建 (包含 Python 环境)
pnpm run dist

# 平台特定构建
pnpm run build:mac    # macOS
pnpm run build:win    # Windows
pnpm run build:linux  # Linux
```

#### Python 环境管理
```bash
# 准备嵌入式 Python 环境 (生产构建用)
pnpm run prepare:python:embedded

# 测试 Python 环境
pnpm run test:python

# 使用系统 Python
pip install funasr modelscope torch torchaudio librosa numpy
python download_models.py
```

#### 开发工具
```bash
# 代码检查
pnpm run lint

# 清理环境
pnpm run clean
```

## 开发注意事项

### FunASR 集成
- FunASR 模型文件默认下载到 `~/.cache/modelscope/`
- Python 服务通过 stdin/stdout 进行进程间通信
- 支持热加载和模型状态监控
- 开发时确保 Python 环境正确配置 FunASR 依赖

### AI 模型配置
- 支持任何兼容 OpenAI API 的服务
- 优先适配国内模型 (通义千问、Kimi、智谱AI等)
- 配置信息存储在本地 SQLite 数据库

### 跨平台兼容性
- macOS: 支持系统通知、全局快捷键、自动启动
- Windows: 支持 Windows 10+
- Linux: 基础功能支持

### 调试和日志
- 主进程日志: 使用 `src/helpers/logManager.js`
- FunASR 服务日志: 输出到用户数据目录的 logs 文件夹
- 开发模式下 Electron DevTools 可用

### 性能优化
- FunASR 模型常驻内存，避免重复加载
- React 组件使用 lazy loading 减少初始包大小
- 音频处理使用 Worker 模式避免阻塞 UI

### 安全考虑
- 所有语音数据本地处理，不上传云端
- API Key 等敏感信息本地加密存储
- 生产环境启用 Hardened Runtime 和代码签名