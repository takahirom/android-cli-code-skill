---
name: android-cli-code-skill
description: Navigate and verify Android/Kotlin code through a running Android Studio, using the Android CLI's `android studio` subcommands (find-declaration, find-usages, analyze-file, render-compose-preview, version-lookup). Use this whenever working in an Android codebase and you need to find where a symbol is declared or used, see a file's lint/inspection warnings before building, confirm a Compose preview still renders, or look up the latest version of a dependency. Prefer it over plain text search (grep) for anything symbol-related, since it uses the IDE index and won't conflate same-named symbols — and reach for it especially during refactors and renames and in large codebases.
---

Work against a **running** Android Studio instance through `android studio` subcommands, so you get the IDE's semantic index instead of grepping text.

Get the project name first with `android studio check`. It lists each running instance and the projects open in it:

```
$ android studio check
pid: 12345
version: Quail 2 | 2026.1.2 Canary 2
Projects:
    READY     MyApp        /Users/you/git/MyApp
    READY     AnotherApp   /Users/you/git/AnotherApp
```

Use the name from the `Projects` list as `--project=<name>` (here, `MyApp`); it has to be `READY`, meaning the IDE has finished indexing it. Pass `--project=<name>` to every command below. If several Studio instances are running, also pass `--pid=<pid>` to pick one.

## 1. Locate code (instead of grep)

```bash
android studio find-declaration --project=<name> <Symbol>
android studio find-usages      --project=<name> [--short] <Symbol>
```

`find-declaration` returns the defining file, line, and surrounding code.
`find-usages` lists every usage as `file:line` across main and test; `--short` prints locations only.
These use the IDE index, so they tell apart same-named-but-different symbols that a text `grep` would conflate.

A `<Symbol>` is just a class, function, or property name — `LoginViewModel`, `fetchUser`, `isLoggedIn`, and so on. For example:

```
$ android studio find-usages --project=MyApp --short LoginViewModel
app/src/main/kotlin/.../login/LoginScreen.kt:42
app/src/main/kotlin/.../navigation/AppNavHost.kt:88
app/src/test/kotlin/.../login/LoginViewModelTest.kt:15
```

When the same name is reused elsewhere, pass `--context-file=<path>` to `find-declaration` so it resolves the one you mean.

## 2. Check a file before building

```bash
android studio analyze-file --project=<name> <path>
```

Returns the file's lint / inspection findings (INFO / WARNING), for example a `@VisibleForTesting` member read from production code, or a dependency declared twice.
Run it after editing a file to catch issues before a costly Gradle build, and chase a finding to its cause with `find-declaration`.

## 3. Render a Compose preview

```bash
android studio render-compose-preview --project=<name> [--output-image-file=/tmp/preview.png] [--print-semantics] <path> <ComposableName>
```

Writes the rendered preview to an image, and with `--print-semantics` prints the layout/semantics tree (text and coordinates) so you can verify the UI without opening the IDE.

## 4. Look up dependency versions

```bash
android studio version-lookup --project=<name> <id> [<id> ...]
```

Each id is `groupId:artifactId` (such as `androidx.window:window`), a Gradle plugin id, or one of `gradle` / `agp` / `sdk` / `ndk` / `studio`.
Returns current stable and preview versions, so you add or bump a dependency to a real version instead of guessing.

## 5. Open a file in the IDE

```bash
android studio open-file --project=<name> <path>
```

## Loop

Understand, edit, verify, using the index at each step:

1. `find-declaration` / `find-usages` to see what a change touches.
2. Edit the code.
3. `analyze-file` (and `render-compose-preview` for Compose) to verify before building. Re-run `find-usages` if you changed a symbol's shape.

## Rules

- Get the project name from `android studio check` before anything else; the commands need a running Studio with that project open.
- Prefer `find-usages` / `find-declaration` over `grep`: the index understands symbols, not just text. This matters most in large codebases.
- `analyze-file` reports findings but does not fix them. Read each finding and decide.
- Paths are relative to the current directory or absolute.

## Notes

- If multiple Studio instances are running, target one with `--pid=<pid>` (from `android studio check`).
- These commands talk to Android Studio. To drive a device or emulator UI instead, see android-cli-ui-automation-skill.
