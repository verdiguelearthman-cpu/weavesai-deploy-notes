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
- 构建时有 6 个 esbuild WARNING（未实现的 DB 函数引用），不影响运行

### ❌ 步骤 7：启动服务（未完成 — 遇到问题）

**问题 1**：`node dist/index.js` — 文件不存在
- 原因：构建输出路径是 `dist/_core/index.js`，不是 `dist/index.js`

**问题 2**：`node dist/_core/index.js` — worker.js 缺失
```
Error: Cannot find module 'D:\WeavesAI\weavesai-service\dist\_core\lib\worker.js'
```
- 原因分析：某个依赖（可能是 pino 日志相关）在运行时动态加载 worker 线程文件，但 esbuild 打包时没有包含这个文件
- 仓库源码中没有 `server/lib/worker.ts`，CI 工作流中也没有额外的 worker 构建步骤
- **这个文件可能是生产环境通过 CI 流水线额外生成的，开源版本缺失**

**待尝试的解决方案**：
1. 用开发模式启动：`pnpm dev`（绕过 esbuild 打包，直接跑 TypeScript 源码）
2. 如果开发模式也失败，检查 `pino-roll` 或 `pino` 的 worker 线程依赖
3. 可能需要降级到 Node.js v20 LTS（v24 对 worker_threads 行为可能有变化）

## 三、待完成的任务

1. **启动服务** — 尝试 `pnpm dev` 开发模式启动
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

### 子模块
| 目录 | 说明 |
|------|------|
| `weavesai-service` | 核心服务（Express + tRPC + React 前端 + SQLite） |
| `guidevice` | 虚拟 KVM 设备（桌面自动化） |
| `svc-wrapper` | Windows 服务包装器（C 语言） |
| `setup` | Windows 安装器/更新器 |
| `sandbox` | WSL2 Ubuntu 沙箱（Dockerfile） |
| `core-docs` | 架构文档 |

### 技术栈
- **前端**：React 19 + Vite 7 + TailwindCSS 4 + Radix UI
- **后端**：Express + tRPC（所有 API 通过 `/api/trpc`，无 REST 路由）
- **数据库**：SQLite（better-sqlite3） + Drizzle ORM
- **实时通信**：Socket.IO（WebSocket）
- **日志**：Pino + pino-roll（滚动日志文件）
- **认证**：bcryptjs + JWT (jose) + Cookie sessions
- **LLM**：支持 Anthropic 原生协议 + OpenAI 原生协议 + XML 协议

### 重要发现
- **开源版本没有用户系统**：无注册/登录/邀请码功能，本地部署直接可用
- **官方托管版** (ai.weaves.cn) 有额外的闭源用户管理系统
- **安装包分发**：通过阿里云 OSS（`weavesai.oss-cn-beijing.aliyuncs.com/releases/`）
- **版本清单**：`https://weavesai.oss-cn-beijing.aliyuncs.com/releases/latest.json`
- **预置 LLM 模型**：Claude Sonnet/Opus 4.6、GPT 5.4、GPT 5.4 Nano

### 关键命令速查
```bash
# 开发模式（热重载）
pnpm dev

# 生产构建
pnpm build

# 生成数据库
pnpm db:push

# 类型检查
pnpm check

# 生产启动（通过 PM2，端口 3001）
pm2 start ecosystem.config.cjs

# 远程调试
DEBUG_HOST=http://<ip>:3000 npx tsx scripts/debug-cli.ts system
```

### .env 配置说明
```ini
DATABASE_PATH=./data/weavesai.db     # SQLite 路径
JWT_SECRET=<必填>                     # JWT 签名密钥
REDIS_URL=                            # 可选，多进程时需要
DEBUG_CONSOLE_TOKEN=                  # 可选，启用调试控制台
E2B_API_URL=                          # 可选，沙箱服务器地址
OSS_ACCESS_KEY_ID=                    # 可选，阿里云 OSS
```
