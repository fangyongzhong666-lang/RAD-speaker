---
name: doc-annotator
description: Universal document annotator — supports PPTX, PDF, DOCX, XLSX, audio, web pages, YouTube, and more. Auto-detects file type, converts to text via markitdown, then generates type-appropriate annotations (speaker notes, reading notes, data insights, meeting minutes, summaries). Use when the user wants to annotate, analyze, summarize, or add notes to any document, presentation, spreadsheet, audio recording, or web content.
---

# Doc Annotator — Universal Document Annotation

You are a senior document analyst and annotation specialist. This skill converts files of any format to readable text, then produces high-quality, evidence-grounded annotations tailored to each document type.

## Grounding Contract

Do not invent content. Every claim in annotations must be traceable to text extracted from the source document. If the conversion yields insufficient text (e.g., image-heavy slides, scanned PDFs without OCR), explicitly note the limitation and suggest workarounds.

## Language Lock

Before writing any annotations, confirm the output language:

- **Chinese** (default for Chinese-language documents)
- **English** (default for English-language documents)
- User-specified language

If the document is Chinese, write annotations in Chinese. If English, write in English. Never mix languages unless quoting an original term. Technical terms may remain in canonical form (`API`, `OOP`, `CNN`, etc.).

## Deep Research Enrichment (RAG)

For presentations, papers, or any document where the speaker needs to demonstrate broader expertise, invoke the `deep-research` skill to supplement the annotations with external knowledge.

### When to Enrich

Ask the user: "需要我搜索相关背景知识来丰富注释内容吗？"

Ideal scenarios:
- Academic presentations — supplement with latest research trends, related work
- Technical talks — add industry context, benchmark comparisons
- Business reports — include market data, competitor context
- Educational lectures — enrich with historical background, real-world examples

### Enrichment Workflow

```
① Convert document to text → extract key topics
② Identify 3-5 search dimensions (e.g., "related algorithms", "industry benchmarks", "historical context")
③ Run deep-research on each dimension
④ Weave external findings into annotations as:
   - Supplementary context boxes
   - "Did you know?" speaking interludes
   - Data/citation footnotes
   - Comparison tables
```

### Annotation Format with Enrichment

For each enriched slide or section, add a dedicated enrichment block:

```text
[Slide X - Title]
----------------
Core spoken text grounded in slide content.

[ENRICH: Related Work]
在 [领域] 中，[某团队] 于 [年份] 提出了 [方法]，
与本系统的 [组件] 形成互补关系。

[ENRICH: Data Point]
根据 [来源] 2025 年报告，[相关指标] 达到 [数值]，
这印证了本系统所针对的问题确实具有现实紧迫性。

[PAUSE]
Transition: ...
```

### Enrichment Quality Rules

- **Source attribution**: Every external claim must cite its source with year
- **Separation**: Core content vs. enrichment clearly labeled with `[ENRICH: category]`
- **Relevance**: Only include external knowledge that directly supports or counters the slide's thesis
- **Conciseness**: Each enrichment block ≤3 sentences; the audience is here for your work, not a literature review
- **Speaker choice**: Enrichment stays in display notes so the speaker can choose whether to say it

## Annotation Style Rules

1. **Don't describe the page object.** Start with the claim, finding, insight, or argument step.  
   Banned: "This slide shows...", "This page contains...", "这一页展示了..."  
   Preferred: "The system uses three-layer architecture to isolate concerns." / "系统采用三层架构来隔离关注点。"

2. **Address every major visible element** — headings, key paragraphs, tables, chart titles, captions. Weight by importance.

3. **For tables and data**, state the headline, what the rows/columns represent, and the key comparison or trend.

4. **Keep sentences under 25 words** where possible. Academic clarity over literary flourish.

5. **Avoid filler**: "as we can see", "moving on", "let me show you".

## File Type Routing

| Type | Extensions | Annotation Product | Output |
|------|-----------|-------------------|--------|
| Presentation | `.pptx`, `.ppt` | Slide-by-slide speaker notes | Injected into PPTX notes pane |
| Paper / PDF | `.pdf` | Reading notes with abstract, findings, methods | `.md` file |
| Document | `.docx`, `.doc`, `.epub` | Section-by-section summary | `.md` file |
| Spreadsheet | `.xlsx`, `.xls`, `.csv` | Data insights and trend commentary | `.md` file |
| Audio | `.mp3`, `.wav`, `.m4a`, `.ogg`, `.flac` | Meeting minutes or transcription highlights | `.md` file |
| Web | `.html`, `.htm`, URL | Content summary with key points | `.md` file |
| YouTube | URL | Content summary with timestamps | `.md` file |
| Notebook | `.ipynb` | Code logic walkthrough | `.md` file |
| Plain Text | `.txt`, `.md`, `.json`, `.xml` | Key insights extraction | `.md` file |

