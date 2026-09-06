# GrokBot in DeepseekHarness 🐾

> 把 [LaoA-GrokBot](https://github.com/zhulin025/LaoA-GrokBot) 的 GrokBot 宠物带进 DeepSeek Harness Web GUI 的浏览器客户端插件（`dsh.client` + `dsh.bundle`）：可换装、可拖拽甩出、带惯性物理与碰撞反弹的桌宠，随 dsh web 一起启动。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Platform](https://img.shields.io/badge/Platform-DeepSeek%20Harness%20Web-5b7fe5)

本仓库同时是：

- **可安装的 dsh 插件包**：根 `package.json` 声明 `dsh.bundle`（`cordis.patch.yml` 作为一层 patch 插入浏览器客户端行）与 `dsh.client`（`platform: web`）；
- 一个**独立的 npm 包源码**（`@edge-hh/dsh-grokbot-pet`），`pnpm install && pnpm bundle` 即可本地产出 `lib/` 产物。

---

## 安装

前置：装有 DeepSeek Harness（`dsh` CLI），并准备一个 `web` profile。

### 方式一：npm 包（发布后，推荐）

```bash
dsh plugin --profile web add @edge-hh/dsh-grokbot-pet
```

### 方式二：直接从本仓库安装（源码构建）

```bash
dsh plugin --profile web add git+https://github.com/Edge-HH/GrokBot_in_DeepseekHerness.git
```

从源码安装会在 profile 内执行本包的 `prepare` 构建（tsdown），需要能访问 npm registry 拉取构建依赖。预构建发布（npm 包或 Release `.tgz`）可跳过构建步骤。

安装完成后**重启 dsh web 服务并刷新浏览器（Ctrl+Shift+R）**，桌宠出现在对话输入框右上方。卸载：`dsh plugin --profile web remove @edge-hh/dsh-grokbot-pet`（或对应安装源），再重启。

> 本插件的运行依赖（react、cordis、`@deepseek-ai/*` 官方包）由 dsh web 自带提供，声明为 `peerDependencies`，安装时不会重复下载。

---

## ✨ 功能

- **原版 GrokBot 渲染**（严格移植自 LaoA-GrokBot，MIT）：6 组官方表情环（待机/思考/工作/开心/倾听/惊讶）、原版眼睛多边形 + 3D 注视 + 眨眼 + 弹簧变形；bounce / tilt / scan / glitch 状态动画
- **外观自定义**：10 种颜色（官方 hex，0.22s 过渡）、8 种形状（官方 SVG path）、4 件身体部件 + 4 件趣味配饰，可多选叠穿；右键菜单整合「换装 + 控制」，设置页带实时预览
- **拖拽物理**：甩出速度 = 松手瞬间鼠标速度（默认阈值 120 px/s），带惯性滑行、动态模糊、残影与边界/输入框碰撞反弹；拖拽不足 24px 视为移动、不甩出
- **眼睛跟随鼠标**：「始终注视」默认开启（可在设置关闭），鼠标不悬停也看向鼠标
- **6 种快捷动作**（开心/惊讶/倾听/思考/工作/待机）：点击随机触发，设置页可手动触发（带预览）
- **改名**：设置里修改宠物名字，气泡标题、提示文字同步替换
- **用量气泡**：每轮对话结束后弹出，显示该轮 provider 上报的 tokens 用量与估算金额（DeepSeek 官方定价近似值：输入 ¥2 / 输出 ¥8 / 缓存读 ¥0.5 / 缓存写 ¥2，每百万 tokens —— 仅供参考，非精确账单）
- **进度气泡**：工具名、请求次数、步数、用时、进度条
- **WebAudio 合成音效**（请求/工具/回合/弹跳），可关
- **桌面通知**（Chrome → Windows），右键菜单 → 控制 → 通知授权，可关
- **隐藏/显示**：右键菜单即时生效

---

## 构建与发布

```bash
pnpm install     # 安装构建依赖（devDependencies）
pnpm bundle      # tsdown：产出 lib/index.js + lib/invariant.js（node 半段）与 lib/client.js（浏览器半段）
pnpm typecheck   # tsc --noEmit（浏览器侧类型检查）
```

- `lib/` 为构建产物，已 gitignore，不入库；
- 发布 npm 包时 `package.json` 的 `repository` 已指向本仓库，market/storefront 会自动建立映射；
- 不发 npm 也可把 `pnpm pack` 产出的 `.tgz` 附加到 GitHub Release（资产名请勿携带版本号，如 `dsh-grokbot-pet.tgz`）。

---

## 📁 文件结构

```
├── package.json           # dsh.bundle + dsh.client 清单、依赖区间（本插件包即仓库根）
├── cordis.patch.yml       # bundle patch 层：向组合树插入 dsh.client 浏览器客户端行
├── tsdown.config.ts       # 独立构建：node 半段 + 浏览器 client bundle（对齐 harness 客户端加载契约）
├── tsconfig.json          # 独立类型检查（不依赖 deepseek-harness monorepo）
├── src/
│   ├── index.ts           # 宿主侧（node）入口：无行为（Loader 挂载根包所需）
│   ├── invariant.ts       # invariant 伴生注册（无运行时不变量）
│   └── client/
│       ├── index.ts       # 客户端入口：向 shell.overlay / settings.section 槽注入
│       ├── pet.tsx        # PetView + GrokbotFigure（主组件/渲染/物理/音效/通知）
│       ├── store.ts       # 会话快照 → 宠物状态桥接（PetUsage 等）
│       ├── data.ts        # 颜色/形状/部件/配饰/设置
│       ├── pet.module.css # 样式（CSS Modules，构建时内联）
│       └── expressions.json # 原版表情环数据（构建时内联）
└── tests/                 # node-plugin 冒烟测试（随 deepseek-harness 开发环境运行）
```

---

## 🔧 本地开发（在 deepseek-harness 源码树内联调试）

桌面宠物最初以内置客户端插件形态开发。若要在源码树内联调试：

1. 将 `src/` 与 `tests/`、`package.json` 同步到 `<harness根>/packages/client/ui-pet/`；
2. 在 `<harness根>/packages/bundle/web-app/cordis.patch.yml` 与 `package.json` 注册 `ui-pet`（或自定义 id）依赖；
3. 在 harness 根执行 `corepack pnpm install`、`corepack pnpm exec tsc -b packages/client/ui-pet`、`corepack pnpm --filter ... run bundle`、`corepack pnpm run build:web` 后重启。

对外发布形态（本仓库）与之等价：`cordis.patch.yml` 插入的行、`dsh.client` 清单与构建产物契约完全一致，只是不再需要改动官方源码。

---

## 📜 许可

MIT License — 宠物造型数据来自 [LaoA-GrokBot](https://github.com/zhulin025/LaoA-GrokBot)（作者：老A玩AI，MIT）。完整署名见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
