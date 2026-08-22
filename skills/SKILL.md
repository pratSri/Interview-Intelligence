---
name: interview-intelligence
description: Use this skill whenever the user wants the latest publicly reported interview questions, interview experiences, or evidence-based preparation priorities for a specific company, role, seniority level, country, or interview round. Use it for requests such as "latest Staff SWE questions at Coupang," "what was asked in the first screen," or "rank likely system-design questions." The expected outcome is a current, source-backed shortlist of concrete questions ranked by evidence strength, not a generic interview-preparation guide.
compatibility: Requires an AI agent or LLM with live web-search/browsing capability and the ability to open public sources. Works with ChatGPT, Claude, Gemini, and agent frameworks that support web research.
metadata:
   version: "1.0.0"
   category: "interview-research"
   output: "evidence-ranked interview questions"
---

# Interview Intelligence

Research the latest publicly available interview experiences for a supplied company, role, level, country, and interview round. Extract concrete reported questions, deduplicate repeated patterns, score the strength of the evidence, and return only **Very High** and **High** confidence preparation targets.

The score represents **research confidence**, not the literal probability that a question will be asked.

## Required Behavior

1. Use live public web research for every request involving "latest," "recent," "current," or a specific company interview process.
2. Never answer a latest-interview-question request solely from model memory.
3. Search multiple independent source families rather than relying on one website or one candidate report.
4. Open the underlying source whenever accessible; do not rely only on search-result snippets.
5. Distinguish clearly between:
   - **Directly reported** — a candidate publicly stated that the question was asked.
   - **Repeated pattern** — materially similar questions appeared in multiple independent reports.
   - **Evidence-based inference** — not directly reported for the exact target, but supported by adjacent reports, current interview structure, or role requirements.
6. Never fabricate exact wording, interview experiences, dates, citations, or source claims.
7. Never claim access to private question banks, internal recruiter systems, private communities, or unpublished internal notes.
8. Return a finished evidence-ranked preparation result, not merely search instructions.
9. If evidence is sparse, say so explicitly and return fewer questions rather than padding the result.

## Inputs

### Required

- `company_name` — target company, for example `Coupang`
- `level` — target seniority, for example `Staff Software Engineer`, `L6`, `Senior Staff`, `E6`

### Optional

- `role` — default: `Software Engineer`
- `country` — default: `USA`
- `round` — for example `First Screen`, `Coding`, `System Design`, `Hiring Manager`, `Bar Raiser`, `Full Loop`
- `lookback_months` — default: `12`
- `max_questions` — default: `30`
- `language` — default: `English`

If either required input is missing, ask for all missing required values in one request. Do not require optional inputs before researching.

## Defaults

Unless the user specifies otherwise:

```yaml
role: Software Engineer
country: USA
lookback_months: 12
max_questions: 30
language: English
```

If `round` is omitted, research the full publicly reported interview loop and preserve the reported round for each question.

## Execution

### 1. Normalize the target

Resolve aliases before searching, but do not invent company-specific level equivalence.

Examples:

- `Staff SWE` -> `Staff Software Engineer`
- `SDE III`, `L6`, or `Staff` may overlap at some companies, but treat them as equivalent only when public evidence supports it.
- `Senior Staff`, `Principal`, `L7`, and similar senior levels remain distinct unless credible company-specific evidence shows otherwise.

Create an internal target record:

```text
Company:
Role:
Level:
Country:
Round:
Lookback window:
```

### 2. Search the live web

Use several query families. Adapt wording to the company and current year.

#### General queries

```text
"{company_name}" "{level}" interview experience {country}
"{company_name}" "{level}" software engineer interview {year}
"{company_name}" "{role}" interview questions {year}
"{company_name}" staff engineer interview experience {country}
```

#### Round-specific queries

```text
"{company_name}" "{level}" "phone screen"
"{company_name}" "{level}" "first round"
"{company_name}" "{level}" "technical screen"
"{company_name}" "{level}" "coding interview"
"{company_name}" "{level}" "system design"
"{company_name}" "{level}" "hiring manager"
"{company_name}" "{level}" "bar raiser"
```

#### Source-targeted queries

```text
site:leetcode.com/discuss "{company_name}" "{level}"
site:teamblind.com "{company_name}" "{level}" interview
site:glassdoor.com "{company_name}" "{level}" interview
site:reddit.com "{company_name}" interview "{level}"
site:medium.com "{company_name}" interview experience
```

#### Official-source queries

