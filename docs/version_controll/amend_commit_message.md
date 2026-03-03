# Amending Commit Messages After Push

*Why This Matters*

Sometimes mistakes slip into commit messages (typos, unclear descriptions, accidental text like --amend). Clear, consistent commit history is critical for collaboration, code reviews, and release notes. This guide explains how to safely amend commit messages that have already been pushed.

## Step‑by‑Step Procedure

1. Identify the Commit

    Find the commit hash (e.g., 121f0d6) using:
    bash

    git log --oneline

2. Start an Interactive Rebase

    Rebase from the parent of the commit you want to change:
    bash

    git rebase -i <commit-hash>^

    Example:
    bash

    git rebase -i 121f0d6^

3. Mark the Commit for Editing

    In the rebase todo file, change the line from:
    Code

    pick 121f0d6 <old message>

    to:
    Code

    edit 121f0d6

4. Amend the Commit Message

    When Git pauses at that commit:
    bash

    git commit --amend

    Replace the incorrect message with a clear, conventional commit message (e.g., test(variant): add database constraint enforcement specs).

5. Continue the Rebase
bash

git rebase --continue

6. Push the Updated History

    Since history has changed, force push:
    bash

    git push --force-with-lease

⚠️ Important Notes

    Shared branches (e.g., main): Avoid rewriting history. Instead, add a new commit clarifying the old one.

    Feature branches: Safe to amend, but always use --force-with-lease to avoid overwriting teammates’ work.

    Commit message style: Follow Conventional Commits for clarity:

        feat(scope): add new feature

        fix(scope): correct a bug

        test(scope): add or refine tests

        refactor(scope): improve code without changing behavior

Example

Accidental commit message:
Code

--amend

Corrected commit message:
Code

test(variant): add database constraint enforcement specs

- Add specs that bypass Rails validations with insert_all! to verify DB-level CHECK constraints
- Ensure unit_price cannot be negative at the database level
- Ensure stock cannot be negative at the database level
- Confirm ActiveRecord::CheckViolation is raised when constraints are violated

✅ With this procedure, the team can safely fix commit messages while keeping history clean and professional.