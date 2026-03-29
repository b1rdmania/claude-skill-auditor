# Skill Auditor

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Built%20with-Claude%20Code-DA7857?logo=anthropic)](https://code.claude.com)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/b1rdmania/claude-skill-auditor)

A Claude Code skill that audits other Claude Code skills. Give it any `SKILL.md` and get a scored quality report with specific fixes — not just flags.

---

## 🎯 What it does

Scores a skill across **7 dimensions** (0–10 each), identifies critical issues and improvements, and offers to apply fixes directly.

| Score | Verdict |
|-------|---------|
| **> 8.0** | ✅ Production-ready |
| **6.0–8.0** | ⚠️ Reliable, improvements optional |
| **< 6.0** | ❌ Needs significant work before use |

---

## 🚀 Quick Start

### Installation

```bash
mkdir -p ~/.claude/skills/skill-auditor
curl -o ~/.claude/skills/skill-auditor/SKILL.md \
  https://raw.githubusercontent.com/b1rdmania/claude-skill-auditor/main/SKILL.md
```

### Usage

Just ask Claude to audit a skill:

```
"Audit this skill: ~/.claude/skills/my-skill/SKILL.md"
```

```
"Check my skill before I publish it"
```

```
"Will Claude follow this skill?"
```

```
"Score this skill"
```

After the audit, Claude will ask if you want the fixes applied directly to the file.

---

## 📊 How it works

```mermaid
flowchart TD
    A([Give it a skill]) --> A1{Input type}
    A1 -->|File path| B[Read SKILL.md]
    A1 -->|Skill name| B2[Look up in ~/.claude/skills/]
    A1 -->|Pasted content| B3[Use directly]
    B & B2 & B3 --> C

    C[Run 7-dimension audit] --> D

    subgraph D [7 Dimensions]
        D1[1. Frontmatter Quality]
        D2[2. Instruction Followability]
        D3[3. DRY — No Repetition]
        D4[4. KISS — No Over-Engineering]
        D5[5. Dead Content]
        D6[6. Tool & Dependency Risk]
        D7[7. Structure & Navigation]
    end

    D --> E[Score each 0–10\nCalculate mean]
    E --> F[Output: table + critical issues\n+ improvements + what's working]
    F --> G{Overall score}
    G -->|above 8| H[Production-ready]
    G -->|6–8| I[Reliable, fixes optional]
    G -->|below 6| J[Needs significant work]
    F --> K([Offer to apply fixes])
```

---

## 📋 The 7 Audit Dimensions

```mermaid
radar
    title Audit Dimensions
    "Frontmatter" : 10
    "Followability" : 10
    "DRY" : 10
    "KISS" : 10
    "Dead Content" : 10
    "Dependency Risk" : 10
    "Structure" : 10
```

| Dimension | What it checks | Deductions |
|-----------|---------------|-----------|
| **1. Frontmatter Quality** | `name`, `description` completeness, trigger phrase count (need 5+) | -2 per missing check |
| **2. Instruction Followability** | Imperative language, passive deferrals, code-fence output traps, ambiguous placeholders | -2 per deferral, -3 per code-fence trap, -1 per placeholder |
| **3. DRY — No Repetition** | Repeated instructions, paraphrased re-statements, redundant sections | -1 per repeat, -2 per section |
| **4. KISS — No Over-Engineering** | Mode count (>4 = split), conditional complexity, premature flexibility | -2 per excess mode, -1 per complex tree |
| **5. Dead Content** | Unreferenced sections, placeholder content, tips that restate criteria | -1 per dead section |
| **6. Tool & Dependency Risk** | MCP requirements, shell binary assumptions, hardcoded URLs, assumed context | -2 per critical risk, -1 per warning |
| **7. Structure & Navigation** | Section headings, tables vs prose, linear flow, wall-of-text detection | -1 per structure issue |

---

## 💡 Example Output

```
# Skill Audit Report: income-opportunity-scorer

Overall Score: 8.2/10 ✅ Production-ready

## Scores by Dimension

| Dimension | Score | Status |
|-----------|-------|--------|
| Frontmatter Quality | 10/10 | ✅ Perfect |
| Instruction Followability | 7/10 | ⚠️ Issues found |
| DRY — No Repetition | 9/10 | ✅ Clean |
| KISS — No Over-Engineering | 8/10 | ✅ Good |
| Dead Content | 9/10 | ✅ Minimal waste |
| Tool & Dependency Risk | 7/10 | ⚠️ Warnings |
| Structure & Navigation | 8/10 | ✅ Well-organized |

## 🔴 Critical Issues (Fix before publishing)

1. **Code-fence output trap** (Line 87): Output template wrapped in triple backticks — Claude will render it as a code block instead of formatted output. Remove the code fence.

2. **Passive deferral** (Line 45): "Ask the user which mode to use" — skill should infer mode from context, not defer to user.

## 🟡 Improvements (Optional)

1. **Repeated instruction** (Lines 23, 67): "Explain the criterion in context" stated twice — merge into a single note.

2. **Dependency assumption** (Line 102): Assumes Read tool is available without checking — add fallback or document requirement.

## ✅ What's Working

- Strong frontmatter with 8 trigger phrases
- Clear imperative language throughout
- Good use of tables for scoring criteria
- Minimal mode complexity (3 modes, well-justified)
- Linear flow, easy to navigate

## Next Steps

Apply fixes? [Y/n]
```

---

## 🛠️ What it catches

The most common skill failure modes:

### 1. Code-fence output traps

```markdown
❌ BAD: Output template in code fence
Output format:
```
Name: {name}
Score: {score}
```
```

Claude renders this as a literal code block instead of filling in variables.

```markdown
✅ GOOD: Output template without code fence
Output format:

Name: {name}
Score: {score}
```

### 2. Passive deferrals

```markdown
❌ BAD: Ask the user which mode to use
✅ GOOD: Infer mode from context — single idea = auto-score, multiple = batch
```

### 3. Duplicate modes

```markdown
❌ BAD: "portfolio audit" and "batch" as separate modes (same thing, different framing)
✅ GOOD: One "batch" mode with examples for both use cases
```

### 4. Repeated instructions

```markdown
❌ BAD: Same rule in "Overview" and "Execution steps"
✅ GOOD: State once, reference from other sections
```

### 5. Ambiguous placeholders

```markdown
❌ BAD: {variable} or [placeholder] syntax that Claude renders literally
✅ GOOD: Clear template format or explicit variable substitution instructions
```

---

## 🎨 Features

- ✅ **7-dimension framework** — Covers frontmatter, followability, DRY, KISS, dead content, dependencies, structure
- ✅ **Explicit scoring rules** — Deduction per issue type, not subjective judgment
- ✅ **Actionable fixes** — Specific line numbers and replacement suggestions
- ✅ **Auto-apply option** — Claude can fix issues directly if you approve
- ✅ **Three input modes** — File path, skill name lookup, or pasted content
- ✅ **Production-grade threshold** — 8.0+ = ready to publish, 6-8 = good enough, <6 = needs work

---

## 📚 Use Cases

- **Before publishing** — Audit your skill before sharing on GitHub or skill marketplace
- **Quality gate** — Check skills before installing from untrusted sources
- **Learning tool** — Understand what makes a skill followable vs ignored
- **Bulk audit** — Score all installed skills to find low-quality ones

---

## 🤝 Contributing

Found a common skill failure mode not covered? Have ideas for new dimensions or scoring improvements?

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/new-dimension`)
3. Add your changes to SKILL.md
4. Open a PR with examples

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 🔗 Links

- [Claude Code](https://code.claude.com) — The AI coding assistant this skill is built for
- [Skill Documentation](https://code.claude.com/docs/en/skills) — Official Claude Code skills guide
- [Skill Marketplace](https://mcpmarket.com) — Discover more Claude Code skills

---

**Built with [Claude Code](https://code.claude.com)** 🚀
