---
name: reviewer-committee
description: Multi-model code review committee using Codex (gpt-5.4) and Gemini (gemini-3.1-pro-preview) in parallel. TRIGGER when user runs /review, /review-diff, /review-commit, asks to "run the committee", "get committee feedback", "do a multi-model review", "committee review", or wants a thorough AI code review before merging or completing a task. Runs both CLIs simultaneously and synthesizes results into COMMITTEE_REVIEW.md in the current working directory. Use this whenever code review is requested - even if phrased as "just a quick check" or "sanity check the code".
---

# Reviewer Committee

Run parallel code reviews using GPT-5.4 (Codex) and Gemini-3.1-Pro-Preview (Gemini CLI), then synthesize into a `COMMITTEE_REVIEW.md` in the current directory.

## When invoked

1. Determine what to review (see "Review Modes" below)
2. Run the committee script (runs both models in parallel)
3. Read the resulting `COMMITTEE_REVIEW.md`
4. Present a brief synthesis: consensus issues first, then single-reviewer findings
5. Apply `superpowers:receiving-code-review` patterns when helping the user process feedback

## Review Modes

| Mode | Script flag | Use when |
|------|-------------|----------|
| Uncommitted changes | `--mode uncommitted` | Default — staged + unstaged |
| Vs base branch | `--mode base --param <branch>` | Before merging a PR |
| Specific commit | `--mode commit --param <sha>` | After a commit, /review-commit |
| Specific files | `--mode files --param "f1.py f2.py"` | Targeted file review |

## Running the Committee

Use the script at `scripts/run_committee_review.py`. The `REVIEWER_COMMITTEE_DIR` environment variable (set by `install.sh`) points to the installation directory:

```bash
# From inside the project repo (recommended — both CLIs get the right context)
cd /path/to/project
python "$REVIEWER_COMMITTEE_DIR/scripts/run_committee_review.py" --mode uncommitted

# Or from anywhere, pointing at the project with --project-dir
python "$REVIEWER_COMMITTEE_DIR/scripts/run_committee_review.py" \
  --mode uncommitted --project-dir /path/to/project

# Review vs a base branch
python "$REVIEWER_COMMITTEE_DIR/scripts/run_committee_review.py" \
  --mode base --param main --project-dir /path/to/project

# Review a specific commit
python "$REVIEWER_COMMITTEE_DIR/scripts/run_committee_review.py" \
  --mode commit --param <SHA> --project-dir /path/to/project

# Review specific files
python "$REVIEWER_COMMITTEE_DIR/scripts/run_committee_review.py" \
  --mode files --param "src/foo.py src/bar.py" --project-dir /path/to/project
```

`COMMITTEE_REVIEW.md` is written to `--project-dir` by default (or `--output-dir` if specified).
Gemini runs with `--approval-mode plan` (read-only): it reads project files for context but cannot edit or execute.

## After the Script Completes

Read `COMMITTEE_REVIEW.md` and present a summary:

```
## Committee Summary

**Consensus issues** (both reviewers flagged): [list - highest confidence]
**Codex only:** [list]
**Gemini only:** [list]

**Recommendation:** [Approve / Approve with Changes / Needs Work / Major Revisions]
```

Then ask the user how they want to proceed.

## Processing the Review Feedback

Apply `superpowers:receiving-code-review` principles:

- **Verify before implementing** — check each suggestion against the actual codebase
- **Consensus = high confidence** — if both models flag something, it's almost certainly real
- **Single-reviewer findings** — still worth checking, but apply more scrutiny
- **Push back technically** when a suggestion is wrong for this codebase
- **Prioritize**: Critical > Important > Minor

Fix Critical issues before proceeding. Fix Important issues before merging. Minor issues are optional.

## Slash Commands

These are registered globally for quick access:

| Command | Action |
|---------|--------|
| `/review` | Committee review of uncommitted changes |
| `/review-diff` | Committee review vs a base branch (prompts for branch name) |
| `/review-commit` | Committee review of a specific commit (prompts for SHA) |

## Troubleshooting

**"No diff content found"**: Ensure you have staged or unstaged changes (`git status`).

**Codex timeout**: Large diffs can be slow. Consider narrowing scope with `--mode files`.

**Gemini auth error**: Run `gemini` interactively once to ensure you're authenticated.

**Model not available**: Codex model can be overridden via `-c model="..."` in the script's `CODEX_CMD` list.
