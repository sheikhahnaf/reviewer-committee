# Reviewer Committee

**Multi-model AI code review for [Claude Code](https://docs.anthropic.com/en/docs/claude-code)** -- run GPT-5.4 and Gemini-3.1-Pro side-by-side as your personal review panel.

One command. Two frontier models. A synthesized `COMMITTEE_REVIEW.md` dropped into your repo.

> **Platform note:** Works on **macOS**, **Linux**, and **Windows (WSL only)**. PowerShell and CMD are **not supported** -- use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) on Windows.

---

## Why a Committee?

Every AI model has blind spots. GPT might catch a concurrency bug that Gemini misses; Gemini might flag an architectural issue GPT glosses over. Running both in parallel and cross-referencing their findings gives you:

- **Higher recall** -- issues flagged by _either_ model are surfaced
- **Higher precision** -- issues flagged by _both_ models are almost certainly real
- **Diverse perspectives** -- different training data = different intuitions about code quality
- **One report** -- no tab-switching; everything lands in a single Markdown file

## How It Works

```
You run /review
       |
       v
 +-----------+     +-----------+
 | Codex CLI |     | Gemini CLI|
 | (GPT-5.4) |     | (3.1-Pro) |
 +-----------+     +-----------+
       \               /
        \             /
         v           v
   COMMITTEE_REVIEW.md
       |
       v
  Claude synthesizes
  consensus + findings
```

Both models run **in parallel** via Python's `ThreadPoolExecutor`. The script collects their structured reviews, writes a unified report, and Claude presents the synthesis.

---

## Prerequisites

