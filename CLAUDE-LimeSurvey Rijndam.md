# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with LimeSurvey survey structure files for **Rijndam Revalidatiecentrum**.

This is the **general base file** for all LimeSurvey questionnaire projects. Each project directory should reference or copy this file and add a questionnaire-specific section at the bottom.

---

## Project context

This workspace generates **LimeSurvey survey structure files** (`.lss`) for clinical questionnaires at Rijndam Revalidatiecentrum. The output files are XML-based and imported directly into LimeSurvey.

Typical workflow:
1. Write or update an R script (`generate_<NAME>.R`) that builds the XML
2. Run the script to produce a date-stamped `.lss` file
3. Import the `.lss` file into LimeSurvey

Source documents (PDFs, answer keys) and the template `.lss` live in a `Basis documenten <NAME>/` subfolder. Output versions go in date folders (`YYYY-MM-DD/`).

---

## How to run

```bash
"C:/Program Files/R/R-4.5.2/bin/Rscript.exe" generate_<NAME>.R
```

Output: `YYYY-MM-DD/<NAME>.lss`

To generate to a new date folder, update `out_dir` at the top of the R script.

---

## LimeSurvey .lss file structure

A `.lss` file is XML with `DBVersion 640`. The required sections **in order** are:

1. `<answers>` — answer options (per question via `qid`)
2. `<answer_l10ns>` — localized answer display text
3. `<groups>` + `<group_l10ns>` — question group(s)
4. `<questions>` — parent questions
5. `<subquestions>` + `<question_l10ns>` — sub-questions and displayed texts
6. `<question_attributes>` — per-question settings
7. `<surveys>` — survey-level settings
8. `<surveys_languagesettings>` — title, email templates, redirect URL
9. `<themes>` + `<themes_inherited>` — visual theme config

All text content goes inside `<![CDATA[...]]>` blocks.

---

## Rijndam standard settings

These settings are the same for all surveys and are taken from `PDI (simpel).lss`:

| Field | Value |
|---|---|
| Template | `rijndam2-patient` |
| Admin | `Robbert Wouters` |
| Adminemail / bounce_email | `zorgdata@rijndam.nl` |
| Format | `G` (group-by-group) |
| Language | `nl` |
| Redirect URL | `https://zorgdata.rijndam.nl/ask/return/id/{TOKEN}` |
| Redirect label | `Terug naar Rijndam Zorgdata` |
| Access_mode | `C` |
| Tokenlength | `15` |
| Autoredirect / Allowprev / Savetimings / Showprogress / Tokenanswerspersistence | `Y` |

**SID**: Use a placeholder like `100001` or `900001`. LimeSurvey assigns the real ID on import.

---

## Question types

| Type | LimeSurvey code | Use case |
|------|----------------|----------|
| Radio list | `L` | Single choice with answer options as rows; best for items with descriptive text and variable score ranges |
| Array matrix | `F` | Matrix with sub-question rows and answer option columns; used for bilateral items (R/L) with a shared scale |
| Equation | `*` | Computed/hidden field; used for scores and averages |

### Answer options for type L
- Each answer has `aid`, `qid`, `code` (numeric), `sortorder`, `assessment_value` (= code), `scale_id` (= 0)
- Display text goes in `<answer_l10ns>` with matching `aid`

### Sub-questions for type F
- Sub-questions define the rows (e.g. `R` = Rechts, `L` = Links)
- Answer options define the columns
- Referenced in equations as `{V5_R.NAOK}` (title of parent + `_` + title of sub-question)

---

## Expression Manager reference syntax

| Context | Syntax | Example |
|---------|--------|---------|
| Type L question | `{TITLE.NAOK}` | `{V1.NAOK}` |
| Type F sub-question | `{PARENT_SUB.NAOK}` | `{V5_R.NAOK}` |
| Reverse scored item (max=2) | `(2-{Qnnn.NAOK})` | `(2-{Q002.NAOK})` |
| Average of bilateral | `({VA_R.NAOK}+{VA_L.NAOK})/2` | |

Always use `.NAOK` (Not Applicable OK) to avoid errors when a question is unanswered.

---

## XML encoding notes

- Inside `<![CDATA[...]]>` sections, use HTML entities for special characters:
  - `&lt;` → `<`, `&gt;` → `>`, `&amp;` → `&`
  - `&#8211;` → en-dash (–), `&#233;` → `é`
- Answer display text format: prefix with score number and equals sign — `0 = Normaal`, `1 = Lichte moeilijkheden...` — so the numeric score is visible in the radio list.
- Instruction box styling (Rijndam huisstijl):
  ```html
  <div style="border:4px solid #14AE5C;background-color:#ffffff;padding:8px 12px;margin:4px 0;border-radius:3px;">
  ```
  Use this to visually separate the instruction paragraph from the question title.
- Answer `code` values must be numeric strings for equation calculations to work.
- The `id` field in `<answer_l10ns>` must match the `aid` of the corresponding answer row.

---

## R script structure (typical)

An R script generating a `.lss` file is usually divided into:

1. **Item data** — vector or list of question texts, matching the source PDF exactly
2. **Scoring definitions** — reverse items, scale definitions
3. **Helper functions** — e.g. `cd()` (CDATA wrapper), `eq_term()`, `build_eq()`, `add()` (XML line accumulator)
4. **XML builder** — sequentially emits all LimeSurvey XML sections using `add()`

---

## Versioning convention

New output files go in a date folder: `YYYY-MM-DD/filename.lss`.  
Iterative versions in the same session can be named descriptively (e.g. `SARA Halve titel.Vinstructie.Vantwoorden.lss`).

---

## Questionnaire-specific section

> **Fill in this section for each new questionnaire project.**

```
## <QUESTIONNAIRE NAME> specifics

**Source**: `Basis documenten <NAME>/<source file>.pdf`

**SID placeholder**: `<e.g. 100003>`

### Survey groups

| gid | Name | Content |
|-----|------|---------|
| 1 | Instructie | Descriptive group, no questions |
| 2 | Vragen | ... |
| 3 | Scores | Equation questions, all hidden |

### Items

| Item | Type | Scale | Notes |
|------|------|-------|-------|
| ... | L / F | 0–n | ... |

### Score equation(s)

```
{...}
```

### Reverse items (if applicable)

Items n, n, n, ... score reversed (0→max, max→0).

### Answer option format

...
```
