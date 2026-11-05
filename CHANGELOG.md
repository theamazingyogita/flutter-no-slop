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

## 0.3.0

- Added rules on UI checks, fixing a bug everywhere it appears, tracing data to
  the real cause, and cleaning up before finishing
- Added `references/ui-states.md` and `references/async-data.md`
- Oversized files are now flagged, not split in the middle of another task
- `dart format` runs on changed files only, and `dart analyze` checks for new
  issues only
- Simple sections stay inline in `build`, and values used once stay inline
- Feature docs are only required where the project keeps them
- Added `context.mounted` and more vague names to avoid
- Moved who closes a Bloc into `references/bloc.md`