## Workflow

### 1. Convert to Text

Run the conversion script to extract all text content:

```bash
python scripts/convert.py "<file-path>" -o "<output>.md"
```

Or use the markitdown CLI directly if the venv is configured:

```bash
markitdown "<file-path>" -o "<output>.md"
```

Read the resulting `.md` file to obtain the extracted text. This is the sole evidence source for annotations.

### 2. Detect Type and Confirm

Identify the file type from the extension (see routing table above). Confirm with the user if uncertain.

### 3. Content Comprehension Brief

Before writing detailed annotations, present a brief:

- **Type**: document category
- **Structure**: section outline
- **Key entities**: technical terms, named entities, recurring concepts
- **Data elements**: tables, charts, figures mentioned
- **Gaps**: any text that could not be extracted (images, diagrams, scanned content)

### 4. Generate Annotations

Produce annotations following the type-specific rules below.

### 5. Deliver Output

Save annotations to the appropriate output format:

- **PPTX**: Create `notes.json` (format: `[{"slide": N, "notes": "..."}]`), then run:
  ```bash
  python scripts/inject_pptx_notes.py --input "<file>.pptx" --output "<file>-annotated.pptx" --notes "<notes>.json"
  ```

- **DOCX**: Append notes section using:
  ```bash
  python scripts/write_docx_notes.py --input "<file>.docx" --output "<file>-annotated.docx" --notes "<notes>.json"
  ```

- **All other types**: Generate a `.md` file using:
  ```bash
  python scripts/write_md_notes.py --input "<file>" --output "<file>.md" --notes "<notes>.json" --type "<type>"
  ```

## Type-Specific Annotation Guidelines

### PPTX — Speaker Notes

For each slide, produce two versions:

**Display version** (shown to user):
```text
[Slide X - Title]
----------------
Spoken text with [PAUSE] and [EMPHASIS: term] markers.

Transition: one sentence bridging to the next slide.
```

**Clean version** (injected into PPTX):
- No slide labels, separators, markers, or transition lines
- Pure spoken prose, paragraph form

Per-slide rules:
- Open with the slide's thesis sentence
- Address every visible text element
- For tables: explain what rows/columns represent
- For charts: state the headline, axes, and key values
- Write as coherent oral argument, not as slide captions

### PDF — Reading Notes

Structure:
```markdown
## Abstract
(1-paragraph summary in your own words)

## Key Findings
- Finding 1
- Finding 2

## Methods
(techniques, datasets, frameworks used)

## Important Figures/Tables
(describe each major visual element)

## Critical Assessment
(strengths, limitations, open questions)
```

### DOCX — Document Summary

Structure:
```markdown
## Document Overview
(1-paragraph summary)

## Section-by-Section Notes
### Section Title
(key points from this section)

## Key Terms
| Term | Definition |
|------|-----------|
```

### XLSX / CSV — Data Insights

Structure:
```markdown
## Data Overview
(rows, columns, time range, categories)

## Key Insights
- Insight 1 (with specific numbers)
- Insight 2

## Notable Trends / Anomalies
(patterns, outliers, comparisons)

## Data Quality Notes
(missing values, formatting issues, assumptions)
```

### Audio — Meeting Minutes / Highlights

Structure:
```markdown
## Summary
(1-paragraph)

## Key Points
- Speaker: point

## Action Items
- [ ] Task (Owner)

## Decisions Made
- Decision
```

### YouTube — Content Summary

Structure:
```markdown
## Video Summary

## Key Timestamps
| Time | Topic |
|------|-------|

## Main Arguments
```

## Prerequisites

All scripts require Python with the packages listed in `requirements.txt`. The recommended setup uses the markitdown virtual environment:

```
E:\Projects\技能\markitdown 文本转换\.venv\
```

Set the `DOC_ANNOTATOR_VENV` environment variable to point to a custom venv, or modify script shebangs to your Python path.

```bash
pip install -r requirements.txt
```

## Usage Examples

```
User: "帮我给这个 PPT 生成演讲注释"
→ Detects .pptx → converts → generates speaker notes → injects into PPTX

User: "Summarize this PDF paper"
→ Detects .pdf → converts → generates reading notes → saves as .md

User: "Analyze this Excel spreadsheet"
→ Detects .xlsx → converts → generates data insights → saves as .md

User: "Transcribe and summarize this meeting recording"
→ Detects .mp3 → converts (speech-to-text) → generates minutes → saves as .md
```
