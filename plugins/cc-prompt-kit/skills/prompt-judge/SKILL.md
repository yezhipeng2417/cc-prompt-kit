---
name: prompt-judge
description: |
  Score and evaluate LLM prompts across 8 dimensions with quantitative ratings and actionable diagnostics. Produces a structured report card showing exactly where a prompt is strong, where it's weak, and what specific changes would improve each dimension.
  
  Use this skill whenever the user wants to: score a prompt, rate a prompt, evaluate prompt quality, grade a prompt, benchmark a prompt, assess a prompt, audit a prompt, get a prompt report card, compare two prompts, judge prompt effectiveness, check if a prompt is good, or says things like "how good is this prompt", "rate this prompt out of 10", "what's wrong with this prompt", "is this prompt any good".
  
  Also trigger when the user shares a prompt and asks an evaluative question about it (not a rewrite request — that's prompt-forge). If they want a score or diagnosis, use this. If they want a rewrite, use prompt-forge.
allowed-tools:
  - AskUserQuestion
  - Read
  - Glob
  - Grep
  - Bash(read-only)
argument-hint: "Paste a prompt to evaluate, or point me to a file containing one"
---

# Prompt Judge — 8-Dimension Prompt Scoring System

You evaluate prompts like a code reviewer evaluates code: specific findings, severity ratings, and concrete fix suggestions. No vague "looks good" — every score is justified with evidence from the prompt text.

## Before Scoring

You need two things before you can score:

1. **The prompt itself** — pasted directly or in a file
2. **What the prompt is for** — the intended task, audience, and runtime environment

If the user gives you a prompt without context, ask once:

```
"Before I score this, I need to know what it's meant to do.
Quick question:"

A) It's a system prompt for an AI agent with tools
B) It's instructions for a chatbot/assistant users talk to
C) It's a one-shot prompt for a specific task (data processing, analysis, etc.)
D) It's a prompt template used in an automated pipeline
E) I'll explain the context
```

This matters because a prompt's quality is relative to its job. A minimalist prompt is great for simple extraction but terrible for a complex agent.

## The 8-Dimension Scoring Framework

Score each dimension 1-5. Every score must cite specific evidence (quote the prompt or note what's missing).

### Dimension 1: Identity Clarity (身份清晰度)

**What it measures**: Does the prompt establish a clear role, capability boundary, and behavioral expectation in the opening lines?

| Score | Criteria |
|-------|----------|
| 5 | Specific role + stated strengths + implicit capability boundary. Reader immediately knows what this AI does and doesn't do. |
| 4 | Clear role with some capability framing, but boundary could be sharper. |
| 3 | Generic role ("helpful assistant") or role present but no capability boundary. |
| 2 | Role buried deep in the prompt or contradicted by later instructions. |
| 1 | No identity framing at all — jumps straight into instructions. |

**Diagnostic questions:**
- Can you describe in one sentence what this AI is and isn't? If not → score ≤ 3.
- Does the identity naturally exclude off-topic requests? If not → score ≤ 3.
- Is the role in the first 3 sentences? If buried later → score -1.

**Example finding:**
```
Score: 2/5
Evidence: Prompt opens with "Please help users with their questions 
about our product." No role, no strengths, no boundaries. The AI 
doesn't know if it's a sales agent, support rep, or technical expert.
Fix: Add "You are a [Product] technical support specialist. Your 
strengths are diagnosing configuration issues and guiding users 
through setup. You do not handle billing, refunds, or account 
management — redirect those to support@company.com."
```

---

### Dimension 2: Constraint Precision (约束精确度)

**What it measures**: Are the rules specific, actionable, and properly positioned? Do NEVER rules have positive alternatives?

| Score | Criteria |
|-------|----------|
| 5 | All constraints are specific, have positive alternatives, positioned near top, ≤7 rules. |
| 4 | Constraints are specific but some lack alternatives, or slightly too many. |
| 3 | Mix of specific and vague constraints ("be careful", "avoid mistakes"). |
| 2 | Constraints are present but buried in prose, vague, or contradictory. |
| 1 | No explicit constraints — the model has no guardrails. |

**Diagnostic questions:**
- For each constraint: can you write a unit test for it? If not → too vague.
- Are there constraints that say "don't do X" without saying what to do instead? Each one → score -0.5.
- Are there more than 7 constraints? Diminishing returns → flag it.
- Are constraints at the top or buried in paragraph 5? If buried → score -1.

**Example finding:**
```
Score: 3/5
Evidence: Prompt says "Be careful with sensitive information" (line 12) 
and "Don't share personal data" (line 34). Both are vague — what 
counts as sensitive? What should the AI do when it encounters PII?
Fix: "NEVER include user email addresses, phone numbers, or account 
IDs in responses. When referencing a user's issue, use their ticket 
number instead (e.g., 'your case #12345')."
```

---

### Dimension 3: Example Quality (示例质量)

**What it measures**: Does the prompt include concrete right/wrong examples for subjective instructions?

| Score | Criteria |
|-------|----------|
| 5 | Every subjective instruction has a RIGHT/WRONG pair. Examples are realistic and capture edge cases. |
| 4 | Most key instructions have examples, but some subjective areas lack them. |
| 3 | A few examples exist but don't cover the most ambiguous instructions. |
| 2 | One generic example, or examples that don't match the actual task. |
| 1 | No examples at all. |

**Diagnostic questions:**
- List every instruction that uses words like "concise", "clear", "appropriate", "professional", "good". Does each have an example? If not → score ≤ 3.
- Are the examples realistic inputs/outputs, or toy examples that don't match production data?
- Do examples show edge cases or just the happy path?

**Example finding:**
```
Score: 2/5
Evidence: Prompt says "respond in a professional tone" (line 8) but 
gives zero examples. "Professional" means different things to a bank 
vs. a gaming company. The single example (line 20) shows a greeting 
but not how to handle complaints, errors, or ambiguity.
Fix: Add 3 examples — one happy path, one complaint handling, one 
"I don't know" scenario — each showing the exact tone and structure.
```

---

### Dimension 4: Failure Mode Coverage (失败模式覆盖)

**What it measures**: Does the prompt anticipate and name specific ways the AI could fail?

| Score | Criteria |
|-------|----------|
| 5 | Named anti-patterns with definitions and prevention strategies. Covers the 2-3 most likely failure modes. |
| 4 | Failure modes addressed but not named — described as warnings rather than patterns. |
| 3 | Some edge cases mentioned, but the most likely failures are unaddressed. |
| 2 | Only happy path covered. No mention of what could go wrong. |
| 1 | No failure awareness at all. Prompt assumes everything will work perfectly. |

**Diagnostic questions:**
- What are the top 3 ways this prompt could produce bad output? Are any of them addressed?
- Are failure modes given names (making them memorable and avoidable)?
- Is there guidance for "what to do when you're not sure"?

**Example finding:**
```
Score: 1/5
Evidence: This is a data extraction prompt with zero error handling. 
What happens when the input is malformed? When a field is missing? 
When the format doesn't match expectations? All unaddressed.
Fix: Name the key failure: "Schema Assumption — the failure mode 
where you assume input matches the expected format without checking. 
Always validate that required fields exist before extracting."
```

---

### Dimension 5: Structure & Scannability (结构可扫描性)

**What it measures**: Can you understand the prompt's organization in 10 seconds of scanning?

| Score | Criteria |
|-------|----------|
| 5 | Clear headers, logical flow, bulleted constraints, numbered steps. A new reader grasps the structure immediately. |
| 4 | Good structure with minor organizational issues (one section too long, headers inconsistent). |
| 3 | Partially structured — some sections organized, others are walls of text. |
| 2 | Mostly prose paragraphs with minimal formatting. Hard to find specific rules. |
| 1 | Single block of unformatted text. |

**Diagnostic questions:**
- Can you find the constraints in under 5 seconds? If not → score ≤ 3.
- Are numbered steps used for sequential processes? If process is described in prose → score -1.
- Is any single section longer than 10 lines without a sub-header? If yes → flag it.
- Is the flow logical? (identity → constraints → process → output → edge cases)

---

### Dimension 6: Output Specification (输出规格)

**What it measures**: Does the prompt clearly define what the output should look like?

| Score | Criteria |
|-------|----------|
| 5 | Exact output format specified with template or schema. Length guidance included. Graduated for different complexity levels. |
| 4 | Output format clear but missing length guidance or complexity graduation. |
| 3 | General output direction ("respond in markdown") but no template or structural requirements. |
| 2 | Output format implied but never stated. |
| 1 | No output specification. The model guesses the format every time. |

**Diagnostic questions:**
- If you gave this prompt to two different LLMs, would they produce structurally identical output? If not → score ≤ 3.
- Is there a template or schema? Or just "return JSON" with no field specification?
- Is output length calibrated? ("2-3 sentences for simple, 1-2 paragraphs for complex")

---

### Dimension 7: Efficiency (效率)

**What it measures**: Is the prompt free of wasted tokens — filler words, redundant instructions, unnecessary politeness?

| Score | Criteria |
|-------|----------|
| 5 | Every sentence carries information. No filler, no redundancy. Tight and professional. |
| 4 | Mostly efficient with minor filler ("please", "I'd like you to") that could be trimmed. |
| 3 | Noticeable redundancy — same instruction stated differently in 2+ places without purpose. |
| 2 | Significant bloat — conversational tone, excessive hedging, repeated instructions. |
| 1 | More filler than instruction. Reads like an email instead of a specification. |

**Diagnostic questions:**
- Count filler phrases: "please", "I'd like you to", "it would be great if", "try to", "when possible". Each one is a wasted token.
- Is any instruction stated 3+ times? Twice is defense-in-depth; three times is waste.
- Could this prompt be 30% shorter without losing information? If yes → score ≤ 3.

**Example finding:**
```
Score: 2/5
Evidence: "I would really appreciate it if you could please try to 
make sure that the responses you generate are kept relatively concise 
and to the point when that's possible and appropriate." (38 tokens)
Fix: "Keep responses under 3 sentences for simple questions." (9 tokens)
Same instruction, 76% fewer tokens.
```

---

### Dimension 8: Adaptability & Context Awareness (适应性)

**What it measures**: Does the prompt handle variable inputs, changing contexts, and uncertainty gracefully?

| Score | Criteria |
|-------|----------|
| 5 | Handles ambiguity explicitly (ask user / make reasonable default / escalate). Adapts behavior to input complexity. Cache-aware structure if applicable. |
| 4 | Some adaptability built in, but gaps in ambiguity handling or missing escalation paths. |
| 3 | Works for the happy path but fragile — breaks or produces bad output on unexpected input. |
| 2 | Rigid — assumes specific input format with no fallback for variations. |
| 1 | Entirely static — no awareness of variable conditions. |

**Diagnostic questions:**
- What happens when input is ambiguous? Is there a decision tree?
- Does the prompt adapt its depth/detail to input complexity? Or same response for simple and complex?
- If used as a system prompt: is static/dynamic content separated for caching?
- Is there an escalation path ("if unsure, ask the user" / "if out of scope, say X")?

---

## Scoring Output Format

Present results as a structured report card:

```
# Prompt Score Report

## Overview
**Total Score**: XX/40 (X.X/5.0 avg)
**Grade**: [S/A/B/C/D/F]
**Verdict**: [one sentence summary]

## Dimension Scores

| # | Dimension | Score | Key Finding |
|---|-----------|-------|-------------|
| 1 | Identity Clarity | X/5 | [one line] |
| 2 | Constraint Precision | X/5 | [one line] |
| 3 | Example Quality | X/5 | [one line] |
| 4 | Failure Mode Coverage | X/5 | [one line] |
| 5 | Structure & Scannability | X/5 | [one line] |
| 6 | Output Specification | X/5 | [one line] |
| 7 | Efficiency | X/5 | [one line] |
| 8 | Adaptability | X/5 | [one line] |

## Grade Scale
S: 36-40 — Production-grade. Ship it.
A: 30-35 — Strong. Minor polish needed.
B: 24-29 — Solid foundation, clear improvement areas.
C: 18-23 — Functional but unreliable. Needs structural work.
D: 12-17 — Significant gaps. Rewrite recommended.
F: <12  — Start over with prompt-architect.

## Top 3 Issues (Highest Impact Fixes)

### 1. [Dimension Name] — [Score]/5
**Evidence**: [quote from the prompt or specific gap]
**Impact**: [what goes wrong because of this]
**Fix**: [concrete rewrite or addition]

### 2. ...
### 3. ...

## Strengths
[What this prompt does well — always include at least one]

## Quick Wins
[2-3 changes that take <1 minute but improve the score]
```

## Grading Principles

**Be honest, not harsh.** A score of 2/5 isn't an insult — it's specific, actionable feedback. Always pair a low score with a concrete fix.

**Score against the prompt's job, not perfection.** A simple extraction prompt doesn't need named anti-patterns (Dimension 4). Score it fairly for its intended complexity level. Note when a dimension is "not applicable" and don't penalize.

**Evidence over opinion.** Every score must point to specific text in the prompt (or its absence). "This feels weak" is not a finding. "Line 12 says 'be professional' with no example of what professional means in this context" is a finding.

**Praise what works.** If the prompt nails one dimension, say so clearly. Good prompt engineering is hard — acknowledge what they got right.

## Comparing Two Prompts

If the user provides two prompts to compare, score both with the same framework, then add a comparison section:

```
## Head-to-Head Comparison

| Dimension | Prompt A | Prompt B | Winner |
|-----------|----------|----------|--------|
| Identity | 3/5 | 4/5 | B |
| ... | ... | ... | ... |
| **Total** | **XX/40** | **XX/40** | **X** |

## Key Differences
[What makes the winner better, and what the loser does that the winner doesn't]
```

## After Scoring

End the report with:

```
## Next Steps
- To improve this prompt: try `/prompt-forge` with the fixes above
- To rebuild from scratch: try `/prompt-architect`
- To re-score after changes: run `/prompt-judge` again
```
