# Bloc: events in, state out

The point of Bloc is that every state change has a traceable cause. Code that
calls methods on a Bloc from the UI throws that away.

**Structure:**

- The UI dispatches events: `context.read<LoginBloc>().add(LoginSubmitted())`.
  Never call a public method on a Bloc from a widget.
- Business logic lives in the Bloc or Cubit. Widgets do not compute, transform,
  or decide — they render state and dispatch events.
- Page provides, View consumes. The `Page` widget owns the `BlocProvider`; the
  `View` widget below it uses `BlocBuilder` / `BlocListener`.
- No Bloc depends on another Bloc. They communicate through a shared repository
  or through the UI layer.

**Events and states stay small.** An event is a fact that happened
(`CartItemRemoved`), not a command with options. A state is what the screen
needs to render, nothing more. If a state class has eight fields, the screen is
probably doing two jobs.

**Widget selection:**

- `BlocBuilder` — rebuild UI on state change
- `BlocListener` — side effects only: navigation, snackbars, dialogs
- `BlocConsumer` — both
- `BlocSelector` — rebuild on one field only
- `context.read` inside callbacks; `context.watch` or `BlocBuilder` inside
  `build`. Never `context.watch` in a callback.

**Wiring that gets forgotten.** Check each of these — they fail silently or
only at runtime:

- `Bloc.observer` assigned in `main()` if the project has a `BlocObserver`
- A `BlocProvider` actually mounted above every widget that reads that Bloc
- `BlocProvider.value` used only for an existing instance, never where `create`
  was meant — otherwise nothing closes the Bloc
- Every field of a state included in `props` (Equatable) or in the freezed
  definition — a missing field means `emit` fires and the UI never rebuilds
- `emit` guarded by `isClosed` after any `await`
- Error states present in the state hierarchy and handled in the UI, so a
  failure does not render as a permanent spinner

