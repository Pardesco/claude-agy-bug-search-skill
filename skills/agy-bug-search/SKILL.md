---
name: agy-bug-search
description: >
  Run a deep, independent bug search using Google Antigravity CLI (`agy`, a
  different model family) as a secondary reviewer for complex or high-stakes
  tasks. Use when the user asks to "bug search with antigravity", "have agy check
  this", "deep review", "third opinion", or wants a second independent pass after
  Codex on tricky logic, concurrency, or subtle correctness issues. agy runs
  sandboxed and only reports findings — it never edits files.
---

# Antigravity Deep Bug Search

This skill delegates a deep bug-hunt to the locally-installed **Google
Antigravity CLI** (`agy`, a different model family from both Claude and Codex).
Use it as the **secondary/escalation reviewer for complex tasks** — concurrency,
state machines, subtle data flow, security-sensitive logic — where one extra
independent perspective is worth the extra time.

**agy hunts bugs. It does not fix them.** Run sandboxed, report-only. Claude
collects findings, verifies them against the code, and presents the analysis.

## When to use

- A task was complex/high-stakes and you want more than one independent pass.
- Natural companion to the `codex-review` skill: Codex first for a fast general
  pass, `agy` for a deeper hunt on the hard parts (or when Codex and Claude
  disagree and you want a tiebreaker).
- Triggers: "bug search with antigravity", "deep review", "third opinion",
  "have agy look at the concurrency / edge cases".

## Preflight

1. Confirm it's installed: `agy --version`. If missing, tell the user and stop.
2. Scope the review. Identify the specific files/dirs that changed or are risky —
   `agy` is for *depth*, so point it at the hard parts, not the whole tree.
3. Run sandboxed (`--sandbox`) so terminal access is restricted during review.
   Never pass `--dangerously-skip-permissions` for a review.

## How to run

`agy` runs a single non-interactive prompt with `-p` / `--print`. Add relevant
directories with `--add-dir` and keep it sandboxed. On Windows, use the Bash tool
with a heredoc to avoid PowerShell quoting issues.

```bash
agy --sandbox --add-dir "path/to/relevant/dir" -p "$(cat <<'PROMPT'
You are an independent deep-bug-search reviewer. Hunt for real, non-obvious bugs
in the files below. Prioritize: correctness, race conditions / concurrency, state
handling, off-by-one and boundary errors, error/exception paths, resource leaks,
and security issues. Trace data flow rather than skimming.

For each finding give: file:line, severity (critical/high/medium/low), a concrete
reproduction or reasoning, and a one-line suggested fix. Do NOT rewrite the code.
If you find nothing solid in an area, say so rather than padding the list.

Files / areas to focus on:
- <list the changed or risky files here>
PROMPT
)"
```

Notes:
- `--print-timeout` defaults to 5m; deep searches on large changes may need more.
  If it's a big surface, run in the background and report when done.
- Override the model with `--model <name>` only if the user asks
  (`agy models` lists available models).
- `--continue` / `--conversation <id>` can resume a prior agy session for
  follow-up questions about its findings.

## After agy responds

Do **not** paste raw output. Claude must:

1. **Verify each finding** against the actual code — agy can produce
   false positives or theoretical issues that don't apply.
2. **Cross-reference** with any prior Codex review: note where the two reviewers
   agree (high confidence) vs. disagree (investigate).
3. **Present a consolidated analysis** grouped by severity:
   - ✅ Confirmed bugs worth fixing (file:line + why)
   - 🤔 Plausible / worth investigating
   - ❌ Flagged but not a real problem (and why)
4. **Recommend** fixes and ask before changing anything.

## Guardrails

- Sandboxed, report-only. A review must not modify the working tree.
- Never use `--dangerously-skip-permissions` for a review.
- agy output is input to Claude's judgment, not ground truth.
