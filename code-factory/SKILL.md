---
name: code-factory
description: 'Claude与Codex双AI协作开发工作流。Claude负责需求分析和架构设计，Codex负责评审、代码实现和测试。支持多轮讨论收敛、多Codex并行实现、互相Review的全闭环。输入可以是需求文档、文字描述或代码库。'
---

# Code Factory - Claude x Codex 双AI协作开发

Claude（架构师）与 Codex（工程师）协作，完成从需求设计到代码交付的全流程。

## 使用方法

```bash
# 文字描述需求
/code-factory 给用户模块添加 RBAC 权限控制，支持角色继承

# 指向需求文档
/code-factory @docs/requirement.md

# 基于代码库
/code-factory 重构 src/services/auth.ts，拆分为独立的认证和授权模块
```

## 上下文

- 用户输入：$ARGUMENTS
- 输入可能是：文字需求描述、需求文档路径、代码库路径/描述、或以上组合

## 角色分工

| 角色 | 负责 |
|------|------|
| Claude | 需求分析、架构设计、讨论决策、最终裁决、波次调度 |
| Codex | 设计评审、任务拆分、代码实现、测试编写、代码Review |

## 执行流程

---

### Phase 0: 环境检查与会话初始化

1. 检测 codex 是否可用：

```bash
which codex
```

如果不可用，提示用户安装（`npm install -g @openai/codex`）并终止。

2. 创建会话目录：

```
.code-factory/{YYYYMMDD-HHmmss}-{需求关键词2-3字}/
```

3. 将 `.code-factory/` 添加到 `.gitignore`（如尚未添加）。

---

### Phase 1: 需求分析与设计（Claude 主导）

**Step 1: 解析输入**

分析 $ARGUMENTS 确定输入类型，可能的组合：

| 输入类型 | 识别方式 | 处理 |
|----------|---------|------|
| 文字描述 | 纯文本，无文件路径特征 | 直接作为需求 |
| 需求文档 | 包含文件路径（.md/.txt/.doc等） | Read 读取文件内容 |
| 代码库/文件引用 | 包含代码文件路径（.ts/.py/.java等）或目录路径 | Glob + Read 探索相关代码 |
| 组合输入 | 同时包含描述和文件引用 | 分别处理后合并 |

将解析后的需求写入 `{session}/00-requirement.md`。

**Step 2: 代码库探索（如需要）**

如果需求涉及现有代码的修改/重构：
- 使用 Glob 查找相关文件
- 使用 Grep 搜索关键类/函数/接口
- 使用 Read 阅读核心文件
- 将代码上下文摘要追加到 `00-requirement.md`

**Step 3: 写设计方案**

按 `assets/design-template.md` 模板输出到 `{session}/01-design.md`。

设计文档末尾必须包含：
- **评审维度建议**：Claude 判断本次需要哪些评审角度
- **待评审问题**：2-5 个需要 Codex 重点关注的具体问题

评审维度参考（按需勾选，不是每次都全选）：
- 架构合理性与可维护性（必选）
- 安全性与边界情况（涉及权限/数据/输入时选）
- 性能与可扩展性（高并发/大数据量时选）

**Step 4: 判断需求规模**

根据涉及文件数和复杂度，确定后续阶段的并行度：

```
小需求（单文件改动）     → 评审1个Codex, 实现1个Codex
中需求（2-5文件）        → 评审1-2个Codex, 实现2-3个Codex
大需求（跨模块/系统级）  → 评审2-3个Codex, 实现3-5个Codex
```

将规模判断记录在 `01-design.md` 末尾。

---

### Phase 2: Codex 评审

根据 Phase 1 确定的评审维度，启动 Codex 评审。

**单维度评审**（小需求）：

```bash
codex exec --model o4-mini --full-auto \
  "$(cat .code-factory/{session}/01-design.md)

按以下格式输出评审结果，写入 .code-factory/{session}/02-review/r1.md：

DECISION: APPROVE 或 REQUEST_CHANGES
PRIORITY_ISSUES: <数字>
---

## P0 问题（必须改）
每条不超过50字。

## P1 问题（建议改）
每条不超过50字。

## 详细分析
对每个问题的详细说明和改进建议。

## 对待评审问题的回应
逐条回应设计文档中的待评审问题。"
```

**多维度并行评审**（中/大需求）：

