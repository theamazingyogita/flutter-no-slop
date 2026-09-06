---
name: flutter-no-slop
description: Enforces human-quality Flutter and Dart code — matches the project's existing conventions, verifies APIs instead of guessing them, refuses to ship stubs or placeholder data, keeps widget files small, handles errors instead of swallowing them, guards BuildContext across async gaps, bans hardcoded colours and strings, and requires tests and a clean dart analyze before reporting done. Use this skill whenever writing, refactoring, or reviewing any Flutter or Dart code, adding a screen, widget, Bloc or Cubit, or when the user mentions code quality, cleanup, or says generated code looks bloated or over-engineered. Apply it by default on Flutter work even when the user does not ask for it explicitly.
---

# Flutter: write it like a developer, not a generator

Generated Flutter code fails in recognisable ways. It invents constructor
parameters that don't exist. It wraps a `Container` in a `Padding` even though
`Container` takes padding directly. It names a class `AuthStateResolver` when
`LoginState` was the honest name. It leaves comments addressed to the next
agent instead of the next developer.

None of these break the build immediately, which is why they survive. They
surface later as review comments, runtime crashes, and a codebase nobody wants
to open.

Follow the rules below. Each one explains what it prevents, because knowing why
a rule exists is what makes it survive contact with an unusual case.

---

## 0. The project you are in wins

Read `pubspec.yaml`, the existing folder structure, and two or three nearby
files before writing anything. Match what is already there: the state
management library, the mocking library, the folder layout, the import style,
the naming conventions.

Where this document and the project disagree, the project wins — except on
correctness. A codebase that uses Riverpod does not get Bloc because this skill
mentions Bloc. A codebase that uses mockito does not get a second mocking
library added to it.

Consistency inside one codebase is worth more than any individual rule here.
The rules below describe good defaults for new code and for projects with no
established convention.

---

## 1. Verify APIs, never recall them

Model memory of package APIs is unreliable and confidently wrong. A guessed
parameter name compiles in your head and fails on the machine. Verification is
cheap; a hallucinated API costs the developer a debugging session.

Before using any package API:

- Read `pubspec.yaml` and `pubspec.lock` to confirm the package is a dependency
  and to get the exact resolved version. Version matters — APIs move between
  majors.
- For any constructor, parameter, or method you are not certain about, read the
  actual source in `.pub-cache` or the Flutter SDK rather than recalling it.
- Never add a dependency to `pubspec.yaml` that the project does not already
  have without telling the user, naming the package, and saying why.
- If you cannot verify an API, say so and ask. An honest "I need to check
  `go_router`'s redirect signature" is worth more than a plausible guess.

**Never invent:** package names, constructor parameters, enum values, extension
methods, or generated code from `build_runner` that has not actually been run.

---

## 2. Check before you create

Every new class is a maintenance cost. Most "new" widgets already exist in the
project under a different name.

Before adding any class, widget, or helper:

1. Search the codebase for something that already does this — grep for likely
   names and for the widget shape, not just the exact name you had in mind.
2. If something close exists, extend or parameterise it instead of writing a
   sibling.
3. If nothing exists, ask whether it belongs in the shared widget directory or
   the feature folder. A widget used by two features belongs in shared.

The failure this prevents: four near-identical `PrimaryButton`,
`AppButton`, `CustomButton`, and `MainButton` classes in one codebase, each
slightly different, none deletable.

---

## 3. One class per file

Put each class in its own file, named after it in `snake_case`:
`login_form.dart` holds `LoginForm`.

Two exceptions, both idiomatic Dart:

- Bloc `part` files — `login_event.dart` and `login_state.dart` joined to
  `login_bloc.dart` via `part` / `part of`.
- Sealed class hierarchies, where the subtypes must live beside the parent for
  exhaustiveness to work.

Everything else gets its own file. A file with five classes is a file nobody can
find anything in, and it makes every diff look larger than it is.

---

## 4. Do not nest widgets that do not need nesting

This is the most visible marker of generated Flutter code. Widgets that already
carry a property get wrapped in a widget providing that same property.

**Wrong:**
```dart
Padding(
  padding: const EdgeInsets.all(16),
  child: Container(
    color: Colors.blue,
    child: Text('Hello'),
  ),
)
```

