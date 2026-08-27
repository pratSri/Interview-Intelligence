---
name: interview-intelligence-research
description: Use this skill whenever the user wants the latest publicly reported interview questions, interview experiences, or evidence-ranked preparation priorities for a specific company, role, seniority level, country, or interview round. Retrieve all evidence-backed questions, deduplicate them, rank them from Very High through Low confidence, and cite the supporting public sources. If coding questions are found, finish by asking whether the user wants to implement them one by one.
---

# Interview Intelligence Research

Research the latest publicly available interview experiences for a supplied company, role, level, country, and interview round. Extract **all evidence-backed questions found**, deduplicate repeated patterns, score the evidence strength, and rank the complete inventory.

The score represents **research confidence**, not the literal probability that a question will be asked.

## Requirements

- Use live public web search and browsing for current company-specific interview research.
- Never answer a request for latest or recent interview questions solely from model memory.
- Use multiple independent source families whenever possible.
- Open underlying sources when accessible instead of relying only on search-result snippets.
- Use only publicly available information.
- Do not bypass authentication, paywalls, robots restrictions, private communities, or access controls.
- Never fabricate questions, candidate reports, dates, citations, source claims, or private interviewer information.

## Inputs

### Required

- `company_name` — target company, for example `Coupang`
- `level` — target seniority, for example `Staff Software Engineer`, `L6`, `Senior Staff`, `E6`

### Optional

- `role` — default: `Software Engineer`
- `country` — default: `USA`
- `round` — for example `First Screen`, `Coding`, `System Design`, `Hiring Manager`, `Bar Raiser`, `Full Loop`
- `lookback_months` — default: `12`
- `max_questions` — optional result cap; if omitted, return all unique evidence-backed questions found
- `language` — response language; default: `English`

If a required input is missing, request all missing required values together. Do not require optional inputs before researching.

## Defaults

```yaml
role: Software Engineer
country: USA
lookback_months: 12
language: English
```

If `round` is omitted, research the full publicly reported interview loop and preserve the reported round for each question.

## Required Behavior

1. Retrieve **all unique evidence-backed questions** found within the requested research scope.
2. Do not silently discard lower-confidence questions that have genuine source support.
3. Use these confidence labels:
   - **Very High** — score `>= 80`
   - **High** — score `65–79`
   - **Medium** — score `45–64`
   - **Low** — score `< 45`
4. Treat `max_questions` as optional. When omitted, return the full deduplicated evidence-backed inventory.
5. Distinguish evidence types:
   - **Directly reported** — a candidate publicly stated that the question was asked.
   - **Repeated pattern** — materially similar questions appeared in multiple independent reports.
   - **Evidence-based inference** — not directly reported for the exact target, but supported by adjacent reports, current interview structure, or role requirements.
6. Classify questions when supported by the evidence, for example:
   - `Coding`
   - `System Design`
   - `Debugging`
   - `Behavioral`
   - `Technical Discussion`
7. After the research result, if at least one `Coding` question exists, ask:

```text
I found <N> coding questions. Would you like to implement them one by one?
```

8. Do **not** implement coding questions in this skill. Implementation belongs to the separate `interview-coding-implementation` skill.

## Research Workflow

### 1. Normalize the target

Resolve aliases before searching, but do not invent company-specific level equivalence.

Examples:

- `Staff SWE` -> `Staff Software Engineer`
- `SDE III`, `L6`, or `Staff` may overlap at some companies, but treat them as equivalent only when credible public evidence supports it.
- `Senior Staff`, `Principal`, `L7`, and similar senior levels remain distinct unless company-specific evidence shows otherwise.

Internal target:

```text
Company:
Role:
Level:
Country:
Round:
Lookback window:
```

### 2. Search multiple query families

Adapt wording to the company, role, level, round, and current year.

#### General

```text
"{company_name}" "{level}" interview experience {country}
"{company_name}" "{level}" software engineer interview {year}
"{company_name}" "{role}" interview questions {year}
"{company_name}" staff engineer interview experience {country}
```

#### Round-specific

