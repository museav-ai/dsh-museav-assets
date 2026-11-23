<h1 align="center">🖼️ dsh-museav-assets</h1>

<p align="center">
  <strong>在 DSH 里创作、探索灵感并管理素材。</strong><br>
  从左侧「Museav」入口打开独立工作台，查看精选作品和可用模型，<br>
  使用提示词或模板出图，并按项目管理素材与作品。
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@dsh-plugins/dsh-museav-assets"><img src="https://img.shields.io/npm/v/%40dsh-plugins%2Fdsh-museav-assets?style=for-the-badge&color=4d6bfe&logo=npm&label=npm" alt="npm" /></a>
  <img src="https://img.shields.io/badge/runtime_deps-0-5A9E6F?style=for-the-badge" alt="runtime deps" />
  <img src="https://img.shields.io/badge/license-MIT-777?style=for-the-badge" alt="MIT" />
  <a href="https://github.com/deepseek-ai/deepseek-harness"><img src="https://img.shields.io/badge/DeepSeek_Harness-Plugin-4d6bfe?style=for-the-badge" alt="DSH Plugin" /></a>
</p>

<p align="center">
  <a href="README.en.md">English</a> | 中文
</p>

## Why This

| | 本插件 | `mcp__cs__*` 工具 | 直接敲 CLI |
|---|---|---|---|
| 看清「有哪些项目、各有多少素材」 | ✅ 一屏网格 + 计数 | ❌ 单次调用，看不见全貌 | ⚠️ 逐条命令，靠脑记 |
| 素材拿直链当垫图 | ✅ 点一下复制 | ⚠️ 得先知道素材 id | ⚠️ 手抄 URL |
| 出图 / 上传能力 | ✅ 工作台调用本机 CLI | ✅ 完整 | ✅ 完整 |
| 密钥 | ✅ 不读不转发 `~/.museav.json` | ✅ | ⚠️ token 落在本机明文文件 |

工作台通过本机 **museav CLI** 使用账户、模型、项目、素材和生成能力；精选灵感来自 MUSE AV 的公开接口。模型由平台自动路由，展示的列表不代表可以手动指定模型。

## 工作台入口

插件在全局侧栏注册「Museav」，点击后在主区打开独立工作台。原会话顶部的「资产」Tab 不再注册；已有项目、素材和生成记录仍由原 CLI 数据源提供。

## 平台配图

在出图台选择小红书、X（推特）或闲鱼，写一次简报，勾选所需配图，再点「一键出图」。

| 平台 | 默认配图 |
| --- | --- |
| 小红书 | 3:4 笔记封面、内容配图、总结页 |
| X（推特） | 16:9 横版主图和 1:1 方形配图 |
| 闲鱼 | 1:1 商品主图、商品细节、成色说明 |

闲鱼需先从项目素材库选一张商品实拍。每张图显示自己的出图结果，失败时可以只重试失败的图。自由出图和个人模板仍在同一个出图台。工作台随 DSH 切换浅色和深色。

## Web 安装

```sh
dsh plugin --profile web add @dsh-plugins/dsh-museav-assets
dsh --profile web
```

如果 Web 服务已打开，重启后从全局侧栏进入「Museav」。出图与项目素材使用运行 DSH 的主机上已登录的 `museav` CLI（`museav login`）。

## 接口

| 路由 | 做什么 |
|---|---|
| `GET /api/dsh-museav-assets/health` | CLI 在不在、什么版本、工作区上限 |
| `GET /api/dsh-museav-assets/projects` | 工作区列表 + 出图/素材计数 + `used/limit/full` |
| `POST /api/dsh-museav-assets/projects` | `{name}` 新建工作区（≤20 字） |
| `GET /api/dsh-museav-assets/assets?project=<id\|名>` | 该工作区的素材库 |
| `GET /api/dsh-museav-assets/jobs?limit=<n>` | 最近出图记录 |
| `POST /api/dsh-museav-assets/gen` | 单张出图并按所选项目归档 |

失败语义：**上游/CLI 出错一律 200 + `{ok:false,error}`**，tab 内联显示原因；
只有参数不合法才 400。一次网络抖动不该让 tab 变白屏。

## CLI 契约（插件唯一依赖的东西）

`lib/museav.js` 是**唯一**知道 CLI 输出长什么样的文件。museav-cli 的
stdout/stderr 分工就是它的脚本契约：

| 命令 | stdout（真值） | stderr（人看的补充） |
|---|---|---|
| `museav projects` | id 换行 | 名称 / 出图 x/y / 素材 n / brand |
| `museav projects assets --project X` | `id<TAB>url` | 名称 / media_type / tags |
| `museav jobs` | **完整 JSON 数组** | 表格式摘要 |
| `museav projects create --name N` | 新建 id | 成功提示 |

所以：**真值一律取 stdout，stderr 只用来补展示字段**。stderr 解析不出来就降级
（项目名退回 id 前 8 位，素材名退回「未命名」），绝不因为上游改一句中文就让整页空白。
`test/museav-parse.test.mjs` 用真机抓的输出当夹具守着这条线。

> 已知脆点：`museav projects` / `projects assets` **没有 `--json`**，展示字段只能从
> stderr 表格里解析。给 museav-cli 这两个命令加 `--json` 就能把这条脆点彻底消掉 ——
> 真源在 `~/dev/muse/museav-cli/src/commands/projects.ts`，改的是上游仓，不是本插件。

## 测试

```sh
npm run build:client && npm run check && npm test
```

路由测试用 stub 可执行文件顶替 CLI（走 `MUSEAW_BIN` 环境变量），**全程不联网、不碰真账号**。

CLI 路径为什么是环境变量而不是插件配置：这套 Cordis 的函数式插件读不到 `ctx.config`
（声明了会永远 `pending (waiting for service: config)`）。`MUSEAW_BIN` 也正好和 profile
里 MCP 桥的约定一致。改它不用重载插件 —— 每次请求现取。
