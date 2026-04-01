# cc-prompt-kit

A prompt engineering toolkit for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), with techniques extracted from analyzing Claude Code's own internal prompt architecture (512K+ lines of TypeScript source).

This isn't theoretical prompt engineering advice — every technique in this toolkit traces back to a specific pattern found in production-grade AI system prompts that handle millions of interactions daily.

## What's Inside

Five skills forming a complete prompt engineering workflow:

```
prompt-architect ──→ prompt-forge ──→ prompt-judge
   (build)            (refine)         (score)
                         ↑                │
                         │                ↓
                    prompt-debug    prompt-compress
                    (diagnose)      (optimize cost)
```

| Skill | Command | Purpose |
|-------|---------|---------|
| **prompt-architect** | `/prompt-architect` | Build prompts from scratch via guided interview |
| **prompt-forge** | `/prompt-forge` | Refine and polish existing prompts |
| **prompt-judge** | `/prompt-judge` | Score prompts across 8 dimensions (1-5 each, 40 max) |
| **prompt-debug** | `/prompt-debug` | Diagnose why a prompt produces unexpected behavior |
| **prompt-compress** | `/prompt-compress` | Compress prompts 30-50% while preserving effectiveness |

## Installation

### Via Marketplace (Recommended)

In Claude Code, run:

```
/plugin
```

Then enter the marketplace source:

```
louisyeaaah/cc-prompt-kit
```

Select `cc-prompt-kit` from the plugin list and install. Done.

### Manual Installation

```bash
git clone https://github.com/louisyeaaah/cc-prompt-kit.git ~/.claude/plugins/cc-prompt-kit
```

Then in Claude Code, run `/reload-plugins` to activate.

## Skills in Detail

### prompt-architect — Build from Zero

Guides you through a structured interview to build a prompt from nothing. Asks the right questions first, then selects from 5 structural patterns:

| Pattern | Best For |
|---------|----------|
| **Simple Instruction** | Single-step, clear output tasks |
| **Agent Prompt** | Tool-using AI with decision-making |
| **Pipeline Prompt** | Automated, no-human-in-loop systems |
| **Persona Prompt** | User-facing chatbots/assistants |
| **Cached System Prompt** | High-volume, cost-sensitive APIs |