```text
"{company_name}" "{level}" "phone screen"
"{company_name}" "{level}" "first round"
"{company_name}" "{level}" "technical screen"
"{company_name}" "{level}" "coding interview"
"{company_name}" "{level}" "system design"
"{company_name}" "{level}" "hiring manager"
"{company_name}" "{level}" "bar raiser"
```

#### Source-targeted

```text
site:leetcode.com/discuss "{company_name}" "{level}"
site:teamblind.com "{company_name}" "{level}" interview
site:glassdoor.com "{company_name}" "{level}" interview
site:reddit.com "{company_name}" interview "{level}"
site:medium.com "{company_name}" interview experience
```

#### Official

```text
site:{company_domain} interview tips software engineer
site:{company_domain} engineering interview
site:{company_domain} leadership principles interview
site:{company_domain} {role} {level} job description
```

Use official sources for interview format, competencies, and role emphasis. A job description is supporting evidence only and is not proof that a specific question is asked.

### 3. Prefer stronger evidence

Prioritize roughly in this order:

1. Detailed first-person candidate reports with role, level, country, and round context.
2. Recent LeetCode Discuss interview experiences.
3. Blind reports with clear role/level/country context.
4. Glassdoor reports with identifiable role/level/country context.
5. First-person Reddit posts or detailed public engineering blogs.
6. Official company interview guidance.
7. Current job descriptions as supporting domain evidence only.

Anonymous forum claims are evidence, not verified company policy.

### 4. Handle recency

Prefer the **interview date** when available. Otherwise use the **publication date**.

| Evidence age | Recency points |
|---|---:|
| <= 3 months | 15 |
| <= 6 months | 14 |
| <= 12 months | 12 |
| <= 24 months | 8 |
| > 24 months | 4 |

Do not discard older questions when the same pattern continues to appear in newer independent reports.

### 5. Extract structured evidence

For each useful report, capture internally:

```text
Source URL
Source type
Publication date
Interview date, if available
Company
Role
Level
Country
Interview round
Question or question pattern
Exact wording available? yes/no
Candidate outcome, if available
Technical topics
Reported follow-ups
```

When exact wording is not available, label the final item as **Pattern / inferred wording**.

### 6. Deduplicate

Merge semantically equivalent questions before counting independent reports.

Example:

```text
"Design Uber"
"Design a ride-sharing system"
"Build an Uber-like matching platform"
```

becomes:

```text
Design a ride-sharing / Uber-like system.
```

Likewise, `LFU Cache` and `Implement LFU with O(1) get/put` become one question with all evidence-backed follow-ups attached.

Do not count reposts, mirrors, copied reports, or citations of the same original report as independent evidence.

## Evidence Scoring

Score each deduplicated question from `0` to `100`.

### A. Level match — 0 to 20

- `20` — exact level
- `14` — directly adjacent level
- `8` — same role family but materially different seniority
- `0` — unrelated

### B. Role/domain match — 0 to 10

- `10` — exact role/domain
- `7` — closely related backend/platform/full-stack role
- `3` — generic software engineering role
- `0` — unrelated domain

### C. Country match — 0 to 10

- `10` — same country
- `5` — different country but same company and level
- `0` — materially unrelated or unknown country context

### D. Round match — 0 to 10

- `10` — explicitly reported in the requested round
- `6` — same loop, round not explicit
- `2` — different round
- `0` — no round relevance

If no round was requested, give a clearly identified reported round `10`; do not penalize the absence of a round filter.

### E. Recency — 0 to 15

- `15` — within 3 months
- `14` — within 6 months
- `12` — within 12 months
- `8` — within 24 months
- `4` — older than 24 months
- `0` — date cannot be established and there is no current corroboration

### F. Independent repetition — 0 to 20

- `20` — 4+ independent recent reports
- `16` — 3 independent reports
- `12` — 2 independent reports
- `6` — 1 direct report
- `0` — inference only

### G. Evidence directness — 0 to 15

- `15` — exact question explicitly published by a candidate
- `11` — detailed description clearly identifies the problem or pattern
- `6` — topic only
- `3` — inference from interview structure or role requirements
- `0` — unsupported speculation

