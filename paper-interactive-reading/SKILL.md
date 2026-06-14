---
name: paper-interactive-reading
description: "Interactive paper reading through fill-in-the-blank Q&A. Use when the user wants to read, understand, or study a research paper (PDF) through guided conversation. Covers: extracting PDF text, three-pass reading (bird's eye, speed read, rubber duck), generating fillable note templates, quizzing the user to test understanding, and providing targeted feedback. Triggers on: read paper, help me understand this paper, quiz me on this paper, make reading notes."
---

# Paper Interactive Reading

## Overview

Guide the user through reading a research paper using an interactive fill-in-the-blank method. Never dump a full summary - instead, ask questions, let the user answer, then correct and fill gaps.

## Workflow

### Step 1: Extract PDF text

Use `pdfplumber` to extract text from the PDF. If encoding errors occur, set `sys.stdout.reconfigure(encoding='utf-8')`.

```python
import pdfplumber, sys
sys.stdout.reconfigure(encoding='utf-8')
with pdfplumber.open(path) as pdf:
    for i, page in enumerate(pdf.pages):
        text = page.extract_text()
        if text:
            print(f'--- Page {i+1} ---')
            print(text)
```

### Step 2: Three-pass reading

Apply three passes in order. Do NOT read linearly from the beginning.

**Pass 1 - Bird's eye**: Scan title, abstract, headings, figures, tables, conclusion. Produce:
> This paper is mainly about X. It solves Y. Its main method is Z.

**Pass 2 - Speed read**: Extract minimum concept set. For each concept, provide a plain-language explanation first, then technical wording.

**Pass 3 - Rubber duck**: Produce a speakable explanation using this template:
1. This paper mainly talks about...
2. Why it matters...
3. Previous methods fail because...
4. The core idea here is...
5. How it works step by step...
6. What I should remember...

### Step 3: Generate fillable note template

Generate a concise Markdown note template for the user to fill in. See `references/note-template.md` for the structure. Keep it to one page - the user should finish filling in 10-15 minutes.

### Step 4: Guide the user to fill in sections

After generating the template, guide the user through filling in each section interactively:

**Step 4a - One-sentence summary**:
> Formula: "Using [method A + method B], solving [problem], achieving [result on dataset]."

**Step 4b - Problems to solve**:
> Ask the user: "What did previous methods fail at?"
> Common types: low accuracy, low efficiency, poor generalization, missing ability, heavy dependency

**Step 4c - Core method (per module)**:
> Formula per module:
> "What it does = [input] -> [processing] -> [output]"
> "Why needed = without it -> [what problem occurs]"

**Step 4d - Key experimental results**:
> Best result = [dataset] on [metric]=[value]
> Ablation finding = removing [module X] causes [drop], meaning [module X] is key
> Key parameter = optimal [K=?], reason in one sentence

**Step 4e - Innovation**:
> Formula: "Previous methods = [how], but have [problem]. This paper = [how], through [mechanism] solving it."

**Step 4f - Weaknesses** (four angles):
> 1. Depends on what? (model, data, hardware)
> 2. Tested where? (simulation vs real)
> 3. What is not done? (future work)
> 4. What assumption is too ideal?

**Step 4g - Inspiration**:
> Formula: "This paper tells me: [core idea]. I can use this on: [my problem]. Specifically: [one sentence]."

### Step 5: Robot perspective explanation

After filling in notes, ask the user to describe the method as if they were the robot executing the algorithm. This tests deep understanding.

> Prompt: "If you are the robot, describe what happens step by step when you start from the entrance."

This reveals whether the user truly understands the process flow or just memorized concepts.

### Step 6: Quiz the user

After the user has read and filled in notes, quiz them with 5 questions from shallow to deep:

1. **Basic** - A factual question from the paper
2. **Understanding** - Requires explaining a concept in their own words
3. **Mechanism** - Asks about how a specific module works
4. **Detail** - Requires recalling a formula or specific parameter
5. **Critical thinking** - Asks "why" (e.g., why does ablation show X?)

### Step 7: Evaluate and give feedback

After the user answers:
- Score each answer (correct, partial, wrong)
- Give a percentage estimate of their understanding
- Identify weak areas and suggest specific sections to re-read
- Use this formula for weakness diagnosis:

| Weakness | Suggested re-read |
|----------|------------------|
| Overall framework unclear | Figure/Framework section |
| Mechanism details missing | Method section with formulas |
| Experiment analysis weak | Ablation study section |
| Can't explain causally | Re-read with "why" questions |
| Can't describe process | Re-read and practice robot perspective |

### Step 8: Paper comparison (if multiple papers)

When the user has read multiple related papers, generate a comparison table:

| Dimension | Paper A | Paper B |
|-----------|---------|---------|
| Core idea | | |
| Decision method | | |
| Dependency | | |
| Best result | | |

Ask the user: "Could you combine ideas from both papers?"

## Key Principles

- Never dump full summaries - always let the user try first
- Questions should go from shallow to deep (3-5 questions)
- After each answer, give immediate feedback before next question
- Estimate understanding percentage based on answer quality
- Suggest specific sections to re-read based on weak answers
- Keep fill-in templates concise (one page max)
- Use the "robot perspective" to test true understanding vs memorization
- When user asks "how to fill X", provide the formula + this paper's example + one-sentence version
- When asking the user any fill-in, reflection, or quiz question, explicitly mark where that question comes from in the paper before or after the question.

## Source Citation Rule

**Every answer, feedback, or correction MUST include a source citation from the paper.** Format:

```
出处：[Section/章节名], [页码], [Table/Figure 编号]（如适用）
```

Examples:
- "出处：Section 4.2 Experiments, 第 7 页, Table 3"
- "出处：Appendix A1 Limitations, 第 13 页"
- "出处：Section 3.1 Dataset Characteristics, 第 5 页, Figure 4"

**This applies to:**
1. When evaluating the user's answer (state which section supports or contradicts their answer)
2. When providing the correct answer (always show where to find it in the paper)
3. When answering "是什么" or "为什么" questions from the user
4. When generating fillable note templates (each bullet point should note its source)
5. When asking the user questions during guided reading or quizzes (show the source section/page/figure/table for each question)

**Question source format:**
Before or after each question, add:

```
问题出处：[Section/章节名], [页码], [Table/Figure 编号]（如适用）
```

If a question synthesizes multiple parts of the paper, cite all relevant locations, for example:
"问题出处：Section 3 Method，第 4 页；Section 4.3 Ablation，第 8 页，Table 4"

**When the user asks a question directly** (e.g., "帕累托最优是什么？"):
- First answer the question clearly
- Then cite where in the paper this concept appears (page, section, figure/table)
- Optionally quote the relevant passage from the paper

This helps the user build the habit of verifying claims against the original text.
