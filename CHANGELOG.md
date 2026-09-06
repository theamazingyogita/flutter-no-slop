# Changelog

## 0.1.0

- Initial release
- `flutter-no-slop` skill with 12 rules covering API verification, file and
  class structure, widget composition, naming, comments, Bloc wiring,
  feature documentation, testing, and pre-completion verification

## 0.2.0

- Added rule 0: the project's existing conventions take precedence
- Added rules on stubs and placeholder data, task scope, error handling,
  `BuildContext` across async gaps, and hardcoded UI values
- Moved Bloc guidance, testing guidance, and the flow-doc template into
  `references/` so they load only when relevant
