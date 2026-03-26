Run a committee code review comparing the current branch against a base branch using the reviewer-committee skill.

$ARGUMENTS should start with a branch name (e.g. `main`, `develop`), optionally followed by a focus area. If no branch is provided, ask the user which branch to compare against. If a focus area is provided after the branch name, pass it as `--focus` to the review script.

Invoke the `reviewer-committee` skill. Use mode `base` with the provided branch as the param. Run the committee review script and write COMMITTEE_REVIEW.md to the current working directory, then present a brief synthesis of consensus findings.
