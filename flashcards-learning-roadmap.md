# 闪记全栈项目 · 学习指引与任务目录

> 技术栈：TypeScript / React + Vite / NestJS / Prisma / MySQL 8 / Docker
> 使用方式：按 Phase 顺序推进，每个 Task 满足「完成条件」再进入下一个。勾选框可直接在编辑器里打勾。

---

## 总览

| Phase | 主题 | 预估 | 产出 |
|---|---|---|---|
| 0 | 开发环境搭建（含 Docker 入门） | 1 天 | 能跑 `node / pnpm / docker` 三条命令 |
| 1 | 要件定义（你自己写） | 0.5 天 | `docs/requirements.md` |
| 2 | 数据库设计 + Prisma | 0.5 天 | `schema.prisma` + 首次 migration |
| 3 | 后端骨架 + 第一个接口 | 1 天 | `GET /decks` 返回真实数据 |
| 4 | 前端骨架 + 打通第一条链路 | 1 天 | 页面显示 DB 里的 Deck |
| 5 | Deck CRUD（前后端） | 1～2 天 | 增删改查全部可用 |
| 6 | 注册 / 登录 | 2 天 | JWT 鉴权 + 前端登录态 |
| 7 | 数据归属（多用户隔离） | 0.5 天 | A 用户看不到 B 用户数据 |
| 8 | Card CRUD + 复习界面 | 2 天 | 完整闪记功能 |
| 9 | 全项目 Docker 化 | 1 天 | `docker compose up` 起三容器 |
| 10 | 进阶（可选） | 不限 | 复习算法 / 测试 / 部署 |

---

## 遇到卡壳时的通用排查方法（先读这一节）

配置环境和陌生库的报错，90% 靠下面这套流程能自己解决：

1. **完整复制报错的第一行和最后一行**，不要只看中间的堆栈
2. 判断报错来源：是 **Node / pnpm**（包管理）、**Docker**（容器）、**Prisma**（数据库）、还是 **Nest / Vite**（框架）—— 报错前缀通常会写
3. 去该工具的**官方文档**搜报错里的关键名词（不是搜整句）
   - Docker → https://docs.docker.com
   - Prisma → https://www.prisma.io/docs
   - NestJS → https://docs.nestjs.com
4. 对陌生库的 API：打开 `node_modules/<包名>/` 里的 `.d.ts` 文件，直接看类型签名，比看博客准确
5. 20 分钟没进展 → 停下来，把「报错原文 + 你执行的命令 + 相关文件内容」整理好再求助

---

## Phase 0 · 开发环境搭建

### Task 0-1：安装 Node.js 与 pnpm

- [ ] 安装版本管理器（推荐 **fnm** 或 **nvm**），不要直接从官网装 Node
  - macOS / Linux：`curl -fsSL https://fnm.vercel.app/install | bash`
  - Windows：`winget install Schniz.fnm`
- [ ] 安装 Node 22 LTS：`fnm install 22 && fnm use 22`
- [ ] 启用 pnpm：`corepack enable && corepack prepare pnpm@latest --activate`

**完成条件**
```bash
node -v    # v22.x.x
pnpm -v    # 9.x 或 10.x
```

**为什么用版本管理器**：不同项目要求的 Node 版本不同，直接装会导致后面换项目时反复卸载重装。

---

### Task 0-2：安装 Docker

**先理解四个概念（各 10 秒）**

| 概念 | 类比 | 你会在哪里遇到 |
|---|---|---|
| **Image（镜像）** | 软件安装包 | `image: mysql:8.4` |
| **Container（容器）** | 用安装包跑起来的进程 | `docker compose ps` 列出的东西 |
| **Volume（卷）** | 容器外面的硬盘 | 保存 MySQL 数据，容器删了数据还在 |
| **Compose** | 一份 YAML 描述多个容器怎么一起跑 | `docker-compose.yml` |

**安装步骤**

- [ ] **macOS**：下载 Docker Desktop（Apple Silicon 选 arm64），安装后启动，等菜单栏鲸鱼图标停止动画
- [ ] **Windows**：
  1. 以管理员打开 PowerShell：`wsl --install`，重启
  2. 安装 Docker Desktop，设置里确认 **Use the WSL 2 based engine** 已勾选
  3. 之后所有命令在 **WSL 终端**（Ubuntu）里执行，项目文件也放在 WSL 文件系统内（`~/` 下），不要放在 `/mnt/c/` —— 否则文件监听和性能都有问题
- [ ] **Linux**：按官方文档装 Docker Engine + Compose plugin，并执行 `sudo usermod -aG docker $USER` 后重新登录

