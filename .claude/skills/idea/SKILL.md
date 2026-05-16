---
description: Use when starting a new research idea or resuming an existing idea lifecycle — chains ideate → pub-assess → frame → empirical/theoretical path → paper-plan, pausing after each stage for explicit user confirmation before continuing.
argument-hint: [<topic-description>] [--slug <existing-idea-slug>]
---

# /idea

> 研究想法全生命周期编排器。从提出想法到生成论文大纲，每个阶段结束后暂停并询问是否继续，不自动跳转。

## Inputs

- `topic-description`（可选）：新想法的话题描述；省略则先询问用户
- `--slug <slug>`（可选）：已有 idea 页的 slug，从当前 status 恢复流程

## 核心规则

1. **每个阶段完成后必须暂停**，等用户明确回答后才进入下一阶段
2. 用户回答"否"/"不"/"停"时立刻终止，告知恢复命令
3. 用户回答"暂停"/"稍后"时，告知：`/idea --slug <slug>` 可恢复
4. **不得替用户做选择**（理论视角须在 `/frame` 内部由用户确认）
5. 进入下一阶段前，在终端输出当前阶段摘要（不超过3行）

---

## 阶段流程

### 阶段 1 — 提出想法

**适用：** 新想法（无 slug）

执行：调用 `/ideate <topic>`

完成后询问：
> "想法页已创建（slug: `{slug}`）。**下一步：发表可行性评估**——搜索近3年期刊，判断能不能发、推荐投哪里。继续吗？"

---

### 阶段 2 — 发表可行性评估

**适用：** status = `proposed`

执行：调用 `/pub-assess {slug} --write`

完成后询问：
> "可行性评估完成（趋势：{趋势}，可行性：{高/中/低}）。**下一步：分析框架设计**——提出候选理论视角供你选择。继续吗？"

---

### 阶段 3 — 分析框架设计

**适用：** status = `assessed`

执行：调用 `/frame {slug}`（注意：**不加 `--write`**，框架技能内部会等待用户确认理论视角后再写入）

在 `/frame` 内部用户完成理论视角选择后，`/frame` 会询问是否 `--write`；用户同意后执行写入，status 推进为 `framing`。

完成后询问：
> "分析框架已建立。**下一步：确定文章类型。** 这篇文章是 实证研究（需要调研/访谈数据）还是 理论研究（文献推导论证）？"

（若 idea 页已设定 `article_type`，跳过此问，直接告知检测到的类型并询问是否继续对应路径）

---

### 阶段 4a — 实证路径：研究设计与数据收集

**适用：** article_type = `empirical`，status = `framing`

执行：
1. 根据研究问题和分析框架，设计问卷或访谈提纲，输出完整草稿
2. 更新 idea 页"研究设计（实证路径）"节，推进 status 为 `data_collection`
3. 告知用户：

> "问卷/访谈提纲已设计完毕。接下来由你完成实地调研。
> **数据收集完成后：** 将数据文件放入 `raw/data/` 目录（支持 .csv/.xlsx/.json 等格式），然后告诉我"数据已上传"，我继续分析。"

4. **暂停，等待用户告知数据已就绪**（不询问"继续吗"，等待用户主动触发）

数据就绪后执行：
- 读取 `raw/data/` 中的数据文件进行分析
- 将分析结果和结论写入 idea 页"数据与分析结果"节
- 推进 status 为 `analysis`

完成后询问：
> "数据分析完成，结论已写入 idea 页。**下一步：生成论文大纲（/paper-plan）。** 继续吗？"

---

### 阶段 4b — 理论路径：论证与大纲构建

**适用：** article_type = `theoretical`，status = `framing`

执行：
1. 基于分析框架展开深入理论研究，构建完整论证脉络
2. 生成细分到三级标题（###）的论文大纲，写入 idea 页"论文大纲（理论路径）"节
3. 将大纲输出给用户确认

完成后询问：
> "论文大纲已建立，请确认。**下一步：生成正式论文规划（/paper-plan）。** 继续吗？"

---

### 阶段 5 — 论文规划

**适用：** status = `outlining`（理论）或 `analysis`（实证）

执行：
1. 若 idea 页 `target_venue` 为空，询问：目标期刊是哪本？
2. 调用 `/paper-plan {slug} --venue {venue}`

完成后输出：
> "论文规划（PAPER_PLAN.md）已生成。后续可用 `/paper-draft` 开始撰写。
> 本轮 /idea 流程结束。"

---

## 恢复流程（--slug）

读取 `wiki/ideas/{slug}.md` 的 `status` 字段，映射到对应阶段：

| status | 恢复起点 |
|---|---|
| proposed | 阶段 2（pub-assess）|
| assessed | 阶段 3（frame）|
| framing | 阶段 4（询问 article_type）|
| data_collection | 阶段 4a（提示等待数据）|
| analysis | 阶段 5（paper-plan，实证）|
| outlining | 阶段 5（paper-plan，理论）|
| writing / submitted / published | 告知 idea 已进入写作/投稿阶段，/idea 流程已完成 |
| abandoned | 告知 idea 已放弃，询问是否重新激活 |

## 错误处理

- slug 不存在：列出 `wiki/ideas/` 中现有 ideas 供选择
- status 与预期不符：告知当前 status，询问是否强制进入某阶段
- `/frame` 内用户拒绝所有候选视角：让用户自由描述视角，再由 `/frame` 整理
