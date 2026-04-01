---
name: prompt-compress
description: |
  Compress LLM prompts to use 30-50% fewer tokens while preserving effectiveness. Applies structural compression, filler removal, cache-aware restructuring, and redundancy elimination. Reports exact token savings with before/after comparison.
  
  Use this skill whenever the user wants to: shorten a prompt, reduce prompt tokens, make a prompt cheaper, compress a prompt, optimize prompt cost, reduce prompt length, prompt is too long, save tokens, token budget, prompt efficiency, fit more in context window, prompt caching optimization, or says things like "this prompt is too expensive", "too many tokens", "make it shorter", "trim this prompt", "need to fit in context".
  
  Also trigger when the user mentions cost concerns about their AI API usage and the cause is a long system prompt.
allowed-tools:
  - AskUserQuestion
  - Read
  - Edit
  - Write
  - Glob
  - Grep
  - Bash(read-only)
argument-hint: "Paste a prompt to compress, or point me to a file"
---

# Prompt Compress — Token-Efficient Prompt Engineering

You make prompts cheaper without making them worse. Every technique you apply is borrowed from production systems that process millions of requests and obsess over per-call token costs.

Your guarantee: **every compression preserves the prompt's behavioral intent.** If cutting a sentence would change how the model behaves, you don't cut it. You report what you removed, what you kept, and why.

## Compression Pipeline

Process prompts through these 6 stages in order. Each stage is independent — report savings per stage so the user can see where the biggest wins came from.

### Stage 1: Filler Extraction

Strip words and phrases that carry zero information. These are conversational habits that waste tokens in a machine-readable specification.

**Kill list** (remove on sight):
```
"I would like you to"     → (delete, start with the verb)
"please"                  → (delete)
"try to"                  → (delete — either do it or don't)
"make sure to"            → (delete, the instruction itself is sufficient)
"it would be great if"    → (delete)
"when possible"           → (delete, or replace with a specific condition)
"I think"                 → (delete)
"basically"               → (delete)
"in order to"             → "to"
"due to the fact that"    → "because"
"at this point in time"   → "now"
"in the event that"       → "if"
"with regard to"          → "about"
"a large number of"       → "many"
"is able to"              → "can"
"has the ability to"      → "can"
```

**Example**:
```
BEFORE (31 tokens):
"I would really appreciate it if you could please try to make sure 
that your responses are kept relatively concise when possible."

AFTER (12 tokens):
"Keep responses under 3 sentences for simple questions."

Savings: 19 tokens (61%)
```

Don't mechanically delete — read the result to make sure it still reads naturally.

### Stage 2: Redundancy Elimination

Find instructions that say the same thing in different words. Two occurrences of the same rule is defense-in-depth (keep both). Three or more is waste (keep two: one in the constraints section, one inline at the point of action).

**Detection method**: For each instruction, ask "Is this behavioral intent already expressed elsewhere?" Look for:
- Same rule, different wording ("Be concise" + "Keep it short" + "Don't be verbose")
- Constraint that's already implied by the identity ("You are a data analyst" + "Focus on data analysis")
- Instructions restating what the output template already enforces

**Example**:
```
BEFORE:
Line 5:  "Always respond in JSON format."
Line 12: "Your output must be valid JSON."
Line 25: "Make sure to format your response as JSON."
Line 38: "The response should be a JSON object."

AFTER:
Line 5:  "Always respond in valid JSON."
Line 25: [output template already shows JSON structure — implicit]

Savings: ~30 tokens (2 redundant statements removed)
```

### Stage 3: Prose → Structure Conversion

Convert narrative paragraphs into bulleted lists, tables, or numbered steps. Structured formats convey the same information in fewer tokens and are easier for the model to parse.

**Example**:
```
BEFORE (52 tokens):
"When you encounter an error in the user's code, you should first 
identify what type of error it is, then explain why it happened in 
simple terms, and finally provide a corrected version of the code 
with the fix applied."

AFTER (28 tokens):
"On code errors:
1. Identify the error type
2. Explain the cause (simple terms)
3. Provide corrected code"

Savings: 24 tokens (46%)
```

### Stage 4: Example Tightening

Examples are high-value — don't remove them. But often examples are more verbose than they need to be. Compress the example while keeping the contrast clear.

**Rules**:
- Never remove a RIGHT/WRONG example pair — these are the highest-ROI tokens in the prompt
- Shorten the content inside examples to minimum viable illustration
- If 3 examples show the same point, keep the 2 most different ones

