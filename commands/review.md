Run a committee code review on all uncommitted changes (staged and unstaged) in the current directory using the reviewer-committee skill.

$ARGUMENTS is an optional focus area for the review (e.g. "security", "performance", "error handling"). If provided, pass it as `--focus` to the review script.

Invoke the `reviewer-committee` skill. Use mode `uncommitted`. Run the committee review script and write COMMITTEE_REVIEW.md to the current working directory, then present a brief synthesis of consensus findings.
