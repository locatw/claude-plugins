---
name: commit
description: Select files to stage, then create a git commit with a WHY-focused message and user confirmation at each step. Use when the user asks to commit changes, create a git commit, or stage files.
argument-hint: "[scope] (optional — describe what to commit, e.g. \"migration files only\")"
allowed-tools: Bash, AskUserQuestion
---

!`git status --short`

!`git diff --cached`

!`git diff`

Create a git commit following these steps.

## Step 1: Propose which files to stage

Analyze the injected git status, staged diff, and unstaged diff to identify all changed files.

If $ARGUMENTS is provided, use it to filter which files to include (e.g., "migration files only" → include only files under migrations/).

Propose a list of files to stage. If files are already staged, include them in the proposal. Present the list to the user and use AskUserQuestion to confirm or let the user adjust the list.

If there are no changed files at all, inform the user and stop.

## Step 2: Stage the confirmed files

Run git add for each confirmed file:

  git add path/a.go path/b.go

Then run `git diff --cached` to capture the final staged diff. Use this output for all subsequent analysis.

## Step 3: Ask the user for intent

Use AskUserQuestion to show a brief summary of the staged files and ask:

> What is the purpose of this change?
> If there is anything non-obvious that the diff doesn't make clear, share that too.

## Step 4: Draft the commit message

Analyze the staged diff and the user's answer to produce a message in this format:

```
<type>: <why this change was made (imperative mood, ~50 chars)>

<information not visible in the diff: purpose, background, reasoning, trade-offs>
```

Replace `<model>` with the actual model name you are running as (e.g. `Sonnet 4.6`).

Subject line rules:

- Use imperative mood (add, fix, remove, etc.).
- Describe WHY, not WHAT.
  - Bad: "Add Ping() call in RawDB.Connect()"
  - Good: "fix: detect DB connection errors early at startup"
- Aim for 50 characters, hard limit 72 characters.
- No trailing period.
- Type prefix: `fix`, `add`, `remove`, `doc`, `refactor`, `test`, etc.; omit if none fits naturally.
- Covers only the primary change, not secondary or incidental changes.

Body rules:

- Aim for 1–2 sentences; only go longer if the context is genuinely complex.
- Write only what is genuinely non-obvious from the diff: a hidden constraint, the key reason this approach was chosen over an obvious alternative, or a side-effect worth noting.
- Do NOT describe what the diff shows (file lists, function names, mechanical changes).
- Omit body entirely if the subject line is self-sufficient.
- Wrap at 72 characters.
- Do not include a `Co-Authored-By:` trailer.

## Step 5: Confirm the commit message

Display the full message in a code block, then use AskUserQuestion to ask:

> Does this commit message look good? You can approve it or request changes.

If the user requests changes, revise and re-present. Repeat until approved.

## Step 6: Commit

Execute the commit using HEREDOC format:

```bash
git commit -m "$(cat <<'EOF'
<the approved message>
EOF
)"
```

## Step 7: Verify and report

Run `git status` and `git log -1` to confirm the commit was created. Report the commit hash and subject line.
