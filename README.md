# Flutter No Slop

Agent skills that make AI coding assistants write Flutter and Dart the way a
developer would — not the way a generator does.

Works with Claude Code, Cursor, Codex, Gemini CLI, Antigravity, and any other
agent that supports the [Agent Skills](https://agentskills.io) standard.

## The problem

Generated Flutter code fails in recognisable ways. It invents constructor
parameters that don't exist. It wraps a `Container` in a `Padding` even though
`Container` takes padding directly. It names a class `AuthStateResolver` when
`LoginState` was honest. It leaves comments addressed to the next agent instead
of the next developer. It calls `dispose()` on a Bloc that the provider already
closed.

None of that breaks the build immediately, which is why it survives review and
surfaces later as crashes and rewrites.

## Install

**Any agent:**

```bash
npx skills add theamazingyogita/flutter-no-slop --skill '*'
```

**Claude Code, as a plugin:**

```bash
claude plugin marketplace add theamazingyogita/flutter-no-slop
claude plugin install flutter-no-slop@theamazingyogita
```

## What it enforces

| # | Rule |
|---|---|
| 1 | Verify APIs in source — never recall them from memory |
| 2 | Search for an existing widget before creating a new one |
| 3 | One class per file |
| 4 | No widget wrapping another for a property it already has |
| 5 | Widget files under 200 lines; extract anything that appears twice |
| 6 | `StatelessWidget` by default; dispose every controller and subscription |
| 7 | Plain names — no `Resolver`, `Orchestrator`, `Manager` |
| 8 | Comments explain why, for developers, never addressed to an agent |
| 9 | UI dispatches events; logic lives in the Bloc |
| 10 | A flow document per feature |
| 11 | Tests cover the failure path, not just the happy path |
| 12 | `dart analyze` clean before reporting done |

Full details in [`skills/flutter-no-slop/SKILL.md`](skills/flutter-no-slop/SKILL.md).

## Complements, doesn't replace

This is a code-quality constraint skill. It sits alongside domain skills like
the [VGV AI Flutter Plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin)
rather than competing with them — those teach how to build features, this one
stops the agent producing bloat while it does.

## Contributing

Issues and pull requests welcome. If you've hit a generated-code pattern that
isn't covered, open an issue with the before/after.

## Licence

MIT. See [LICENSE](LICENSE).

---
