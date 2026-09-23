# Hanselman Code Review Skill

Adversarial dual-model code review skill for [GitHub Copilot CLI](https://githubnext.com/projects/copilot-cli/).

Runs two AI models (Claude Opus 5.5 + GPT-5.6 Sol with 1M context) in parallel on the same code, cross-references their findings, and produces a consensus severity table with fix confidence ratings.

The tool model IDs are `claude-opus-5.5` and `gpt-5.6-sol`. Sol uses `context_tier: "long_context"`; `1M` is not a model-name suffix. Unavailable models or context tiers are reported rather than silently replaced with older generations.

## How It Works

1. **Two models review independently** — same code, same prompt, different reasoning
2. **Cross-reference findings** — both flagged it? Almost certainly real. Only one? Use judgment.
3. **Consensus table** — HIGH (both agree) vs LOW (one model only) with fix confidence %
4. **Fix prioritization** — HIGH consensus issues first, LOW consensus with user confirmation

Repeat reviews can target the delta: changed code and its affected paths, unresolved findings, or disputed high-risk decisions when upgrading review models. Both reviewers receive the same bounded scope. A newer-model review does not replace missing runtime proof.

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

| Issue | Opus 5.5 | GPT-5.6 Sol | Fix Confidence |
|-------|------|-------|----------------|
| Mutex released before relaunch confirmed | HIGH | HIGH | 95% |
| Authenticode accepts any signer | HIGH | MEDIUM | 90% |

## Only One Model Flagged → LOW consensus

| Issue | Opus 5.5 | GPT-5.6 Sol | Fix Confidence |
|-------|------|-------|----------------|
| Ghost tray icon on exit | LOW | — | 95% |
| WinHTTP redirect handling | — | HIGH | 80% (disputed) |
```

## License

MIT
