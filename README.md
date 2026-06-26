# android-cli-code-skill

An [Agent Skill](https://agentskills.io/) that lets an AI agent understand and edit Android code through a **running Android Studio**, using the Android CLI's `android studio` subcommands. The agent works from the IDE's semantic index instead of plain text search.

The agent follows an **understand → edit → verify** loop:

1. Understand — `android studio find-declaration` / `find-usages` to locate where a symbol is defined and used (the index distinguishes same-named symbols that `grep` would conflate).
2. Edit — change the code.
3. Verify — `android studio analyze-file` for a file's lint/inspection findings before a build, and `android studio render-compose-preview` to render a Compose preview (optionally with `--print-semantics`). Re-run `find-usages` when a symbol's shape changes.

Get the project name with `android studio check`, then pass `--project=<name>` to every command (or `--pid=<pid>` to target a specific running instance).

## Install

Use the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add takahirom/android-cli-code-skill
```

Pick your agents when prompted; the CLI copies the skill into each selected agent's skills directory.

> [!NOTE]
> In the "Additional agents" list, Claude Code and others are **not selected by default** — press **Space** to toggle (`○` → `●`) before pressing Enter. Hitting Enter while the row is only highlighted (`❯ ○`) confirms without selecting, and the skill ends up in `~/.agents/skills/` only, which Claude Code does not read.

## Requirements

- [Android CLI](https://developer.android.com/tools/agents/android-cli) with the `android studio` subcommands (`check`, `find-declaration`, `find-usages`, `analyze-file`, `render-compose-preview`, `version-lookup`, `open-file`)
- A **running** Android Studio with the target project open

## Try it

In an agent chat, ask something like:

> Find every usage of `NiaAppState` and check `NiaAppState.kt` for warnings before I build.

The agent should activate the skill and drive the `android studio` commands for you.
