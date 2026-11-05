# Checking a screen

Most UI bugs pass `dart analyze` and every test. Go through this list for any
screen you build or change. If you could not check something, say which one.

**Against the design:**

- Read the design file or screenshot before writing code, not after
- Compare position, size, colour, spacing, and which elements exist at all
- If the design and the existing app disagree, say so instead of picking quietly

**Light and dark mode:**

- Check both. Faded subtitles, dark icons on dark backgrounds, and glowing shadows
  mostly show up in the mode nobody tested
- Gradients, SVGs, and shadows do not follow the theme by themselves
- Change only what was asked. If the icon colour is wrong, the button stays as it is

**Loading:**

- Use the loader the app already uses. A lone spinner on a shimmer app looks broken
- Shimmer mirrors the loaded layout and fills the same space, not half the screen
- Every element of the screen is part of the shimmer. A real button sitting on a
  loading screen is a bug
- Changing a sort or filter shows loading too, not the old list

**Empty:**

- Show an empty state only after a request succeeded with no items
- Hide actions that mean nothing with no data, like "See all"
- The icon and text match this screen, not whichever empty state was nearby

**Error and retry:**

- A failure gets an error state, never an empty state and never an endless loader
- Retry shows loading straight away and really sends the request again. A retry
  button that does nothing visible looks dead

**Real content:**

- Try long names and long text. Popups and chips must fit their content
- Missing data takes no space, no blank row left behind
- Every tappable card goes somewhere. Check the tap, not just the layout

**Lists:**

- Check pull to refresh, pagination, and sort with real data
- Overlays and popups dismiss the normal way, by tapping outside or moving the map

**Consistency:**

- Reuse the existing button, chip, card, and dialog. A one-off version is a bug
  waiting for its second report
- Same size and style for the same element everywhere it appears
- Match the app's capitalisation for titles and buttons

**Testing empty and error states by hand:**

Forcing a state in the code is fine while testing. Remove it before you finish.
A widget test for the same state lasts longer and cannot be left behind.
