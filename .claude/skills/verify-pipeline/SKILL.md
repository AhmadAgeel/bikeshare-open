---
name: verify-pipeline
description: Verify the bikeshare pipeline after changes using the established tests and Git status.
allowed-tools: Read, Bash(make*), Bash(git status*)
---

# Verify pipeline: $ARGUMENTS

Use the argument to choose the verification workflow:
- `tests`: run `make test`, then `git status --short`.
- `all`: run `make all`, then `make test`, then `git status --short`.

Report actual command outputs, failures, and whether the working tree is clean.
Do not change code, repair failures silently, or commit anything.
Do not use test-set metrics for feature selection.
