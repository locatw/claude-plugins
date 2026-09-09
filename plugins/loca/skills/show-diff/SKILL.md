---
name: show-diff
description: Show the repository's change set in full as raw git output, so the operator reviews the diff itself rather than a summary of it. Use whenever the operator asks to see the diff, the changes, or what was modified, and takes --base to add the diff against a base ref.
argument-hint: "[--base [ref]]"
allowed-tools: Bash(git status *), Bash(git diff *), Bash(git fetch *), Bash(git ls-files *), Read
---

Show the repository's change set so the operator can read it.

The diff is the deliverable here rather than evidence for a claim, so it is pasted whole.
Summarising it, or showing only the part this session happened to touch, leaves the operator with nothing to review.

## The working tree

This section runs whether or not an argument was given.

1. Run `git status -uall`, and paste its output.
   - Without `-uall` an untracked directory collapses to one line, and the files inside it never appear.
2. Run `git diff --shortstat HEAD`, and keep its line for the diff that follows.
   - The count comes from git rather than from counting the pasted hunks, because it is what the reader checks the paste against.
3. Run `git diff HEAD`, and paste its output in full.
   - `HEAD` rather than no argument, because a bare `git diff` drops whatever is already staged.
4. Enumerate untracked paths with `git ls-files --others --exclude-standard`.
   - No diff shows an untracked file, because a path absent from both the index and HEAD is never walked.
   - This rather than reading them out of step 1, because `git status` prints a path relative to the current directory and quotes a non-ASCII one.
5. Paste the content of each untracked path, except where [Withholding](#withholding) applies.

## Against a base ref

This section runs only when the argument starts with `--base`.

1. Take the base from the ref given after `--base`, defaulting to `main`, or to `origin/main` when `main` is itself the branch out.
   - The three-dot range is empty when the base and `HEAD` name the same branch, and what is worth showing on `main` is the commits the operator has not published.
   - Use the ref exactly as written, because a remote branch, a tag, and a commit id are all refs the argument admits.
   - Where it does not resolve, retry as `origin/<ref>` and say which form was used.
   - Run `git fetch origin --quiet` first whenever the base resolves to a remote-tracking ref, that retry included, and on failure carry on and say the base may be stale.
2. Run `git diff --shortstat <base>...HEAD`, then `git diff <base>...HEAD`, and paste the diff in full.
   - The three-dot form measures from where the branch left the base, so a base that moved since does not enter the diff.

## Withholding

Pasting is the default, and the cases below are the only ones that withhold content.
A withheld item is named with its size and the reason, because a silent omission is the failure this skill exists to prevent.

- A path the repository denies reading, listed under `permissions.deny` in its `.claude/settings.json`.
- A path whose name suggests a credential, such as a key, a certificate, a token, or a kubeconfig.
- A file that is not text.
- A tool result that was truncated or written to a file instead of returned, whose path is given so the operator reads it directly.
- A diff or a file large enough to fill the session, whose size is reported so the operator decides before it is pasted.

## Output

- Give each command its own fenced block, with the command written on the line above it.
- Open a block holding a diff with more backticks than any fence inside it, because a diff of Markdown carries fences of its own.
- Put the `--shortstat` line above each diff, so the reader knows the size before starting.
- Paste every diff whole, and elide nothing outside [Withholding](#withholding).
- Say where a command produced no output, because an omitted block reads as a forgotten one.
- Say that there is nothing to show where the tree is clean and no path is untracked.
