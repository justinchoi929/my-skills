# My Skills

个人 Skills 集合，遵循 [Skills 格式](https://github.com/anthropics/skills) 开放标准。

## Skills 格式规范

Skills 格式是一种用于封装 AI 执行特定任务能力的标准化结构，由 Anthropic 发起并已成为开放标准，被 Claude Code、Cursor、VS Code 等主流工具支持。

### 目录结构

每个 Skill 是一个独立目录，至少包含一个 `SKILL.md` 文件：

```
my-skills/                     # 技能集合根目录
├── skill-a/                   # 单个技能（推荐 kebab-case 命名）
│   ├── SKILL.md               # 必需：包含元数据和指令
│   ├── scripts/               # 可选：可执行脚本（如 .py、.sh）
│   ├── references/            # 可选：参考文档
│   ├── assets/                # 可选：模板、配置文件等资源
│   └── examples/              # 可选：使用示例
├── skill-b/
│   └── SKILL.md
└── README.md
```

### SKILL.md 文件格式

`SKILL.md` 是 Skill 的核心文件，由 **YAML frontmatter（元数据）** 和 **Markdown 正文（指令）** 两部分组成：

```markdown
---
name: my-skill-name
description: '清晰描述这个 Skill 做什么以及什么时候使用它'
---

# Skill 标题

正文内容：详细的指令、上下文、执行流程等。

## Examples
- 示例 1
- 示例 2

## Guidelines
- 规则 1
- 规则 2
```

#### Frontmatter 字段说明

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | 是 | 技能唯一标识符，小写 + 连字符（kebab-case） |
| `description` | 是 | 完整描述该技能做什么、什么时候用，会显示在工具列表中 |

#### 正文内容建议

- **指令**：AI 需要遵循的具体规则和要求
- **示例（Examples）**：使用场景和用法示例
- **规范（Guidelines）**：执行约束、风格要求等
- **执行流程**：分步骤说明 Skill 的工作流程

### 可选目录说明

| 目录 | 用途 | 示例内容 |
|------|------|----------|
| `scripts/` | 可执行脚本 | `.py`、`.sh`、`.js` 脚本文件 |
| `references/` | 参考文档 | API 文档、规范说明、风格指南 |
| `assets/` | 静态资源 | 模板文件、配置文件、图片素材 |
| `examples/` | 使用示例 | 输入输出样例、示范文件 |

### 命名规范

- 目录名使用 **kebab-case**（如 `write-blog`、`code-review`）
- 核心文件固定为 `SKILL.md`（全大写）
- 文件和目录名应清晰表达 Skill 的功能

## 当前 Skills 列表

| Skill | 说明 |
|-------|------|
| [write-blog](./write-blog/) | 以博主 JustinChoi 的个人写作风格撰写博客文章 |
