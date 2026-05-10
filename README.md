# WeavesAI 部署笔记

本仓库存放 WeavesAI 平台本地部署的会话记忆和参考文档，用于 AI 任务交接。

## 文件说明

| 文件 | 内容 |
|------|------|
| [SESSION-MEMORY.md](SESSION-MEMORY.md) | 完整会话记忆：用户环境、已完成步骤、遇到的问题、待完成任务、项目关键信息 |
| [DEPLOY-GUIDE.md](DEPLOY-GUIDE.md) | WeavesAI 本地部署完整指南（从零开始） |

## 当前状态

**部署进度：70%** — 构建完成，启动时遇到 `lib/worker.js` 缺失问题，需要继续排查。

### 下一个 AI 需要做的

1. 阅读 `SESSION-MEMORY.md` 了解完整上下文
2. 帮助用户解决启动问题（尝试 `pnpm dev` 或排查 worker.js）
3. 启动成功后，指导用户在 Web UI 配置 LLM API Key
4. 可选：部署 GUIDevice、WSL2 沙箱、Windows 服务

### 用户环境速查
- Windows 11, Node.js v24.13.1, pnpm 10.29.3
- 项目路径：`D:\WeavesAI\weavesai-service`
- 需要 Clash 代理（127.0.0.1:7890）访问 GitHub