## Confidence Labels

- **Very High** — `>= 80`
- **High** — `65–79`
- **Medium** — `45–64`
- **Low** — `< 45`

The score is a research-confidence score, not a guaranteed probability.

A weak single source should not become **Very High** merely because contextual fields match. When raw scoring conflicts with evidence quality, conservatively downgrade and explain why.

## Output

Begin with:

```text
Research scope: <lookback window>
Target: <company> — <role/level> — <country> — <round or full loop>
Sources reviewed: <number of useful sources reviewed, when known>
Most recent relevant interview report: <date or unavailable>
Evidence quality: <Strong | Moderate | Sparse>
```

Then return the **complete evidence-backed question inventory**:

| Rank | Likelihood | Score | Question / Problem | Category | Round Evidence | Evidence Type | Most Recent Evidence | Independent Reports | Why It Ranks Here | Sources |
|---:|---|---:|---|---|---|---|---|---:|---|---|

### Table Rules

- Sort by score descending.
- Preserve label order: **Very High**, **High**, **Medium**, **Low**.
- When `max_questions` is omitted, return all unique evidence-backed questions found.
- Apply `max_questions` only when explicitly supplied.
- If the cap truncates the result, state how many additional lower-ranked questions were found.
- Do not pad the table merely to reach a count.
- Keep questions concrete enough to prepare directly.
- Mark unknown exact wording as `Pattern / inferred wording`.
- Cite every row.
- Prefer multiple independent citations for **Very High** rows when available.
- Keep direct reports separate from inference in `Evidence Type`.

## Follow-Up Questions

For every **Very High** technical question, add `3–8` follow-ups when evidence supports them.

Always label each follow-up as:

- **Reported follow-up** — explicitly described in a public candidate report.
- **Likely Staff-level follow-up** — inferred from the problem and seniority expectations.

Never present inferred follow-ups as directly reported.

## Preparation Priority

Finish with:

```text
Prepare first:
1. <highest-confidence question>
2. <second-highest-confidence question>
3. <third-highest-confidence question>
4. <fourth-highest-confidence question>
5. <fifth-highest-confidence question>

If only 2 hours:
- <highest-value subset>

If only 1 day:
- <broader ordered plan>
```

Prioritize **Very High** and **High** questions first.

## Implementation Handoff

If one or more `Coding` questions were found, end with:

```text
I found <N> coding questions. Would you like to implement them one by one?
```

Do not include implementation code in this skill.

If no coding questions were found, state that clearly and do not invent coding questions.

## Failure Handling

### Live web access unavailable

State:

```text
I cannot guarantee the latest interview questions without live web access.
```

Do not silently fall back to model memory and label the answer as current.

### Sources blocked or inaccessible

Use other public sources. Do not bypass restrictions. A search snippet alone is weak evidence and is not sufficient for **Very High** by itself.

### Sparse evidence

Say that evidence is sparse and return only genuinely supported questions. Lower-confidence supported rows may still appear as **Medium** or **Low**.

### Conflicting reports

Prefer more recent, exact-level, exact-country, and exact-round evidence. Preserve disagreement when it materially affects ranking.

## Scope

- Public interview research only.
- No private, leaked, confidential, paywalled, authenticated, or access-controlled question banks.
- No guaranteed-question claims.
- No fabricated citations or candidate reports.
- Job descriptions may influence domain weighting but cannot prove exact questions.
- Reposts and copied reports count once.

## Example Invocation

```yaml
company_name: Coupang
level: Staff Software Engineer
role: Backend Software Engineer
country: USA
round: First Screen
lookback_months: 12
language: English
```

Expected behavior:

1. Search current public sources.
2. Extract all concrete evidence-backed questions.
3. Deduplicate semantically equivalent reports.
4. Score evidence from `0–100`.
5. Return the complete inventory using **Very High**, **High**, **Medium**, and **Low** labels.
6. Add evidence-labeled follow-ups for Very High technical questions.
7. Provide preparation priorities.
8. If coding questions exist, ask whether the user wants to implement them one by one.
