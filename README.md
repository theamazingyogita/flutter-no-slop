# Flutter No Slop

*Leave the Container alone.*

## Why this exists
Using AI to write code is settled now. Whether we picked it or the industry picked it for us is a different argument 🥹, and either way, most of us gave something up. The joy of working a problem out. The sound of actually typing. 😭
What companies want is the feature and a working UI, and those do show up faster now. Great. I guess that's a win.
Then you open the code the worst part.
It is that exact feeling of taking handover of a project some other developer built which is not always very pleasing. 
You explore the code for some time and then you try not to get angry or laugh with tears of pain of the mess you got.
When AI generates the code, the developer's role quietly shifts from creator to code reviewer and maintainer

The annoying part is having to explain the same things every time. Use the
architecture that is already here. Do not make a `StatefulWidget` for something
with no state. Check whether that widget exists before writing another one. Do
not stack three wrappers to move something eight pixels. Handle the error
instead of hiding it. Write the test for when it breaks, not only for when it
works.

The agent listens, usually. Then the context fills up, or you start a new
session, or a teammate opens the project, or you switch agents. And you are
explaining it all again.

If you repeat an instruction every time an agent starts working, that
instruction belongs in the project.

## What a skill is

A skill is a markdown file of instructions that lives with your project
configuration. When the agent works on Dart or Flutter code, it reads the file
and follows what is in it.

There is no package to add, no runtime code, no build step, and nothing that
ships with your app. It is a way to hand the agent your rules without pasting
them into every conversation. Install it once and everyone working on the
project gets the same rules.

Works with Claude Code, Cursor, Codex, Gemini CLI, Antigravity, and other agents
that support the [Agent Skills](https://agentskills.io) standard.

## What it tries to stop

Generated Flutter has predictable habits. It likes wrappers, abstractions, and
inventing architecture. It writes a new widget without checking for the one that
already exists. It turns an ordinary class into a `Manager`, a `Resolver`, or an
`Orchestrator`.

Widgets are not free, either. Every one is another node in the tree, another
thing to lay out and paint each frame, another level of indentation, another
bracket at the bottom of the file.

The worse habit is finishing things that are not finished. A repository returns
fake data, an exception becomes a `print`, a `build` method drifts past four
hundred lines, and the tests cover the path that was always going to work. It
compiles, so the agent reports success.

Eighteen rules against all of that.

## Before

Ask for a profile header and you can get this:

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

Nothing here is a disaster, which is the problem. It looks fine once. Two
hundred files like it is a codebase nobody wants to open.

## After

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

Nothing was added. The state, the `Container`, the `Center`, the `SizedBox` and
half the class name were removed. That is the whole idea.

## Install

```bash
npx skills add theamazingyogita/flutter-no-slop --skill '*'
```

For Claude Code:

```bash
claude plugin marketplace add theamazingyogita/flutter-no-slop
claude plugin install flutter-no-slop@theamazingyogita
```

## The rules

| # | Rule |
|---|---|
| 1 | The project's existing conventions always win |
| 2 | Verify APIs from source instead of guessing from memory |
| 3 | Look for an existing widget before creating another one |
| 4 | One class per file |
| 5 | No wrapper for a property the widget already supports |
| 6 | Keep widget files under 200 lines and extract repeated code |
| 7 | Prefer `StatelessWidget` and `const`, dispose every resource you create |
| 8 | Plain names, no `Resolver`, `Orchestrator`, `Manager` |
| 9 | Comments explain why, and not every file needs the same shape |
| 10 | Never call a stub or placeholder data done |
| 11 | Do the task that was asked, no unrelated refactors |
| 12 | Handle errors, do not swallow exceptions or reach for `print` |
| 13 | Guard `BuildContext` across async gaps |
| 14 | No hardcoded colours, text styles, or user-facing strings |
| 15 | Keep business logic out of widgets |
| 16 | Keep a flow document for each feature |
| 17 | Test failure paths, not only the successful ones |
| 18 | Run `dart analyze` before reporting the work complete |

Rule 10 matters more than it looks. An agent that hands you fake data and calls
the feature complete has not saved you an afternoon. It has moved the afternoon
to next week, and added the job of working out what it actually did.

Full detail in [`SKILL.md`](skills/flutter-no-slop/SKILL.md). Stack-specific
guidance sits in [`references/`](skills/flutter-no-slop/references) and loads
only when the task needs it.

## It reads the project before changing it

Rule 1 is first on purpose. A generic set of Flutter rules should not bulldoze
the conventions of a codebase that already has its own. A Riverpod project does
not get told to introduce Bloc. A project on mockito does not get a second
mocking library because this file mentions one. Where your project and this
skill disagree, your project wins.

The point is to remove decisions, not add them.

## Still evolving

This is early. The rules came from watching AI write Flutter in real projects,
not from an attempt to define the correct way to write Flutter, so they will
change. Some will miss a pattern, some will fire too aggressively, and some will
turn out to be personal taste dressed up as a principle. Those should go.

The aim is not two hundred rules and another instruction manual. It is a small
set that reliably stops bad generated code.

If you use it and the agent gets something wrong, open an issue. A before and
after is the most useful thing you can send.

## Numbers

None yet. Benchmarks are coming, and they will be published whether or not the
result is flattering. A benchmark you only publish when it looks good is
marketing.

## Not a Flutter tutorial

Other projects teach agents how to build things properly, and the
[VGV AI Flutter Plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin)
is a good one. Use those. This has a different job: keeping the agent from
making the codebase worse while it works. Less abstraction, less duplication,
less boilerplate, less unfinished work described as finished.

## Credit

The idea came from [ponytail](https://github.com/DietrichGebert/ponytail), which
does this for code in general and is worth installing on its own. Ponytail
argues for the laziest solution that works. Flutter No Slop takes the same
instinct and points it at the things Flutter gets wrong specifically: the widget
tree, disposal, `BuildContext` across async gaps, and state management.

They work together. Ponytail asks whether the code needs to exist. This one
deals with what is left.

## Contributing

Found a generated pattern that keeps appearing and is not covered? Open an issue
with the generated code, what is wrong with it, and what it should have been.
One rule per pull request.

Keep `SKILL.md` under 500 lines. Anything longer belongs in `references/`. The
skill should get more useful, not longer.

## License

MIT. See [LICENSE](LICENSE).

---

Flutter and the related logo are trademarks of Google LLC. This project is not
affiliated with or otherwise sponsored by Google LLC.
