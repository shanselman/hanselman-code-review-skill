# Hanselman Code Review Skill

Adversarial dual-model code review skill for [GitHub Copilot CLI](https://githubnext.com/projects/copilot-cli/).

Runs Claude Opus 5 (`claude-opus-5`) and GPT-5.3-Codex (`gpt-5.3-codex`) at high reasoning effort in parallel on the same code, cross-references their findings, and produces a consensus severity table with fix confidence ratings.

## How It Works

1. **Two models review independently** — same code, same prompt, different reasoning
2. **Cross-reference findings** — both flagged it? Almost certainly real. Only one? Use judgment.
3. **Consensus table** — HIGH (both agree) vs LOW (one model only) with fix confidence %
4. **Fix prioritization** — HIGH consensus issues first, LOW consensus with user confirmation

## Installation

Clone into your Copilot skills directory:

```powershell
cd ~/.copilot/skills
git clone https://github.com/shanselman/hanselman-code-review-skill.git hanselman-code-review
```

## Usage

In Copilot CLI, say any of:
- "Do a Hanselman review of my changes"
- "Run an adversarial code review"
- "Dual-model review this PR"
- "Review with confidence ratings"

## Output Format

```
## Both Models Agree → HIGH consensus (fix all)

| Issue | Opus | Codex | Fix Confidence |
|-------|------|-------|----------------|
| Mutex released before relaunch confirmed | HIGH | HIGH | 95% |
| Authenticode accepts any signer | HIGH | MEDIUM | 90% |

## Only One Model Flagged → LOW consensus

| Issue | Opus | Codex | Fix Confidence |
|-------|------|-------|----------------|
| Ghost tray icon on exit | LOW | — | 95% |
| WinHTTP redirect handling | — | HIGH | 80% (disputed) |
```

## License

MIT
