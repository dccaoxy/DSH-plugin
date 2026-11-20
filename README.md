# DSH-plugin

本机（macOS）DeepSeek Harness 的**已装清单**：装了哪些插件、哪些技能，装在哪，怎么在新机器上装回去。
**不含源码、不含配置和密钥**，只记包名、版本、路径、用途。

- 采集时间：2026-11-20
- 环境：macOS（Apple Silicon）+ DSH App `0.2.2-rc.5`
- 机器可读清单：[`plugins.json`](plugins.json)、[`skills.json`](skills.json)

---

## 0. 先分清两类东西

| 类型 | 是什么 | 装在哪 | 怎么装 |
|---|---|---|---|
| 插件 plugin | npm 包，给 DSH 加工具、界面、模型路由 | `~/.dsh/profiles/<profile>/node_modules/` | `dsh plugin add <包名>` |
| 技能 skill | 一个目录，里面至少有 `SKILL.md`（可带 scripts/、references/） | `~/.dsh/skills/<技能名>/` | 把目录复制进去 |

本机 profile 是 `desktop`（另有 `headless`）。插件按 profile 隔离安装。

DSH 命令行（App 自带，不在 PATH 里）：

```bash
DSH="/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh"
"$DSH" plugin --help    # 实际是把参数转给 profile 目录里的 pnpm
```

---

## 1. 插件（11 个，启用 10 个）

