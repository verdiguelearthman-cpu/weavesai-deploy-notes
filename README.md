# WeavesAI 部署笔记

本仓库存放 WeavesAI 平台本地部署的会话记忆和参考文档，用于 AI 任务交接。

## 文件说明

| 文件 | 内容 |
|------|------|
| [SESSION-MEMORY.md](SESSION-MEMORY.md) | 完整会话记忆：用户环境、已完成步骤、遇到的问题、待完成任务、项目关键信息 |
| [DEPLOY-GUIDE.md](DEPLOY-GUIDE.md) | WeavesAI 本地部署完整指南（从零开始） |

## 当前状态

**部署进度：100%** — 前后端服务均已成功运行，DeepSeek R1 模型对接完成，平台可正常使用。

### 已完成
- 环境准备、仓库克隆、依赖安装
- 数据库生成、环境变量配置、项目构建
- ESM 兼容性修复（__dirname -> import.meta.dirname）
- 启动入口修复（Windows 路径匹配问题）
- Vite 前端加载修复（CMD set 空格问题）
- chat-agent.json 配置文件创建
- DeepSeek R1 模型配置（供应商 + 令牌 + 模型绑定）
- 对话测试通过

### 已知问题
- Agent 会通过宿主机访问权限自动打开谷歌浏览器搜索，可在运行环境页面将"AI 访问宿主机"设为"禁止"来解决

### 可选后续任务
- 部署 GUIDevice（桌面自动化）
- 部署 WSL2 沙箱（隔离执行环境）
- 注册 Windows 服务（开机自启）
- 添加 DeepSeek-V3（deepseek-chat）模型，速度更快

### 用户环境速查
- Windows 11, Node.js v24.13.1, pnpm 10.29.3
- 项目路径：`D:\WeavesAI\weavesai-service`
- 需要 Clash 代理（127.0.0.1:7890）访问 GitHub
- 启动命令（CMD，注意无空格）：`set NODE_ENV=development&& node --import tsx -e "import('./server/_core/index.ts').then(m => m.startWebServer({ logger: console, version: '1.14' }))"`