**Right:**
```dart
Container(
  padding: const EdgeInsets.all(16),
  color: Colors.blue,
  child: const Text('Hello'),
)
```

Apply the same check to these common cases:

- `Container` takes `padding`, `margin`, `alignment`, `color`, `decoration`,
  `width`, `height` — do not wrap it in `Padding`, `Center`, `Align`,
  `ColoredBox`, or `SizedBox` for any of those.
- Use `Padding` alone when there is nothing else to configure — a bare
  `Container` used only for padding should be a `Padding`.
- `Column` and `Row` have `mainAxisAlignment` and `crossAxisAlignment` — do not
  wrap children in `Center` to achieve the same thing.
- `Column` and `Row` take `spacing` — use it instead of inserting `SizedBox`
  between every child.
- `SizedBox.shrink()` over `Container()` for an empty widget.
- Do not wrap a single child in a `Column` or `Stack`.

Before adding a wrapper, check whether the child widget already exposes the
property you want. It usually does.

---

## 5. Keep UI files small, and extract anything that repeats

A `build` method that runs for hundreds of lines cannot be read, reviewed, or
safely edited. It also rebuilds as one unit, so a change to any part of it
rebuilds all of it.

**File size:**

- Under 200 lines is healthy for a widget file.
- 200–500 lines means look for something to extract, and usually there is
  something obvious.
- Over 500 lines is a defect. Split it before doing anything else in that file.

**Extract a widget when any of these is true:**

- The same visual block appears twice anywhere in the project. Twice is the
  threshold — not three times, not "when it gets messy."
- A section of the tree has a name you would say out loud: the order summary,
  the avatar row, the empty state. If it has a name, it is a widget.
- Nesting in `build` passes about four levels.
- A part of the screen rebuilds on state the rest does not care about. Extract
  it so the rebuild stays local.

**Where the extracted widget goes:**

- Used by one feature → `lib/<feature>/view/widgets/`
- Used by two or more features → the shared UI directory the project already
  uses. Never copy it into a second feature.

**Extract to a class, not a method.** A `Widget _buildHeader()` method looks
tidy but is not a widget — it has no element in the tree, cannot be `const`,
cannot have its own state, and rebuilds with its parent every time. A private
`class _Header extends StatelessWidget` in the same file costs three extra
lines and is strictly better. Reach for a method only for something trivial and
used once.

This rule and "check before you create" (section 2) pull in opposite directions
on purpose. Section 2 stops you inventing a fourth button class. This section
stops you inlining the same block twice because you were reluctant to create
anything. The resolution is the same in both cases: **search first, then
either reuse what exists or extract once and reuse it everywhere.** What you
must never do is duplicate.

---

## 6. Prefer StatelessWidget, and dispose what you create

**Default to `StatelessWidget`.** Reach for `StatefulWidget` only when the
widget genuinely owns a lifecycle: a controller, an animation, a focus node, a
subscription. If the thing you want to hold is feature state — the user, the
cart, the loading flag — it belongs in a Bloc or Cubit, not in `setState`.

A screen that keeps its data in `setState` looks simpler for one afternoon and
then cannot be tested, restored, or shared. If you find yourself reaching for
`StatefulWidget` to store data rather than to manage a resource, the state is
in the wrong place.

**Use `const` constructors everywhere they are possible.** A `const` widget is
not rebuilt when its parent rebuilds. This is the cheapest performance win in
Flutter and generated code almost always misses it. `dart analyze` will point
out the ones you missed if the project has the lint enabled.

**Dispose resources in `StatefulWidget`.** Anything you create in `initState`
or as a field gets released in `dispose`, or it leaks:

```dart
@override
void dispose() {
  _controller.dispose();
  _subscription.cancel();
  super.dispose();
}
```

This covers `TextEditingController`, `ScrollController`, `AnimationController`,
`PageController`, `TabController`, `FocusNode`, `StreamSubscription`, and
`Timer`. Call `super.dispose()` last.

**Blocs are different, and this is where mistakes happen.** A Bloc has
`close()`, not `dispose()`, and you usually should not call it yourself:

- Created with `BlocProvider(create: (_) => MyBloc())` — the provider closes it
  automatically when it leaves the tree. Do not close it manually.
