Run a committee code review on a specific git commit using the reviewer-committee skill.

$ARGUMENTS should start with a commit SHA (full or short), optionally followed by a focus area. If no SHA is provided, show recent commits with `git log --oneline -10` and ask the user which commit to review. If a focus area is provided after the SHA, pass it as `--focus` to the review script.

Invoke the `reviewer-committee` skill. Use mode `commit` with the provided SHA as the param. Run the committee review script and write COMMITTEE_REVIEW.md to the current working directory, then present a brief synthesis of consensus findings.
