# WeavesAI 本地部署指南

## 一、环境要求

| 项目 | 要求 |
|------|------|
| **操作系统** | Windows 10/11（完整功能）或 Linux/macOS（仅 Web 服务） |
| **Node.js** | v20 LTS |
| **包管理器** | pnpm ≥ 10.4.1 |
| **数据库** | SQLite（内置，无需额外安装） |
| **可选** | Redis（多进程部署时需要）、Docker（构建沙箱时需要） |

> **说明**：GUIDevice（桌面自动化）和 Windows 服务仅支持 Windows。如果只需要 Web 界面 + AI Agent 对话功能，Linux/macOS 也可以运行核心服务。

---

## 二、克隆仓库

```bash
git clone https://github.com/WeavesAI/WeavesAI.git
cd WeavesAI
```

---

## 三、部署核心服务（weavesai-service）

这是平台的主体，包含后端 API、前端 UI、WebSocket 网关。

### 3.1 安装依赖

```bash
cd weavesai-service
pnpm install
```

> 如果没有安装 pnpm，先执行：`npm install -g pnpm@10.4.1`

### 3.2 生成模板数据库

```bash
pnpm db:push
```

这会根据 `drizzle/schema.ts` 中的表结构，生成 `db/template.db`。首次启动时，服务会自动复制模板到 `data/weavesai.db` 作为运行数据库。

### 3.3 配置环境变量

```bash
cp .env.example .env
```

编辑 `.env` 文件，以下是各项说明：

```ini
# ─── 数据库 ───
DATABASE_PATH=./data/weavesai.db

# ─── 认证 ───
# 随机字符串，用于 JWT 签名（必填）
JWT_SECRET=your-random-secret-string-here

# ─── Redis（可选）───
# 单进程部署可留空；多进程（PM2 多实例）时需要配置
REDIS_URL=

# ─── 调试控制台（可选）───
# 设置后可通过 /debug 路径访问远程调试控制台
DEBUG_CONSOLE_TOKEN=
DEBUG_SSH_HOST=
DEBUG_SSH_PORT=2202
DEBUG_SSH_USER=
DEBUG_SSH_PASSWORD=

# ─── 沙箱（可选）───
# 如果需要 AI Agent 执行 Shell 命令的隔离环境
# 不配置则沙箱功能不可用，不影响其他功能
E2B_API_KEY=admin-987654321
E2B_API_URL=http://<你的沙箱服务器IP>:7788
E2B_SANDBOX_URL=http://<你的沙箱服务器IP>:7788

# ─── 阿里云 OSS（可选）───
# 用于组件分发，本地部署可留空
OSS_ACCESS_KEY_ID=
OSS_ACCESS_KEY_SECRET=
OSS_BUCKET=weavesai
OSS_REGION=oss-cn-beijing

# ─── CI 组件上传（可选）───
UPLOAD_SECRET=
```

**最小化配置**：只需填写 `JWT_SECRET` 即可启动，其他全部留空。

### 3.4 构建项目

```bash
# 构建前端（React + Vite）和后端（esbuild）
pnpm build
```

构建产物：
- `dist/public/` — 前端静态文件
- `dist/index.js` — 后端服务入口

### 3.5 启动服务

**开发模式**（热重载，适合调试）：

```bash
pnpm dev
```

**生产模式**（推荐）：

```bash
# 方式 A：直接启动
NODE_ENV=production node dist/index.js

# 方式 B：通过 PM2 启动（推荐，支持守护进程和自动重启）
npm install -g pm2
pm2 start ecosystem.config.cjs
pm2 save
pm2 startup   # 设置开机自启（可选）
```

### 3.6 访问服务

启动后，服务默认监听端口 **3000**（如被占用会自动递增，最高到 3020）。
PM2 模式下端口为 **3001**。

打开浏览器访问：

```
http://localhost:3000
```

---

## 四、配置 LLM 模型（必做）

平台本身不包含 AI 模型，需要在 Web UI 中配置你的 LLM API。

### 4.1 预置模型

系统预置了以下模型定义（`config/models.json`）：

| 模型 | 协议 | 思考模式 |
|------|------|----------|
| Claude Sonnet 4.6 | Anthropic 原生 | 支持 (low/medium/high/xhigh/max) |
| Claude Opus 4.6 | Anthropic 原生 | 支持 |
| GPT 5.4 | OpenAI 原生 | 不支持 |
| GPT 5.4 Nano | XML | 不支持 |

### 4.2 配置步骤

在 Web UI 的设置页面中：

1. **添加 Provider**（提供商）：
   - 名称：如 "Anthropic"、"OpenAI" 或你的中转服务商
   - Base URL：API 端点（如 `https://api.anthropic.com`）

2. **添加 API Key**：
   - 关联到对应 Provider
   - 填入你的 API Key

3. **绑定 Provider Model**：
   - 将 Provider + API Key 与预置模型关联
   - 填写 API Model ID（如 `claude-sonnet-4-20250514`）
   - 配置价格（可选，用于成本统计）

