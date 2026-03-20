# My Skills

JustinChoi 的个人 Skills 集合，遵循 [Anthropic Agent Skills 标准](https://agentskills.io)。

每个 Skill 是一个自包含的目录，包含 `SKILL.md` 指令文件和可选的资源文件。浏览下方技能列表了解详情，或直接安装使用。

## Skills 一览

### [write-blog](./write-blog)

以博主 JustinChoi 的个人写作风格撰写博客文章，模拟其语气、结构和表达习惯。

- **调用方式：** `/zcf:write-blog <文章主题> [--type <类型>]`
- **支持类型：** 技术教程 (`tech`)、经验分享 (`experience`)、随笔 (`essay`)
- **核心特点：** 实用型学习笔记风格，正式与随意结合，大量使用反引号标记技术术语

```
write-blog/
└── SKILL.md
```

---

### [code-factory](./code-factory)

Claude 与 Codex 双 AI 协作开发工作流。Claude 负责需求分析和架构设计，Codex 负责评审、代码实现和测试。

- **调用方式：** `/code-factory <需求描述>` 或 `/code-factory @docs/requirement.md`
- **核心角色：** Claude（架构师）+ Codex（工程师）
- **执行流程：** 环境检查 → 需求分析 → Codex 评审 → 多轮讨论 → 并行实现 → 交叉 Review → 最终报告
- **核心特点：** 多轮讨论收敛、多 Codex 并行实现、互相 Review 全闭环

```
code-factory/
├── SKILL.md
├── assets/            # 设计/计划/报告/评审模板
└── references/        # Codex 调用提示词
```

## 安装使用

在 Claude Code 中运行：

```bash
# 注册 marketplace
claude plugin marketplace add justinchoi929/my-skills

# 安装插件
claude plugin install my-skills
```

安装后通过 `/skill-name` 调用。更新插件：

```bash
claude plugin update my-skills
```

## 创建新 Skill

1. 新建目录，添加 `SKILL.md`（包含 YAML frontmatter + Markdown 指令）：

```markdown
---
name: my-skill-name
description: '描述这个 Skill 做什么以及什么时候用'
---

# Skill 标题

指令正文...
```

2. 在 `.claude-plugin/marketplace.json` 的 `skills` 数组中注册路径：

```json
"skills": [
  "./write-blog",
  "./code-factory",
  "./my-new-skill"
]
```

3. 提交推送后运行 `claude plugin update my-skills` 更新。

## 目录结构

```
my-skills/
├── .claude-plugin/
│   └── marketplace.json       # 插件清单
├── write-blog/                # 博客写作技能
│   └── SKILL.md
├── code-factory/              # 双AI协作开发技能
│   ├── SKILL.md
│   ├── assets/
│   └── references/
└── README.md
```

每个 Skill 目录支持以下可选子目录：

| 目录 | 用途 |
|------|------|
| `assets/` | 模板、配置文件等资源 |
| `references/` | 参考文档 |
| `scripts/` | 可执行脚本 |
| `examples/` | 使用示例 |

更多信息参见 [Agent Skills 规范](https://agentskills.io) 和 [创建自定义 Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)。