```text
site:{company_domain} interview tips software engineer
site:{company_domain} engineering interview
site:{company_domain} leadership principles interview
site:{company_domain} {role} {level} job description
```

Use official material to establish interview format, competencies, or current role emphasis. Do **not** treat a job description as proof that a specific question is asked.

### 3. Prefer the strongest sources

Prioritize, in roughly this order:

1. Detailed first-person candidate reports with role, level, country, and round context.
2. Recent LeetCode Discuss interview experiences.
3. Blind reports with clear role/level/country context.
4. Glassdoor reports with identifiable role/level/country context.
5. First-person Reddit posts or detailed public engineering blogs.
6. Official company guidance for process, competencies, and format.
7. Current job descriptions as supporting domain evidence only.

Treat anonymous forum claims as evidence, not verified company policy.

Do not bypass authentication, paywalls, robots restrictions, private groups, or other access controls.

### 4. Prioritize recency correctly

Prefer the **interview date** when the source provides it. Otherwise use the **publication date**.

Use these recency points in scoring:

| Evidence age | Points |
|---|---:|
| <= 3 months | 15 |
| <= 6 months | 14 |
| <= 12 months | 12 |
| <= 24 months | 8 |
| > 24 months | 4 |

Do not discard an older question when the same pattern continues to appear in newer independent reports.

### 5. Extract structured evidence

For every useful report, capture internally:

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

When a source gives only a topic and not exact wording, label the final question as **Pattern / inferred wording**.

### 6. Deduplicate questions

Merge semantically equivalent questions before counting independent reports.

Examples:

```text
"Design Uber"
"Design a ride-sharing system"
"Build an Uber-like matching platform"
```

becomes:

```text
Design a ride-sharing / Uber-like system.
```

Similarly:

```text
"LFU Cache"
"Implement LFU with O(1) get/put"
```

becomes one LFU Cache question with all evidence-backed follow-ups attached.

Do not count reposts, mirrors, copied interview reports, or citations of the same original report as independent evidence.

### 7. Score evidence strength

Score each deduplicated question from `0` to `100`.

#### A. Level match — 0 to 20

- `20` — exact level
- `14` — directly adjacent level
- `8` — same role family but materially different seniority
- `0` — unrelated

#### B. Role/domain match — 0 to 10

- `10` — exact role/domain
- `7` — closely related backend/platform/full-stack role
- `3` — generic software engineering role
- `0` — unrelated domain

#### C. Country match — 0 to 10

- `10` — same country
- `5` — different country but same company and level
- `0` — materially unrelated or unknown country context

#### D. Round match — 0 to 10

- `10` — explicitly reported in the requested round
- `6` — same loop, round not explicit
- `2` — different round
- `0` — no round relevance

If the user did not request a round, score a clearly identified reported round as `10`; do not penalize it for the absence of a round filter.

#### E. Recency — 0 to 15

- `15` — within 3 months
- `14` — within 6 months
- `12` — within 12 months
- `8` — within 24 months
- `4` — older than 24 months
- `0` — date cannot be established and no current corroboration exists

#### F. Independent repetition — 0 to 20

- `20` — 4+ independent recent reports
- `16` — 3 independent reports
- `12` — 2 independent reports
- `6` — 1 direct report
- `0` — inference only

#### G. Evidence directness — 0 to 15

- `15` — exact question explicitly published by a candidate
- `11` — detailed description clearly identifies the problem or pattern
- `6` — topic only
- `3` — inference from interview structure or job requirements
- `0` — unsupported speculation

### 8. Assign likelihood labels

Use only these labels in the primary result:

- **Very High** — score `>= 80`
- **High** — score `65–79`
- **Below 65** — omit from the primary table

The score is a research-confidence score. Never describe it as a guaranteed probability of being asked.

A single weak source should not become **Very High** merely because other contextual fields match. Use judgment when evidence quality conflicts with the raw score and explain any conservative downgrade.

## Output

Begin with a concise research summary:

```text
Research scope: <lookback window>
Target: <company> — <role/level> — <country> — <round or full loop>
Sources reviewed: <number of useful sources reviewed, when known>
Most recent relevant interview report: <date or unavailable>
Evidence quality: <Strong | Moderate | Sparse>
```

Then return this table:

| Rank | Likelihood | Score | Question / Problem | Category | Round Evidence | Evidence Type | Most Recent Evidence | Independent Reports | Why It Ranks High | Sources |
|---:|---|---:|---|---|---|---|---|---:|---|---|
| 1 | Very High | 86 | ... | Coding | First Screen | Direct + repeated | Aug 2026 | 3 | Exact level, U.S., recent repetition | citations |

