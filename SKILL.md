---
name: paper-reader
description: Quickly read and deeply understand a single academic paper (PDF) in economics, finance, or quantitative research. Produces a structured "Paper Brief" and saves it as a markdown file. Designed for practitioners who read papers for industry insight and research ideas, not for academic writing. Use this skill whenever the user provides a PDF paper and asks you to read, summarize, extract findings, understand methodology, or discuss it. Triggers on phrases like "read this paper", "summarize this paper", "help me understand this paper", "这篇论文讲什么", "帮我看看这篇paper", or when the user provides a PDF path and asks about its contents. Responds in whichever language (EN/CN) the user initiates with.
---

# Paper Reader

Read an academic paper once, understand it deeply, answer questions fluently.

## Why This Skill Exists

The user is a quantitative researcher in economics/finance who reads papers to spot industry trends, borrow methodological ideas, and find data sources — not to write academic papers themselves. Every minute spent on exhaustive analysis of tangential sections is wasted. The goal is a single efficient pass that captures everything needed to discuss the paper intelligently and decide whether to dig deeper.

This matters because the user's time is the bottleneck, not comprehensiveness. A 600-word brief that nails the core argument is worth more than a 3000-word walkthrough that buries the insight.

## Workflow

### Step 1: Convert PDF to Markdown

Use markitdown to convert the PDF into markdown. This avoids loading the full PDF binary into context, which would consume far more tokens for the same information. The markdown output preserves headings, tables, and structure while being compact.

```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("<path_to_pdf>")
paper_text = result.text_content
```

If the Python approach fails (missing dependency, corrupted PDF), fall back to the CLI:

```bash
markitdown "<path_to_pdf>"
```

If markitdown is not installed:

```bash
uv add 'markitdown[pdf]'
```

If markitdown completely fails (some scanned PDFs lack text layers), fall back to Claude's native PDF reading via the Read tool — but warn the user that this consumes more tokens for long papers.

### Step 2: Read with Purpose

Not all sections of an academic paper carry equal information density. The table below reflects how a practitioner actually reads — heavy on "what did you find and how" and light on "let me re-derive your proofs."

| Section | Attention | What to Extract |
|:--------|:----------|:----------------|
| Title, Authors, Abstract | Full | Core claim, scope, contribution |
| Introduction | Moderate | Research gap, motivation, hypothesis |
| Literature Review | Skim | Only note foundational or frequently-cited papers |
| Methodology | Full | Model spec, estimation, identification strategy, key assumptions |
| Data | Full | Source, time period, frequency, sample size, geography |
| Results | Full | Main findings, coefficient magnitudes, significance levels |
| Discussion / Conclusion | Full | Practical implications, limitations authors acknowledge |
| Appendices / Proofs | Skip | Unless the user specifically asks |
| References | Scan | Flag 3-5 high-signal papers (seminal works, recent related, data sources) |

The goal of this pass is to reconstruct the paper's argument chain: What question → Why it matters → How they attacked it → What they found → How confident we should be. If you can articulate this chain clearly, you understand the paper well enough to answer follow-up questions without re-reading.

### Step 3: Produce the Paper Brief

Output a structured brief following the template below. Be concise within each section but do not sacrifice important data to hit a word count — completeness of key results and sample details matters more than brevity. Use the same language the user used in their request.

**On formatting**: Prefer tables over dense prose whenever the data involves cross-period, cross-group, or cross-market numerical comparisons. A 4×2 table is always more scannable than 8 numbers crammed into one sentence.

---

**Template (EN):**

