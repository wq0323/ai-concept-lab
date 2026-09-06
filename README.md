# ai-concept-lab

> AI 概念学习资料生成 Skill 与示例。

本仓库包含一个项目级 Skill `concept-explainer`，以及用它生成的若干概念学习资料（HTML）。

## 仓库用途

把"理解一个陌生 AI / 技术概念"这件事沉淀成可复用的工作流：

1. **Skill 模板**：`concept-explainer` 给出统一的五段式资料结构（个人解释 / 核心机制 / 应用场景 / 混淆与边界 / 资料来源），不只为某一个概念服务。
2. **示例资料**：调用该 Skill 生成的三份概念学习资料（Agent、大模型的上下文、Skill），以及一份概念关系说明。
3. **可核查**：每份资料都附资料来源链接，并标记可信度（官方 / 学术 / 工程 / 一般）。

## 目录结构

```
ai-concept-lab/
├── .workbuddy/
│   └── skills/
│       └── concept-explainer/
│           └── SKILL.md        # 项目级 Skill 入口
├── learning-materials/
│   ├── agent.html              # 概念：Agent
│   ├── llm-context.html        # 概念：大模型的上下文
│   ├── skill.html              # 概念：Skill
│   └── concept-relationship.html  # 三概念关系说明（含 Mermaid 图）
├── README.md
└── .gitignore
```

## Skill：concept-explainer

**存放路径**：`.workbuddy/skills/concept-explainer/SKILL.md`

**当前版本**：v1.1（8 段式；v1.0 为 5 段式，已被 v1.1 取代）。

**作用**：当被加载时，按统一模板为任意 AI / 技术概念生成体系化、可自学、可自测的个人学习资料 HTML。

**包含的元数据**：

- `name`: `concept-explainer`
- `description`: 触发条件描述——见 SKILL.md 头部

**正文包含**：

- 设计意图（为什么要做这件事 / 卡点分析）
- 适用场景（含不适用场景）
- 输入信息（必填 / 可选）
- 生成步骤（11 步：概念锁定 → 学习目标 → 个人解释 → 核心问题 → 结构化解释 → 应用案例 → 概念辨析 → 自测问题 → 参考来源 → 自检 → 写入文件）
- 输出结构（HTML **8 段式**，锚点 ID 固定：`goals` / `personal` / `core-questions` / `mechanism` / `scenario` / `contrast` / `quiz` / `sources`）
- 参考来源硬约束
- 自检清单（11 条）
- 工作约定与版本

## 如何在 WorkBuddy 中调用

### 方式 A：通过 Skill 工具自动加载

把本仓库放在工作区下任意路径，并在对话中说：

> "用 `concept-explainer` Skill 帮我整理一下 RAG 这个概念的学习资料。"

WorkBuddy 会扫描 `.workbuddy/skills/` 下的 SKILL.md，加载元数据与正文，按其指示生成 HTML。

### 方式 B：手动加载

直接在对话中引用 Skill 路径：

> "请按 `.workbuddy/skills/concept-explainer/SKILL.md` 的规范，生成 Self-Attention 的学习资料。"

### 方式 C：复制 Skill 到用户级目录

如果想跨项目复用，把 `.workbuddy/skills/concept-explainer` 整目录拷到 `~/.workbuddy/skills/` 下即可。

## 已生成的学习资料

| 资料 | 文件 | 调用方式 |
|------|------|----------|
| Agent | [`learning-materials/agent.html`](./learning-materials/agent.html) | `concept-explainer` Skill |
| 大模型的上下文 | [`learning-materials/llm-context.html`](./learning-materials/llm-context.html) | `concept-explainer` Skill |
| Skill | [`learning-materials/skill.html`](./learning-materials/skill.html) | `concept-explainer` Skill |
| 三概念关系 | [`learning-materials/concept-relationship.html`](./learning-materials/concept-relationship.html) | 手工整理 + Mermaid 图 |

## 人工核查与修改记录

AI 生成 → 人工核查，并非"全自动可信"。本仓库的人工核查点：

- **链接核查**：所有资料来源链接均为人工或 AI 检索所得的可点开 URL；论文号（arXiv ID）已核对（ReAct 2210.03629 / Attention 1706.03762 / HuggingGPT 2303.17580）。
- **风格统一**：四份 HTML 共享同一份 CSS（卡片化、浅色主题、面包屑），保证视觉一致。
- **第一人称**：每份资料的"个人解释"段都已调整为亲历者口吻，去掉了"本文将..."这种论文腔。
- **边界与混淆**：每份资料的"概念辨析"段都列出了 3–5 条与相邻概念的边界，避免概念被混用。
- **概念解释非搬运**：每份 HTML 的"个人解释 / 核心问题 / 应用案例 / 自测问题"段均为本概念定制撰写，未从对话历史中整段照搬其他概念的解释。
- **Mermaid 图渲染**：`concept-relationship.html` 引用的 Mermaid CDN 是公开免费的 jsdelivr 镜像；如离线使用可下载 mermaid.min.js 到本地。
- **v1.1 升级记录**：将 Skill 从 5 段式升级为 8 段式（新增学习目标 / 核心问题 / 自测问题三段），同步刷新三份单概念 HTML 与 README。Skill 头部 `description`、自检清单（11 条）、生成步骤（11 步）也已同步更新。
- **每页核查记录**：每份 HTML 底部 `<footer id="meta">` 都列出了本次人工核查的具体项目，链接 / 论文号 / 章节自检一一对应。

## 安全与隐私

- 本仓库不包含任何 API Key、密码、个人隐私信息。
- `.gitignore` 已排除常见的本地环境文件与敏感文件。
- 之前用于 GitHub API 操作的 Personal Access Token 仅在创建/恢复仓库时使用过一次，已建议用户撤销。

## 维护

- 欢迎通过 Issue 或 PR 补充更多概念的资料（按 `concept-explainer` Skill 模板生成即可）。
- Skill 的版本演进记录在 `SKILL.md` 末尾的"版本"小节。