```bash
# 多个 Codex 并行，每个关注不同维度
codex exec --model o4-mini --full-auto \
  "...从【架构合理性】角度评审...写入 02-review/r1-architecture.md" &

codex exec --model o4-mini --full-auto \
  "...从【安全性】角度评审...写入 02-review/r1-security.md" &

wait
```

具体 prompt 模板见 `references/codex-prompts.md`。

---

### Phase 3: 讨论循环（最多 3 轮）

**Step 1: 完整读取评审内容**

Claude 完整读取所有评审文件，逐条理解 Codex 提出的每个问题、分析和建议。
包括 P0/P1 问题列表、详细分析、以及对待评审问题的回应。

**Step 2: 判断是否需要讨论**

- 所有评审 DECISION 均为 APPROVE → 写共识文档，跳到 Phase 4
- 存在 REQUEST_CHANGES → 进入讨论：

**Step 3: Claude 逐条回应**

Claude 认真分析每个 REQUEST_CHANGES 点：
- 认同的：修订设计，说明如何改
- 不认同的：给出具体的技术反驳理由
- 部分认同的：说明采纳了哪部分、保留了哪部分及原因

将回应和修订后的设计写入：
- `02-review/r{N}-response.md`：Claude 对每条评审的逐条回应
- `01-design-v{N}.md`：修订后的设计文档

**Step 4: 发起下一轮评审**

将修订后的设计和 Claude 的回应一起发给 Codex：

```bash
codex exec --model o4-mini --full-auto \
  "$(cat .code-factory/{session}/01-design-v2.md)

以下是设计者对上一轮评审的回应：
$(cat .code-factory/{session}/02-review/r1-response.md)

请重新评审修订后的设计：
- 上轮提出的问题是否已妥善解决
- 设计者的反驳理由是否成立
- 是否有新的问题

输出到 .code-factory/{session}/02-review/r2.md，格式同前。"
```

**Step 5: 收敛规则**

- 轮次 1-2：正常讨论，Claude 和 Codex 充分交换意见
- 轮次 3：Claude 强制裁决，对仍存分歧的点写明取舍理由
- 如遇根本性架构分歧（技术路线完全不同）→ 暂停，征求用户意见

**Step 6: 输出共识文档**

写 `02-review/consensus.md`，记录：
- 所有达成共识的决策点
- Claude 的仲裁结论（如有）
- 讨论中涌现的改进点

**用户检查点：展示讨论摘要（共识点 + 仲裁点），等用户确认后继续。**

---

### Phase 4: 任务拆分与多 Codex 并行实现

**Step 1: Codex 拆任务**

让 Codex 读取最终设计和共识，拆分实现任务：

```bash
codex exec --model o4-mini --full-auto \
  "读取以下文件：
- .code-factory/{session}/01-design-final.md（最终设计）
- .code-factory/{session}/02-review/consensus.md（评审共识）

将实现拆分为独立任务，分析依赖关系。

输出格式（严格遵守），写入 .code-factory/{session}/03-plan.md：

## TASKS
- TASK:{id} DEPENDS:{none|task-ids} COMPLEXITY:{low|medium|high} FILES:{file-paths}
  {一句话描述该任务要做什么}

## WAVES
- WAVE-1: {可并行的task-id列表}
- WAVE-2: {依赖WAVE-1的task-id列表}
- ...

## ACCEPTANCE
- [ ] {验收标准1}
- [ ] {验收标准2}"
```

**Step 2: Claude 读取任务列表**

只读 `03-plan.md` 的 `## TASKS` 和 `## WAVES` 部分（Read + limit），解析任务和波次。

**用户检查点：展示任务列表和波次划分，等用户确认后继续。**

**Step 3: 按波次并行派发**

```bash
# Wave 1: 无依赖任务并行
codex exec --model o4-mini --full-auto \
  "你是一名高级工程师。

任务：{从 plan 中提取的 Task-1 描述}
涉及文件：{file-paths}
设计要求：{从 design 中提取的相关部分}

实现所有代码变更，编写对应测试。
完成后写摘要到 .code-factory/{session}/04-impl/task-{id}-summary.md

摘要格式：
STATUS: DONE 或 PARTIAL
FILES_CHANGED: {逗号分隔的文件路径}
---
{简述做了什么，遇到什么问题}" &

codex exec --model o4-mini --full-auto \
  "...(Task-2)..." &

wait
```

