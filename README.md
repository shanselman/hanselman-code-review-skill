# Hanselman Code Review Skill

Adversarial dual-model code review skill for GitHub Copilot CLI.

Runs **Claude Opus 5.5** (`claude-opus-5.5`) and **GPT-6 Astra**
(`gpt-6-astra`) at high reasoning effort, in parallel on the same code. Astra
uses `context_tier: "long_context"`; the context tier is not part of the model
ID. Unavailable models or context tiers are reported rather than silently
replaced with older generations.

## How It Works

1. **Pin the scope**: record the exact head and base revisions.
2. **Review independently**: same code, prompt, constraints, and severity rules.
3. **Verify findings**: check model claims against code, tests, and contracts.
4. **Cross-reference**: both flagged it is high consensus; one model only is
   low or disputed consensus.
5. **Assess fixes**: the coordinator assigns fix confidence and asks before
   acting on low-consensus findings.

Repeat reviews can target the delta: changed code and its affected paths,
unresolved findings, or disputed high-risk decisions when upgrading review
models. Both reviewers receive the same bounded scope. A newer-model review
does not replace missing runtime proof.

Both reviewers must complete on the same revision. If either reviewer fails or
the head changes, report the result as incomplete or stale rather than clean.

## Installation

Clone this repository with your normal GitHub-authenticated repository URL:

```powershell
cd ~/.copilot/skills
git clone <repository-url> hanselman-code-review
```

## Usage

In Copilot CLI, say any of:

- "Do a Hanselman review of my changes"
- "Run an adversarial code review"
- "Dual-model review this PR"
- "Review with confidence ratings"

## Output Format

```text
Both models agree: HIGH consensus

| Issue | Opus 5.5 | GPT-6 Astra | Fix Confidence |
|---|---|---|---|
| Concrete verified issue | HIGH | HIGH | 95% |

Only one model flagged: LOW consensus

| Issue | Opus 5.5 | GPT-6 Astra | Fix Confidence |
|---|---|---|---|
| Verified edge case | not flagged | MEDIUM | 80% |
```

## License

MIT
