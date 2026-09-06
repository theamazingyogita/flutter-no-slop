# Flutter No Slop

![version](https://img.shields.io/badge/version-0.2.0-blue)
![license](https://img.shields.io/badge/license-MIT-green)
![agent skills](https://img.shields.io/badge/agent%20skills-compatible-7c3aed)
![flutter](https://img.shields.io/badge/flutter-dart-02569B)

Agent skills that stop AI from writing bloated Flutter code.

Works with Claude Code, Cursor, Codex, Gemini CLI, Antigravity, and any agent
that supports the [Agent Skills](https://agentskills.io) standard.

## Install

```bash
npx skills add theamazingyogita/flutter-no-slop --skill '*'
```

Claude Code, as a plugin:

```bash
claude plugin marketplace add theamazingyogita/flutter-no-slop
claude plugin install flutter-no-slop@theamazingyogita
```

## The difference

Same prompt, same model, same day, one run with the skill installed, one
without.

### Without the skill

![Generated without the skill](docs/before.png)

Stateful with no state. `Padding` wrapping a `Container` that already takes
padding. `SizedBox` where `Column` has `spacing`. No `const`. A class name no
developer would choose. A comment restating what the code says.

### With the skill

![Generated with the skill](docs/after.png)

## What it enforces

| # | Rule |
|---|---|
| 0 | The project's existing conventions win over anything here |
| 1 | Verify APIs in source, never recall them from memory |
| 2 | Search for an existing widget before creating a new one |
| 3 | One class per file |
| 4 | No widget wrapping another for a property it already has |
| 5 | Widget files under 200 lines; extract anything that appears twice |
| 6 | `StatelessWidget` and `const` by default; dispose every resource |
| 7 | Plain names, no `Resolver`, `Orchestrator`, `Manager` |
| 8 | Comments explain why, for developers, never addressed to an agent |
| 9 | Never ship a stub or placeholder data and call it done |
| 10 | Stay inside the task, no unrequested refactors |
| 11 | Handle errors; no swallowed exceptions, no `print` |
| 12 | Guard `BuildContext` across async gaps |
| 13 | No hardcoded colours, text styles, or user-facing strings |
| 14 | Logic lives outside widgets |
| 15 | A flow document per feature |
| 16 | Tests cover the failure path, not just the happy path |
| 17 | `dart analyze` clean before reporting done |

Full detail in [`SKILL.md`](skills/flutter-no-slop/SKILL.md). Stack-specific
guidance lives in [`references/`](skills/flutter-no-slop/references) and loads
only when relevant.

## Works with your stack

Rule 0 means the skill reads your `pubspec.yaml` and existing structure first,
and defers to them. A Riverpod project does not get Bloc pushed onto it. A
project using mockito does not get a second mocking library. The rules apply as
defaults for new code and where no convention exists.

## Complements, doesn't replace

This is a constraint skill, not a tutorial. It sits alongside domain skills like
the [VGV AI Flutter Plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin)
,  those teach how to build features, this one stops the agent producing bloat
while it does.

## Contributing

Found a generated-code pattern that isn't covered? Open an issue with the before
and after. One rule per pull request, and keep `SKILL.md` under 500 lines , 
anything longer belongs in `references/`.

## Licence

MIT. See [LICENSE](LICENSE).

---

Flutter and the related logo are trademarks of Google LLC. This project is not
affiliated with or otherwise sponsored by Google LLC.
