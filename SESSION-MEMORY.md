# WeavesAI 部署会话记忆

> 本文件记录了 2026-05-10 的 AI 辅助部署会话，供下一个 AI 接手时快速了解上下文。

## 一、用户环境

| 项目 | 详情 |
|------|------|
| **操作系统** | Windows 11 (版本 10.0.26200.8246) |
| **机器** | Lenovo，用户名 Lenovo |
| **Node.js** | v24.13.1（项目要求 v20 LTS，但用户选择先用 v24 试） |
| **pnpm** | v10.29.3（满足 ≥10.4.1 要求） |
| **网络** | 需要 Clash 代理（端口 7890）才能访问 GitHub |
| **Git 代理已配置** | `git config --global http.proxy http://127.0.0.1:7890` |
| **项目路径** | `D:\WeavesAI\weavesai-service` |

## 二、已完成的步骤

### ✅ 步骤 1：环境检查
- Node.js v24.13.1 和 pnpm 10.29.3 已安装
- 确认 Clash 代理端口 7890

### ✅ 步骤 2：克隆仓库
- 配置 Git 代理后成功克隆到 `D:\WeavesAI`
- 命令：`git clone https://github.com/WeavesAI/WeavesAI.git`

### ✅ 步骤 3：安装依赖
- `pnpm install` 成功（1m 27.7s）
- 初次安装时 5 个原生模块被跳过（cpu-features, node-datachannel, onnxruntime-node, sharp, ssh2）
- 通过 `pnpm approve-builds` 批准后重新安装，全部编译成功

### ✅ 步骤 4：生成模板数据库
- `pnpm db:push` 成功
- 生成 `D:\WeavesAI\weavesai-service\db\template.db`

### ✅ 步骤 5：配置环境变量
- 复制 `.env.example` 为 `.env`
- 用户已设置 `JWT_SECRET`，其他配置项保持默认（留空）

### ✅ 步骤 6：构建项目
- `pnpm build` 成功
- 前端（Vite）构建 16.81s，输出到 `dist/public/`
- 后端（esbuild）构建 1033ms，输出 `dist/_core/index.js` 和 `dist/entry/service-entry.js`

### ✅ 步骤 7：修复 __dirname ESM 兼容性问题
- Node.js v24 以 ESM 模式运行，`__dirname` 不可用
- 已修复 4 个文件，将 `__dirname` 替换为 `import.meta.dirname`：
  - `server/_core/chat-agent-config.ts` 第 31 行
  - `server/db/connection.ts` 第 35、38 行
  - `server/db/migrations.ts` 第 21 行
  - `server/engine/yolo-detector.ts` 第 353 行

### ✅ 步骤 8：解决启动入口问题
- `npx tsx watch server/_core/index.ts` 无输出 — 原因是 `index.ts` 末尾的 `import.meta.url === file://...` 路径匹配在 Windows 上失败
- 解决方法：直接调用导出函数绕过路径判断

### ✅ 步骤 9：服务成功启动
- 后端服务在 `http://localhost:3000/` 成功运行
- YOLO 检测器、Socket.IO 网关、沙箱管理器均初始化成功

### ✅ 步骤 10：修复 Vite 前端加载
- CMD 中 `set NODE_ENV=development &&` 尾部空格导致 Vite 开发模式未激活
- 改为 `set NODE_ENV=development&&`（无空格）后修复

### ✅ 步骤 11：创建 chat-agent.json
- 报错 `ENOENT: no such file or directory, open '...\config\chat-agent.json'`
- 手动创建 `server\_core\config\chat-agent.json`

### ✅ 步骤 12：配置 DeepSeek R1 模型
- 供应商：deepseek，Base URL：`https://api.deepseek.com`
- 令牌：`sk-ec8ef...`（已设为默认）
- 模型绑定：协议 OpenAI 兼容，API Model ID `deepseek-reasoner`
- thinkModelOverrides 全部填 `deepseek-reasoner`（系统不允许留空）

### ✅ 步骤 13：对话测试成功
- 发送 "1+1等于几"，收到正确回复 "1 + 1 等于 2。"
- 任务已完成，部署 100% 成功

## 三、已知问题

1. **Agent 自动打开谷歌浏览器** — 当问开放性问题时，Agent 会通过宿主机访问权限打开 Chrome 搜索。解决方法：在运行环境页面（`localhost:3000/runtime`）将"AI 访问宿主机"从"访问时授权"改为"禁止"

## 四、可选后续任务

1. **添加 DeepSeek-V3 模型** — API Model ID 填 `deepseek-chat`，速度更快、更便宜
2. **部署 GUIDevice** — 桌面自动化组件（仅 Windows）
3. **部署 WSL2 沙箱** — 隔离执行环境
4. **注册 Windows 服务** — 开机自启

## 五、项目关键信息

### 技术栈
- **前端**：React 19 + Vite 7 + TailwindCSS 4 + Radix UI
- **后端**：Express + tRPC（所有 API 通过 `/api/trpc`）
- **数据库**：SQLite（better-sqlite3） + Drizzle ORM
- **实时通信**：Socket.IO（WebSocket）
- **LLM**：支持 Anthropic 原生协议 + OpenAI 原生协议 + XML 协议

### 启动命令（正确写法）
```bash
# Windows CMD（注意 development 和 && 之间无空格）
set NODE_ENV=development&& node --import tsx -e "import('./server/_core/index.ts').then(m => m.startWebServer({ logger: console, version: '1.14' }))"
```

### 踩过的坑总结
1. **__dirname 不可用**：Node.js v24 + tsx 以 ESM 模式运行，需要用 `import.meta.dirname` 替代
2. **PowerShell Set-Content 破坏编码**：PowerShell 的反引号是转义符，会吞掉 JS 模板字符串的反引号
3. **tsx watch 无输出**：`import.meta.url` 路径匹配在 Windows 上失败，需要直接调用 `startWebServer()`
4. **CMD set 变量尾部空格**：`set VAR=value &&` 会在 value 后加空格，必须写成 `set VAR=value&&`
5. **pnpm dev 不兼容 Windows CMD**：`NODE_ENV=development tsx watch ...` 是 Linux 语法
6. **chat-agent.json 缺失**：需要手动创建 `server\_core\config\chat-agent.json`
7. **模型绑定 thinkModelOverrides 不能留空**：系统校验要求字符串类型，全部填 `deepseek-reasoner`
8. **令牌需设为默认**：供应商令牌必须点"设为默认"，否则模型绑定中"使用供应商默认令牌"找不到可用令牌
