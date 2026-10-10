# make-knowledge-cards

把一段文章（或本地 Markdown/TXT 文件）转换成 5~8 张知识卡片，每张卡片只讲一个知识点。

---

## 1. 项目解决什么问题

长文章、论文、教程往往信息密度高，读完却很难记住，复习时又得重新翻原文。手动整理成卡片很耗时，而常见的自动摘要工具容易把多个知识点揉在一起、重复啰嗦，甚至会“脑补”原文没有的内容。

`make-knowledge-cards` 把一篇完整文章提炼成 5~8 张结构化知识卡片，帮助用户快速复习与记忆。它的取舍原则是：

- 只提取**真正重要**的知识点；
- **一张卡片只讲一个知识点**，不把多个概念糅在一张；
- 自动**删除重复内容**；
- **不编造**原文没有的信息（数字、结论、例子、引用一律以原文为准）；
- 信息不足时**不强行凑数量**（可少于 5 张）。

## 2. 主要功能

- **输入**：用户直接粘贴的文本，或本地 `.md` / `.markdown` / `.txt` 文件。
- **输出**：5~8 张知识卡片，每张包含四个固定字段：
  - **标题**：概括该知识点的短语
  - **核心知识**：最凝练的一句结论/概念
  - **简明解释**：在核心知识基础上补充细节的 2~4 句说明
  - **例子 / 自测问题**：优先用原文例子，否则给一个可自测的问题
- **质量保障**：内置“识别知识点 → 去重合并 → 定数量 → 生成卡片 → 自检”五步工作流，并强制自检（一卡一点、字段完整、不编造、无重复、数量合规）。
- **语言跟随原文**：中文原文输出中文卡片。
- **数量规则**：默认 5~8 张；原文重要知识点不足 5 个时按实际数量输出并说明原因；知识点过多时只保留最重要的 5~8 个。
- **可选写文件**：用户要求时，可将卡片写入本地 `.md` 文件。
- 纯提示词（prompt-only）实现，**不含脚本、无第三方依赖**。

## 3. 安装方法

本技能是纯提示词的 Agent Skill，**无需安装依赖，也无需构建**，把技能文件夹放到技能目录即可。

**方式一：用户级（推荐，所有项目可用）**

把 `make-knowledge-cards` 文件夹复制到用户技能目录：

```bash
# macOS / Linux
cp -r skills/make-knowledge-cards ~/.workbuddy/skills/

# Windows (Git Bash)
cp -r skills/make-knowledge-cards "$USERPROFILE/.workbuddy/skills/"
```

**方式二：项目级（仅当前项目可用）**

把 `make-knowledge-cards` 文件夹复制到目标项目的技能目录：

```bash
cp -r skills/make-knowledge-cards <目标项目>/.workbuddy/skills/
```

放置后的目录结构应为：

```
<技能目录>/make-knowledge-cards/
├── SKILL.md            # 技能主文件（YAML frontmatter + 工作流 + 质量规则）
└── agents/
    └── openai.yaml     # 界面元数据（display_name / short_description / default_prompt）
```

> 复制完成后，在 WorkBuddy 中开启一个新会话即可发现该技能；`agents/openai.yaml` 为可选的界面元数据，缺失也不影响技能运行。

## 4. 使用方法

技能通过“知识卡片 / 学习卡片 / 卡片笔记 / 抽卡 / 把这段内容做成卡片”等意图触发。两种典型用法：

**方式一：直接粘贴文本**

> 使用 make-knowledge-cards，把下面这段内容做成知识卡片：〈粘贴你的文章〉

**方式二：指定本地文件**

> 使用 make-knowledge-cards，把 `docs/xxx.md` 转成知识卡片

说明：

- 支持的文件类型为 `.md` / `.markdown` / `.txt`；相对路径基于当前工作目录解析。
- 可附加要求，例如“做成 6 张”“输出到 `cards.md`”。若指定数量超出原文可支撑范围，会先说明再按实际数量输出。
- 未指定输出位置时，卡片默认以 Markdown 直接在对话中返回。

## 5. 输入输出示例

**输入**（节选，来自 `tests/fixtures/01-technical-http-cache.md`）：

> no-cache 比较容易被误解：它并不表示“不缓存”，而是表示“每次使用前必须向源服务器验证”。

**输出**：

```
### 卡片 3：no-cache 不等于“不缓存”
- **核心知识**：no-cache 表示“每次使用前必须向源服务器验证”，而不是禁止缓存。
- **简明解释**：no-cache 容易被误解为不缓存；实际上它允许缓存，只要求每次使用前先验证。真正表示“任何位置都不得缓存”的指令是 no-store。
- **例子 / 自测问题**：想禁止任何位置缓存该响应，应使用哪个指令？（no-store）
```

完整的输入/输出对照见 `tests/`，包含 4 类不同文章的实测结果：

| 输入文件（`tests/fixtures/`） | 类型 | 输出文件（`tests/outputs/`） | 卡片数 |
| --- | --- | --- | --- |
| `01-technical-http-cache.md` | 技术 | `01-technical-http-cache.cards.md` | 6 |
| `02-science-sleep-memory.md` | 科普 | `02-science-sleep-memory.cards.md` | 6 |
| `03-howto-pomodoro.md` | 方法步骤 | `03-howto-pomodoro.cards.md` | 6 |
| `04-short-insufficient.md` | 短文本（信息不足） | `04-short-insufficient.cards.md` | 2（未凑数） |

## 目录结构

```
.
├── README.md
├── skills/
│   └── make-knowledge-cards/
│       ├── SKILL.md
│       └── agents/
│           └── openai.yaml
└── tests/
    ├── fixtures/     # 4 篇不同类型的测试输入
    └── outputs/      # 对应的知识卡片输出
```

## 暂不支持

- 网页抓取（从 URL 抓取正文）——请先粘贴正文文本
- PDF / 扫描件 / 图片的文字提取——请提供 Markdown/TXT 或直接粘贴
- Anki（`.apkg` / Anki 专用 CSV）导出
- 图形界面（GUI / 网页应用）

## 测试与校验

测试用例位于 `tests/`，按 `tests/fixtures/<输入>` 与 `tests/outputs/<输出>` 组织。

技能结构使用 WorkBuddy 官方 skill-creator 提供的 `quick_validate.py` 校验，通过时输出：

```
Skill is valid!
```
