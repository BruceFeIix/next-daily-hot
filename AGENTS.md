# AGENTS.md — 今日热榜（daily-trending）开发指南

> 入口文件：只保留项目定位、常用命令与硬性规则；详细规范按任务拆分在 `docs/`，动手前先按「文档索引」读对应分册。

## 项目概览

- **定位**：聚合微博、知乎、B 站、抖音、GitHub 等全网热门平台热榜的单页应用；纯聚合前端，无数据库、无自建后端，数据由 Next.js Route Handlers（`src/app/api/<platform>/route.ts`）在服务端抓取各平台上游接口，统一映射为 `HotListItem[]` 后返回客户端渲染
- **仓库**：<https://github.com/baiwumm/daily-trending>；线上：<https://hot.baiwumm.com>（Vercel 部署）
- **页面**：唯一业务页面 `src/app/page.tsx`，一次渲染全部榜单卡片（数据进入视口后按需拉取）

## 常用命令

| 任务 | 命令 |
| --- | --- |
| 安装依赖（仅 pnpm，有 pnpm-lock.yaml） | `pnpm install` |
| 开发服务器（localhost:3000） | `pnpm dev` |
| 生产构建（`.env` 已提交，clone 即有） | `pnpm build` |
| 运行生产构建 | `pnpm start` |
| Lint 单文件 | `pnpm eslint src/path/to/file.ts` |
| Lint 全量 / 自动修复 | `pnpm lint` / `pnpm lint:fix` |
| 类型检查（无独立 typecheck 脚本） | `npx tsc --noEmit` |
| 本地发版（备用，需本地 `GITHUB_TOKEN`） | `pnpm release` |

无测试框架，且 PR / push 无 CI 门禁——提交前必须自查 `pnpm lint` + `pnpm build`（build 内含类型检查，详见 `docs/architecture.md`）。

## 文档索引

| 需求 | 文件 |
| --- | --- |
| 架构 / 目录 / 数据流 / 缓存 / 状态 / 动画 / SEO | `docs/architecture.md` |
| 代码风格 / 命名 / 注释 / 提交规范 / UI / 环境变量 / 本地调试 | `docs/development.md` |
| 新增榜单 / 上游抓取与反爬 / 健康巡检 | `docs/add-hotlist.md` |
| 发版流程 / tag 规则 / README 徽章同步 | `docs/release.md` |
| 路线图 / 维护约定 | `docs/ROADMAP.md` |
| 项目说明 / 部署 / 开放 API | `README.md` |

## AI Agent 代码生成规范

项目级 Skill 统一安装在 `.agents/skills/`，涉及对应技术产出前必须先读对应 `SKILL.md` 并遵循其规则：

- **`heroui-react`**：任何 HeroUI 组件任务（组件 API、主题与暗色模式、安装配置）先读它；仅适用 v3，勿凭 v2 记忆写 API
- **`vercel-react-best-practices`**：新建组件/页面、数据获取、重构、性能与 bundle 优化前读
- **`vercel-composition-patterns`**：组件设计与 API 设计、重构布尔 prop 泛滥 / prop drilling 前读（React 19 规则：禁 `forwardRef`，用 `use()` 替代 `useContext()`）

通用规则：

- **冲突裁决**：本文件、`docs/` 分册与既有代码架构 > Skill 规则；规则细节以 Skill 目录内文件为准（部分 Skill 为单文件，部分含 `rules/` 分册）
- **框架 API**：Next.js 框架行为以 `node_modules/next/dist/docs/` 内置文档为准（版本随依赖精确匹配），先查文档再写
- **不回退**：改造时保持既有代码同等水准（`useRequest` 竞态保护、zustand persist 版本化迁移、`fetchJson` 统一超时）

## 禁止事项

1. 禁止修改 `IResponse` 响应结构与 `successResponse` / `errorResponse` 语义（前端全局依赖）
2. 禁止绕过 `src/lib/request.ts` 的 `fetchJson` / `fetchText` 直接裸 `fetch` 上游（丢失 UA / 超时 / 缓存策略）
3. 禁止 `hotItemsConfig` 的 `value` 与 `src/app/api/` 目录名不一致——value 即路由路径，改名必须两处同步
4. 禁止引入替代性基础库（UI 非 HeroUI、状态非 zustand、请求非原生 fetch + 自研 `useRequest`），除非用户明确要求并评估影响
5. 禁止改动 `eslint.config.mjs` 与 Prettier 风格基线；新增代码以 `pnpm lint:fix` 适配
6. 禁止手工编辑 `.heroui-docs/`（工具生成目录）；禁止把工具生成的文档索引块（HeroUI / Next.js）写入本文件，保持本文件纯手写
7. 禁止提交 `.env.local` 或任何私密密钥；`.env` 中只允许公开变量
8. 禁止在未沟通的情况下升级 Next.js / HeroUI / React 大版本（历史上均为独立 `chore:` 提交，需配套回归验证）
9. 禁止在仓库内把 Skill 复制到 `.agents/skills/` 之外的位置（避免多副本漂移）