**完成条件**
```bash
docker --version           # Docker version 2x.x
docker compose version     # Docker Compose version v2.x
docker run --rm hello-world   # 看到 "Hello from Docker!"
```

**常见报错**
- `Cannot connect to the Docker daemon` → Docker Desktop 没启动
- Windows 下 `docker` 命令找不到 → 在 Docker Desktop 设置 Resources → WSL Integration 里勾选你的发行版

---

### Task 0-3：编辑器与工具

- [ ] VS Code 扩展：`Prisma`、`ESLint`、`Prettier`、`Docker`、`Thunder Client`（或单独装 Postman / Bruno，用来手动测 API）
- [ ] 数据库 GUI：任选 **TablePlus** / **DBeaver** / **MySQL Workbench**，用来直接看表里的数据
- [ ] Git 已配置 `user.name` / `user.email`，GitHub 建一个空仓库

---

### Task 0-4：初始化项目目录

- [ ] 建目录并初始化
  ```bash
  mkdir flashcards && cd flashcards
  git init
  pnpm init
  mkdir -p apps packages docs
  ```
- [ ] 建 `pnpm-workspace.yaml`
  ```yaml
  packages:
    - "apps/*"
    - "packages/*"
  ```
- [ ] 建 `.gitignore`（至少包含 `node_modules/`、`.env`、`dist/`）
- [ ] 建 `.env.example`（提交）和 `.env`（不提交）

**完成条件**：`git status` 里看不到 `.env`。

---

### Task 0-5：用 Docker 起 MySQL（你的第一个容器）

- [ ] 建 `docker-compose.yml`

```yaml
services:
  db:
    image: mysql:8.4
    container_name: flashcards-db
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  db-data:
```

- [ ] `.env` 写入
```env
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=flashcards
MYSQL_USER=app
MYSQL_PASSWORD=apppass
DATABASE_URL="mysql://root:rootpass@localhost:3306/flashcards"
```

- [ ] 启动并验证
```bash
docker compose up -d db
docker compose ps          # STATUS 要变成 (healthy)，约 20 秒
docker compose logs db     # 看到 "ready for connections"
```

- [ ] 用 GUI 工具连 `localhost:3306` / `root` / `rootpass`，能看到空的 `flashcards` 库

**逐行理解 YAML（必读）**
| 行 | 含义 |
|---|---|
| `image: mysql:8.4` | 从 Docker Hub 拉 MySQL 8.4 的镜像 |
| `${MYSQL_ROOT_PASSWORD}` | 从同目录 `.env` 读取变量 |
| `ports: "3306:3306"` | 左边宿主机端口 : 右边容器端口，让你电脑能访问容器里的 MySQL |
| `volumes: db-data:/var/lib/mysql` | 把容器内 MySQL 数据目录挂到命名卷 `db-data`，容器删了数据不丢 |
| `healthcheck` | 定期 ping，让 Docker 知道 MySQL 真正可用（而非仅进程启动） |

**日常命令**
```bash
docker compose up -d db        # 启动
docker compose stop            # 停止（数据保留）
docker compose down            # 删除容器（数据保留在卷里）
docker compose down -v         # 删除容器 + 卷 = 彻底清库
docker compose exec db mysql -uroot -prootpass flashcards   # 进容器里的 mysql 命令行
```

**常见报错**
- `port is already allocated` → 本机已有 MySQL 占 3306，改左边端口为 `"3307:3306"`，`DATABASE_URL` 也改 3307
- `Access denied for user` → `.env` 改过密码但卷里还是旧的，`down -v` 重建

**Phase 0 完成标志**：`node`、`pnpm`、`docker compose ps` 三条命令正常，MySQL 容器 healthy，GUI 能连上。

---

## Phase 1 · 要件定义（你自己写）

写到 `docs/requirements.md`。下面是模板，**每一项都要填，填不出来说明还没想清楚**。

### 模板

```markdown
# 要件定義

## 1. 目的
- 一句话：这个应用解决什么问题、给谁用

## 2. 用户与场景
- 主要用户：
- 典型使用场景（3 个）：
  1.
  2.
  3.

## 3. 功能一览
| ID | 功能 | 优先级 (MVP / 后续) | 备注 |
|---|---|---|---|
| F-01 | 注册 | MVP | |
| F-02 | 登录 / 登出 | MVP | |
| F-03 | Deck 增删改查 | MVP | |
| F-04 | Card 增删改查 | MVP | |
| F-05 | 复习模式（翻卡） | MVP | |
| F-06 | 复习算法（间隔重复） | 后续 | |
| F-07 | ... | | |

## 4. 画面一览
| 画面ID | 名称 | 主要元素 | 对应功能 |
|---|---|---|---|
| S-01 | 登录页 | | F-02 |
| S-02 | Deck 列表 | | F-03 |
| ... | | | |

## 5. API 一览（暂定）
| Method | Path | 说明 | 需登录 |
|---|---|---|---|
| POST | /auth/register | | ✗ |
| POST | /auth/login | | ✗ |
| GET | /decks | | ✓ |
| ... | | | |

## 6. 数据模型（实体与关系）
- User 1 — N Deck
- Deck 1 — N Card
- （复习记录？暂缓）

## 7. 非功能要件
- 响应时间：
- 数据保护：密码必须哈希存储；用户只能访问自己的数据
- 浏览器支持：

## 8. 明确不做的事（MVP 阶段）
-
```