| # | 包名 | 版本 | 用途 | 上游 | 许可 | 本机状态 |
|---|---|---|---|---|---|---|
| 1 | `@nanmicoder/dsh-agent-teams` | 0.1.30 | 多 agent 团队协作：队长/成员、任务依赖、消息、Web 树形监控 | [NanmiCoder/dsh-agent-teams](https://github.com/NanmiCoder/dsh-agent-teams) | MIT | 启用 |
| 2 | `@perrylink/dsh-github` | 0.7.28 | GitHub：PR/issue 读写、CI 状态、PR 评审 bot、审批门控 | [PerryLink/dsh-github](https://github.com/PerryLink/dsh-github) | Apache-2.0 | 启用 |
| 3 | `@xmanrui/dsh-im` | 5.9.1 | 即时通讯接入：微信、飞书、企微、钉钉、QQ、Telegram、WhatsApp、Slack、Discord、Matrix | [xmanrui/dsh-im](https://github.com/xmanrui/dsh-im) | MIT | 启用 |
| 4 | `dsh-browser-cdp` | 1.7.12 | 可见浏览器：用 CDP 驱动 Chromium，也可挂接已开的 Chrome/Edge | [drscrewdriver/dsh-browser-cdp](https://github.com/drscrewdriver/dsh-browser-cdp) | MIT | 启用 |
| 5 | `dsh-email` | 0.29.0 | 邮件工作台：多账号收发、草稿、定时发送需确认、OAuth | [STARDUSTLC666/dsh-email](https://github.com/STARDUSTLC666/dsh-email) | MIT | 启用 |
| 6 | `dsh-mac-computer-use` | 1.5.1 | macOS 桌面操作：窗口列表、截图、OCR 找文字、后台点击/输入 | [OrderG-X/dsh-mac-computer-use](https://github.com/OrderG-X/dsh-mac-computer-use) | MIT | 启用 |
| 7 | `dsh-prime-memory` | 0.26.41 | 分层记忆：L0 采集 → L1 原子记忆 → L2 场景 → L3 画像，按步召回注入 | [drscrewdriver/dsh-prime-memory](https://github.com/drscrewdriver/dsh-prime-memory) | MIT | 启用 |
| 8 | `dsh-reference-session-collector` | 0.2.9 | 只读扫描会话日志，投影进 SQLite 做检索面（不写会话文件） | [drscrewdriver/dsh-reference-session-collector](https://github.com/drscrewdriver/dsh-reference-session-collector) | MIT | 启用 |
| 9 | `dshmarket` | 1.73.5 | 可视化插件市场，界面里搜、点一下装 | [dsh-market/dsh-market](https://github.com/dsh-market/dsh-market) | MIT | 启用 |
| 10 | `dsh-soul-md` | 0.10.3 | soul.md 风格人设 + 记忆 | [Scorp1o117/dsh-soul-md](https://github.com/Scorp1o117/dsh-soul-md) | MIT | **装了但停用** |
| 11 | `@deepseek-ai/dsh-base`、`@deepseek-ai/dsh-web-app` | 随 App | 基础能力与 Web 界面 | App 自带 | — | 自带，不用装 |

`dsh-soul-md` 停用原因是人设已迁到 `dsh-prime-memory`；配置还留在
`cordis.patch.yml` 的 `soul-md` 段里，删掉 `disabled: true` 就能恢复。

### 怎么装

```bash
DSH="/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh"

"$DSH" plugin add @nanmicoder/dsh-agent-teams --profile desktop
"$DSH" plugin add @perrylink/dsh-github         --profile desktop
"$DSH" plugin add @xmanrui/dsh-im               --profile desktop
"$DSH" plugin add dsh-browser-cdp               --profile desktop
"$DSH" plugin add dsh-email                     --profile desktop
"$DSH" plugin add dsh-mac-computer-use          --profile desktop
"$DSH" plugin add dsh-prime-memory              --profile desktop
"$DSH" plugin add dsh-reference-session-collector --profile desktop
"$DSH" plugin add dshmarket                     --profile desktop
# 可选（本机已停用）："$DSH" plugin add dsh-soul-md --profile desktop
```

装完要确认两处，缺一个都不会生效：

1. **装载名单**：`~/.dsh/profiles/desktop/package.json` 的 `dsh.profile.bundles` 数组里要有这个包名。
   DSH 是按这个数组逐层组装运行树的（`cordis.yml` 注释：the tree is composed as patches: each bundle in package.json's `dsh.profile.bundles`, then `cordis.patch.yml`）。
2. **配置/开关**：`~/.dsh/profiles/desktop/cordis.patch.yml`，按 `id` 覆盖配置，`disabled: true` 即停用。

也可以不敲命令：装好 `dshmarket` 后，在 DSH 设置里的插件市场搜索安装。

### 装完各自还需要什么

| 插件 | 额外配置 |
|---|---|
| `dsh-email` | 账号与口令（`cordis.patch.yml` 的 `tool-email` 段，明文），或走 Google/Outlook OAuth |
| `@xmanrui/dsh-im` | 各 IM 通道的 token / webhook，在本机 `~/.dsh/integrations/` 下 |
| `dsh-prime-memory` | 数据在 `~/.dsh/memory/`，可指定蒸馏模型；默认跟随主模型 |
| `@perrylink/dsh-github` | GitHub 登录（`gh auth login` 或 token） |
| `dsh-browser-cdp` | 首次启动会拉起自带 Chromium；挂接已开 Chrome 需 CDP 端口 |
| `dsh-mac-computer-use` | macOS 需给 App 系统权限（屏幕录制/辅助功能），不授权时只能截屏不能点 |

---

## 2. 技能（10 个）

都放在 `~/.dsh/skills/<名字>/`，入口文件是 `SKILL.md`。

| 技能 | 版本 | 用途 | 上游 | 本机体积 |
|---|---|---|---|---|
| `archify` | 3.2.0 | 架构/流程/时序/数据流/状态/ERD/类图/时间线等，产出可交互单页 HTML，可导出 PNG/SVG/WebM | [tt-a1i/archify](https://github.com/tt-a1i/archify) | 18M（206 文件，带渲染器） |
| `unslop` | — | 把"AI 味"的界面改成有设计感的界面：间距、配色、图标、文案、排版、动效 | Damir Sokolovsky（CC-BY-4.0） | 1.4M（16 文件） |
| `defuddle` | — | 网页转干净 Markdown，依赖同名 CLI | 未记录 | 4K（1 文件） |
| `knap` | — | 用模板 + JSON/CSV 批量生成 Markdown，依赖同名 CLI | 未记录 | 8K（1 文件） |
| `obsidian-markdown` | — | Obsidian 风格 Markdown：wikilink、embed、callout、属性 | 未记录 | 20K（4 文件） |
| `obsidian-bases` | — | `.base` 文件：视图、筛选、公式、汇总 | 未记录 | 28K（2 文件） |
| `obsidian-cli` | — | 用 Obsidian CLI 读写库、管属性/任务、调插件主题 | 未记录 | 8K（1 文件） |
| `obsidian-headless` | — | 无桌面端时用 `ob` 命令行做同步/发布 | 未记录 | 8K（1 文件） |
| `json-canvas` | — | 生成/编辑 `.canvas` 文件（节点、连线、分组） | 未记录 | 20K（2 文件） |
| `shop-route` | — | 多店最短驾车路线 + 高德 App 可扫的完整路线二维码（本机自用，含 scripts） | 本机自建 | 12K（3 文件） |

"未记录"= 本机是手工放入的，原始下载地址没有留下来；要复现得找用户要原始目录。
`json-canvas` 的 SKILL.md 只引用了画布格式规范 [obsidianmd/jsoncanvas](https://github.com/obsidianmd/jsoncanvas)，不是技能本体的来源。

### 怎么装

技能没有安装命令，是"放进去就生效"：

```bash
mkdir -p ~/.dsh/skills
# 把技能目录整个复制进去，保证 <技能名>/SKILL.md 存在
cp -R /path/to/archify ~/.dsh/skills/
# 检查：每个目录都应有 SKILL.md
ls ~/.dsh/skills/*/SKILL.md
```

优先级（DSH 文档口径）：项目里的 `.dsh/skills/` 或 `.agents/skills/` 会盖过用户级 `~/.dsh/skills/`。
放好后新开一个会话即可看到技能出现在技能清单里。

`unslop` 是 CC-BY-4.0，复制或改写时保留作者署名。

---

## 3. 技能/插件用到的外部命令行工具

装在 `~/.local`（npm 前缀 `/Users/<user>/.local`），软链在 `~/.local/bin`。

| 命令 | 版本 | 装法 | 被谁用 |
|---|---|---|---|
| `defuddle` | 0.19.7 | `npm install -g defuddle` | `defuddle` 技能 |
| `knap` | 0.6.0 | `npm install -g knap` | `knap` 技能 |
| `lark-cli` | 1.0.99 | `npm install -g @larksuite/cli` | `lark-*` 系列技能 |
| `obsidian` | 随 App | 来自 `/Applications/Obsidian.app` | `obsidian-*` 技能 |
| `gh` | 2.103.3 | arm64 压缩包解到 `~/.local/opt/gh-2.103.3/` 后软链 | GitHub 操作 |
| `pnpm` | — | 随 DSH 运行时 | `dsh plugin add` 内部使用 |

`lark-*` 系列（approval、base、calendar、doc、drive、im、mail、sheets 等 20+ 个）是 DSH 自带的技能，不用另装；但它们的实际动作要靠 `lark-cli`。第一次用需要登录：

```bash
lark-cli auth login
lark-cli auth status
```

---

## 4. 校验清单（接手时照着打）

```bash
# 1) 插件：列出版本
python3 -c "import json;d=json.load(open('$HOME/.dsh/profiles/desktop/package.json'));print(d['dependencies'])"

# 2) 装载名单
python3 -c "import json;d=json.load(open('$HOME/.dsh/profiles/desktop/package.json'));print(d['dsh']['profile']['bundles'])"

# 3) 插件实体是否真的落到磁盘
ls ~/.dsh/profiles/desktop/node_modules | wc -l

# 4) 技能
ls ~/.dsh/skills

# 5) 命令行
for c in defuddle knap lark-cli gh; do printf '%s: ' $c; command -v $c || echo MISSING; done
```

---

## 5. 注意事项

- **不要把本机配置搬上来**：`~/.dsh/profiles/desktop/cordis.patch.yml` 里有邮箱账号和明文口令、各家 token，属于本机私密配置。本仓库只记包名和路径。
- 插件装完没生效，先看 `bundles` 数组，再看 `cordis.patch.yml` 里是不是 `disabled: true`，最后看日志 `~/.dsh/profiles/desktop/.plugin-manager/logs/`。
- 版本号会漂：`package.json` 里 `^` 前缀的包（`dsh-prime-memory`、`dsh-reference-session-collector`、`dshmarket`）升级会自己走；要钉死版本就写死版本号。
- DSH 升级后插件可能要重建，重建只跑 `dsh plugin add` 即可，配置在 `cordis.patch.yml` 里不会丢。

---

Last verified: 2026-11-20
