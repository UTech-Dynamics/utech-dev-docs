# Amending Commit Messages After Push

## Why This Matters

Sometimes mistakes slip into commit messages (typos, unclear descriptions, accidental text like --amend). Clear, consistent commit history is critical for collaboration, code reviews, and release notes. This guide explains how to safely amend commit messages that have already been pushed.

## Step-by-Step Procedure

1. Identify the commit.

Find the commit hash (for example, `121f0d6`) using:

```bash
git log --oneline
```

2. Start an interactive rebase.

Rebase from the parent of the commit you want to change:

```bash
git rebase -i <commit-hash>^
```

Example:

```bash
git rebase -i 121f0d6^
```

3. Mark the commit for editing.

In the rebase todo file, change the line from:

```text
pick 121f0d6 <old message>
```

to:

```text
edit 121f0d6
```

4. Amend the commit message.

When Git pauses at that commit:

```bash
git commit --amend
```

Replace the incorrect message with a clear, conventional commit message (for example, `test(variant): add database constraint enforcement specs`).

5. Continue the rebase.

```bash
git rebase --continue
```

6. Push the updated history.

Since history has changed, force push:

```bash
git push --force-with-lease
```

## Important Notes

- Shared branches (for example, `main`): avoid rewriting history. Add a new commit clarifying the old one instead.
- Feature branches: amending is usually safe, but always use `--force-with-lease` to avoid overwriting teammates' work.
- Commit message style: follow Conventional Commits for clarity.

Examples:

- `feat(scope): add new feature`
- `fix(scope): correct a bug`
- `test(scope): add or refine tests`
- `refactor(scope): improve code without changing behavior`

## Example

Accidental commit message:

```text
--amend
```

Corrected commit message:

```text
test(variant): add database constraint enforcement specs
```

Suggested description points:

- Add specs that bypass Rails validations with `insert_all!` to verify DB-level CHECK constraints.
- Ensure `unit_price` cannot be negative at the database level.
- Ensure `stock` cannot be negative at the database level.
- Confirm `ActiveRecord::CheckViolation` is raised when constraints are violated.

With this procedure, the team can safely fix commit messages while keeping history clean and professional.