### 检查清单
- [ ] 每个画面都能对应到至少一个功能 ID
- [ ] 每个 API 都标注了是否需要登录
- [ ] 「明确不做的事」至少写 3 条（防止范围膨胀）
- [ ] MVP 功能不超过 6 个

---

## Phase 2 · 数据库设计 + Prisma

### Task 2-1：创建 NestJS 项目

```bash
cd apps
npx @nestjs/cli new api --package-manager pnpm --strict
cd api
```

- [ ] `pnpm run start:dev`，浏览器打开 `localhost:3000` 看到 `Hello World!`

### Task 2-2：引入 Prisma

```bash
pnpm add prisma -D
pnpm add @prisma/client
npx prisma init --datasource-provider mysql
```

- [ ] 删除 `apps/api/.env` 里生成的 `DATABASE_URL`，改为读根目录 `.env`（或在 `package.json` scripts 里用 `dotenv -e ../../.env --`）

### Task 2-3：写 schema

- [ ] `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id           Int      @id @default(autoincrement())
  email        String   @unique
  passwordHash String
  decks        Deck[]
  createdAt    DateTime @default(now())
}

model Deck {
  id        Int      @id @default(autoincrement())
  title     String
  userId    Int
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  cards     Card[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([userId])
}

model Card {
  id        Int      @id @default(autoincrement())
  front     String   @db.Text
  back      String   @db.Text
  deckId    Int
  deck      Deck     @relation(fields: [deckId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([deckId])
}
```

- [ ] 对照你 Phase 1 写的数据模型，有出入就以要件定义为准修改

### Task 2-4：首次迁移

```bash
npx prisma migrate dev --name init
npx prisma studio      # 浏览器里的表编辑器
```

- [ ] `prisma/migrations/` 下出现一个带时间戳的目录，里面是生成的 SQL —— **打开读一遍**，确认与你预想的 DDL 一致
- [ ] 在 Prisma Studio 里手动插入 1 个 User、2 个 Deck

**完成条件**：GUI 工具里能看到三张表 + `_prisma_migrations` 表，且有测试数据。

---

## Phase 3 · 后端骨架 + 第一个接口

### Task 3-1：PrismaService（你的第一个 Provider）

- [ ] `src/prisma/prisma.service.ts`
```ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() { await this.$connect(); }
  async onModuleDestroy() { await this.$disconnect(); }
}
```
- [ ] `src/prisma/prisma.module.ts`：`@Module({ providers: [PrismaService], exports: [PrismaService] })`

**在这里停下来理解**
- `@Injectable()`：告诉 Nest 这个类可以被 DI 容器管理
- `providers`：这个 Module **内部**能注入什么
- `exports`：允许**其他** Module 注入什么 —— 不写 exports 就会报 `Nest can't resolve dependencies`

### Task 3-2：DecksModule

```bash
npx nest g module decks
npx nest g controller decks
npx nest g service decks
```

- [ ] `DecksModule` 的 `imports: [PrismaModule]`
- [ ] `DecksService` 构造函数注入 `PrismaService`，实现 `findAll()`
- [ ] `DecksController` 注入 `DecksService`，`@Get()` 返回 `findAll()`

**完成条件**
```bash
curl localhost:3000/decks
# [{"id":1,"title":"...","userId":1,...},{...}]
```

### Task 3-3：开启 CORS 与全局校验

- [ ] `main.ts`
```ts
app.enableCors({ origin: 'http://localhost:5173' });
app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
```
- [ ] `pnpm add class-validator class-transformer`

---

## Phase 4 · 前端骨架 + 打通链路

### Task 4-1：创建 Vite 项目

```bash
cd apps
pnpm create vite web --template react-ts
cd web && pnpm install
pnpm add @tanstack/react-query react-router-dom
```

### Task 4-2：API 客户端

