[![npm version](https://img.shields.io/npm/v/@itrocks/air-datepicker?logo=npm)](https://www.npmjs.org/package/@itrocks/air-datepicker)
[![npm downloads](https://img.shields.io/npm/dm/@itrocks/air-datepicker)](https://www.npmjs.org/package/@itrocks/air-datepicker)
[![GitHub](https://img.shields.io/github/last-commit/itrocks-ts/air-datepicker?color=2dba4e&label=commit&logo=github)](https://github.com/itrocks-ts/air-datepicker)
[![issues](https://img.shields.io/github/issues/itrocks-ts/air-datepicker)](https://github.com/itrocks-ts/air-datepicker/issues)
[![discord](https://img.shields.io/discord/1314141024020467782?color=7289da&label=discord&logo=discord&logoColor=white)](https://25.re/ditr)

# air-datepicker

Simplifies the integration of the air-datepicker component into a web page.

*This documentation was written by an artificial intelligence and may contain errors or approximations.
It has not yet been fully reviewed by a human. If anything seems unclear or incomplete,
please feel free to contact the author of this package.*

## Installation

```bash
npm i @itrocks/air-datepicker
```

`@itrocks/air-datepicker` ships with `air-datepicker` and
`@itrocks/asset-loader` as dependencies. The helper automatically loads the
CSS, JS and locale files for you; you only need to make sure your HTTP server
exposes the usual `/lib` static paths.

## Usage

`@itrocks/air-datepicker` provides a single helper function
`airDatePicker(input: HTMLInputElement): void`.

It takes a plain text input and turns it into an
[air-datepicker](https://air-datepicker.com/) date picker with the correct
locale, inferred from the `<html lang="…">` attribute.

Behind the scenes it:

- loads the `air-datepicker.css` stylesheet,
- loads the `air-datepicker.js` script,
- loads the locale file matching `document.documentElement.lang`,
- instantiates a date picker bound to the given input,
- slightly tweaks keyboard handling to better integrate with the
  it.rocks UI stack.

### Minimal example

HTML:

```html
<!doctype html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>Example</title>
</head>
<body>
	<label for="start-date">Start date</label>
	<input id="start-date" name="startDate" type="text">

	<script type="module" src="/main.js"></script>
</body>
</html>
```

JavaScript / TypeScript (`main.ts`):

```ts
import { airDatePicker } from '@itrocks/air-datepicker'

const input = document.querySelector<HTMLInputElement>('#start-date')

if (input) {
	airDatePicker(input)
}
```

When the page loads, `airDatePicker` will dynamically load the
`air-datepicker` assets and attach the picker to `#start-date`.

### Complete example with localisation and multiple fields

In a typical it.rocks application you might have several date fields on the
same page. The locale is chosen once from the `<html lang>` attribute (for
example `fr`), and the corresponding `/node_modules/air-datepicker/locale/fr.js`
file is fetched automatically.

```html
<!doctype html>
<html lang="fr">
<head>
	<meta charset="utf-8">
	<title>Booking form</title>
</head>
<body>
	<form>
		<label for="check-in">Arrivée</label>
		<input id="check-in" name="checkIn" type="text">

		<label for="check-out">Départ</label>
		<input id="check-out" name="checkOut" type="text">
	</form>

	<script type="module" src="/booking.js"></script>
</body>
</html>
```

```ts
// booking.ts
import { airDatePicker } from '@itrocks/air-datepicker'

function enhanceDateField (selector: string) {
	const input = document.querySelector<HTMLInputElement>(selector)
	if (input) {
		airDatePicker(input)
	}
}

enhanceDateField('#check-in')
enhanceDateField('#check-out')
```

`airDatePicker` takes care of:

- loading the `fr` locale only once, even if you call it for several
  inputs;
- creating a separate date picker instance for each field;
- preventing the Enter key from submitting the form while the picker is
  open, and keeping arrow navigation inside the calendar.

If you need advanced configuration (range selection, custom buttons, inline
mode, etc.) you can still use the underlying `air-datepicker` library
directly; this package focuses on the simple "attach a date picker to an
input with the right locale" case and provides typed helpers for the
upstream options.

## API

### `function airDatePicker(input: HTMLInputElement): void`

Initialises an `air-datepicker` instance on the given text input.

#### Parameters

- `input`: the `HTMLInputElement` to enhance.

#### Behaviour

1. Calls `loadCss('/lib/air-datepicker/air-datepicker.css')` via
   `@itrocks/asset-loader`.
2. Calls
   `loadScript('/lib/air-datepicker/air-datepicker.js', callback)`
   and waits for the script to be loaded.
3. Loads the locale file
   `/lib/air-datepicker/locale/{lang}.js`, where `lang` is
   `document.documentElement.lang` (for example `en`, `fr`, `de`).
4. Extracts the locale object from the loaded script and caches it for the
   current `lang`.
5. Instantiates a new
   `AirDatepicker(input, { locale })` from the global `window.AirDatepicker`
   constructor provided by the `air-datepicker` library.
6. Adds a `keydown` listener on the input to:
   - prevent the default behaviour of the Enter key while the calendar is
     visible, avoiding accidental form submission;
   - stop propagation of left/right arrow keys so they navigate inside the
     calendar instead of interacting with global shortcuts.

The function returns `void`; the created `AirDatepicker` instance is not
exposed by this helper.

### Exported types

The package also exports several TypeScript types that mirror the official
`air-datepicker` API. These are mainly useful when you work directly with
`AirDatepicker` from the original library and still want strong typing in
your project.

They are re‑exported from this package so you can import them from
`'@itrocks/air-datepicker'` instead of referencing
`'air-datepicker/air-datepicker'` yourself.

#### `type AirDatepickerLocale`

Shape of a locale object used by `air-datepicker`.

Key fields:

- `days`, `daysShort`, `daysMin` – arrays of day names;
- `months`, `monthsShort` – arrays of month names;
- `today`, `clear` – labels for the Today / Clear buttons;
- `dateFormat`, `timeFormat` – default display formats;
- `firstDay` – index (0–6) of the first day of the week.

#### `type AirDatepickerButton`

Describes a custom button that can appear in the date picker footer when you
use the underlying `AirDatepicker` options.

- `content`: string or function that returns the label;
- `tagName`: tag to use (defaults to `button`);
- `className`: CSS class(es) to apply;
- `attrs`: additional HTML attributes;
- `onClick`: callback invoked when the button is pressed.

#### `type AirDatepickerButtonPresets = 'clear' | 'today'`

Named button presets recognised by `air-datepicker`.

#### `type AirDatepickerPosition`

String literals controlling where the picker is displayed relative to the
input (for example `'top'`, `'bottom left'`, `'right bottom'`, etc.).

#### `type AirDatepickerViews` / `type AirDatepickerViewsSingle`

Represent the possible views (`'days'`, `'months'`, `'years'`) and single
cell types (`'day'`, `'month'`, `'year'`).

#### `type AirDatepickerDate`

Accepted date input formats: `string`, `number` (timestamp) or `Date`.

#### `type AirDatepickerNavEntry`

Type for navigation title entries (a simple string or a function that
returns a string given the current picker instance).

#### `type AirDatepickerDecade = [number, number]`

Represents a decade range as `[startYear, endYear]`.

#### `type AirDatepickerPositionCallback`

Callback signature used when you provide a function to fully control the
popup position instead of a simple `AirDatepickerPosition` string.

You receive the date picker elements and must position them, then call the
`done` callback once you are finished. You may optionally return a cleanup
function.

#### `type AirDatepickerOptions<E extends HTMLElement = HTMLInputElement>`

Strongly‑typed configuration object for the `AirDatepicker` constructor. It
groups all the available options from the upstream library:

- **Display & behaviour**: `classes`, `inline`, `startDate`, `firstDay`,
  `isMobile`, `visible`, `fixedHeight`, `weekends`.
- **Formatting**: `dateFormat`, `altField`, `altFieldDateFormat`,
  `dateTimeSeparator`, `timeFormat`.
- **Selection rules**: `selectedDates`, `minDate`, `maxDate`,
  `multipleDates`, `multipleDatesSeparator`, `range`, `dynamicRange`,
  `toggleSelected`, `disableNavWhenOutOfRange`.
- **Layout / positioning**: `container`, `position`, `offset`, `view`,
  `minView`, `monthsField`.
- **Visibility of other months / years**: `showOtherMonths`,
  `selectOtherMonths`, `moveToOtherMonthsOnSelect`, `showOtherYears`,
  `selectOtherYears`, `moveToOtherYearsOnSelect`.
- **Time picker**: `timepicker`, `onlyTimepicker`, `minHours`, `maxHours`,
  `minMinutes`, `maxMinutes`, `hoursStep`, `minutesStep`.
- **Buttons & navigation**: `buttons`, `prevHtml`, `nextHtml`, `navTitles`.
- **Lifecycle callbacks**:
  - `onSelect` – called when the selection changes;
  - `onChangeViewDate` – called when the current view date moves to
    another month / year / decade;
  - `onChangeView` – called when switching between day / month / year
    views;
  - `onRenderCell` – lets you customise each cell (disable specific
    dates, add CSS classes, change HTML, add attributes);
  - `onShow` / `onHide` – called when the picker is shown / hidden;
  - `onClickDayName` – called when a day name (Mon, Tue, …) is clicked;
  - `onBeforeSelect` – lets you veto a selection by returning `false`;
  - `onFocus` – called when a date cell receives focus.

You typically use `AirDatepickerOptions` when constructing
`new AirDatepicker(input, options)` directly in your own code, while still
benefiting from the same typings as the helper.

> Note: the helper `airDatePicker` defined in this package does **not**
> accept an options object; it always uses the default `air-datepicker`
> configuration enriched with the computed `locale`. Use `AirDatepicker`
> from the underlying library if you need full control.

## Typical use cases

- Add a calendar widget to one or more date fields in a plain HTML form
  without worrying about loading assets or locales.
- Automatically localise the date picker based on the `<html lang>`
  attribute (`en`, `fr`, etc.) in a multi‑lingual web application.
- Integrate `air-datepicker` in an it.rocks front‑end using the same
  asset‑loader conventions as other UI modules.
- Prepare more advanced usages of `air-datepicker` (range selection,
  custom buttons, inline calendars) while keeping strong TypeScript typing
  for options via the exported `AirDatepickerOptions` and related types.
