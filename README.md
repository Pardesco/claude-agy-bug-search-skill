# Claude Code → Antigravity Deep Bug Search Skill

A [Claude Code](https://code.claude.com) **skill** that delegates a deep,
independent bug hunt to the [Google Antigravity CLI](https://antigravity.google)
(`agy`) — a *different* model family from both Claude and Codex. It's built as the
**escalation reviewer for complex, high-stakes tasks**: concurrency, state
machines, subtle data flow, and security-sensitive logic, where one more
independent perspective is worth the extra minutes.

Antigravity **hunts bugs; it does not fix them.** It runs sandboxed, reports
findings, and Claude verifies every finding against the real code before showing
you anything.

## Where this fits: a tiered review stack

This skill is the deepest tier of a layered, multi-model review workflow:

| Tier | Reviewer | Role |
|------|----------|------|
| 1 | **Claude** | Writes the code + self-review |
| 2 | **[Codex](https://github.com/Pardesco/claude-codex-review-skill)** | Fast, independent general pass on recent changes |
| 3 | **Antigravity (`agy`)** | Deep hunt on the hard parts; tiebreaker when Claude and Codex disagree |

Using three different model families means each tier catches a class of bugs the
others are blind to. The companion Codex skill lives here:
**[claude-codex-review-skill](https://github.com/Pardesco/claude-codex-review-skill)**.

## Why this one is different

Most CLI-bridge skills dump the sub-agent's raw output back to you. This skill
adds two things:

1. **Triage** — Claude verifies each finding against the actual code and sorts it
   into ✅ confirmed / 🤔 plausible / ❌ false alarm. An un-reviewed reviewer is
   just noise.
2. **Cross-referencing** — when run after the Codex pass, Claude flags where the
   two independent reviewers *agree* (high confidence) vs. *disagree*
   (investigate). Agreement across model families is a strong signal.

## Requirements

- [Claude Code](https://code.claude.com)
- [Google Antigravity CLI](https://antigravity.google) installed and
  authenticated (`agy --version`).

## Install

```bash
# user-level (all projects)
cp -r skills/agy-bug-search ~/.claude/skills/

# or project-level
cp -r skills/agy-bug-search .claude/skills/
```

## Usage

After a complex task, ask Claude:

- "deep review this"
- "bug search with antigravity"
- "have agy check the concurrency / edge cases"
- "third opinion" (after a Codex pass)

Claude scopes the review to the risky files, runs `agy --sandbox --add-dir ... -p`
with a deep-bug-hunt prompt, verifies the findings, cross-references any prior
Codex review, and presents a severity-grouped analysis — asking before it changes
anything.

## How it runs (under the hood)

- `agy -p` / `--print` for a single non-interactive prompt
- `--sandbox` to restrict terminal access during review (never
  `--dangerously-skip-permissions`)
- `--add-dir` to scope the workspace to the relevant code
- Points at the *hard parts*, not the whole tree — `agy` is tuned for depth
- `--print-timeout` may need raising for large diffs; long runs can go in the
  background

## Safety

- Read/report-only. A review never modifies the working tree.
- `agy` output is **input to Claude's judgment**, not ground truth.

## License

MIT — see [LICENSE](LICENSE).
