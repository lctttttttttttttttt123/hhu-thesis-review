# hhu-thesis-review

一个 Claude Agent Skill，对河海大学博士学位论文做写作审查，把问题批成一份**可逐行落实的 Word 表格**。

它审查论文的五个固定部分——**前言、摘要、问题的提出（绪论 1.1）、结论（结论与展望）、参考文献**——逐块给出带严重度分级（硬伤/建议/可选）的修改意见，最终交付一个 .docx 文档：顶部是无法定位到单句的跨段问题，下面是一张四列表（原文定位 / 问题 / 修改后 / 修改理由），用户照着就能改。

审稿规则归纳自多篇河海博士论文真实样本。这个 skill 只看**写作质量与著录格式**，不评判创新点的学科价值、方法对错或论证是否严密——那些依赖具体领域知识，超出它的范围。

## 能做什么

装上之后，把论文（或其中某一部分）丢给 Claude，说"帮我看看摘要""按 GB/T 7714 查一下参考文献""审一下我的前言和结论"，它会：

- 先查**跨部分红线**，最典型的是同一段研究背景在前言/摘要/结论三处逐句复用——河海博士论文头号通病，置顶标为硬伤
- 按每一块的专门清单逐条审：前言的创新点要素与拔高词、摘要的结论数据化与关键词、问题提出的"名实相符"与问题—内容—结论闭环、结论的过程复述与展望空泛、参考文献的著录格式与正文交叉校验
- 把结果生成一份 Word 文档供下载

几个有意为之的设计：

- **结论数据化区分类型**：实测/实验类结论要求给定量结果，方法构建/机理类只查是不是过程复述，不一刀切要求数字
- **拔高词降级**：单个有数据支撑的"首次"不报，只在缺证据或同段密集（三个以上）时提示，避免误伤
- **单块聚焦**：用户只审某一块时，跨段红线压成一句话脚注，不喧宾夺主
- **数字防呆**：示范句里的定量结果一律用占位符（X%、Δ天），并在文档显著位置提示"数字为示意，请替换为你的实测值"，防止用户误抄
- **参考文献交叉校验如实声明限度**：拿不到原始排版时，"引而未列/列而未引"只能尽力而为，会明确告知并建议用 Word 自查
- **范例合规**：反例只摘极短脱标识片段，正例一律按规则现场生成，不摘真实论文

## 安装

### Claude（网页版 / 桌面 / 移动端）

需要 Pro、Max、Team 或 Enterprise 订阅，并在 Settings → Capabilities 中开启 Code execution。

1. 从本仓库 [Releases](../../releases) 下载 `hhu-thesis-review.skill`（或自行把 `hhu-thesis-review/` 文件夹压缩成 zip，文件夹需位于压缩包根目录）
2. 打开 claude.ai → Settings → Capabilities → Skills（部分版本路径为 Customize → Skills）
3. 点击 "+" 上传，开启开关即可

详细说明见 [Anthropic 官方文档](https://support.claude.com/en/articles/12512180-use-skills-in-claude)。

### Claude Code

把 `hhu-thesis-review/` 文件夹放进项目的 `.claude/skills/`，或放进 `~/.claude/skills/` 供所有项目使用。

## 使用示例

> 这是我博士论文的摘要，帮我按河海大学的要求看看哪里需要改。

> 帮我按 GB/T 7714—2015 核对一下参考文献格式有没有问题。

> 我把前言、摘要、结论放在一个文件里了，帮我审一下这三部分。

第三种会触发跨段红线检查（背景模板三处复用）。审完后 Claude 会生成一份 .docx 给你下载。

## 仓库结构

```
hhu-thesis-review/
├── SKILL.md                        # 工作流程、跨部分红线、严重度分级、输出格式
├── references/
│   ├── section-checklists.md       # 前言/摘要/问题的提出/结论 四块审查清单
│   ├── gb-t-7714.md                # 参考文献清单（GB/T 7714—2015 顺序编码制）
│   └── docx-output.md              # 审稿 Word 的 JSON spec 结构与字段规则
└── scripts/
    └── build_review_docx.py        # 由 spec.json 生成审稿 .docx（python-docx）
```

## 配套

本 skill 负责审写作。如果还需要按河海格式把论文**排成合规的 Word 文档**（封面、页眉页脚、图表编号、固定行距等），见配套的 [hhu-thesis-docx](https://github.com/lctttttttttttttttt123/hhu-thesis-docx)。一个审内容，一个排格式。

## 注意事项

审稿规则归纳自有限的真实样本，反映的是河海博士论文的常见体例与通病，不能替代导师意见和研究生院的最新规定。各学院对图表编号、参考文献等细节可能有自己的要求，以学院和研究生院的正式规定为准。

skill 对参考文献的交叉校验受限于能否从文本可靠提取引用序号，结论会如实声明限度，请配合 Word 的交叉引用功能自查。

## License

本仓库内容（SKILL.md、references、scripts）以 [MIT License](LICENSE) 发布。

---

## English

A Claude Agent Skill that reviews the writing of Hohai University doctoral dissertations and delivers the findings as a row-by-row actionable Word table. It reviews five fixed sections — foreword, abstract, problem statement (intro §1.1), conclusion, and references — giving severity-tagged edits (critical / suggested / optional) and producing a .docx whose top holds cross-section issues that can't be pinned to one sentence, followed by a four-column table (original sentence / problem / revised / rationale). Rules were distilled from real Hohai dissertation samples; it judges writing quality and citation format only, not the scholarly merit of contributions or the soundness of methods. Notable behaviors: it treats reuse of the same background paragraph across foreword/abstract/conclusion as the top "red-line" defect; requires quantitative results for empirical conclusions but not for method/mechanism conclusions; downgrades buzzword detection to avoid false positives on a single evidence-backed "first"; uses placeholder numbers in examples to prevent users copying fabricated figures; and is honest about the limits of reference cross-checking when it only has extracted text. Install by uploading the zipped `hhu-thesis-review/` folder under Settings → Capabilities → Skills on claude.ai, or drop the folder into `~/.claude/skills/` for Claude Code. A companion skill, hhu-thesis-docx, handles formatting the dissertation into a compliant Word document. Not endorsed by Hohai University; defer to your school's official requirements.
