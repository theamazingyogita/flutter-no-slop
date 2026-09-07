# Flutter No Slop

Agent skills for writing Flutter and Dart code without the usual AI-generated bloat.

Works with Claude Code, Cursor, Codex, Gemini CLI, Antigravity, and any other agent that supports the [Agent Skills](https://agentskills.io) standard.

## The problem

AI-generated Flutter code has some pretty consistent problems.

It invents constructor parameters that don't exist. It wraps a `Container` in a `Padding` even though `Container` already has a `padding` property. It creates a class called `AuthStateResolver` when `LoginState` would do. It leaves comments for the next agent instead of the developer who has to maintain the code. It calls `dispose()` on a Bloc that its provider already closed.

Most of this doesn't stop the app from building.

It just leaves behind code that is harder to read, harder to change, and more likely to cause problems later.

## What changes

Take the same prompt:

> Build a profile header with an avatar and name.

Without the skill:

```dart
class ProfileHeaderWidgetBuilder extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    // Build the profile header
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Container(
        child: Column(
          children: [
            Center(child: CircleAvatar(radius: 40)),
            SizedBox(height: 8),
            Text(user.name),
          ],
        ),
      ),
    );
  }
}
```

There are several problems here.

The widget is stateful without having any state. `Container` doesn't need to be there. `Center` can be replaced with the appropriate `Column` alignment. `SizedBox` is unnecessary when `Column.spacing` is available. The class name is doing far too much work. The comment doesn't tell the developer anything they couldn't already see.

With the skill:

```dart
class ProfileHeader extends StatelessWidget {
  const ProfileHeader({required this.user, super.key});

  final User user;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        spacing: 8,
        children: [
          const CircleAvatar(radius: 40),
          Text(user.name),
        ],
      ),
    );
  }
}
```

Less code. Fewer widgets. A normal class name. No unnecessary state.

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

| #  | Rule                                                                            |
| -- | ------------------------------------------------------------------------------- |
| 1  | Verify APIs in source instead of guessing                                       |
| 2  | Search for an existing widget before creating another one                       |
| 3  | One class per file                                                              |
| 4  | Don't wrap a widget just to provide a property it already has                   |
| 5  | Keep widget files under 200 lines,extract repeated code                        |
| 6  | Prefer `StatelessWidget`,clean up every owned controller and subscription      |
| 7  | Use plain names; avoid `Resolver`, `Orchestrator`, `Manager`, and similar names |
| 8  | Comments explain why something exists, not what the code already says           |
| 9  | UI dispatches events,business logic stays in the Bloc                          |
| 10 | Keep a flow document for each feature                                           |
| 11 | Test failure paths, not only the happy path                                     |
| 12 | Run `dart analyze` before reporting the work as done                            |

Full details are in [`skills/flutter-no-slop/SKILL.md`](skills/flutter-no-slop/SKILL.md).

## Works alongside other skills

Flutter No Slop is focused on code quality.

It doesn't try to replace domain-specific skills such as the [VGV AI Flutter Plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin). Those can help an agent build a feature. This skill is there to keep the implementation from turning into unnecessary widgets, abstractions, and boilerplate.

## Contributing

Found a generated-code pattern that isn't covered? Open an issue or pull request with the before and after :)

## Licence
MIT. [LICENSE](LICENSE).
