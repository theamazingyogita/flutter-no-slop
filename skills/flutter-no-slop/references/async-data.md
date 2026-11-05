# Data on screen

"The backend sends it but the screen is empty" has one real cause. Find it before
changing anything.

**Trace it in order:**

1. The response. Is the field really there, with the name and type you expect?
2. The model. Does parsing keep it, or drop it because of a name or a null?
3. The state. Does it reach the state, and is it in `props` if the state uses
   Equatable?
4. The widget. Does this widget read it, and does it rebuild when it changes?

Report which step broke. Do not add a fallback that hides it.

**Stale responses.** The user switches city, date, or tab while a request is in
flight. The old response lands last and the screen shows the wrong data. Bloc runs
events concurrently by default, so this happens without anything looking wrong.

Check the response still belongs to what is selected before you use it:

```dart
Future<void> _onCityChanged(
  CityChanged event,
  Emitter<SummaryState> emit,
) async {
  emit(state.copyWith(cityId: event.cityId, isLoading: true));
  final summary = await repository.fetchSummary(event.cityId);
  if (state.cityId != event.cityId) return;
  emit(state.copyWith(summary: summary, isLoading: false));
}
```

If the project already uses `bloc_concurrency`, `restartable()` does this job. In
any other state library the idea is the same. Error handling is left out above,
the failure path still needs a state.

**Do not call what you do not need:**

- Load a tab when it first opens, not when its parent opens
- If the previous screen already has the data, pass it. Call the API when the user
  changes something
- Ask why before adding a second call for data another response already carries
- Fix the request in the app before asking for a backend change

**Lists that change:**

- A new sort or filter starts from page one and drops the old items
- A late page from the old query must not be added to the new list
- After a create, edit, or delete, update every screen showing that item
