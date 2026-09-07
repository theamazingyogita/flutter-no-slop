# Flutter No Slop

*Leave the `Container` alone.*

## Why this exists

AI coding assistants are useful. That part is settled.

The annoying part is having to explain the same things to them over and over.

Use the project's existing architecture.
Do not create a `StatefulWidget` for something that has no state.
Check whether that widget already exists.
Do not add three wrappers just to move something eight pixels.
Handle the error instead of hiding it.
Write the test for when it breaks, not just when everything works.

The agent follows the instructions. Usually.

Then the context gets too large. You start a new session. Someone else opens the project. Or you switch agents.

Now you are explaining everything again.

If you have to repeat an instruction every time an agent starts working, that instruction probably belongs in the project.

That's what this is for.

## What is a skill?

A skill is a Markdown file containing instructions for the coding agent.

The file lives alongside your project configuration. When the agent needs to work on Dart or Flutter code, it can read those instructions and follow them.

There is no package to add.

No runtime code.

No build step.

No dependency.

Nothing gets shipped with your app.

It is simply a way to give the agent the project's rules without having to paste them into every conversation.

Install it once and the rules are available to everyone working with the project.

Works with Claude Code, Cursor, Codex, Gemini CLI, Antigravity, and other agents that support the [Agent Skills](https://agentskills.io/) standard.

## What it tries to stop

AI-generated Flutter code has some predictable habits.

It likes wrappers.

It likes unnecessary abstractions.

It likes inventing architecture.

It likes creating a new widget before checking whether one already exists.

It likes turning a simple class into a `Manager`, `Resolver`, `Orchestrator`, or whatever other impressive-sounding name happens to fit.

And when you ask it to implement something, it can be very good at making something that *looks* finished without actually finishing it.

A repository returns fake data.

An exception becomes a `print`.

A `build()` method quietly grows past 400 lines.

A test checks the happy path while the actual failure case has never been exercised.

The code compiles, so the agent reports success.

That is the kind of "done" this project is trying to avoid.

There are currently 18 rules.

## Before

Ask an agent to build a profile header and you might get something like this:

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

Nothing here is catastrophic.

That's the problem.

It is the kind of code that looks reasonable when generated once, then slowly makes an entire codebase harder to work with.

## After

With the rules applied:

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

The important part is not what was added.

It is what was removed.

No unnecessary state.

No unnecessary `Container`.

No unnecessary `Center`.

No unnecessary `SizedBox`.

No unnecessarily complicated class name.

That is the general idea behind Flutter No Slop.

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

| #  | Rule                                                                            |
| -- | ------------------------------------------------------------------------------- |
| 1  | The project's existing conventions always win                                   |
| 2  | Verify APIs from source instead of guessing from memory                         |
| 3  | Look for an existing widget before creating another one                         |
| 4  | One class per file                                                              |
| 5  | Do not add a wrapper just to set a property the widget already supports         |
| 6  | Keep widget files under 200 lines and extract repeated code                     |
| 7  | Prefer `StatelessWidget` and `const`; dispose every resource you create         |
| 8  | Use plain names; avoid `Resolver`, `Orchestrator`, `Manager`, and similar noise |
| 9  | Comments explain why; do not force every file into the same template            |
| 10 | Never call a stub, fake implementation, or placeholder data "done"              |
| 11 | Do the task that was requested; do not sneak in unrelated refactors             |
| 12 | Handle errors explicitly; do not swallow exceptions or use `print`              |
| 13 | Guard `BuildContext` across async gaps                                          |
| 14 | Do not hardcode colours, text styles, or user-facing strings                    |
| 15 | Keep business logic out of widgets                                              |
| 16 | Keep a flow document for each feature                                           |
| 17 | Test failure paths, not only successful ones                                    |
| 18 | Run `dart analyze` before reporting the work as complete                        |

### Rule 10 matters more than it looks

An agent that gives you fake data, leaves half the implementation unfinished, and tells you the feature is complete has not saved you time.

It has simply postponed the work until you discover it.

And now you have to figure out what the agent actually did before you can continue.

## It reads the project before changing it

Rule 1 is deliberately first.

A generic set of Flutter rules should not bulldoze the conventions of an existing codebase.

If the project uses Riverpod, this does not tell the agent to introduce Bloc.

If the project already uses Mockito, it does not add another mocking library because that happens to be mentioned in some example.

If the project has its own naming conventions, those conventions matter more than this file.

The agent should look at the project first.

This skill is there to reduce unnecessary decisions, not create more of them.

## This is still evolving

This is early.

The rules came from seeing AI-generated Flutter code in actual projects, not from trying to define the one true way to write Flutter.

That means some rules will change.

A rule might miss a pattern.

Another might be too aggressive.

Something that seemed useful might turn out to be nothing more than personal preference.

When that happens, the rule should change.

The goal is not to accumulate 200 rules until the skill becomes another giant instruction manual. The goal is to keep a small set of rules that consistently prevents bad generated code.

If you use it and the agent gets something wrong, open an issue.

A before-and-after example is especially useful.

## Numbers

None yet.

Benchmarks are coming.

And they will be published whether they make this project look good or not.

A benchmark that only gets published when the result is flattering is marketing, not a benchmark.

## This is not a Flutter tutorial

There are already projects that teach agents how to work with Flutter and particular stacks.

For example, the [VGV AI Flutter Plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin) provides more domain-specific guidance.

Use those.

Flutter No Slop has a different job.

It is not trying to teach an agent everything about Flutter.

It is trying to stop the agent from making the codebase worse while it is doing its job.

Less abstraction.

Less duplication.

Less boilerplate.

Less pretending unfinished work is finished.

## Contributing

Found a generated-code pattern that keeps showing up and is not covered?

Open an issue with:

1. The generated code
2. What is wrong with it
3. What the code should have looked like

If it belongs in the skill, add one rule.

Keep `SKILL.md` under 500 lines. Detailed, stack-specific guidance belongs in `references/`.

The skill should become more useful, not simply become longer.

## License

MIT. See `LICENSE`.

---

Flutter and the related logo are trademarks of Google LLC. This project is not affiliated with or otherwise sponsored by Google LLC.
