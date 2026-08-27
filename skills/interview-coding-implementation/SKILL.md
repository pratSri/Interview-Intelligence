
---
name: interview-coding-implementation
description: Use this skill when the user asks to implement coding questions from an interview-question list or interview research result, including requests such as "yes, implement them," "implement the coding questions," "solve the first coding question," "next coding question," or "work through the follow-ups." Ask for the preferred programming language unless it is already known, implement one coding question at a time in ranked order, keep the solution concise and interview-ready, and preserve the distinction between reported and inferred follow-ups.
---

# Interview Coding Implementation

Implement interview coding questions sequentially, one problem at a time. This skill is designed to continue from an evidence-ranked question inventory produced by the `interview-intelligence-research` skill, but it can also implement a coding question explicitly supplied by the user.

## Requirements

- Do not dump solutions for every coding question at once.
- Implement only one coding question per turn unless the user explicitly asks for multiple implementations.
- Prefer the highest-ranked unimplemented coding question from the available interview-research context.
- Keep implementations concise, runnable, and interview-ready.
- Do not invent missing reported details. When a public interview report provides only a pattern, state the assumptions used to make the problem implementable.
- Keep **Reported follow-up** separate from **Likely Staff-level follow-up**.

## Trigger and Context Selection

Use this skill when the user asks to implement, solve, code, continue, or work through coding questions from the current interview-preparation context.

Determine the current coding question in this order:

1. If the user explicitly names a coding question, use that question.
2. Otherwise, if an evidence-ranked interview inventory is available in the conversation, select the **highest-ranked unimplemented coding question**.
3. If the user says `next`, select the next-highest-ranked unimplemented coding question.
4. If no coding question or ranked inventory is available, ask the user to provide the coding question or the interview-question inventory.

Preserve ranking and evidence labels from the research result when they are available.

## Preferred Programming Language

Before the first implementation, determine `preferred_coding_language`.

- If the user already specified a preferred programming language in the current conversation, reuse it.
- Otherwise ask:

```text
What programming language would you like to use?
```

- Once chosen, reuse the same language for later coding questions unless the user changes it.
- Do not confuse response language such as English with the programming language.

## Sequential Implementation Workflow

### 1. Implement only the current question

For the first implementation, choose the **highest-ranked coding question** available.

For later turns, choose the next-highest-ranked unimplemented coding question unless the user selects a different one.

### 2. Use short format

Keep each implementation compact. Use this structure:

#### Question

State the concrete problem briefly.

If exact wording was unavailable in the source, say:

```text
Pattern / reconstructed statement based on the reported question.
```

#### Approach

Use `2–5` concise bullets covering:

- core data structure or algorithm;
- key invariant or insight;
- why it satisfies the required constraints.

Avoid long tutorial-style explanations unless the user asks for more depth.

#### Complexity

Use one compact line, for example:

```text
Time: O(n) | Space: O(n)
```

For data structures with multiple operations, state complexity per operation.

#### Implementation

Provide complete runnable code in `preferred_coding_language`.

Prefer:

- standard library types where appropriate;
- modern language features when they improve clarity;
- interview-readable naming;
- minimal helper classes;
- no unnecessary framework or boilerplate.

#### Edge Cases

List the `3–6` most important edge cases.

#### Quick Tests / Walkthrough

Provide `2–4` focused tests or one concise walkthrough that validates the main behavior and edge cases.

#### Interview Explanation

Give a short explanation the candidate could say aloud, generally `3–6` sentences. Cover:

- why this approach was selected;
- the invariant or key idea;
- complexity;
- one likely tradeoff or follow-up.

### 3. Stop after one implementation

After finishing the current question, do not automatically solve the next one.

Ask:

```text
Would you like to:
1. Implement the next coding question, or
2. Work through follow-up questions for this question?
```

## Next Coding Question

If the user chooses the next coding question:

1. Keep the same preferred programming language unless changed by the user.
2. Select the next-highest-ranked unimplemented coding question.
3. Use the same short implementation format.
4. Stop after that one implementation.
5. Ask the same two-option continuation question again.

When all coding questions are complete, state that the coding inventory is exhausted and offer to work through follow-ups or another category from the research result.

## Follow-Up Workflow

If the user chooses follow-ups for the current question, preserve evidence provenance.

### Reported follow-up

Use this label only when the research evidence explicitly reported that follow-up.

Example:

```text
Reported follow-up — Why is get/put O(1)?
```

### Likely Staff-level follow-up

Use this label for an inferred question based on the problem, target seniority, or common engineering depth.

Example:

```text
Likely Staff-level follow-up — How would you make this thread-safe?
```

Never present an inferred follow-up as directly reported.

### Interactive handling

- Prefer discussing one follow-up at a time when the user is practicing interactively.
- For conceptual follow-ups, answer concisely first.
- For implementation-oriented follow-ups such as thread safety, distributed design, optimization, API changes, or alternate constraints, modify or extend code only when the user asks for implementation.
- After each follow-up, ask whether the user wants another follow-up or the next coding question.

## Assumptions and Incomplete Problem Statements

Interview reports often mention only a problem name or pattern. When the full statement is unavailable:

1. Preserve the original evidence label.
2. State only the minimum assumptions necessary to implement the problem.
3. Do not claim reconstructed constraints were directly reported.
4. Prefer standard canonical constraints when needed, but clearly label them as assumptions.

Example:

```text
Assumption: the report names "LFU Cache" but does not publish the complete prompt. I will use the standard get/put LFU formulation with LRU tie-breaking.
```

## Quality Rules

- Prefer optimal or interview-appropriate solutions.
- Explain why the complexity target is met.
- Use complete runnable code rather than pseudocode unless the user requests pseudocode.
- Avoid unnecessary abstraction.
- Use built-in collections when they meet the complexity requirement.
- Create helper structures only when standard collections cannot satisfy the required operations cleanly.
- Call out tradeoffs that are likely to matter at Senior, Staff, or Principal level.
- Do not change the original question silently to make the implementation easier.

## Example Continuation

Research result contains ranked coding questions:

```text
1. Very High — LFU Cache
2. High — All O(1) Data Structure
3. Medium — Course Schedule
```

User:

```text
Yes, implement them.
```

Expected behavior:

1. Ask for programming language if not already known.
2. Implement `LFU Cache` only.
3. Include approach, complexity, runnable code, edge cases, quick tests/walkthrough, and interview explanation.
4. Ask whether to implement `All O(1) Data Structure` next or work through LFU follow-ups.
5. Continue one question per turn.