### Table rules

- Sort by score descending.
- Show **Very High** before **High**.
- Return no more than `max_questions`.
- Do not force the output to reach `max_questions`.
- Keep questions concrete enough to prepare from directly.
- Mark unknown exact wording as `Pattern / inferred wording`.
- Cite every row with the strongest available evidence.
- Prefer multiple independent citations for **Very High** rows when available.
- Keep direct reports separate from inference in the `Evidence Type` column.

## Follow-Up Questions

For every **Very High** technical question, add `3–8` follow-ups when the evidence supports them.

Label each follow-up as one of:

- **Reported follow-up** — explicitly described in a public candidate report.
- **Likely Staff-level follow-up** — inferred from the problem and seniority expectations.

Never present inferred follow-ups as directly reported.

Example:

```text
### LFU Cache

1. Reported follow-up — Why is get/put O(1)?
2. Reported follow-up — How are equal-frequency ties resolved?
3. Likely Staff-level follow-up — How would you make it thread-safe?
4. Likely Staff-level follow-up — How would you distribute this across multiple servers?
```

## Preparation Priority

Finish with a compact evidence-driven preparation plan.

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

Base the ordering on research evidence for this target, not generic interview popularity.

## Failure Handling

### Live web access unavailable

State:

```text
I cannot guarantee the latest interview questions without live web access.
```

Do not silently fall back to model memory and label the result as current.

### Sources blocked or inaccessible

Use other public sources. Do not bypass the restriction. If a search snippet is the only available evidence, label that evidence as weak and do not treat it as sufficient for a **Very High** result by itself.

### Sparse evidence

Say that evidence is sparse, explain the closest evidence available, and return only rows that satisfy the confidence threshold. Do not fill space with generic LeetCode, system-design, or behavioral questions.

### Conflicting reports

Prefer the most recent, exact-level, exact-country, and exact-round evidence. Preserve disagreement when it materially affects ranking.

## Scope

- Public interview research only.
- No private, leaked, confidential, paywalled, authenticated, or access-controlled question banks.
- No claims that a question is guaranteed.
- No fabricated citations or candidate reports.
- Current job descriptions may influence domain weighting but cannot prove an exact interview question.
- Reposts and copied reports count once.

## Example Invocation

```yaml
company_name: Coupang
level: Staff Software Engineer
role: Backend Software Engineer
country: USA
round: First Screen
lookback_months: 12
max_questions: 20
language: English
```

Expected behavior:

1. Search current public sources.
2. Find recent U.S. Staff-level reports for the target company and round.
3. Extract concrete coding, behavioral, system-design, and relevant follow-up questions actually supported by sources.
4. Deduplicate semantically equivalent reports.
5. Score the evidence.
6. Return only **Very High** and **High** rows with citations.
7. Add evidence-labeled follow-ups for **Very High** technical questions.
8. Finish with prioritized preparation guidance.

## Compact Prompt Fallback

Use this only when the host platform cannot load the full skill file:

```text
Act as an Interview Intelligence Researcher.

Inputs:
Company: {{company_name}}
Level: {{level}}
Role: {{role | default: Software Engineer}}
Country: {{country | default: USA}}
Round: {{round | optional}}
Lookback: {{lookback_months | default: 12}} months
Max questions: {{max_questions | default: 30}}

Use live public web research to find the latest interview experiences for this exact company, role, level, country, and round. Prioritize detailed firsthand reports and recent LeetCode Discuss, Blind, Glassdoor, Reddit, public blogs, and official company interview guidance. Open underlying sources when accessible rather than relying only on snippets.

Extract each reported question, interview/publication date, role, level, country, round, evidence type, follow-ups, and source. Deduplicate semantically equivalent questions and do not double-count reposts.

Rank each question using exact-level match, role/domain match, country, round, recency, independent repetition, and evidence directness.

Return only:
- Very High: score >= 80
- High: score 65-79

The score is research confidence, not a guaranteed probability.

Output:
| Rank | Likelihood | Score | Question / Problem | Category | Round Evidence | Evidence Type | Most Recent Evidence | Independent Reports | Why It Ranks High | Sources |

For every Very High technical question, add evidence-labeled follow-ups: Reported follow-up or Likely Staff-level follow-up.

Finish with the top five preparation priorities plus a 2-hour and 1-day preparation plan. If evidence is sparse, say so and return fewer questions rather than inventing or padding results.
```
