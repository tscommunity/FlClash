---
name: ui-work
description: Use when changing FlClash Flutter UI, widgets, screens, color and surface styling, icons or glyphs, shapes and corner radii, navigation surfaces, async feedback, or user-facing interactions.
---

# UI Work

## When To Use

Use this for user-facing Flutter UI changes in `lib/`, including widgets, screens, navigation surfaces, settings rows,
dialogs, and interaction behavior.

## Conventions

- Reuse the patterns of nearby widgets, providers, notifiers, and helpers before adding an abstraction, and keep to the
  look the rest of the app has: its own seeded color system, with Apple's interaction conventions (square corners,
  sheets that snap between detents, a floating tab bar), all drawn by `prism_ui`. `Card`, `PressSurface`, and `ListRow`
  press with `PressLook.ink`, a state layer plus, on Android, a sparkle drawn by `PressSparkles`, and a desktop hover is
  faint (Press in `.agents/surfaces.md`). Do not introduce a visual system for one screen or rewrite a layout the
  request does not need.
- Widgets come from `package:flutter/widgets.dart` and components from `prism_ui`; no design library is imported. The
  Generated Code section of `.agents/rules.md` has the rule.
- `.agents/rules.md` is binding here: the lint set (`child:` last, `const`, final locals), the square-corner API,
  and the widget-test facts under Testing Rules (`pumpAndSettle` never returns on `EditorView`, `dispose()` must not
  await, auto-dispose providers need a container-level hold, platform decisions are passed in as parameters).
- A component that knows nothing of FlClash goes in the `prism_ui` package, the submodule `plugins/prism_ui`; what FlClash adds to it without reading app
  state (its glyphs, fonts, page illustrations) goes in `lib/ui/`; one built for an app feature goes in
  `lib/widgets/`. The UI Layer section of `.agents/architecture.md` draws the boundary and says how a `prism_ui`
  component gets app state.
- Settings screens are built from the `ConfigToggleItem` family in `lib/widgets/config_item.dart`; the Settings Rows
  section of `.agents/architecture.md` says when a file-local helper or a named widget is warranted.
- Every icon is a glyph, never an icon font: reuse or add one in `AppGlyphs` (`Glyphs` when a
  `prism_ui` component draws it) and show it
  with `GlyphIcon`, or with `AnimatedGlyph` where it fills on selection. Glyphs are outlined, except inside a filled
  button, where they take `fill: 1`. The Icons section of `.agents/architecture.md` covers shape roles, the stroke, the
  fill animation, and when to fill.
- Text sizes and weights come from `AppTextStyle` in `prism_ui/lib/typography.dart`, Apple's iOS Dynamic Type scale, never a
  bare `fontSize`, or a `fontWeight`; the Typography section of `.agents/rules.md` has the
  rule.
- Modals dim what they cover and never blur it: sheets with the Material 3 scrim (`ColorScheme.modalScrim`, `scrim` at
  32%), alerts with `ColorScheme.alertScrim`, the lighter dim iOS 27 puts behind an alert. Every modal brings its own
  scrim, including one opened from inside another, so do not make a barrier transparent to keep scrims from stacking.
- A sheet's size belongs to its route, never to its content or call site: a bottom sheet rests at
  `SheetProps.detents`, a side sheet is as tall as the window and as wide as `ModalSideSheetRoute` makes it from
  the window width (`prism_ui/lib/side_sheet.dart`, pinned by `test/widgets/side_sheet_width_test.dart`), and a form sheet
  is a card `FormSheetRoute` caps in width and height (`prism_ui/lib/form_sheet.dart`). Do not wrap a sheet's content in a
  fixed width or height to resize one sheet.
- A form with more than one value opens with `showFormSheet` and lays its values out in `FormRow`s, never as labeled
  fields in an `AlertDialog`; a prompt for one value stays an alert. Forms in `.agents/architecture.md` has the rule.
- A modal scrolls only once the window and keyboard leave it no room, and then through one scroll view: `AlertDialog`
  caps its body at nothing but the space it is given, and a multi-line field in a dialog or sheet grows with its text
  (`maxLines: null`) rather than scrolling inside a line count. With the keyboard up, a bottom sheet keeps what it
  shows above the keyboard, rising past its tallest detent as far as a peek below the top (`_top` in
  `prism_ui/lib/snap_sheet.dart`).
