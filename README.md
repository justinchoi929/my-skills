# My Skills

个人 Skills 集合，遵循 [Anthropic Skills 格式](https://github.com/anthropics/skills) 开放标准。

## 安装使用

```bash
# 1. 注册 marketplace
claude plugin marketplace add justinchoi929/my-skills

# 2. 安装插件
claude plugin install my-skills

# 3. 重启 Claude Code 后，通过 /skill-name 调用
```

更新已安装的插件：

```bash
claude plugin update my-skills
```

## Skills 格式规范

### 目录结构

每个 Skill 是一个独立目录，至少包含一个 `SKILL.md` 文件：

```
my-skills/                     # 技能集合根目录
├── .claude-plugin/
│   └── marketplace.json       # 插件清单（Claude Code Plugin 注册）
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

由 **YAML frontmatter（元数据）** 和 **Markdown 正文（指令）** 两部分组成：

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

#### Frontmatter 字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | 是 | 技能唯一标识符，小写 + 连字符（kebab-case），即 `/` 后的调用名 |
| `description` | 是 | 完整描述该技能做什么、什么时候用 |

#### 命名规范

- 目录名使用 **kebab-case**（如 `write-blog`、`code-review`）
- 核心文件固定为 `SKILL.md`（全大写）

## 新增 Skill

1. 创建新目录并添加 `SKILL.md`：

```
my-new-skill/
└── SKILL.md
```

2. 在 `.claude-plugin/marketplace.json` 的 `skills` 数组中添加路径：

```json
"skills": [
  "./write-blog",
  "./my-new-skill"
]
```

3. 提交推送后，运行 `claude plugin update my-skills` 更新。
