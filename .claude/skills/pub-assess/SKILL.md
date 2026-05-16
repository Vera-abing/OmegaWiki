---
description: 评估研究想法在目标社科期刊近3年的发表可行性 — Semantic Scholar + WebSearch 搜索近期发文 → 判断话题趋势 → 推荐期刊 → 输出可行性报告。可选 --write 将报告写入 idea 页。
argument-hint: <idea-description-or-slug> [--write] [--venue <journal-name>]
---

# /pub-assess

> 评估一个研究想法在社科期刊发表的可行性。搜索 Semantic Scholar 和目标期刊近3年发文情况，判断话题是否处于上升期、平稳期、饱和期或冷门期，并给出发表可行性评级和推荐期刊列表。
> 可独立使用，也可由 /ideate 在想法通过新颖性验证后调用。

## Inputs

- `target`：以下之一：
  - 研究想法的自由文本描述（一段话或几句话）
  - wiki 中 ideas/ 页面的 slug（如 `vocational-skill-ecosystem-africa`）
- `--write`（可选，默认**关闭**）：将报告写入目标 idea 页的"发表可行性评估"节。仅当 target 是 idea slug 时生效。
- `--venue <journal-name>`（可选）：指定优先搜索的期刊名称；若省略，使用 VERA 的默认期刊列表。

## Outputs

- **发表可行性报告**（输出到终端）：
  - 话题趋势评级：上升期 / 平稳期 / 饱和期 / 冷门期
  - 近期代表性文献（≤5篇，含标题、期刊、年份）
  - 推荐投稿期刊（中文2-3本 + 英文2-3本）
  - 发表可行性预判（高 / 中 / 低）及理由
- **Idea 页写入**（仅当 `--write` 且 target 是 idea slug 时）：更新 `wiki/ideas/{slug}.md` 的"发表可行性评估"节，并将 idea status 推进为 `assessed`。

## 默认期刊列表

**中文期刊（CSSCI）：**
- 教育研究、职业技术教育、比较教育研究、教育发展研究、华东师范大学学报（教育科学版）

**英文期刊：**
- Journal of Vocational Education & Training (JVET)
- Compare: A Journal of Comparative and International Education
- International Journal of Educational Development (IJED)
- Journal of Education and Work
- Globalisation, Societies and Education

## Wiki Interaction

### Reads
- `wiki/ideas/{slug}.md`（当 target 是 slug 时）— 提取研究问题和研究背景
- `wiki/papers/*.md` — 检查已收录文献，避免重复搜索
- `context/user.md` — 了解 VERA 的研究背景和兴趣领域

### Writes
- `wiki/ideas/{slug}.md`（仅当 `--write` 且 target 是 slug）— 更新"发表可行性评估"节
- `wiki/log.md`（仅当发生写入）— 追加记录

### Graph edges created
- 无。

## Workflow

**前置条件**：确认工作目录为项目根目录（包含 `wiki/`、`raw/`、`tools/`）。

### Step 1: 提取研究主题

1. 若 target 是 slug：读取 `wiki/ideas/{slug}.md`，提取"研究问题与文献缺口"节的内容
2. 若 target 是自由文本：直接使用
3. 读取 `context/user.md` 了解 VERA 的研究领域背景
4. 提炼3-5个核心搜索关键词（中英文各一组）

### Step 2: Semantic Scholar 搜索

使用 `tools/fetch_s2.py` 搜索近3年（当前年份-3年至今）相关文献：

```bash
python3 tools/fetch_s2.py search "<关键词>" --year-start <year-3>
```

- 搜索2-3组关键词，合并去重结果
- 重点关注在目标期刊发表的论文
- 记录：标题、期刊、年份、引用数

### Step 3: WebSearch 补充搜索

用 WebSearch 工具补充搜索，重点检索：
- 目标中文期刊近3年该主题的发文情况
- 目标英文期刊近3年该主题的发文情况
- 搜索格式示例：`site:cnki.net "职业教育" "技能生态" 2022-2025`

### Step 4: 判断话题趋势

综合 Step 2-3 的结果，判断话题趋势：

| 趋势 | 判断标准 |
|------|----------|
| 上升期 | 近3年发文量逐年增加，或有政策热点驱动 |
| 平稳期 | 近3年发文量稳定，仍有持续关注 |
| 饱和期 | 近3年发文量高但同质化严重，创新空间有限 |
| 冷门期 | 近3年发文极少，需评估是空白机会还是无人问津 |

### Step 5: 生成发表可行性报告

输出报告，包含：

```markdown
## 发表可行性评估报告

**话题趋势：** [上升期 / 平稳期 / 饱和期 / 冷门期]

**近期代表性文献（近3年）：**
1. [标题] — [期刊] ([年份])
2. ...（最多5篇）

**推荐投稿期刊：**
- 中文：[期刊1]（理由）、[期刊2]（理由）
- 英文：[期刊1]（理由）、[期刊2]（理由）

**发表可行性预判：** [高 / 中 / 低]
**理由：** [2-3句话说明判断依据]
```

### Step 6: 写入 Idea 页（仅 --write 时）

1. 将 Step 5 的报告写入 `wiki/ideas/{slug}.md` 的"发表可行性评估"节
2. 用 `tools/research_wiki.py transition` 将 idea status 推进为 `assessed`：
   ```bash
   python3 tools/research_wiki.py transition wiki/ ideas/{slug} assessed
   ```
3. 追加日志：
   ```bash
   python3 tools/research_wiki.py log wiki/ "pub-assess | 完成发表可行性评估 [[ideas/{slug}]] | 趋势: {趋势} | 可行性: {高/中/低}"
   ```

## Constraints

- 搜索结果必须限定在近3年（非全时段）
- 推荐期刊必须来自中文 CSSCI 或英文 SSCI/SCOPUS 期刊，不推荐预印本平台
- `--write` 仅修改 idea 页的"发表可行性评估"节，不修改其他节
- 状态推进仅限 proposed → assessed，不跨越状态

## Dependencies

### Tools（via Bash）
- `python3 tools/fetch_s2.py search "<关键词>" --year-start <year>` — Semantic Scholar 搜索
- `python3 tools/research_wiki.py transition wiki/ ideas/{slug} assessed` — 推进状态
- `python3 tools/research_wiki.py log wiki/ "<message>"` — 追加日志

### Claude Code Native
- `WebSearch` — 补充期刊发文搜索
- `Read` — 读取 wiki 页面
- `Edit` — 更新 idea 页（--write 时）

### Called by
- 用户手动调用
- `/ideate` Phase 4 完成新颖性验证后可选调用
