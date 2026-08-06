# obsidian-anki-cards

把学习域里的单元学习笔记提炼成 [flashcards-obsidian](https://github.com/reuseman/flashcards-obsidian) 插件格式的 Anki 卡片，写入该域自己的中间文件，在 Obsidian 里手动同步进 Anki。

这是给支持 [Agent Skills](https://agentskills.io) 规范的 AI 编码工具（**Claude Code、OpenCode、Codex、Cursor** 等 76+ 种）用的 skill。

## 它解决什么问题

学习笔记最大的坑是"读的时候都懂，三个月后全忘"。Anki 的间隔重复能解决遗忘，但有一个更高的门槛——**把一段笔记提炼成一张好卡片**。这个 skill 把三件事固化成一套可复用的流程：

1. **插件语法**：flashcards-obsidian 的语法（`#card`、行内 `::`、cloze 挖空、context-aware 路径、`^id` 追踪机制）细节多、一个空行就碎卡。skill 内置完整语法参考，不用每次重新查。
2. **提炼判断标准**："这段话删掉后，三个月后的我还能想起这一课吗？"——能想起来的才值得做卡，避免把笔记逐字抄成卡片。
3. **问法设计铁律**：好问法带"场景钩子"（用具体处境起头），禁指代、禁题面泄答案、禁逆向提问、禁开放式问法。这四条决定了卡片复习时能不能真正考住你。

**跨域通用**：同时服务多个采用"单元学习笔记"模式的学习域（如哲学启蒙、产品方法论……），只在开头做一次"这次是哪个域"的定位，不为每个域各建一个 skill。

**边界**：只负责生成卡片文本、写入中间文件；真正同步到 Anki 的那一步（`Ctrl+P → Flashcards: generate for the current file`）由用户在 Obsidian 里手动执行。

## 依赖

- [Obsidian](https://obsidian.md/)
- [flashcards-obsidian](https://github.com/reuseman/flashcards-obsidian) 插件
- [Anki](https://apps.ankiweb.net/) 桌面版 + [AnkiConnect](https://ankiweb.net/shared/info/2055492159) 插件
- 任一支持 Agent Skills 的 AI 编码工具（Claude Code、OpenCode、Codex、Cursor……）

## 安装

### 一键安装（推荐）

用 [npx skills](https://github.com/vercel-labs/skills)（Vercel 官方 CLI，支持 76+ 种 agent）安装，无需先装任何东西：

```bash
# 自动检测你已装的工具，一键装到全局（所有项目可用）
npx skills add ShanwaPine/obsidian-anki-cards -g

# 或精确指定要装到哪些工具
npx skills add ShanwaPine/obsidian-anki-cards -g -a claude-code -a opencode -a codex

# 只装到当前项目（不带 -g）：
npx skills add ShanwaPine/obsidian-anki-cards
```

装了 bun 或 pnpm 的，把 `npx` 换成 `bunx` / `pnpm dlx` 即可。装完新开一个会话即可自动发现。

### 手动安装（备选）

把整个仓库克隆到想用的工具的 skills 目录：

```bash
git clone https://github.com/ShanwaPine/obsidian-anki-cards ~/.claude/skills/obsidian-anki-cards   # Claude Code
git clone https://github.com/ShanwaPine/obsidian-anki-cards ~/.config/opencode/skills/obsidian-anki-cards  # OpenCode
git clone https://github.com/ShanwaPine/obsidian-anki-cards ~/.codex/skills/obsidian-anki-cards  # Codex
```

### 更新与卸载

```bash
npx skills update obsidian-anki-cards   # 更新
npx skills remove obsidian-anki-cards   # 卸载
```

## 使用

对 AI 助手说"做闪卡""把单元3做成卡片""同步到Anki"这类话就会触发；也可以直接说"用 obsidian-anki-cards"。核心工作流：

1. **定位域**：确定要处理哪个学习域（用户明说 / 当前目录在域下 / 询问用户）
2. **确认单元**：读取中间文件已有列表，跳过已做过的单元
3. **提炼知识点**：按内置判断标准挑值得做成卡片的内容，通常一个单元 4~8 张
4. **写问法**：按四条铁律写，每张卡回看一遍
5. **选格式**：`#card`（多行答案）/ `::`（一句话）/ `#card-reverse`（双向）/ cloze（原句挖空）
6. **写入中间文件**：追加到该域的 `XX学习域-Anki卡片.md`，绝不覆盖
7. **全文自检**：通读整份文件，检查空行、标题写法、`^id` 规则
8. **用户手动同步**：在 Obsidian 里跑 `Ctrl+P → Flashcards: generate for the current file`

## 目录结构

```
obsidian-anki-cards/
├── SKILL.md                    # skill 定义与完整工作流
├── references/
│   └── flashcards-obsidian-语法.md    # 插件语法完整参考（含踩坑记录）
└── README.md
```

## 设计背景

skill 里的规则大多来自真实使用中踩过的坑，值得说明为什么这么严格：

- **`#card` 答案内不能有空行**：答案到第一个空行就截止，空行后的内容会被甩出卡片、变成孤儿文本，下次同步长出新 ID，一张卡裂成两半。
- **卡片问题不用 markdown 标题写法**：`### 问题 #card` 会变成文档里真实的标题，插件的 context-aware 逻辑会把紧跟其后的所有卡片都拼到这张"标题卡"下面，正面越叠越长。
- **`^id` 不删不生成**：ID 只能由插件在真实同步后写入，人工删了会重复建卡、人工编了会去更新一张不存在的笔记。
- **cloze 不能跟 `::` 混用**：两套卡片机制写在同一行会生成两张互相打架的卡。

## License

[MIT](LICENSE)