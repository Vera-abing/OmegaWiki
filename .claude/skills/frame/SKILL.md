---
description: 为通过可行性评估的研究想法设计分析框架 — 提出候选理论视角 → 用户确认 → 构建分析框架 → 可选写入 idea 页。
argument-hint: <idea-slug> [--write]
---

# /frame

> 为一个已完成发表可行性评估的研究想法设计理论视角和分析框架。
> 从 wiki 检索相关理论，提出2-3个候选视角供用户选择，确认后构建完整分析框架。
> 可选 --write 将框架写入 idea 页并推进状态至 framing。

## Inputs

- `idea-slug`（必填）：目标 ideas/ 页面的 slug
- `--write`（可选，默认**关闭**）：将理论视角和分析框架写入 idea 页，并将 status 推进为 `framing`

## Outputs

- **框架设计报告**（输出到终端）：
  - 2-3个候选理论视角（含理论来源、核心概念、契合点、局限性）
  - 基于用户选定视角构建的分析框架（含核心维度和分析逻辑）
- **Idea 页写入**（仅当 `--write` 时）：更新 idea 页的"理论视角"和"分析框架"节，推进 status 为 `framing`

## Wiki Interaction

### Reads
- `context/user.md` — 了解 VERA 的研究背景、学科立场和思维偏好
- `wiki/ideas/{slug}.md` — 提取研究问题、文献缺口、article_type
- `wiki/concepts/*.md` — 检索相关理论概念
- `wiki/topics/*.md` — 检索相关研究话题的理论背景
- `wiki/papers/*.md` — 检索已收录文献中使用的理论视角

### Writes
- `wiki/ideas/{slug}.md`（仅当 `--write`）— 更新"理论视角"和"分析框架"节
- `wiki/log.md`（仅当发生写入）— 追加记录

### Graph edges created
- 无。

## Workflow

**前置条件**：
1. 确认工作目录为项目根目录
2. 读取 `wiki/ideas/{slug}.md`，确认 idea 存在
3. 若 idea status 不是 `assessed` 或 `framing`，发出警告但继续执行

### Step 1: 读取研究背景

1. 读取 `context/user.md`，了解 VERA 的学科立场（职业技术教育学、比较教育、劳动经济学交叉）
2. 读取 `wiki/ideas/{slug}.md`，提取：
   - 研究问题与文献缺口
   - article_type（empirical / theoretical，如已设定）
   - 目标期刊（如已设定）

### Step 2: 检索相关理论

从 wiki 中检索可能适用的理论：
- 搜索 `wiki/concepts/*.md` 中与研究问题关键词匹配的概念
- 搜索 `wiki/papers/*.md` 中已收录文献的"理论视角"节
- 综合已知的社科理论库（技能生态系统理论、人力资本理论、制度主义、世界体系理论、能力方法、依附理论等）

### Step 3: 提出候选理论视角

提出2-3个候选理论视角，每个包含：

```markdown
### 视角 A：[理论名称]

- **理论来源**：[提出者 / 代表文献]
- **核心概念**：[2-4个关键概念]
- **与本研究的契合点**：[为什么这个视角适合这个研究问题]
- **局限性**：[这个视角在本研究中的不足之处]
- **代表性应用文献**：[wiki 中已有或建议检索的文献]
```

**候选视角选取原则：**
- 优先选择在 VERA 研究领域已有一定应用基础的理论
- 至少包含一个批判性/政治经济学视角（符合 VERA 的批判性思维偏好）
- 视角之间应有差异性，不选择高度同质的理论

### Step 4: 呈现给用户确认

将候选视角以清晰格式输出，**暂停并等待用户选择**。

询问用户：
1. 选择哪个理论视角（或组合使用）
2. 是否需要调整某个视角的侧重点

### Step 5: 构建分析框架

基于用户确认的理论视角，构建分析框架：

```markdown
## 分析框架

**理论视角**：[选定的理论]

**核心分析维度：**
1. [维度1]：[定义和分析逻辑]
2. [维度2]：[定义和分析逻辑]
3. [维度3]（如有）：[定义和分析逻辑]

**分析逻辑**：[各维度之间的关系和分析推进路径]

**操作化方向**（empirical 路径）/ **论证路径**（theoretical 路径）：
[根据 article_type 给出相应的下一步方向]
```

### Step 6: 写入 Idea 页（仅 --write 时）

1. 将 Step 5 的框架写入 `wiki/ideas/{slug}.md` 的"理论视角"和"分析框架"节
2. 推进 idea status 为 `framing`：
   ```bash
   python3 tools/research_wiki.py transition wiki/ ideas/{slug} framing
   ```
3. 追加日志：
   ```bash
   python3 tools/research_wiki.py log wiki/ "frame | 完成分析框架设计 [[ideas/{slug}]] | 理论视角: {视角名称}"
   ```

## Constraints

- Step 4 必须等待用户确认，不得跳过
- `--write` 仅修改"理论视角"和"分析框架"节，不修改其他节
- 状态推进仅限 assessed → framing
- 不在 frame 技能内做文献搜索（文献检索属于 /discover 和 /ingest 的职责）

## Dependencies

### Tools（via Bash）
- `python3 tools/research_wiki.py transition wiki/ ideas/{slug} framing` — 推进状态
- `python3 tools/research_wiki.py log wiki/ "<message>"` — 追加日志

### Claude Code Native
- `Read` — 读取 wiki 页面和 context/user.md
- `Edit` — 更新 idea 页（--write 时）
- `Glob` — 检索 wiki/concepts/ 和 wiki/papers/

### Called by
- 用户手动调用（在 /pub-assess 完成后）