After selecting a pattern, it constructs the prompt by applying all 8 techniques (see [Theoretical Foundation](#theoretical-foundation) below) in sequence.

### prompt-forge — Refine Existing Prompts

Takes a prompt that works but could be better and applies targeted improvements. The process:

1. **Understand** — What is this prompt's job? Where does it fail?
2. **Diagnose** — Evaluate against 8 dimensions
3. **Rewrite** — Apply structural improvements
4. **Present** — Show changes with rationale for each

Uses `AskUserQuestion` to clarify ambiguous areas before making changes.

### prompt-judge — Score & Grade

Quantitative evaluation across 8 dimensions, each scored 1-5:

| # | Dimension | What It Measures |
|---|-----------|-----------------|
| 1 | **Identity Clarity** | Is the role specific enough to constrain behavior? |
| 2 | **Constraint Precision** | Are rules specific, testable, with positive alternatives? |
| 3 | **Example Quality** | Do subjective instructions have RIGHT/WRONG pairs? |
| 4 | **Failure Mode Coverage** | Are likely failure modes named and addressed? |
| 5 | **Structure & Scannability** | Can you find any rule in under 5 seconds? |
| 6 | **Output Specification** | Would two LLMs produce structurally identical output? |
| 7 | **Efficiency** | Is every token earning its place? |
| 8 | **Adaptability** | Does it handle ambiguity and variable complexity? |

**Grade scale:**

| Grade | Score | Meaning |
|-------|-------|---------|
| S | 36-40 | Production-grade. Ship it. |
| A | 30-35 | Strong. Minor polish needed. |
| B | 24-29 | Solid foundation, clear improvement areas. |
| C | 18-23 | Functional but unreliable. Needs work. |
| D | 12-17 | Significant gaps. Rewrite recommended. |
| F | < 12 | Start over with prompt-architect. |

Each low score comes with evidence (quotes from the prompt), impact analysis, and a concrete fix.

### prompt-debug — Symptom-Driven Diagnosis

When your prompt misbehaves ("it does X but I want Y"), prompt-debug traces the symptom to a structural root cause. Nine fault patterns:

| Fault | Symptom | Root Cause |
|-------|---------|------------|
| **Buried Constraint** | AI ignores a rule that IS in the prompt | Rule positioned too late or in prose |
| **Naked Subjective** | Output quality varies wildly between runs | Subjective terms ("concise") without examples |
| **Identity Vacuum** | Generic, shallow responses | No role/capability framing |
| **Constraint Without Alternative** | AI avoids the bad thing but does nothing useful | NEVER rule without a "do this instead" |
| **Process Ambiguity** | Steps done in wrong order or skipped | Workflow described in prose, not numbered steps |
| **Format Drift** | Output structure varies between runs | No output template |
| **Tool Blindness** | AI won't use available tools | Tool descriptions don't say WHEN to use them |
| **Context Starvation** | Correct but shallow answers | Dynamic context not injected, or lost in long prompt |
| **Overconstraint Paralysis** | Overly safe, bland, minimal output | Too many rules (>7) or contradictions |

Prescribes the minimum fix for the specific symptom — doesn't rewrite the whole prompt.

### prompt-compress — Token Optimization

6-stage compression pipeline targeting 30-50% token reduction:

| Stage | Technique | Typical Savings |
|-------|-----------|-----------------|
| 1 | **Filler Extraction** | Remove "please", "I'd like you to", "try to" | 
| 2 | **Redundancy Elimination** | Same rule stated 3+ times → keep 2 |
| 3 | **Prose → Structure** | Paragraphs → bulleted lists (40-50% per section) |
| 4 | **Example Tightening** | Compress example content, keep the contrast |
| 5 | **Abbreviations** | Domain-safe shorthand (e.g. → "config", "repo") |
| 6 | **Cache Boundary** | Separate static/dynamic for API prompt caching |

Outputs a compression report with exact token savings per stage and a before/after comparison.

**Hard rules**: Never removes RIGHT/WRONG example pairs, NEVER/ALWAYS keywords, identity sentences, or anti-pattern names — these are "power tokens" with the highest behavioral ROI.

## Theoretical Foundation

All techniques in this toolkit are derived from analyzing the internal prompt architecture of Claude Code. Here's what we found and how each skill applies it:

### The 8 Core Techniques

#### 1. Identity Framing
**Source**: Every Claude Code agent (Explore, Plan, Verification, General-Purpose) opens with a precise `role + strengths + constraints` definition. The Verification Agent opens with "You are a verification specialist. Your job is to try to BREAK it" — the identity itself defines the behavioral boundary.

**Applied in**: prompt-architect (Phase 3, Technique 1), prompt-forge (Dimension 1), prompt-judge (Dimension 1), prompt-debug (Fault 3: Identity Vacuum)

#### 2. Negative Constraints with Positive Alternatives
**Source**: Claude Code places all NEVER rules in the top 20% of the system prompt, and every prohibition has a corresponding "instead, do Y". Example from the source: `"NEVER skip hooks (--no-verify). If a hook fails, investigate and fix the underlying issue."` — the prohibition and the alternative in the same sentence.

**Applied in**: prompt-architect (Technique 2), prompt-forge (Dimension 2), prompt-judge (Dimension 2), prompt-debug (Fault 1 & 4), prompt-compress (preserved as "power tokens")

#### 3. Contrastive Examples (Not X, But Y)
**Source**: Claude Code's BashTool description pairs every instruction with concrete right/wrong examples. The tool description parameter guidance includes: `ls → "List files in current directory"` as RIGHT, and for complex commands shows the detailed alternative. Every subjective quality standard is anchored to examples, not adjectives.

**Applied in**: prompt-architect (Technique 3), prompt-forge (Dimension 3), prompt-judge (Dimension 3), prompt-debug (Fault 2: Naked Subjective), prompt-compress (Stage 4 — compress but never remove)

#### 4. Anti-Pattern Naming
**Source**: The Verification Agent's prompt directly names failure modes: "Verification Avoidance" (reading code instead of running it) and "Seduced by 80%" (declaring pass after first few checks). By giving failures memorable labels, the model treats them as concepts to actively avoid.

**Applied in**: prompt-architect (Technique 4), prompt-debug (all 9 named fault patterns), prompt-judge (Dimension 4)

#### 5. Process Templates
**Source**: Claude Code enforces rigid step sequences everywhere — the Plan Agent's 4-phase process (Understand → Explore → Design → Detail), the Compact prompt's 9-section summary structure, the commit command's numbered workflow. Numbered steps prevent reordering and skipping.

**Applied in**: prompt-architect (Technique 5 + Pattern selection), prompt-forge (Phase 3), prompt-judge (Dimension 5), prompt-debug (Fault 5: Process Ambiguity)

#### 6. Graduated Specificity
**Source**: Claude Code's BashTool description gives brief guidance for simple commands ("5-10 words") and detailed guidance for complex ones. The Explore Agent accepts three thoroughness levels: "quick", "medium", "very thorough". The system matches instruction detail to situation complexity.

**Applied in**: prompt-architect (Technique 6), prompt-debug (Fault 9: Overconstraint Paralysis), prompt-compress (Stage 3)

#### 7. Economic Reasoning
**Source**: The Agent Tool prompt explains "Forks are CHEAP — they share the parent's prompt cache" and "Don't set model on fork — different model can't reuse parent cache." The system prompt says "The Grep tool is optimized for this environment" rather than "You must use Grep." Explaining WHY an approach is efficient persuades better than mandating it.

**Applied in**: prompt-architect (Technique 7), prompt-debug (Fault 7: Tool Blindness), prompt-compress (Stage 6 — cache boundary ROI)

#### 8. Cache-Aware Structure (Defense in Depth)
**Source**: Claude Code's system prompt uses a `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` marker that splits the prompt into a cacheable static section and a per-request dynamic section. Critical constraints appear both in the constraints section (top) AND inline at the relevant process step. This redundancy is by design — different positions in a long prompt receive different attention weight.

**Applied in**: prompt-architect (Pattern E + Technique 8), prompt-forge (Phase 3), prompt-judge (Dimension 8), prompt-compress (Stage 6)

### Three Architectural Insights

Beyond the 8 techniques, we identified three higher-level design principles:

#### Insight 1: Prompts Are API Contracts, Not Natural Language
Claude Code's prompts read like interface documentation — identity as type definition, MUST/NEVER as contracts, examples as test cases, output templates as return type specs. The lesson: write prompts like specifications, not emails.

#### Insight 2: Defense in Depth at the Prompt Layer
Critical rules in Claude Code appear at multiple layers: system prompt → tool-specific prompt → code-level validation → permission system. The same constraint ("don't force push") is stated in the system prompt, restated in BashTool's prompt, and enforced again in `bashSecurity.ts`. For prompts, this means: state critical rules twice — once in the constraints section, once at the point of action.

#### Insight 3: Separation of Concerns
Claude Code separates Context (git status, memory files), Capability (tool definitions), and Behavior (system prompt instructions) into independent injection paths. They're composed at runtime but authored separately. This principle applies to any complex prompt: don't mix "what the AI knows" with "what it can do" with "how it should behave."

## Usage Examples

```
# Build a prompt from scratch
/prompt-architect "I need a prompt for a code review bot"

# Polish an existing prompt  
/prompt-forge [paste your prompt]

# Get a score report
/prompt-judge [paste your prompt]

# Debug unexpected behavior
/prompt-debug "My prompt tells the AI to be concise but it writes essays"

# Reduce token cost
/prompt-compress [paste your long system prompt]
```

## Recommended Workflow

```
1. /prompt-architect    → Build initial prompt
2. /prompt-judge        → Score it, find weak spots
3. /prompt-forge        → Fix the weak spots
4. Test in production   → Observe behavior
5. /prompt-debug        → Diagnose any issues
6. /prompt-compress     → Optimize for cost
7. /prompt-judge        → Final score check
```

## License

MIT
