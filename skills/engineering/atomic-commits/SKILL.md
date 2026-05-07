---
name: atomic-commits
description: Prepare clean atomic commits using Conventional Commits. Use when asked to commit changes, split work into commits, write commit messages, review commit history, or create a PR-ready branch. Enforces small single-purpose commits whose message explains the motivation and why the change exists, not just what is visible in the diff.
---

# Atomic Commits

Create commits that are easy to review, revert, bisect, and release.

## Required Standard

Follow Conventional Commits 1.0.0:

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Use `feat` for user-visible features and `fix` for bug fixes. Other useful types include `docs`, `test`, `refactor`, `perf`, `build`, `ci`, `chore`, and `style`. Use `!` after the type or scope for breaking changes, or add a `BREAKING CHANGE:` footer.

Reference: https://www.conventionalcommits.org/en/v1.0.0/

## Commit Rules

1. Inspect the diff before committing: `git status --short`, then `git diff` and `git diff --staged`.
2. Make each commit atomic: one intent, one behavioral or structural reason, one coherent review unit.
3. Split unrelated edits even when they are in the same file.
4. Do not mix mechanical formatting with behavior changes unless the formatting is required for the behavior change.
5. Do not hide risky changes inside `chore`, `style`, or `refactor`.
6. Write the subject in imperative mood and keep it concise.
7. Use the body to explain motivation, tradeoffs, context, and consequences.
8. Do not use the body to narrate file-by-file changes that the diff already shows.

## Message Quality

The subject tells the reader the commit category and summary. The body answers why this change is necessary.

Good commit messages explain:

- the problem or risk being addressed
- why this approach was chosen
- why alternatives were rejected, if relevant
- compatibility or migration concerns
- operational, security, or performance implications

Poor commit messages only repeat the diff:

- files changed
- functions renamed
- variables moved
- tests added
- dependencies updated

Those facts are visible from the patch. Put the reason in the message.

## Examples

### Bad: bundled and diff-driven

```text
fix: update auth files

Changed auth middleware, renamed token helpers, updated tests, and formatted
the user repository.
```

Why it is bad:

- mixes middleware behavior, helper renames, tests, and formatting
- says what changed, not why the bug mattered
- cannot be safely reverted without losing unrelated cleanup

### Good: atomic and motivation-driven

```text
fix(auth): reject expired session tokens before user lookup

Expired tokens were still reaching the repository layer, which made auth
failures look like missing users and obscured the real cause in logs. Checking
expiry at the middleware boundary keeps invalid sessions out of downstream
code and preserves the existing 401 behavior.
```

Why it is good:

- one behavioral change
- clear scope
- explains the failure mode and boundary choice
- states compatibility impact

### Bad: vague chore

```text
chore: cleanup
```

Why it is bad:

- no scope
- no review intent
- no clue whether the change is safe, mechanical, or behavioral

### Good: mechanical refactor

```text
refactor(storage): isolate blob key construction

Blob keys are built in multiple call paths that need to stay byte-for-byte
compatible during the bucket migration. Centralizing construction lets the
next change add the migration prefix in one place without changing current
object names.
```

Why it is good:

- honestly labels a non-behavioral change
- explains why the refactor exists now
- makes the follow-up change and compatibility constraint clear

### Good: breaking change

```text
feat(api)!: require idempotency keys for payment creation

Duplicate payment requests are currently deduplicated best-effort by payload,
which fails when clients retry with generated metadata. Requiring an explicit
idempotency key makes retry behavior deterministic across process restarts.

BREAKING CHANGE: payment creation requests without Idempotency-Key are rejected.
```

## Splitting Workflow

1. Group changes by intent: bug fix, feature, refactor, tests, docs, generated files.
2. Stage only one group at a time. Use `git add -p` when a file contains multiple intents.
3. Re-read the staged diff and ask: "Could this commit be reverted alone?"
4. If the answer is no, unstage and split smaller.
5. Write the subject last, after the staged diff has a clear intent.
6. Add a body whenever the reason is not obvious from the issue title or surrounding history.

## Final Check

Before committing, verify:

- the commit has exactly one purpose
- the type matches the actual change
- the scope is useful and stable
- the body explains why, not just what
- tests or verification match the change
- no unrelated local edits are staged
