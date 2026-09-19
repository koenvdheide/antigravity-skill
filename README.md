# antigravity-skill

A [Claude Code](https://claude.ai/code) plugin that invokes the local [Antigravity CLI](https://antigravity.google/docs/cli/using) (`agy`) as an independent analysis partner, by default from a different model family.

## What it does

Gives Claude Code a structured way to delegate analysis to Antigravity: brainstorming, red-teaming, diff review, or anything that benefits from a non-Claude perspective.

The skill runs `agy` headless (`--print`) in plan mode. Google documents [the permission model](https://antigravity.google/docs/cli/permissions) and leaves print mode undocumented, so the skill covers what headless operation actually does: how to get content in (stdin piping doesn't work), and how to tell a finished review from one that stopped early.

## Why the completion contract matters

Headless `agy` can stop partway through a run and still look like it succeeded: exit code 0, empty stderr, and stdout containing narration that reads as though the work happened. The skill defends against this by asking for a sentinel line at the end of every response and discarding any output that lacks it.

## Convergence mode (iterative review)

For artifacts that evolve across revisions (specs, plans, designs), the skill runs a convergence loop: review, fix, re-review, until the reviewer gives an affirmative verdict or you stop. Rounds resume a pinned conversation ID when one can be captured, and fall back to a stateless round with a prior-findings block when none can. Each round re-supplies the current artifact, so it never critiques a stale version.

By default a round asks you for two decisions: which fixes to apply, then whether to continue. A standing instruction to iterate to convergence waives both. See the Convergence Mode section in `skills/antigravity/SKILL.md` for the loop shape and the scope-drift guidance that tells Claude when to stop and re-confirm scope.

## Prerequisites

- [Claude Code](https://claude.ai/code)
- Antigravity CLI (`agy`) installed and on PATH. If the binary is installed but the `agy` command is not found, run the installer by its absolute path and restart the shell (on Windows under Git Bash that is `"$LOCALAPPDATA/agy/bin/agy.exe" install`).
- A logged-in Antigravity account.

Print mode is undocumented upstream and its flags drift between releases, so the skill tells Claude to trust `agy --help` and `agy models` over its own tables when they disagree.

## Installation

Via the `agent-tools` marketplace:

```text
/plugin marketplace add koenvdheide/agent-tools
/plugin install antigravity@agent-tools
/reload-plugins
```

Refresh later with `/plugin marketplace update agent-tools`, then `/plugin update antigravity@agent-tools` and `/reload-plugins`.

To update the plugin itself from the CLI, use the qualified id. The bare name reports "not
found" even when the plugin is installed:

```bash
claude plugin update antigravity@agent-tools
```

## Migration from the `gemini` plugin

The plugin name is its installation identity, so editing the manifest does not convert an installed copy. Uninstall the old one and install the new one.

Check first which scope the old plugin is installed at, because uninstall defaults to `user` and a project- or local-scoped copy will survive an unscoped removal:

```bash
claude plugin list --json
```

Then remove it at that scope and install the replacement there:

```bash
claude plugin uninstall gemini --scope user
claude plugin install antigravity@agent-tools --scope user
```

Substitute `project` or `local` if that is where the old copy lives. From inside a session the equivalents are `/plugin uninstall gemini`, `/plugin install antigravity@agent-tools`, then `/reload-plugins`.

Invocation changes from `/gemini:gemini` to `/antigravity:antigravity`. The old skill targeted the Gemini CLI (`gemini`), which this release no longer supports.

## Summary QA

In the high-stakes modes (`red-team`, `diff-review`, `exhausted-hypotheses`, `attack-surface`) the skill re-reads its own summary against the fidelity rules before presenting it.

## Usage

Claude invokes the skill automatically when a task matches, or you can invoke it directly:

```text
/antigravity:antigravity red-team my API design before I start implementing
```

## License

MIT