- Passed with `BlocProvider.value(value: existingBloc)` — the provider does
  **not** close it. Whoever created it owns closing it. This is the common
  source of both "used after close" errors and leaked Blocs.
- Registered globally at app root, or in a service locator for the whole app
  lifetime — it lives as long as the app. Do not close it.

The one time you write cleanup inside a Bloc is when the Bloc itself started a
subscription:

```dart
@override
Future<void> close() {
  _subscription.cancel();
  return super.close();
}
```

Everything a Bloc opens, the Bloc closes. Everything a widget opens, the widget
closes. Do not reach across that line.

---

## 7. Names a developer would choose

Name things after what they are in the product, not after a design-pattern
vocabulary. The reader should be able to guess what a class does from its name
without opening it.

**Avoid these suffixes** unless the class genuinely is one: `Resolver`,
`Orchestrator`, `Coordinator`, `Processor`, `Handler`, `Manager`, `Executor`,
`Provider` (when it is not a real provider), `Helper`, `Utils`, `Base` (with a
single subclass).

| Instead of | Write |
|---|---|
| `AuthenticationStateResolver` | `LoginState` |
| `UserDataFetchOrchestrator` | `UserRepository` |
| `ItemListRenderingHelper` | `ItemList` |
| `performDataRetrievalOperation()` | `fetchOrders()` |
| `handleUserInteractionEvent()` | `onSubmitPressed()` |

Do not add an abstraction until there is a second caller. One interface with one
implementation, or a factory that constructs exactly one type, is indirection
with no benefit — it makes the reader chase a hop for nothing.

Methods are verbs (`fetchOrders`, `submitForm`). Booleans read as assertions
(`isLoading`, `hasError`). Follow effective Dart: `UpperCamelCase` for types,
`lowerCamelCase` for members, `snake_case` for files.

---

## 8. Comments for developers only

Write comments that explain **why**. The code already says what.

Delete or never write:

- Narration: `// Initialize the controller`, `// Loop through the items`,
  `// Build the widget`
- Anything addressed to an AI: `// Note for the agent:`,
  `// This section handles the logic as requested`, `// TODO: agent should
  verify`
- `TODO`s with no owner or ticket
- Commented-out code — delete it, git has it

Write instead:

```dart
// The API returns createdAt in UTC but the design shows local time.
final localTime = order.createdAt.toLocal();
```

Use `///` doc comments on public APIs of shared widgets and repositories, where
someone will read them from another file.

---

## 9. Never ship a stub and call it done

The most damaging thing a coding agent does is report a feature complete when
it returns hardcoded data. The code compiles, the screen renders, and the bug
is found in QA or production instead of in review.

- Never return placeholder or invented data from a repository or data source to
  make a screen render.
- Never write `// TODO: implement` and describe the feature as finished.
- Never write an empty method body to satisfy an interface without saying so.

If something cannot be completed — an endpoint does not exist yet, a design is
ambiguous, a credential is missing — stop and say exactly what is blocking.
A clear "the orders endpoint is not in the API spec, so I stubbed the
repository and marked it" is useful. A silent stub is not.

---

## 10. Stay inside the task

Change what was asked and the files that change requires. Nothing else.

- Do not reformat, rename, or restructure files unrelated to the task.
- Do not upgrade dependencies, change lint rules, or alter build configuration
  unless that is the task.
- Do not delete code you do not understand. If something looks wrong but is out
  of scope, mention it and leave it.

A large diff hides the actual change. The developer has to review every line,
and the real work gets lost in noise they did not ask for.

---

## 11. Handle errors, do not swallow them

Generated code catches exceptions and discards them, which turns a clear
failure into a silent one.

**Never write:**

```dart
try {
  await repository.fetch();
} catch (e) {
  print(e);
}
```

That has three problems: `print` does not exist in a release build's logs, the
caller has no idea anything failed, and the UI keeps spinning.

**Instead:**

- Catch specific exceptions where you can act on them, not bare `catch (e)`.
- Surface failures as state the UI renders — an error state, not a swallowed
  log line.
- Use the project's logging setup. Never `print` in application code.
- Never use `!` to force-unwrap a nullable just to silence the analyser. Handle
  the null case or explain in a comment why it cannot happen.