- The keyboard animates `MediaQuery.viewInsets` every frame. Read MediaQuery by aspect (`paddingOf`, `viewInsetsOf`)
  rather than `MediaQuery.of`, and wrap the page of a new route type in `KeyboardInsetHold`; Keyboard Insets in
  `.agents/architecture.md` has the reasons.
- A page loads what it shows as it mounts, never once its route has settled; what waits out a transition is work
  that repeats, such as a live feed's refreshes (`RouteMotionHoldMixin`), and the rows past the viewport
  (`ArrivalCacheExtentMixin`). Page Arrival in `.agents/architecture.md` has the rule.
- A page that waits on its data shows nothing until all of it is ready (`NullStatusSwitcher.isLoading`), and a load
  that can run long adds its illustration or a page spinner as `loadingStatus`, never a skeleton screen; Page Arrival in
  `.agents/architecture.md` has the rule.
- App bar buttons reach `Scaffold` as data (`primaryAction`, `iconActions`, `menuItems`) so it can fold them into
  its overflow menu; App Bar Actions in `.agents/architecture.md` has the rule.
- A row's secondary actions open from a context menu, never a trailing more button; a list whose rows have one
  reorders in a sort mode, any other from the row itself, a mouse dragging it and a finger long pressing first. Row
  Actions And Sort Mode in `.agents/architecture.md` has the rule.
- Quick edit belongs only on a page that edits a whole list or section, or one isolated item; never on one that edits a
  part of something other settings also write. Quick Edit in `.agents/architecture.md` has the rule.
- A page or sheet opens on the navigator of the context it comes from, never on the root navigator unless Where A
  Route Opens in `.agents/architecture.md` allows it.
- User-facing text goes through ARB; use `localization` when text changes are non-trivial.
- An asynchronous control keeps four things apart: authoritative provider/domain state; display-only state such as a
  minimum progress duration; tap policy while work or a display hold is active; and failure/disposal cleanup, in
  `finally` for animations and timers. `CoreStatusButton` and the proxies delay test action are the reference implementations,
  described under User-Facing Core And Delay Feedback in `.agents/architecture.md`.

## Corner Radii

Every corner is square. `AppCorner`, `AppRadius`, and `AppShape` in `prism_ui/lib/shape.dart` resolve to a zero radius,
`full` and `circle` included, and their builders ignore any magnitude passed to them. Do not add a non-zero radius at a
call site, a `StadiumBorder`, a `CircleBorder`, or `BoxShape.circle`.

- Use an `AppShape` token for a shape, `AppRadius.none` where a `BorderRadius` is required. The old tokens (`xs` to
  `xxl`, `alert`, `full`) remain as names so call sites keep their intent, but every one is square.
- `AppCorner.fit(shortestSide)` and the `AppRadius`/`AppShape` builders stay in the API for compatibility; they always
  return a square result. Do not reintroduce a size-dependent radius behind them.
- The components of `prism_ui` already take their shapes from these tokens. Leave a field's `border` unset so it keeps
  `AppShape.input`, which now outlines a rectangle.
- Colors of surfaces follow `.agents/surfaces.md`: fills by `SurfaceLevel`, no structural borders, and a selected
  item keeping its fill under a `SelectionCheck` or `SelectionBadge`, with no container role on a card or row.

Outlined inputs use `AppInputBorder`, which paints a rectangular border through the floating label's notch.

On Windows the native window corner is disabled at startup in `lib/manager/window_manager.dart` and stays disabled
across maximize and unmaximize. The glass shader takes its corner curves from the same zero radii, so glass panes,
menus, and dock surfaces are square too.

## Verification

- A behavior change gets a focused widget test covering rendering states, taps, scrolling, and empty/error states,
  including the exception path of any async action. Run `flutter analyze` and `flutter test test/widgets/` plus the
  tests of the touched view.
- The scans in `test/lint/` fail on the common UI mistakes and run as part of `flutter test`: an `IconButton`
  whose icon is an icon needs a `tooltip`; a field that constructs its own controller, notifier, or focus node is released in
  the same file; a public declaration nothing references is dead. `.agents/rules.md` explains each rule and its
  allowlist.

## Pitfalls

- A list, row, or animation whose cost grows with the user's data follows UI Frame Budget in `.agents/rules.md`; use
  the `ui-performance` skill to measure it.
- Do not mutate provider/domain state merely to smooth a transition. Keep presentation holds local and let real errors
  bypass them immediately.
- Do not leave loading animations active when callbacks throw.
- Do not manually edit generated localization or provider files.