- [ ] `src/api/client.ts`：封装 `fetch`，base URL 读 `import.meta.env.VITE_API_URL`
- [ ] `.env` 加 `VITE_API_URL=http://localhost:3000`（Vite 只读以 `VITE_` 开头的变量）

### Task 4-3：QueryClient + 首个 useQuery

- [ ] `main.tsx` 包 `<QueryClientProvider>`
- [ ] `src/features/decks/useDecks.ts`
```ts
export const useDecks = () =>
  useQuery({ queryKey: ['decks'], queryFn: () => api.get<Deck[]>('/decks') });
```
- [ ] `DeckListPage` 渲染列表，处理 `isPending` / `isError`

**完成条件**：浏览器 `localhost:5173` 显示 Prisma Studio 里插的那两条 Deck。

**这条链路的每一跳（画出来贴在墙上）**
```
浏览器 → fetch → Nest Controller → Service → PrismaService → MySQL 容器
        ←  JSON ←              ←         ←               ←
```

---

## Phase 5 · Deck CRUD

### 后端
- [ ] `dto/create-deck.dto.ts`：`@IsString() @MinLength(1) title`
- [ ] `dto/update-deck.dto.ts`：`PartialType(CreateDeckDto)`（来自 `@nestjs/mapped-types`）
- [ ] Controller 补齐 `POST /decks`、`GET /decks/:id`、`PATCH /decks/:id`、`DELETE /decks/:id`
- [ ] `:id` 用 `@Param('id', ParseIntPipe)`
- [ ] 找不到时抛 `NotFoundException`
- [ ] **暂时把 `userId` 写死为 1**（Phase 7 再改）
- [ ] 用 Thunder Client / Bruno 手测 5 个接口，保存成 collection

### 前端
- [ ] `useCreateDeck` / `useUpdateDeck` / `useDeleteDeck`，每个都是 `useMutation` + `onSuccess: () => queryClient.invalidateQueries({ queryKey: ['decks'] })`
- [ ] 新建 / 编辑表单（受控组件即可，先不上表单库）
- [ ] 删除前 `confirm()`
- [ ] 错误提示：mutation 的 `error` 显示到页面上

**完成条件**：不刷新页面完成增 → 改 → 删，列表实时更新。

**这一 Phase 的核心概念**
- `useQuery` 负责读、`useMutation` 负责写，写完 `invalidateQueries` 触发重读 —— 这是 React Query 的基本节奏
- `whitelist: true` 会剥掉 DTO 里没声明的字段，试一下多传一个字段看效果

---

## Phase 6 · 注册 / 登录

### Task 6-1：注册
- [ ] `pnpm add bcrypt @nestjs/jwt @nestjs/passport passport passport-jwt` + 对应 `@types`
- [ ] `AuthModule` / `AuthService` / `AuthController`
- [ ] `POST /auth/register`：校验 email 格式 → `bcrypt.hash(password, 10)` → 建 User → 返回不含 passwordHash 的对象
- [ ] email 重复时返回 `409 Conflict`

### Task 6-2：登录与 JWT
- [ ] `POST /auth/login`：查 User → `bcrypt.compare` → 签发 JWT（payload 只放 `sub: user.id`）
- [ ] `JWT_SECRET` 放 `.env`，用 `ConfigModule` 读取，**不要硬编码**
- [ ] `JwtStrategy`（passport-jwt）：从 `Authorization: Bearer <token>` 取 token，验证后把 `{ id }` 挂到 `request.user`
- [ ] `JwtAuthGuard extends AuthGuard('jwt')`

### Task 6-3：自定义装饰器
- [ ] `@CurrentUser()`：用 `createParamDecorator` 从 `ctx.switchToHttp().getRequest().user` 取值
- [ ] `GET /auth/me`：加 `@UseGuards(JwtAuthGuard)`，返回当前用户

### Task 6-4：前端登录态
- [ ] `AuthContext`：保存 token（先放 `localStorage`，理解它的 XSS 风险后再决定是否换 httpOnly cookie）
- [ ] `api/client.ts` 每次请求自动加 `Authorization` 头
- [ ] 收到 401 → 清 token → 跳登录页
- [ ] `ProtectedRoute` 组件包住需要登录的路由
- [ ] 登录页 / 注册页

**完成条件**：未登录访问 `/decks` 被重定向；登录后 `GET /auth/me` 返回自己。

**这一 Phase 的核心概念**
- Guard 在 Controller 方法**之前**执行，返回 `false` 或抛异常就拦住请求
- `ExecutionContext` 是 Guard / Decorator 拿到当前请求的入口
- JWT 是「签名过的声明」不是「加密」，payload 任何人可读，所以不放敏感信息

---

## Phase 7 · 数据归属