**Step 4: 波次间集成检查（Claude 轻量执行）**

每个 Wave 完成后，Claude 只读各 summary 的前 3 行（STATUS + FILES_CHANGED）：
- 检查是否有多个 Task 修改了同一文件（文件冲突）
- 检查 STATUS 是否全部 DONE
- 如有冲突 → Claude 手动合并
- 如有 PARTIAL → 评估是否影响下一波次
- 确认无问题后启动下一个 Wave

---

### Phase 5: 互相 Review

**并行执行两个 Review：**

1. **Claude 完整代码审查**：
   - 读取所有变更文件的实际代码
   - 对照设计文档和实施计划，逐项检查：
     - 功能完整性：设计要求是否全部实现
     - 接口一致性：函数签名、类型、数据结构是否与设计对齐
     - 代码质量：是否符合项目风格、有无明显坏味道
     - 逻辑正确性：核心业务逻辑是否正确
   - 将审查结果写入 `{session}/05-review/claude-review.md`

2. **Codex 全面 Review**：

```bash
codex exec --model o4-mini --full-auto \
  "你是一名代码评审专家。

读取 .code-factory/{session}/03-plan.md 了解设计意图。
审查所有已实现的代码变更（文件列表：{从summaries收集的FILES_CHANGED}）。

检查维度：
- 功能正确性：是否符合设计要求
- 边界情况：空值、异常、并发
- 代码质量：命名、结构、重复
- 测试覆盖：是否覆盖核心路径

输出格式，写入 .code-factory/{session}/05-review/codex-review.md：

VERDICT: PASS 或 ISSUES_FOUND
CRITICAL_ISSUES: <数字>
---

## 关键问题
{如有，每条包含：文件、行号、问题描述、修复建议}

## 改进建议
{如有}

## 总体评价
{一段话}"
```

大需求可拆为多个 Codex 分模块并行 Review。

**Step 3: 交叉比对**

Claude 完整读取 Codex 的 Review 结果，与自己的审查结论交叉比对：
- 双方都发现的问题 → 高优先级修复
- 仅一方发现的问题 → Claude 判断是否需要修复
- 双方意见冲突 → Claude 做最终裁决

**Step 4: 结果处理**

- 双方 Review 均无关键问题 → 生成最终报告
- 存在关键问题 → 触发修复循环：
  - 将关键问题发给 Codex 修复（最多 2 轮）
  - 每轮修复后 Codex 重新验证

---

### Phase 6: 最终报告

Claude 产出简短报告展示给用户：

```markdown
## Code Factory 完成报告

### 需求概要
{一句话}

### 讨论
- 评审轮次: N 轮
- 结论: {共识/仲裁}

### 实现
| Task | Status | Files |
|------|--------|-------|

### Review
- Codex: {PASS/ISSUES_FOUND}
- Claude: {通过/有建议}

### 变更文件汇总
{所有变更文件列表}

### 遗留问题
{如有}
```

---

## 错误处理

| 场景 | 处理 |
|------|------|
| codex 未安装 | Phase 0 检测，提示安装命令，终止 |
| codex exec 超时（>5min） | 重试 1 次，仍失败则标记该任务为 FAILED，不阻塞其他并行任务 |
| Codex 输出格式错误 | 尝试解析可用部分，完全不可解析则 Claude 降级读全文 |
| 评审无法收敛 | 第 3 轮 Claude 强制裁决 |
| 并行任务文件冲突 | Claude 在波次间检测并手动合并 |
| 修复循环超限 | 报告用户，列出未解决的关键问题 |

## Guidelines

- 会话文件全部存放在 `.code-factory/` 目录
- Phase 3 讨论：Claude 完整读取 Codex 评审，认真逐条回应，不省略
- Phase 5 审查：Claude 完整读取变更代码，做实质性代码评审，不走过场
- Phase 4 波次间检查：Claude 只读 summary 头部（STATUS + FILES_CHANGED），这里可以省
- Codex 的所有输出必须遵循指定格式（DECISION/STATUS/VERDICT 头部）
- 需求规模由 Claude 在 Phase 1 自适应判断，不过度并行
- 每个用户检查点必须等用户明确确认后才继续下一阶段
- Codex prompt 中直接内联必要上下文（cat 文件内容），避免让 Codex 自己读大量文件
- 代码注释语言与现有代码库保持一致
