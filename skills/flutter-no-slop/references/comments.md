# Comments and written style

Write comments that explain **why**. The code already says what.

Delete or never write:

- Narration: `// Initialize the controller`, `// Loop through the items`,
  `// Build the widget`
- Anything addressed to an AI: `// Note for the agent:`,
  `// This section handles the logic as requested`, `// TODO: agent should
  verify`
- `TODO`s with no owner or ticket
- Commented-out code, delete it, git has it

Write instead:

```dart
// The API returns createdAt in UTC but the design shows local time.
final localTime = order.createdAt.toLocal();
```

Use `///` doc comments on public APIs of shared widgets and repositories, where
someone will read them from another file.

**Do not over-document.** Exhaustive comments on every line, and doc comments
on every private method and obvious parameter, are the clearest sign code was
machine-written. Document the public API of shared code and anything genuinely
surprising. Leave the rest alone.

**Punctuate comments the way a developer types.** Long dashes, semicolons in
prose, and carefully balanced clauses are the fingerprint of generated text.
Developers type commas, full stops, and parentheses. Write short sentences.

```dart
// Wrong, this reads as machine-written:
// The API returns UTC, however the design requires local time; therefore we
// convert here, ensuring consistency across all screens.

// Right:
// API sends UTC but the design shows local time.
```

The same applies to commit messages, pull request descriptions, and any
markdown you write into the repository. If a sentence would not survive being
typed quickly by a tired person, rewrite it.

**Do not make every file the same shape.** Generated code is recognisable by
its uniformity: every widget structured identically, every method the same
length, the same three sections in the same order in file after file. Real
codebases vary because problems vary. Let a simple widget be short and a
complex one be long. Do not add a section to a file because the last file had
one.

