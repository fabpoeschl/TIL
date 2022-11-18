# How to safely rewrite Git commit history

Working in a team using feature branches for multiple developers, or with small branches developed in parallel which may change during review phase while a new branch is based on these changes, rewriting the commit history with `--force-push` potentially has disastrous effects.

In order to avoid this, there are two options:
- use `--force-push` only before merging feature branch to main branch
- use `--force-with-lease` to automatically check if a commit is affected that was branched out of
