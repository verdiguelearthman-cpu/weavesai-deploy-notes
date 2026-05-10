# WeavesAI 部署笔记

本仓库存放 WeavesAI 平台本地部署的会话记忆和参考文档，用于 AI 任务交接。

## 文件说明

| 文件 | 内容 |
|------|------|
| [SESSION-MEMORY.md](SESSION-MEMORY.md) | 完整会话记忆：用户环境、已完成步骤、遇到的问题、待完成任务、项目关键信息 |
| [DEPLOY-GUIDE.md](DEPLOY-GUIDE.md) | WeavesAI 本地部署完整指南（从零开始） |

## 当前状态

**部署进度：90%** — 核心服务和前端均已成功启动运行，剩余配置 LLM API Key 即可使用。

### 下一个 AI 需要做的

1. 阅读 `SESSION-MEMORY.md` 了解完整上下文
2. 指导用户在 Web UI 设置页面配置 LLM Provider + API Key
3. 可选：部署 GUIDevice、WSL2 沙箱、Windows 服务

### 用户环境速查
- Windows 11, Node.js v24.13.1, pnpm 10.29.3
- 项目路径：`D:\WeavesAI\weavesai-service`
- 需要 Clash 代理（127.0.0.1:7890）访问 GitHub
- 启动命令（CMD，注意无空格）：`set NODE_ENV=development&& node --import tsx -e "import('./server/_core/index.ts').then(m => m.startWebServer({ logger: console, version: '1.14' }))"`