> **提示**：支持 OpenAI 兼容协议，可以接入任何兼容 OpenAI API 格式的服务商。

---

## 五、部署 GUIDevice（仅 Windows，可选）

GUIDevice 提供桌面自动化能力（截图 + 键鼠模拟），让 AI Agent 能操作你的电脑。

```bash
cd ../guidevice
pnpm install
npx tsc          # 编译 TypeScript → app/
pnpm start       # 启动服务
```

启动后，核心服务会自动发现并注册 GUIDevice。

**依赖**：需要 `node-interception`（键鼠）和 `node-screenshots`（截图）两个原生模块，仅 Windows 可用。

---

## 六、部署 WSL2 沙箱（仅 Windows，可选）

沙箱提供隔离的 Linux 环境，让 AI Agent 安全执行 Shell 命令，不影响宿主系统。

### 6.1 从源码构建

```bash
cd ../sandbox

# 构建 Docker 镜像
docker build -t weavesai-sandbox:v1.5 .

# 导出为 rootfs tar
docker create --name weavesai-tmp weavesai-sandbox:v1.5
docker export weavesai-tmp > weavesai-sandbox-v1.5.tar
docker rm weavesai-tmp
```

### 6.2 导入 WSL2

在 Windows PowerShell 中：

```powershell
wsl --import weavesai-sandbox C:\WeavesAI\sandbox .\weavesai-sandbox-v1.5.tar
wsl -d weavesai-sandbox -u weavesai   # 验证启动
```

### 6.3 沙箱环境内容

- Ubuntu 22.04
- Node.js 20 LTS + pnpm
- Python 3 + pip
- Git, curl, wget, build-essential
- 默认用户：`weavesai` (uid 1000)

---

## 七、注册为 Windows 服务（可选）

将 WeavesAI 注册为 Windows 系统服务，实现开机自启。

### 7.1 编译服务包装器

```bash
cd ../svc-wrapper
gcc -O2 -s -municode -o WeavesAIService.exe svc-wrapper.c -ladvapi32
```

### 7.2 配置

编辑 `service.cfg`：

```ini
[Service]
ServiceName=WeavesAIService
NodePath=..\node\node.exe       # 指向你的 Node.js 可执行文件
EntryPath=service-entry.js      # 入口脚本
LogPrefix=service               # 日志文件前缀
```

### 7.3 注册服务

```powershell
# 以管理员身份运行
.\WeavesAIService.exe install
net start WeavesAIService
```

---

## 八、远程调试（可选）

部署完成后，可通过 Debug CLI 远程诊断：

```bash
cd weavesai-service

# 本地调试
npx tsx scripts/debug-cli.ts ping

# 远程调试
DEBUG_HOST=http://<服务器IP>:3000 npx tsx scripts/debug-cli.ts system   # 系统信息
DEBUG_HOST=http://<服务器IP>:3000 npx tsx scripts/debug-cli.ts logs     # 查看日志
DEBUG_HOST=http://<服务器IP>:3000 npx tsx scripts/debug-cli.ts tables   # 数据库表
DEBUG_HOST=http://<服务器IP>:3000 npx tsx scripts/debug-cli.ts sql "SELECT * FROM chats"
```

> 需要在 `.env` 中设置 `DEBUG_CONSOLE_TOKEN` 才能使用。

---

## 九、快速启动清单

最小化部署只需以下步骤：

```bash
# 1. 克隆
git clone https://github.com/WeavesAI/WeavesAI.git
cd WeavesAI/weavesai-service

# 2. 安装依赖
pnpm install

# 3. 生成数据库
pnpm db:push

# 4. 配置环境变量（至少填 JWT_SECRET）
cp .env.example .env
# 编辑 .env，设置 JWT_SECRET=<任意随机字符串>

# 5. 构建
pnpm build

# 6. 启动
NODE_ENV=production node dist/index.js

# 7. 打开浏览器
# http://localhost:3000
```

启动后在 Web UI 设置页面配置你的 LLM API Key，即可开始使用。

---

## 十、注意事项

1. **许可证**：本项目采用 CC BY-NC-SA 4.0，**禁止商业使用**。商业授权联系：tw889508@gmail.com

2. **无用户系统**：开源版本不包含注册/登录/邀请码功能，本地部署后直接可用，无需账号。

3. **数据存储**：
   - 数据库：`data/weavesai.db`（SQLite 文件）
   - 用户文件：`workspace/` 目录
   - 上传文件：`uploads/` 目录
   - 建议定期备份以上目录

4. **端口冲突**：如果 3000 端口被占用，可通过 `PORT=8080 node dist/index.js` 指定端口，或让服务自动检测 3000-3020 范围内的可用端口。

5. **Native 模块**：`better-sqlite3` 需要编译原生模块，确保系统安装了 C++ 编译工具链：
   - Windows：Visual Studio Build Tools
   - Linux：`sudo apt install build-essential python3`
   - macOS：`xcode-select --install`