```
## Paper Brief

**Title**: [Full title]  
**Authors**: [First author et al., year] — [affiliation if notable]  
**Published in**: [Journal / Working Paper (NBER/SSRN/etc.)]

### One-Liner
[What this paper is about in one sentence — plain language, no jargon]

### Research Question
[What gap or puzzle does this paper address? Why should a practitioner care?]

### Main Result
[The single most important finding — 1-2 sentences with magnitude and direction.
If the result is a quantitative comparison across periods/groups, use a table:]

| Period | Addition CAR | Deletion CAR |
|:-------|:-------------|:-------------|
| ...    | ...          | ...          |

### Mechanism
[How/why does the main result arise? 2-4 sentences tracing the causal chain.
E.g., "1980s: X → 1990s: Y → 2000s: Z." If the paper has a memorable
illustrative example (e.g., a specific firm or event), mention it in one sentence.]

### Supporting Findings
- [Finding that supports or qualifies the main result]
- [Finding 2]
[2-3 bullet points for secondary findings. Use tables when comparing numbers
across categories; prefer tables over cramming numbers into prose.]

### Core Equation
$$[The 1-2 most important equations, displayed prominently]$$
where $X$ = ..., $Y$ = ..., ...
[Variable definitions on the line below. If the paper truly has no equation
worth showing (rare), state that explicitly.]

### Methodology
[Model type, estimation approach, identification strategy. 2-3 sentences.
Flag any methodological innovations or known weaknesses.]

### Data
- **Source**: [Database / provider name]
- **Sample**: [Time period, frequency, N observations, geographic scope]
- **Key Variables**: [Dependent variable, main independent variables]

### Statistical Rigor
- **Significance**: [Typical confidence levels reported — 1% / 5% / 10%]
- **Robustness**: [What robustness checks? Subsamples, alternative specs, placebo? If a check involves cross-sample or cross-market validation with quantitative results, show the actual numbers (e.g., a small comparison table), don't just say "they also tested on other indices."]
- **Limitations**: [What authors acknowledge + anything you notice they didn't]

### Key Results in Detail
[Show the paper's most important regression/decomposition results as tables.
For each key table, include: variable names, coefficient direction, magnitude,
significance level (*, **, ***), and how results change across specifications
(e.g., "with controls" vs "without controls"). This section should let the
reader see the actual evidence, not just the author's verbal summary.
Typically 1-3 small tables covering: the main result, the robustness to
controls, and any decomposition/subgroup analysis.]

### Sample Construction
- **Universe**: [Starting population and data source]
- **Filters**: [Each exclusion criterion, listed explicitly — e.g., "exclude
  firms acquired within 100 days of event", "require matching to X database"]
- **Attrition**: [How many observations survive each filter. Show final N vs
  starting N, and retention rate. Flag if any subgroup has severe attrition
  (e.g., deletions losing 2/3 of sample) — this affects statistical power.]
- **Sensitivity**: [Did the authors test whether results change with/without
  these filters? If yes, summarize. If no, note the gap.]

### Notable References
[3-5 papers worth reading next — one line each on why it matters to the reader]
- Author (Year) — [why this is useful]

### Practitioner Takeaway
[1-2 sentences: What does this mean for someone doing quantitative research
or working in the industry? Is the finding actionable? Novel? Confirmatory?]

### Project Implications (only if PDF is inside the project directory)
[If the PDF path is within the current working directory, read CLAUDE.md and
any relevant project context to produce specific, actionable implications:
- Which research questions (Q1-Q7) does this paper inform?
- What specific methodology or data can be borrowed?
- Does it challenge or confirm assumptions in the current project design?
- Concrete next steps: "use X framework for Q4", "add Y as control variable in event panel"
Keep to 3-5 bullet points, each tied to a specific project element.]
```

---

**Template (CN):**

