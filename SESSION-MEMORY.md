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
- **注意**：不能用 PowerShell 的 `Set-Content` 替换，会破坏文件编码（反引号被吃掉）。应使用 VS Code/PyCharm 等编辑器的查找替换功能。
- **替换范围**：仅限 `server/` 目录下的 `*.ts` 文件，不能替换 `node_modules` 中的文件。

### ✅ 步骤 8：解决启动入口问题
- `npx tsx watch server/_core/index.ts` 无输出 — 原因是 `index.ts` 末尾的 `import.meta.url === file://...` 路径匹配在 Windows 上失败，`startWebServer()` 未被调用
- 解决方法：直接调用导出函数绕过路径判断：
  ```
  set NODE_ENV=development&& node --import tsx -e "import('./server/_core/index.ts').then(m => m.startWebServer({ logger: console, version: '1.14' }))"
  ```
- **关键注意**：CMD 中 `set NODE_ENV=development&&` 后面不能有空格，否则 NODE_ENV 值会带空格，导致 Vite 开发模式不激活

### ✅ 步骤 9：服务成功启动
- 后端服务在 `http://localhost:3000/` 成功运行
- YOLO 检测器、Socket.IO 网关、沙箱管理器均初始化成功
- 日志输出正常

### ⚠️ 步骤 10：前端加载问题（进行中）
- 浏览器访问 `localhost:3000` 报错：`ENOENT: no such file or directory, stat 'D:\WeavesAI\weavesai-service\server\_core\public\index.html'`
- 原因：CMD 中 `set NODE_ENV=development &&`（development 后有空格）导致 NODE_ENV 实际值为 `"development "`，Vite 开发模式条件判断失败，回退到静态文件模式
- **解决方法**：确保 `set NODE_ENV=development&&`（无空格）

## 三、待完成的任务

1. **修复 Vite 前端** — 用无空格的命令重新启动，确保 Vite HMR 正常工作
2. **配置 LLM 模型** — 用户尚未决定用哪个 LLM 服务商，需要在 Web UI 设置页面配置 Provider + API Key
3. **可选：部署 GUIDevice** — 桌面自动化组件（仅 Windows）
4. **可选：部署 WSL2 沙箱** — 隔离执行环境
5. **可选：注册 Windows 服务** — 开机自启

## 四、项目关键信息

### 仓库结构
- **源码**：https://github.com/WeavesAI/WeavesAI （CC BY-NC-SA 4.0，禁止商用）
- **主语言**：TypeScript 97.3%
- **总提交**：2,588
- **Release**：45 个（最新 Hot Update v2.0.60）

### 技术栈
- **前端**：React 19 + Vite 7 + TailwindCSS 4 + Radix UI
- **后端**：Express + tRPC（所有 API 通过 `/api/trpc`，无 REST 路由）
- **数据库**：SQLite（better-sqlite3） + Drizzle ORM
- **实时通信**：Socket.IO（WebSocket）
- **日志**：Pino + pino-roll（滚动日志文件）
- **认证**：bcryptjs + JWT (jose) + Cookie sessions
- **LLM**：支持 Anthropic 原生协议 + OpenAI 原生协议 + XML 协议

### 启动命令（正确写法）
```bash
# Windows CMD 中启动开发模式（注意 development 和 && 之间无空格）
set NODE_ENV=development&& node --import tsx -e "import('./server/_core/index.ts').then(m => m.startWebServer({ logger: console, version: '1.14' }))"
```

### 踩过的坑总结
1. **__dirname 不可用**：Node.js v24 + tsx 以 ESM 模式运行，需要用 `import.meta.dirname` 替代
2. **PowerShell Set-Content 破坏编码**：PowerShell 的反引号是转义符，会吞掉 JS 模板字符串的反引号，建议用编辑器替换
3. **tsx watch 无输出**：`import.meta.url` 路径匹配在 Windows 上失败，需要直接调用 `startWebServer()`
4. **CMD set 变量尾部空格**：`set VAR=value &&` 会在 value 后加空格，必须写成 `set VAR=value&&`
5. **pnpm dev 不兼容 Windows CMD**：`NODE_ENV=development tsx watch ...` 是 Linux 语法，Windows 需要分开写

### .env 配置说明
```ini
DATABASE_PATH=./data/weavesai.db     # SQLite 路径
JWT_SECRET=<必填>                     # JWT 签名密钥
REDIS_URL=                            # 可选，多进程时需要
DEBUG_CONSOLE_TOKEN=                  # 可选，启用调试控制台
E2B_API_URL=                          # 可选，沙箱服务器地址
OSS_ACCESS_KEY_ID=                    # 可选，阿里云 OSS
```