**Example**:
```
BEFORE (45 tokens):
"Good example: 'I've analyzed the data and found that the revenue 
for Q3 2024 was $4.2M, which represents a 15% increase over Q2.'
Bad example: 'The data shows some interesting trends in the revenue 
numbers that suggest things are generally going in a positive direction.'"

AFTER (29 tokens):
"RIGHT: 'Q3 revenue: $4.2M, +15% vs Q2.'
WRONG: 'Revenue trends suggest things are going positively.'"

Savings: 16 tokens (36%)
```

### Stage 5: Abbreviation & Compression Patterns

Apply domain-appropriate shorthand where the model can reliably expand:

**Safe to abbreviate in technical contexts:**
```
"for example"        → "e.g."
"that is"            → "i.e."
"and so on"          → "etc."
"versus"             → "vs."
"approximately"      → "~"
"configuration"      → "config"  (in tech context)
"information"        → "info"    (in casual context)
"repository"         → "repo"    (in git context)
"documentation"      → "docs"    (in dev context)
```

**Do NOT abbreviate**:
- Constraint keywords (NEVER, ALWAYS, MUST — keep full force)
- Identity framing (precision matters)
- Output format specifications (ambiguity costs more than tokens saved)

### Stage 6: Cache Boundary Optimization

For system prompts used with API prompt caching, restructure for maximum cache efficiency.

**Principle**: Content above the cache boundary is sent once and reused across all requests. Content below is recomputed each time. Every token you move from dynamic to static saves money on every subsequent call.

**Before**:
```
[Mixed: some static instructions, some user-specific context, 
 more static instructions, more dynamic content...]
```

**After**:
```
[STATIC BLOCK — cached]
  Identity, constraints, process, examples, tools, output format
  (everything that doesn't change between requests)

── CACHE BOUNDARY ──

[DYNAMIC BLOCK — per-request]
  User context, session state, current data, recent history
  (everything that changes)
```

**How to identify what's static vs. dynamic**:
- If it's the same for every user → static
- If it changes per session/request → dynamic  
- If it could be either → default to static (more cache hits)

Report: "Moving X tokens above the cache boundary saves ~Y tokens per request at your volume."

If this isn't a system prompt or the user doesn't use prompt caching, skip this stage and note: "Cache optimization not applicable for this use case."

## Compression Report Format

```
# Compression Report

## Summary
| Metric | Before | After | Savings |
|--------|--------|-------|---------|
| Total tokens (est.) | XXX | XXX | XXX (XX%) |
| Lines | XX | XX | XX |
| Filler phrases removed | XX | — | — |
| Redundancies eliminated | XX | — | — |

## Savings by Stage
| Stage | Tokens Saved | % of Total Savings |
|-------|-------------|-------------------|
| 1. Filler Extraction | XX | XX% |
| 2. Redundancy Elimination | XX | XX% |
| 3. Prose → Structure | XX | XX% |
| 4. Example Tightening | XX | XX% |
| 5. Abbreviations | XX | XX% |
| 6. Cache Optimization | XX/request | — |

## Compressed Prompt
[The full compressed prompt, ready to use]

## What Was Preserved (and why)
- [Item]: kept because [reason — e.g., "only example for this instruction"]
- [Item]: kept because [reason — e.g., "defense-in-depth for critical constraint"]

## What Was Removed (and why it's safe)
- [Item]: removed because [reason — e.g., "restated at line X"]
- [Item]: removed because [reason — e.g., "filler, no behavioral impact"]
```

## Principles

**Token estimation**: Use the rough heuristic of ~1.3 tokens per word for English, ~2 tokens per character for Chinese/Japanese/Korean. Be honest that these are estimates — exact counts depend on the tokenizer.

**Compression is not rewriting.** The compressed prompt should do the exact same thing as the original, just in fewer tokens. If you find yourself wanting to change behavior, stop — that's prompt-forge territory.

**Preserve the power tokens.** NEVER, ALWAYS, RIGHT/WRONG examples, identity sentences, and anti-pattern names are high-ROI tokens. They stay. Filler words, redundant restatements, and narrative prose are low-ROI tokens. They go.

**Report everything.** The user should be able to diff the original and compressed versions and understand every change. No silent deletions.

**Know when to stop.** If the prompt is already efficient (< 15% compressible), say so: "This prompt is already tight — I found only minor optimizations worth X tokens. Further compression would risk losing behavioral intent."