| Dependency | Install | Purpose |
|------------|---------|---------|
| **Claude Code** | [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code) | Host environment for the skill |
| **Codex CLI** | `npm install -g @openai/codex` | GPT-5.4 code review |
| **Gemini CLI** | `npm install -g @google/gemini-cli` | Gemini-3.1-Pro code review |
| **Python 3.9+** | System or conda | Runs the orchestration script |
| **Node.js 18+** | [nodejs.org](https://nodejs.org) | Required by Codex & Gemini CLIs |
| **Unix shell** | macOS / Linux / WSL | **Not compatible with PowerShell or CMD** |

Make sure both `codex` and `gemini` are authenticated before first use:
```bash
# Authenticate Codex (uses OpenAI API key)
codex auth

# Authenticate Gemini (opens browser OAuth)
gemini
```

---

## Installation

### One-liner

```bash
git clone git@github.com:sheikhahnaf/reviewer-committee.git
bash reviewer-committee/install.sh
```

### What `install.sh` does

1. Copies slash command definitions to `~/.claude/commands/`:
   - `reviewer-committee.md` -- main skill (the brain)
   - `review.md` -- `/review` slash command
   - `review-diff.md` -- `/review-diff` slash command
   - `review-commit.md` -- `/review-commit` slash command
2. Sets `REVIEWER_COMMITTEE_DIR` in `~/.claude/settings.json` so commands can locate the Python script
3. Checks that `codex`, `gemini`, and `python3` are available (warns if missing)

### Verify installation

Open Claude Code in any project and type:
```
/review
```

If you see "Running Codex and Gemini in parallel...", you're set.

---

## Usage

### Slash Commands (recommended)

| Command | What it reviews | Example |
|---------|-----------------|---------|
| `/review` | All uncommitted changes (staged + unstaged) | Just type `/review` |
| `/review-diff main` | Current branch vs a base branch | `/review-diff develop` |
| `/review-commit abc123` | A specific commit by SHA | `/review-commit HEAD~1` |

### Natural Language (also works)

Claude recognizes these phrases and triggers the skill automatically:

- _"run the committee"_
- _"get committee feedback"_
- _"do a multi-model review"_
- _"sanity check the code"_
- _"committee review this"_

### Direct Script Usage

You can also run the Python script directly:

```bash
# Review uncommitted changes
python scripts/run_committee_review.py --mode uncommitted

# Review against a base branch
python scripts/run_committee_review.py --mode base --param main

# Review a specific commit
python scripts/run_committee_review.py --mode commit --param abc123f

# Review specific files
python scripts/run_committee_review.py --mode files --param "src/app.py src/utils.py"

# Review a different project (run from anywhere)
python scripts/run_committee_review.py --mode uncommitted --project-dir /path/to/project
```

---

## Output: `COMMITTEE_REVIEW.md`

The script writes a structured Markdown report to your project root:

```markdown
# Committee Review Report

**Subject:** Uncommitted changes (staged + unstaged)
**Date:** 2025-03-06 14:32:01
**Panel:** GPT-5.4 (Codex CLI) . Gemini-3.1-Pro-Preview (Gemini CLI)

---

## Codex Review (GPT-5.4)

### Strengths
...
### Critical Issues
...
### Important Issues
...

---

## Gemini Review (Gemini-3.1-Pro-Preview)

### Strengths
...
### Critical Issues
...
### Important Issues
...

---

## How to Process This Review
...
```

After writing the report, Claude reads it and presents a **synthesis**:

```
## Committee Summary

**Consensus issues** (both reviewers flagged): [highest confidence]
**Codex only:** [...]
**Gemini only:** [...]

**Recommendation:** Approve with Changes
```

---

## Review Modes

### `uncommitted` (default)
Reviews all staged + unstaged changes via `git diff --cached` and `git diff`. Use this during active development to catch issues before committing.

### `base`
Reviews all commits on your branch vs a base branch via `git diff <branch>...HEAD`. Ideal before opening or merging a PR.

### `commit`
Reviews a single commit via `git show <SHA>`. Good for post-commit audits or reviewing someone else's work.

### `files`
Reviews specific files by reading their full contents (not a diff). Useful for targeted reviews of specific modules.

---

## Severity Guide

Both reviewers use a consistent severity framework:

| Severity | Action | Examples |
|----------|--------|----------|
| **Critical** | Fix before anything else | Bugs, security vulnerabilities, data loss, crashes |
| **Important** | Fix before merging | Correctness, design flaws, missing error handling |
| **Minor** | Address if time permits | Style, naming, docs, minor optimizations |

**Consensus issues** (flagged by both models) have the highest confidence and should be prioritized first.

---

## Project Structure

```
reviewer-committee/
  README.md            # You are here
  LICENSE              # MIT License
  SKILL.md             # Skill definition (copied to ~/.claude/commands/)
  install.sh           # Installation script
  .gitignore           # Ignores .DS_Store, __pycache__, COMMITTEE_REVIEW.md
  commands/
    review.md          # /review slash command
    review-diff.md     # /review-diff slash command
    review-commit.md   # /review-commit slash command
  scripts/
    run_committee_review.py   # Core orchestration script
```

---

## Troubleshooting

### "No diff content found"
You have no staged or unstaged changes. Check with `git status`. If you want to review committed code, use `/review-commit <SHA>` or `/review-diff main`.

### Codex or Gemini times out
Large diffs can take a while. Narrow the scope:
- Use `--mode files --param "specific_file.py"` to review only what matters
- Split large PRs into smaller reviews

### "codex CLI not found" / "gemini CLI not found"
Install the missing CLI:
```bash
npm install -g @openai/codex    # for Codex
npm install -g @google/gemini-cli  # for Gemini
```

### Gemini authentication error
Run `gemini` interactively once to complete the OAuth flow, then retry.

### One reviewer fails but the other succeeds
The report still generates with the successful review. The failed reviewer's section shows the error message. This is by design -- partial results are better than no results.

### Script not found after install
Make sure the repo directory hasn't moved since installation. If it has, re-run `bash install.sh` to update the path in `~/.claude/settings.json`.

---

## Customization

### Changing models

Edit `scripts/run_committee_review.py`:

- **Codex model**: Change `'model="gpt-5.4"'` in the `run_codex_review` function
- **Gemini model**: Change `"gemini-3.1-pro-preview"` in the `run_gemini_review` function

### Changing the review prompt

The `GEMINI_REVIEW_PROMPT` variable at the top of the script controls the structure Gemini follows. Edit it to match your team's review standards.

### Changing the output location

```bash
python scripts/run_committee_review.py --mode uncommitted --output-dir ./reviews/
```

---

## Uninstall

```bash
rm ~/.claude/commands/reviewer-committee.md
rm ~/.claude/commands/review.md
rm ~/.claude/commands/review-diff.md
rm ~/.claude/commands/review-commit.md
```

Optionally remove `REVIEWER_COMMITTEE_DIR` from `~/.claude/settings.json`.

---

## Contributing

1. Fork the repo
2. Create a feature branch
3. Run `/review` on your own changes (dogfooding encouraged)
4. Open a PR

---

## License

[MIT](LICENSE)
