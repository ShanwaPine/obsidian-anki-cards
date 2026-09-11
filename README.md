# obsidian-anki-cards

> **版本**：`v2.0.0`（适配 flashcards-obsidian v2.0.1+）

把学习域里的单元学习笔记提炼成 [flashcards-obsidian](https://github.com/reuseman/flashcards-obsidian) 插件格式的 Anki 卡片，写入该域自己的中间文件，在 Obsidian 里手动同步进 Anki。

这是给支持 [Agent Skills](https://agentskills.io) 规范的 AI 编码工具（**Claude Code、OpenCode、Codex、Cursor** 等 76+ 种）用的 skill。

> **语言**：skill 正文与示例均为中文，面向中文笔记使用者。方法论本身与语言无关，英文用户可自行翻译 `SKILL.md`。English summary at the bottom.

## 它解决什么问题

学习笔记最大的坑是"读的时候都懂，三个月后全忘"。Anki 的间隔重复能解决遗忘，但有一个更高的门槛——**把一段笔记提炼成一张好卡片**。这个 skill 把三件事固化成一套可复用的流程：

1. **插件语法**：flashcards-obsidian 的语法（`#card`、fenced 块 ` ```flashcard `、行内 `::`、cloze 挖空、context-aware 路径、`^id` 追踪机制）细节多、AST 节点与空行规则严。skill 内置完整语法参考和四条铁律，不用每次重新查。
2. **提炼判断标准**："这段话删掉后，三个月后的我还能想起这一课吗？"——能想起来的才值得做卡，避免把笔记逐字抄成卡片。
3. **问法设计铁律**：好问法带"场景钩子"（用具体处境起头），禁指代、禁题面泄答案、禁逆向提问、禁开放式问法。这四条决定了卡片复习时能不能真正考住你。

**跨域通用**：同时服务多个采用"单元学习笔记"模式的学习域，只在开头做一次"这次是哪个域"的定位，不为每个域各建一个 skill。目录名、文件命名一概不假设，skill 现场问、现场读。

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

把整个仓库克隆到对应工具的 skills 目录（各工具的实际路径以其官方文档为准）：

```bash
git clone https://github.com/ShanwaPine/obsidian-anki-cards ~/.claude/skills/obsidian-anki-cards            # Claude Code
git clone https://github.com/ShanwaPine/obsidian-anki-cards ~/.config/opencode/skills/obsidian-anki-cards   # OpenCode
git clone https://github.com/ShanwaPine/obsidian-anki-cards ~/.codex/skills/obsidian-anki-cards             # Codex
```

### 更新与卸载

**scope 要跟安装时一致**——用 `-g` 装到全局的，更新/卸载也要带 `-g`，否则 CLI 会去当前项目目录里找、报告找不到：

```bash
npx skills update obsidian-anki-cards -g   # 更新（全局）
npx skills remove obsidian-anki-cards -g   # 卸载（全局）
```

装在项目里的（安装时没带 `-g`）去掉 `-g` 即可。

## 使用

对 AI 助手说"做闪卡""把单元3做成卡片""同步到Anki"这类话就会触发；也可以直接说"用 obsidian-anki-cards"。核心工作流：

1. **定位域**：确定要处理哪个学习域（用户明说 / 当前目录在域下 / 全库只有一个 / 询问用户）；中间文件不存在时会先问你卡组名，再按模板新建
2. **确认单元**：读取中间文件已有列表，跳过已做过的单元
3. **提炼知识点**：按判断标准挑值得做成卡片的内容，通常一个单元 4~8 张
4. **写问法**：按四条禁忌写，每张卡回看一遍
5. **选格式**：fenced 块（多行/复杂列表） / `#card`（紧邻列表或单段） / `::`（一句话） / `#card-reverse`（双向） / cloze（原句挖空）
6. **写入中间文件**：追加到该域的 `XX学习域-Anki卡片.md`，绝不覆盖
7. **全文自检**：通读整份文件，按四条铁律 + 四条禁忌逐张过
8. **用户手动同步**：在 Obsidian 里跑 `Ctrl+P → Flashcards: generate for the current file`

## 目录结构

```
obsidian-anki-cards/
├── SKILL.md                              # skill 定义与完整工作流
├── references/
│   └── flashcards-obsidian-语法.md       # 插件语法完整参考（含踩坑记录）
├── examples/
│   └── 示例-Anki卡片.md                  # 一份可直接照抄的中间文件范例
├── README.md
└── LICENSE
```

## 示例文件导读

`examples/示例-Anki卡片.md` 是一份格式完全正确的中间文件（示例域为"经济学入门"），可以直接当模板照抄。它示范的东西：

| 位置 | 示范什么 |
|------|---------|
| frontmatter + 加粗域名 + 修改前必读 | 新建中间文件的标准头部（**注意域名没有用 `# ` 一级标题**，否则会污染每张卡的正面） |
| 单元3 第 1 张（`#card`） | 行尾 `#card` + 紧邻纯列表：分点列表 + 关键词加粗 + 末尾一句点睛，**全块无空行** |
| 单元3 第 2 张（行内 `::`） | 一句话问完答完的简单事实，最省地方 |
| 单元3 第 3 张（`#card-reverse`） | 答案首句就是单一术语定义，才配得上反转卡 |
| 单元3 第 4 张（cloze） | 想记准确措辞的金句，单空用 `==挖空==`，**整行不带 `::`**（多空用 `{1:词}` 绑定） |
| 单元4 两张卡（fenced 块） | v2 推荐的 ` ```flashcard ` 代码块写法：多单元区块追加，复杂多行列表结构清晰、互不干扰 |
| 全文没有任何 `^id` 行 | ID 只能由插件同步后自动写入，人工绝不手动补 |

所有问题都遵守"场景钩子"原则：先给具体处境，再问答案，题面里不出现答案关键词。

> ⚠️ 如果你把这个 skill 装在**项目级**目录（安装时不带 `-g`）而该项目正好是你的 Obsidian 库，这份示例文件也会出现在库里。它本身是安全的（没有伪造 ID），但如果你对它跑了同步命令，Anki 里会多出一个「经济学入门」卡组。不想要就删掉这个文件，或者改用全局安装。

## 设计背景

skill 里的规则大多来自真实使用中踩过的坑，值得说明为什么这么严格：

- **`#card` 答案在 v2 下的节点规则**：v2 会严格按 AST 顶层节点收集答案。行尾 `#card` 紧邻纯列表是合法的；但 marker 后面同段写了引言文字再接列表，列表会被甩出卡片；复杂多行推荐使用 fenced 代码块（` ```flashcard `）。
- **卡片问题不用 markdown 标题写法**：`### 问题 #card` 会变成文档里真实的标题，插件的 context-aware 逻辑会把紧跟其后的所有卡片都拼到这张"标题卡"下面，正面越叠越长。
- **`^id` 不删不生成**：ID 只能由插件在真实同步后写入（v2 为 `^q-xxxx`），人工删了会重复建卡、人工编了会去更新一张不存在的笔记。
- **cloze 规则严谨化**：挖空绝不能跟行内 `::` 写在同一行（两套机制冲突）；在 v2 下单空用 `==词==`，一句话挖多个词必须用 `{1:词}` 复用编号绑成一张卡，避免 Anki 不自动补卡导致明文泄露。

## English

An [Agent Skill](https://agentskills.io) that turns study notes into [flashcards-obsidian](https://github.com/reuseman/flashcards-obsidian) cards for Anki. It encodes three things that are easy to get wrong: the plugin's exact v2 syntax (AST node rules, fenced blocks, multi-cloze binding), a filter for what actually deserves a card, and four hard rules for writing prompts that still work six months later.

The skill only writes the card file — you run `Ctrl+P → Flashcards: generate for the current file` in Obsidian yourself.

Install: `npx skills add ShanwaPine/obsidian-anki-cards -g`

**Note:** `SKILL.md` and the example file are written in Chinese. The methodology is language-agnostic; translate `SKILL.md` if you want English cards.

## License

[MIT](LICENSE)
