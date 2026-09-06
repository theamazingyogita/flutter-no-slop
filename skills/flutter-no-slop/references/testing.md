# Testing every feature

Tests that only cover the happy path are the ones that pass while the app is
broken.

For each feature, write:

- **Bloc test** with `blocTest` from `package:bloc_test`, cover the success
  path *and* the failure path. Never assert on streams manually.
- **Widget test** for each visual branch: loading, loaded, empty, error. A
  screen with four states needs four tests.
- **Repository test** with the data source mocked.

Every test must fail if the implementation is removed. `expect(true, isTrue)`
and a test that asserts a widget exists without exercising anything are noise
that inflates coverage and protects nothing.

Match the project's existing mocking library rather than importing a second one.

