---
name: commit
description: Select files to stage, then create a git commit with a WHY-focused body and user confirmation at each step. Use when the user asks to commit changes, create a git commit, or stage files.
argument-hint: "[scope] (optional — describe what to commit, e.g. \"migration files only\")"
allowed-tools: Bash, AskUserQuestion
---

!`git status --short`

!`git diff --cached`

!`git diff`

Create a git commit following these steps.

Whenever a step shows the user something and asks about it, put the full content inside the AskUserQuestion question text. Do not print it as text before the call, and do not put it in an option preview: text written ahead of a tool call in the same turn can be dropped before it reaches any client, and previews are cut or not rendered.

## Step 1: Propose which files to stage

Analyze the injected git status, staged diff, and unstaged diff to identify all changed files.

If $ARGUMENTS is provided, use it to filter which files to include (e.g., "migration files only" → include only files under migrations/).

Propose a list of files to stage. If files are already staged, include them in the proposal. Use AskUserQuestion with the full list inside the question text, and let the user confirm or adjust it.

If there are no changed files at all, inform the user and stop.

## Step 2: Stage the confirmed files

Run git add for the confirmed files. Quote every path, and pass `--` before them so a path starting with `-` is not read as an option:

  git add -- "path/a.go" "docs/release notes.md"

Then run `git diff --cached` to capture the final staged diff. Use this output for all subsequent analysis.

## Step 3: Ask the user for intent

Use AskUserQuestion with a brief summary of the staged files inside the question text, followed by:

> What is the purpose of this change?
> If there is anything non-obvious that the diff doesn't make clear, share that too.

## Step 4: Draft the commit message

Analyze the staged diff and the user's answer to produce a message in this format:

```
<type>: <subject>

<body>
```

Subject line rules:

- Use imperative mood (add, fix, remove, etc.).
- Describe WHAT the change does, in terms of behavior — not the reasoning behind it (that goes in the body), and not literal code mechanics (specific function, method, or variable names).
  - Bad (that is the why): "fix: detect DB connection errors early at startup"
  - Bad (names code, not behavior): "fix: call db.Ping() in RawDB.Connect()"
  - Good: "fix: ping the DB when opening a connection"
- Aim for 50 characters, hard limit 72 characters.
- No trailing period.
- Type prefix: `fix`, `add`, `remove`, `doc`, `refactor`, `test`, etc.; omit if none fits naturally.
- The prefix is the one part of the subject that names intent. Pick it from what the change does to the codebase, not from the motivation behind it.
- Covers only the primary change, not secondary or incidental changes.

Body rules:

The body records what the diff cannot show.

- Required — WHY the change was made: the problem it solves, the constraint behind it, or the reason this approach was chosen over an obvious alternative.
- When it applies — a consequence the reader would otherwise hit by surprise: a side effect on callers, a trade-off knowingly accepted, or a limitation left in place.
- Use only what the diff and the user's answer actually establish. Never invent a reason, constraint, or rejected alternative to fill the body.
- Omit the body when the subject line already makes the reason self-evident (e.g. a typo fix), or when nothing beyond the diff was established.
- Do NOT describe what the diff shows (file lists, function names, mechanical changes).
- Aim for 1–2 sentences; only go longer if the context is genuinely complex.
- Wrap at 72 characters.
- Do not append any trailer, including `Co-Authored-By:` and `Claude-Session:`.

## Step 5: Review the draft

Before showing the draft, check it against every rule in Step 4, one rule at a time, reading it as a future reader who has the diff but not this conversation. Three checks decide whether the content is right:

- Reread the staged diff and confirm the subject describes what the diff actually does, and that it names the primary change, not a secondary one.
- For each statement in the body, name the line of the diff or the part of the user's answer that establishes it. A statement with no source is invented; remove it.
- Confirm the body tells that reader something the diff cannot show. If it only restates the diff, remove it.

Fix every violation and check again. Only a draft that passes every rule goes to Step 6.

## Step 6: Confirm the commit message

Use AskUserQuestion with the full message inside the question text, followed by:

> Does this commit message look good? You can approve it or request changes.

If the user requests changes, go back to Step 4 and treat the request as part of the user's answer. What the user asked for is not a violation in Step 5.

## Step 7: Commit

Execute the commit using HEREDOC format:

```bash
git commit -m "$(cat <<'EOF'
<the approved message>
EOF
)"
```

## Step 8: Verify and report

Run `git status` and `git log -1` to confirm the commit was created. Report the commit hash and subject line.