- Rethrow if you cannot handle it: `catch (e) { ... rethrow; }`.

---

## 12. Guard BuildContext across async gaps

Using a `BuildContext` after an `await` crashes if the widget was disposed
while the future was in flight. This is a real runtime failure, not a style
preference, and `dart analyze` flags it as `use_build_context_synchronously`.

```dart
Future<void> _submit() async {
  await context.read<LoginBloc>().stream.first;
  if (!mounted) return;
  Navigator.of(context).pop();
}
```

Capture what you need before the `await` where possible, and check `mounted`
after it where not. Never silence this lint with an ignore comment.

---

## 13. No hardcoded values in UI

Hardcoded colours, sizes, and strings are why a design change becomes a
find-and-replace across fifty files.

- Colours come from `Theme.of(context).colorScheme`, never `Color(0xFF...)`
  inline.
- Text styles come from `Theme.of(context).textTheme`, never a bare `TextStyle`
  with a hardcoded size.
- Spacing uses the project's existing constants if it has them.
- User-facing strings go through the project's localisation setup. Never a
  hardcoded English string in a widget if the project has l10n configured.

If the project has no theme or l10n setup, follow what it does today and say
that adding one would help — do not introduce one uninvited.

---

## 14. State management

Match whatever the project already uses. Read `pubspec.yaml` first.

If the project uses **flutter_bloc**, read `references/bloc.md` before writing
any Bloc, Cubit, event, state, or provider. It covers event-driven structure,
widget selection, and the wiring that fails silently — missing providers,
incomplete `props`, unguarded `emit` after `await`, and who owns closing a Bloc.

If the project uses Riverpod, Provider, signals, or anything else, follow that
library's conventions and the project's existing patterns. Do not migrate a
project to a different state management library unless asked.

Regardless of library, two rules hold: business logic never lives in a widget,
and the UI describes what happened rather than calling logic directly.

---

## 15. A flow document per feature

When you build or substantially change a feature, write or update
`docs/features/<feature>.md` in the same change. It records what the feature
does, the flow from user action to result, the files involved, the states, and
the edge cases — so the next person does not have to read every file to
understand it. See `references/flow-doc.md` for the template.

A stale flow doc is worse than none, so update it with the code, not after.

---

## 16. Test every feature, including failure

Read `references/testing.md` before writing tests.

The short version: every feature gets tests covering the failure path, not just
the happy path. Every visual branch — loading, loaded, empty, error — gets a
widget test. Every test must fail if the implementation is removed. Match the
project's existing test and mocking libraries rather than introducing a second
one.

---

## 17. Verify before reporting done

Do not tell the user the work is complete until this passes:

1. `dart analyze` — zero errors and zero warnings. Fix them; do not silence
   them with ignore comments unless the user asks.
2. `dart format .`
3. `flutter test` — all tests pass, including the ones you just wrote.

If any step fails, fix it and run again. Reporting completion on code that does
not analyse cleanly is the single fastest way to lose the developer's trust.

---

## Quick self-check

Before finishing any Flutter change, confirm:

- [ ] Matched the project's existing libraries, structure, and conventions
- [ ] Every API used was verified in source, not recalled
- [ ] No stubs, placeholder data, or unfinished work reported as done
- [ ] Nothing changed outside the scope of the task
- [ ] Searched for an existing widget before creating a new one
- [ ] One class per file; no widget file over 500 lines
- [ ] No visual block appearing twice — extracted once and reused
- [ ] No widget wrapping another for a property it already has
- [ ] `StatelessWidget` and `const` wherever possible
- [ ] Every controller, subscription, and timer disposed
- [ ] No swallowed exceptions, no `print`, no unexplained `!`
- [ ] `mounted` checked before using `BuildContext` after an `await`
- [ ] No hardcoded colours, text styles, or user-facing strings
- [ ] Plain names; no `Resolver` / `Manager` / single-use abstractions
- [ ] Comments explain why, addressed to developers
- [ ] Logic lives outside widgets
- [ ] Flow doc written or updated
- [ ] Tests cover success and failure for every state
- [ ] `dart analyze` clean, `dart format` run, `flutter test` green