- [ ] `DecksController` 全部加 `@UseGuards(JwtAuthGuard)`
- [ ] 所有 Service 方法多接一个 `userId` 参数，来源是 `@CurrentUser()`
- [ ] `findAll` → `where: { userId }`
- [ ] `findOne / update / remove` → `where: { id, userId }`，查不到统一 `NotFoundException`（不要返回 403 泄露「存在但不是你的」）
- [ ] 删掉 Phase 5 写死的 `userId = 1`

**完成条件**：注册两个账号，A 建的 Deck，B 用 A 的 Deck id 访问 `GET /decks/:id` 得到 404。

---

## Phase 8 · Card CRUD + 复习界面

- [ ] `CardsModule`，路由设计为嵌套：`/decks/:deckId/cards`
- [ ] 每个操作先确认 `deckId` 属于当前用户（复用 DecksService 的查询）
- [ ] 前端 `DeckDetailPage`：卡片列表 + 新建卡片表单
- [ ] `ReviewPage`：一次显示一张 front，点击翻到 back，「下一张」按钮，结束显示统计
- [ ] （可选）快捷键：空格翻卡、方向键切换

**完成条件**：要件定义里 MVP 标注的功能全部可用。

---

## Phase 9 · 全项目 Docker 化

> 到这一步你已经用了两三周 `docker compose up -d db`，对容器有手感了，现在把 api 和 web 也装进去。

### Task 9-1：后端 Dockerfile

- [ ] `apps/api/Dockerfile`（多阶段：deps → dev → build → prod，见前文方案）
- [ ] `apps/api/.dockerignore`：`node_modules`、`dist`、`.env`
- [ ] 单独测试：`docker build -t flashcards-api --target dev ./apps/api`

### Task 9-2：前端 Dockerfile

- [ ] `apps/web/Dockerfile`（dev → build → nginx prod）
- [ ] `.dockerignore` 同上

### Task 9-3：compose 加 api / web 服务

- [ ] `api` 服务：`build.target: dev`、`depends_on: db: condition: service_healthy`、`DATABASE_URL` 的 host 改成 **`db`**
- [ ] `web` 服务：`build.target: dev`、`--host 0.0.0.0`
- [ ] bind mount 源码 + 命名卷隔离 `node_modules`
- [ ] `docker compose up --build`，一条命令起三容器

**完成条件**：关掉宿主机所有 `pnpm dev`，只靠 `docker compose up`，浏览器功能全部正常，且改代码后热重载生效。

### 必踩坑速查
| 现象 | 原因 | 处理 |
|---|---|---|
| api 报 `ECONNREFUSED 127.0.0.1:3306` | 容器内 `localhost` 是容器自己 | `DATABASE_URL` host 改 `db` |
| 浏览器请求 `http://api:3000` 失败 | 浏览器不在 compose 网络里 | 前端仍用 `localhost:3000` |
| 容器里 `Cannot find module` | 宿主机 node_modules 覆盖了容器的 | 加命名卷 `/app/node_modules` |
| 装了新包容器不认 | 命名卷里还是旧依赖 | `docker compose build api` |
| 改代码不热重载 | 文件事件穿不过 bind mount | 环境变量 `CHOKIDAR_USEPOLLING=true` |
| Prisma 报 `binaryTargets` | alpine 镜像需要 musl 版引擎 | schema 加 `binaryTargets = ["native", "linux-musl-openssl-3.0.x"]` |

---

## Phase 10 · 进阶（可选，按兴趣选）

| 方向 | 内容 | 练到 |
|---|---|---|
| 复习算法 | 实现 SM-2，加 `Review` 表记录每次评分与下次复习时间 | 事务、日期计算、索引设计 |
| 测试 | Nest e2e 测试（supertest）跑在临时 MySQL 容器上 | 测试隔离、CI |
| 类型共享 | `packages/shared` 放 DTO 类型，前后端同时引用；或后端出 OpenAPI，前端用 Orval 生成 | monorepo 依赖管理 |
| 安全 | token 改 httpOnly cookie + refresh token；rate limit | Web 安全基础 |
| 部署 | prod 镜像推到 VPS / Fly.io，Nginx 反代，HTTPS | 运维入门 |
| CI | GitHub Actions：lint + test + build image | 自动化 |

---

## 每个 Phase 结束时做三件事

1. **Git 提交**，message 写清楚 Phase 编号
2. 回到 `docs/requirements.md`，实际做出来和写的不一致的地方，**改文档**（要件是活的）
3. 在 `docs/learning-log.md` 记 3 行：这阶段最卡的地方 / 怎么解决的 / 下次能更快的方法
