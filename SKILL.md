---
name: hanselman-code-review
description: "Adversarial dual-model code review that runs two AI models in parallel, cross-references findings, and produces a consensus severity table with fix confidence ratings. Use when asked for a thorough code review, adversarial review, dual-model review, confidence-rated review, or 'Hanselman review'."
---

# Hanselman Adversarial Code Review Skill

Run a rigorous, adversarial code review using two AI models in parallel. Cross-reference their findings to separate signal from noise, producing a consensus severity table with fix confidence ratings.

## When to Use

Invoke this skill when the user asks for:
- A thorough or deep code review
- An adversarial review
- A dual-model review
- A "Hanselman review"
- A confidence-rated review
- Cross-model validation of code changes

## Philosophy

Single-model reviews have blind spots. Two models reviewing independently surface different classes of issues — security vs correctness vs edge cases. By cross-referencing, we get:
- **HIGH consensus** (both models flag it) → almost certainly a real issue
- **LOW consensus** (only one model flags it) → may be noise, use judgment
- **Disputed** (one model flags, evidence contradicts) → investigate before acting

## Review Workflow

### Step 1: Identify the Scope

Determine what code to review:
- If the user specifies files, use those
- If the user says "review my changes", use `git diff` or `git diff --staged`
- If reviewing a PR, use the GitHub MCP to get the diff
- For a branch, diff against the base branch

### Step 2: Launch Two Parallel Reviews

Launch two `rubber-duck` agents via the `task` tool, running in **background** mode with different models:

**Agent 1 — Opus (deep reasoning):**
```
task({
  name: "opus-review",
  agent_type: "rubber-duck",
  mode: "background",
  model: "claude-opus-5",
  reasoning_effort: "high",
  prompt: "<full context + code + review instructions>"
})
```

**Agent 2 — Codex (different perspective):**
```
task({
  name: "codex-review",
  agent_type: "rubber-duck",
  mode: "background",
  model: "gpt-5.3-codex",
  reasoning_effort: "high",
  prompt: "<same context + code + review instructions>"
})
```

**Important:** Both agents get the EXACT same prompt so findings are comparable. The prompt must include:
- Full file contents or diffs being reviewed
- Project context (language, framework, constraints)
- Instruction: "For each issue found, classify severity as: CRITICAL (will crash/corrupt/security hole), HIGH (likely bug in real use), MEDIUM (edge case that could bite), LOW (minor improvement)"

### Step 3: Wait for Both to Complete

Wait for both background agents to finish. Read their results with `read_agent`.

### Step 4: Cross-Reference and Build the Consensus Table

Create a SQL table to track findings:

```sql
CREATE TABLE review_findings (
    id TEXT PRIMARY KEY,
    issue TEXT NOT NULL,
    opus_severity TEXT,
    codex_severity TEXT,
    consensus TEXT NOT NULL,
    fix_confidence INTEGER NOT NULL
);
```

For each finding from either model:
1. Check if the other model found the same or similar issue
2. Assign consensus:
   - **Both models flagged it** → `HIGH` consensus (real issue, fix it)
   - **Only one model flagged it** → `LOW` consensus (may be valid, use judgment)
   - **One model flagged it but evidence contradicts** → `LOW (disputed)` (investigate)
3. Assign fix confidence (0-100%):
   - 99% = trivial fix (typo, missing null check, wrong flag)
   - 90-95% = straightforward fix requiring some design thought
   - 70-80% = complex fix, may need research or trade-off decisions
   - <70% = hard problem, might be intentional, or requires architectural change

### Step 5: Present the Results

Present findings in two tables:

**Both Models Agree → HIGH consensus (fix all)**

| Issue | Opus | Codex | Fix Confidence |
|-------|------|-------|----------------|
| Description of issue | SEVERITY | SEVERITY | XX% |

**Only One Model Flagged → LOW consensus**

| Issue | Opus | Codex | Fix Confidence |
|-------|------|-------|----------------|
| Description of issue | SEVERITY or — | SEVERITY or — | XX% |

Use `—` when a model didn't flag the issue.

### Step 6: Act on Findings

After presenting the table:
- Ask the user if they want to fix all issues or select specific ones
- Fix HIGH consensus issues first (highest priority)
- For LOW consensus / disputed issues, briefly explain why one model flagged it and why the other didn't
- For disputed findings, explain your assessment of whether it's a real issue

## Review Instructions Template

Use this as the base prompt for both review agents (customize the project context):

```
Review the following code changes for [PROJECT NAME]. This is a [LANGUAGE/FRAMEWORK] project with these constraints: [CONSTRAINTS].

Focus on bugs, security issues, race conditions, and correctness problems. Ignore style/formatting.

[FULL CODE OR DIFF HERE]

For each issue found, classify severity as:
- CRITICAL: will crash, corrupt data, or create a security vulnerability
- HIGH: likely bug that will manifest in real use
- MEDIUM: edge case that could bite in specific scenarios
- LOW: minor improvement, theoretical concern, or robustness enhancement

For each issue, include:
1. What the issue is
2. Where it is (file + line/region)
3. Why it matters (impact)
4. Suggested fix
```

## Model Selection

The default pairing uses these pinned model IDs at high reasoning effort:
- **Claude Opus 5:** `claude-opus-5`
- **GPT-5.3-Codex:** `gpt-5.3-codex`

Use this exact cross-family pairing when both models are available. Their different training and reasoning patterns maximize the chance of catching different classes of issues.

Alternative pairings if a model is unavailable:
- Claude Opus + Claude Sonnet (different capability tiers, same family)
- GPT Codex + Claude Haiku (fast but different perspectives)
- Any two models from different families for maximum diversity

## Important Notes

- **Never skip the cross-reference step.** The value is in the comparison, not individual reviews.
- **Log disputed findings with your reasoning.** The user should understand why you agree or disagree.
- **Fix confidence is YOUR assessment**, not the models'. Base it on your understanding of the codebase and the complexity of the fix.
- **Don't fix LOW consensus issues without asking** unless the user said "fix it all."