```
## 论文速览

**标题**: [完整标题]  
**作者**: [第一作者 et al., 年份] — [机构(如有名校/名机构)]  
**发表于**: [期刊 / 工作论文 (NBER/SSRN等)]

### 一句话总结
[用一句话说清楚这篇论文讲什么]

### 研究问题
[这篇论文解决什么空白或疑问?对从业者来说为什么重要?]

### 主结论
[最重要的发现 — 1-2句话,带方向和量级.
如果结论涉及跨时期/跨组别的量化对比,用表格展示:]

| 时期 | Addition CAR | Deletion CAR |
|:-----|:-------------|:-------------|
| ...  | ...          | ...          |

### 核心机制
[主结论背后的因果链是什么?2-4句话.
如"1980s: X → 1990s: Y → 2000s: Z".
如果论文中有一个突出的案例(如某只具体股票或事件),用一句话提及.]

### 支撑发现
- [支撑或限定主结论的发现]
- [发现2]
[2-3条次要发现.涉及跨类别数字对比时用表格,不要把数字堆在一句话里.]

### 核心方程
$$[1-2个最重要的方程,独立成行]$$
其中 $X$ = ..., $Y$ = ..., ...
[变量定义紧跟方程下方.如果论文确实没有值得展示的方程(少见),明确说明.]

### 方法论
[模型类型、估计方法、识别策略.2-3句话.
标注方法论创新点或已知局限.]

### 数据
- **来源**: [数据库/数据提供商]
- **样本**: [时间区间、频率、观测数量、地理范围]
- **核心变量**: [因变量、主要自变量]

### 统计严谨性
- **显著性**: [论文通常报告的置信水平 — 1% / 5% / 10%]
- **稳健性检验**: [做了哪些?子样本、替代模型、安慰剂检验?如果涉及跨样本/跨市场验证且有具体数据,列出关键数字,不要只说"做了"]
- **局限性**: [作者自己承认的 + 你发现但作者没提的]

### 核心结果详表
[展示论文最重要的回归/分解结果,以表格形式呈现.
每个关键table包含:变量名、系数方向、量级、显著性水平(*, **, ***),
以及不同specification下结果如何变化(如"有控制变量" vs "无控制变量").
这个section让读者看到实际证据,而不仅是作者的文字总结.
通常1-3个小表格,覆盖:主结论的回归结果、控制变量的影响、
分组/分解分析.]

### 样本构建
- **全集**: [初始总体和数据来源]
- **筛选条件**: [逐条列出排除标准 — 如"排除指数变动100天内被收购的公司"、
  "要求能匹配到X数据库"]
- **损耗率**: [每个筛选步骤保留了多少观测.展示最终N vs 初始N和保留率.
  如果某个子样本损耗严重(如deletions丢失2/3样本),标注出来 — 这影响统计功效.]
- **敏感性**: [作者是否测试了不同筛选标准下结果是否变化?如果测了,总结.
  如果没测,指出这个gap.]

### 值得跟读的参考文献
[3-5篇值得进一步阅读的论文 — 每篇一句话说明为什么值得读]
- 作者 (年份) — [对读者有什么用]

### 实操启示
[1-2句话:对做量化研究或行业从业者意味着什么?
这个发现是否可操作?是否新颖?还是验证了已知结论?]

### 项目关联(仅当PDF位于项目目录内时)
[如果PDF路径在当前working directory下,先读取CLAUDE.md等项目核心信息,
然后给出与项目直接挂钩的具体启示:
- 这篇论文对项目的哪些research questions (Q1-Q7) 有启发?
- 有哪些具体的方法论或数据可以借鉴?
- 是否挑战或验证了项目当前的设计假设?
- 可落地的下一步:如"将X框架用于Q4"、"在event panel中加入Y作为控制变量"
控制在3-5条,每条必须对应项目中的具体元素.]
```

### Step 4: Save the Brief

After displaying the brief in the conversation, save it as a markdown file:

- **Path**: `docs/paper_briefs/{first_author_lastname}_{year}.md`
  - Use the first author's last name, lowercased, ASCII-only (e.g., `müller` → `muller`)
  - Year is the publication year (or the most recent working paper version year)
  - If a file with the same name already exists, append a short disambiguator: `petajisto_2011_indexing.md`
- **Create the directory** `docs/paper_briefs/` if it does not exist yet
- **File content**: the brief itself (the markdown output from Step 3), without code fences

This creates an accumulating knowledge base of paper briefs the user can grep, reference, and revisit across sessions.

## Follow-Up Readiness

After producing the brief, be ready for deeper questions without re-reading the entire paper. Common follow-ups:

- "This methodology — how exactly does it differ from [X]?"
- "What data would I need to replicate this?"
- "Are their results robust to [some alternative specification]?"
- "Which of the referenced papers should I read first?"
- "How does this relate to [my own research / index rebalancing project]?"

If a follow-up requires details from a section you skimmed (e.g., a specific table in the appendix), re-read that section on demand. Front-loading every detail wastes tokens on information the user may never ask about.

## What to Avoid

These anti-patterns waste the user's time or obscure the paper's actual contribution:

- **Section-by-section summaries**: "Section 2 discusses..." format mirrors the paper's structure, not the reader's needs. Organize by information type instead — the reader wants "the data comes from..." not "Section 3 describes the data."
- **Excessive hedging**: "The authors find X" beats "The authors seem to suggest that perhaps X might be the case." Caveats belong in the Limitations field, not sprinkled through every finding.
- **Unsolicited critique**: The brief reports what the paper says and how rigorous it is. If the user wants a methodological critique, they will ask — and then the `scientific-critical-thinking` skill is better suited for that depth.
- **Inflated reference lists**: 3-5 references that genuinely help the reader's next step, not a dump of every citation in the bibliography.
- **Verbatim abstract copying**: The one-liner and key findings should be rewritten — more concise and more practical than the original abstract.

## Integration with Other Skills

- **Critical evaluation** of methodology → `scientific-critical-thinking`
- **Find related papers** → `literature-review` or `arxiv-database`
- **Replicate statistical analysis** → `statsmodels` or `statistical-analysis`
- **Paper is a web page or preprint link** (not a local PDF) → use `markitdown` to convert the URL content first, then follow this skill's workflow
