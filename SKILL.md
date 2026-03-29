---
name: skill-auditor
description: >
  Audit, validate, and score Claude Code skill files (SKILL.md). Use this skill when the
  user wants to check skill quality, improve a skill, validate skill structure, audit a
  skill before publishing, or asks "is this skill well-written?", "will Claude follow this
  skill?", "audit this skill", "check my skill", "validate this skill", or "score this skill".
  Works on a single SKILL.md file or a named installed skill.
---

# Skill Auditor

A structured quality framework for Claude Code skill files. Evaluates whether a skill is
clear, followable, efficient, and well-formed — and gives concrete fixes, not just flags.

---

## Input

Either:
- A file path to a SKILL.md
- A skill name (look it up in `~/.claude/skills/`)
- The skill content pasted directly into the conversation

Read the full file before starting any analysis.

---

## The 7 Audit Dimensions

Run all 7. Each is scored 0–10. Final score is the mean, rounded to one decimal.

---

### 1. Frontmatter Quality (0–10)

Check the YAML frontmatter block.

| Check | Pass condition |
|-------|---------------|
| `name` present | Matches directory/file name |
| `description` present | At least 3 sentences covering: what the skill does, when to trigger, what modes exist |
| Trigger phrases | At least 5 concrete example phrases in the description |
| No junk fields | No fields that Claude Code doesn't use (e.g. `version`, `author` are fine but irrelevant) |

**Scoring:** 10 = all checks pass. Deduct 2 per failed check.

---

### 2. Instruction Followability (0–10)

Will Claude actually do what the skill says? Check for:

- **Imperative language**: Instructions should say "do X", not "you might want to X" or "consider X"
- **Passive mode detection**: Flag any "ask the user which mode" patterns — the skill should infer and act, not defer
- **Ambiguous placeholders**: Any `{variable}` or `[placeholder]` syntax that Claude might treat as literal text
- **Output format traps**: Output templates wrapped in triple-backtick code fences — Claude will output the template as a code block rather than rendering it. Flag every instance.
- **Conflicting instructions**: Any two sections that contradict each other

**Scoring:** Start at 10. Deduct 2 per passive deferral, 3 per code-fence output trap, 1 per ambiguous placeholder, 2 per contradiction.

---

### 3. DRY — No Repetition (0–10)

Check for instructions stated more than once, paraphrased re-statements of the same rule, and redundant examples that don't add new information.

Threshold: flag any instruction repeated more than once. Identical intent in different wording counts.

**Scoring:** 10 = no repetition. Deduct 1 per repeated instruction, 2 per repeated section.

---

### 4. KISS — No Over-Engineering (0–10)

Is the skill more complex than its task requires?

- **Mode count**: More than 4 distinct modes in one skill is a signal to split
- **Conditional trees**: Deeply nested "if X then Y else if Z" logic that could be simplified to a default + one override
- **Premature flexibility**: Optional parameters, weight systems, preset configurations that add complexity but rarely get used
- **Length relative to task**: A skill that does one simple thing but has 200+ lines likely has bloat

**Scoring:** 10 = appropriately simple. Deduct 2 per unnecessary mode, 1 per unnecessary conditional branch, 2 for length bloat (>250 lines for a single-task skill).

---

### 5. Dead Content (0–10)

Sections that exist but serve no purpose:

- **Unreferenced sections**: Headings that are never triggered by any mode or flow
- **Tips that restate criteria**: "Tips for better scoring" that just paraphrase already-defined criteria
- **Attribution/boilerplate** at the bottom that adds length but no instruction value (note: attribution is fine, just flag if it's excessive)
- **Commented-out or placeholder content**: `[TODO]`, `...`, or empty bullet points

**Scoring:** 10 = no dead content. Deduct 2 per unreferenced section, 1 per redundant tip, 1 per placeholder.

---

### 6. Tool & Dependency Risk (0–10)

Check for:

- **MCP tool references**: Any `mcp__toolname__action` calls. Flag each one — if the MCP isn't installed, the skill silently fails. Note which MCPs are required.
- **Shell commands**: Any Bash instructions that assume specific binaries are installed
- **External URLs hardcoded**: Links that may rot
- **Assumed context**: Instructions that assume Claude has memory, a specific project structure, or prior conversation state without a fallback

**Scoring:** 10 = no external dependencies. Deduct 2 per required MCP, 1 per shell binary assumption, 1 per hardcoded URL, 2 per assumed context with no fallback.

---

### 7. Structure & Navigation (0–10)

Is the skill easy for Claude to parse and navigate mid-execution?

- **Clear section headings**: Each major step or mode has its own `##` or `###` heading
- **Tables for structured data**: Criteria, scoring bands, and comparisons should be in tables, not prose
- **Flow is linear**: A model reading top-to-bottom can follow the skill without backtracking
- **No wall-of-text**: No paragraph longer than ~6 lines without a break or list

**Scoring:** 10 = well structured. Deduct 1 per missing heading for a major section, 2 per table replaced by prose, 1 per non-linear flow issue, 1 per wall-of-text block.

---

## Output Format

Render directly as markdown (not inside a code block):

## Skill Audit: [Skill Name]

**Overall Score: X.X / 10**

| Dimension | Score | Summary |
|-----------|-------|---------|
| 1. Frontmatter Quality | X/10 | ... |
| 2. Instruction Followability | X/10 | ... |
| 3. DRY | X/10 | ... |
| 4. KISS | X/10 | ... |
| 5. Dead Content | X/10 | ... |
| 6. Tool & Dependency Risk | X/10 | ... |
| 7. Structure & Navigation | X/10 | ... |

### Critical Issues (fix before using)
[List only issues scoring -2 or worse. Be specific: quote the offending text, explain why it's a problem, give the fix.]

### Improvements (nice to have)
[List minor issues scoring -1. Same format: quote → problem → fix.]

### What's Working Well
[2–3 specific things done right. Not generic praise.]

---

## After the Audit

Always ask: "Want me to apply the fixes?" If yes, edit the file directly — don't just describe changes.

If the overall score is below 6: flag that the skill needs significant work before it will be reliable.
If 6–8: reliable for most uses, improvements optional.
If above 8: production-ready, only apply fixes if the user wants them.
