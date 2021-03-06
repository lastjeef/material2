The datepicker allows users to enter a date either through text input, or by choosing a date from
the calendar. It is made up of several components and directives that work together:

<!-- example(datepicker-overview) -->

### Current state
Currently the datepicker is in the beginning stages and supports basic date selection functionality.
There are many more features that will be added in future iterations, including:
 * Support for datetimes (e.g. May 2, 2017 at 12:30pm) and month + year only (e.g. May 2017)
 * Support for selecting and displaying date ranges
 * Support for custom time zones
 * Infinite scrolling through calendar months
 * Built in support for [Moment.js](https://momentjs.com/) dates

### Connecting a datepicker to an input
A datepicker is composed of a text input and a calendar pop-up, connected via the `mdDatepicker`
property on the text input.

```html
<input [mdDatepicker]="myDatepicker">
<md-datepicker #myDatepicker></md-datepicker>
```

An optional datepicker toggle button is available. A toggle can be added to the example above:

```html
<input [mdDatepicker]="myDatepicker">
<md-datepicker-toggle [for]="myDatepicker"></md-datepicker-toggle>
<md-datepicker #myDatepicker></md-datepicker>
```

This works exactly the same with an input that is part of an `<md-form-field>` and the toggle
can easily be used as a prefix or suffix on the material input:

```html
<md-form-field>
  <input mdInput [mdDatepicker]="myDatepicker">
  <md-datepicker-toggle mdSuffix [for]="myDatepicker"></md-datepicker-toggle>
  <md-datepicker #myDatepicker></md-datepicker>
</md-form-field>
```

### Setting the calendar starting view
By default the calendar will open in month view, this can be changed by setting the `startView`
property of `md-datepicker` to `"year"`. In year view the user will see all months of the year and
then proceed to month view after choosing a month.

The month or year that the calendar opens to is determined by first checking if any date is
currently selected, if so it will open to the month or year containing that date. Otherwise it will
open to the month or year containing today's date. This behavior can be overridden by using the
`startAt` property of `md-datepicker`. In this case the calendar will open to the month or year
containing the `startAt` date.

<!-- example(datepicker-start-view) -->

### Date validation
There are three properties that add date validation to the datepicker input. The first two are the
`min` and `max` properties. In addition to enforcing validation on the input, these properties will
disable all dates on the calendar popup before or after the respective values and prevent the user
from advancing the calendar past the `month` or `year` (depending on current view) containing the
`min` or `max` date.

<!-- example(datepicker-min-max) -->

The second way to add date validation is using the `mdDatepickerFilter` property of the datepicker
input. This property accepts a function of `<D> => boolean` (where `<D>` is the date type used by
the datepicker, see section on
[choosing a date implementation](#choosing-a-date-implementation-and-date-format-settings)).
A result of `true` indicates that the date is valid and a result of `false` indicates that it is
not. Again this will also disable the dates on the calendar that are invalid. However, one important
difference between using `mdDatepickerFilter` vs using `min` or `max` is that filtering out all
dates before or after a certain point, will not prevent the user from advancing the calendar past
that point.

<!-- example(datepicker-filter) -->

In this example the user can back past 2005, but all of the dates before then will be unselectable.
They will not be able to go further back in the calendar than 2000. If they manually type in a date
that is before the min, after the max, or filtered out, the input will have validation errors.

Each validation property has a different error that can be checked:
 * A value that violates the `min` property will have a `mdDatepickerMin` error.
 * A value that violates the `max` property will have a `mdDatepickerMax` error.
 * A value that violates the `mdDatepickerFilter` property will have a `mdDatepickerFilter` error.
 
### Input and change events
The input's native `input` and `change` events will only trigger due to user interaction with the
input element; they will not fire when the user selects a date from the calendar popup. Because of
this limitation, the datepicker input also has support for `dateInput` and `dateChange` events.
These trigger when the user interacts with either the input or the popup.
  
```html
<input [mdDatepicker]="d" (dateInput)="onInput($event)" (dateChange)="onChange($event)">
<md-datepicker #d></md-datepicker>
```

### Touch UI mode
The datepicker normally opens as a popup under the input. However this is not ideal for touch
devices that don't have as much screen real estate and need bigger click targets. For this reason
`md-datepicker` has a `touchUi` property that can be set to `true` in order to enable a more touch
friendly UI where the calendar opens in a large dialog.

<!-- example(datepicker-touch) -->

### Manually opening and closing the calendar
The calendar popup can be programmatically controlled using the `open` and `close` methods on the
`md-datepicker`. It also has an `opened` property that reflects the status of the popup.

<!-- example(datepicker-api) -->

### Internationalization
In order to support internationalization, the datepicker supports customization of the following
three pieces via injection:
 1. The date implementation that the datepicker accepts.
 2. The display and parse formats used by the datepicker.
 3. The message strings used in the datepicker's UI.

#### Setting the locale code
By default, the `MAT_DATE_LOCALE` injection token will use the existing `LOCALE_ID` locale code
from `@angular/core`. If you want to override it, you can provide a new value for the
`MAT_DATE_LOCALE` token:

```ts
@NgModule({
  providers: [
    {provide: MAT_DATE_LOCALE, useValue: 'en-GB'},
  ],
})
export class MyApp {}
```

It's also possible to set the locale at runtime using the `setLocale` method of the `DateAdapter`.

```ts
import { DateAdapter, NativeDateAdapter } from '@angular/material';

@Component({
  selector:    'foo',
  template: ''
})
export class FooComponent {
  constructor(dateAdapter: DateAdapter<NativeDateAdapter>) {
    dateAdapter.setLocale('de-DE');
  }
}
```

#### Choosing a date implementation and date format settings
The datepicker was built to be date implementation agnostic. This means that it can be made to work
with a variety of different date implementations. However it also means that developers need to make
sure to provide the appropriate pieces for the datepicker to work with their chosen implementation.
The easiest way to ensure this is just to import one of the pre-made modules (currently
`MdNativeDateModule` is the only implementation that ships with material, but there are plans to add
a module for Moment.js support):
 * `MdNativeDateModule` - support for native JavaScript Date object

These modules include providers for `DateAdapter` and `MD_DATE_FORMATS`

```ts
@NgModule({
  imports: [MdDatepickerModule, MdNativeDateModule],
})
export class MyApp {}
```

Because `DateAdapter` is a generic class, `MdDatepicker` and `MdDatepickerInput` also need to be
made generic. When working with these classes (for example as a `ViewChild`) you should include the
appropriate generic type that corresponds to the `DateAdapter` implementation you are using. For
example:

```ts
@Component({...})
export class MyComponent {
  @ViewChild(MdDatepicker) datepicker: MdDatepicker<Date>;
}
```

*Please note: `MdNativeDateModule` is based off of the functionality available in JavaScript's
native `Date` object, and is thus not suitable for many locales. One of the biggest shortcomings of
the native `Date` object is the inability to set the parse format. We highly recommend using a
custom `DateAdapter` that works with the formatting/parsing library of your choice.*

#### Customizing the date implementation
The datepicker does all of its interaction with date objects via the `DateAdapter`. Making the
datepicker work with a different date implementation is as easy as extending `DateAdapter`, and
using your subclass as the provider. You will also want to make sure that the `MD_DATE_FORMATS`
provided in your app are formats that can be understood by your date implementation.

```ts
@NgModule({
  imports: [MdDatepickerModule],
  providers: [
    {provide: DateAdapter, useClass: MyDateAdapter},
    {provide: MD_DATE_FORMATS, useValue: MY_DATE_FORMATS},
  ],
})
export class MyApp {}
```

#### Customizing the parse and display formats
The `MD_DATE_FORMATS` object is just a collection of formats that the datepicker uses when parsing
and displaying dates. These formats are passed through to the `DateAdapter` so you will want to make
sure that the format objects you're using are compatible with the `DateAdapter` used in your app.
This example shows how to use the native `Date` implementation from material, but with custom
formats.

```ts
@NgModule({
  imports: [MdDatepickerModule],
  providers: [
    {provide: DateAdapter, useClass: NativeDateAdapter},
    {provide: MD_DATE_FORMATS, useValue: MY_NATIVE_DATE_FORMATS},
  ],
})
export class MyApp {}
```

#### Localizing labels and messages
The various text strings used by the datepicker are provided through `MdDatepickerIntl`.
Localization of these messages can be done by providing a subclass with translated values in your
application root module.

```ts
@NgModule({
  imports: [MdDatepickerModule, MdNativeDateModule],
  providers: [
    {provide: MdDatepickerIntl, useClass: MyIntl},
  ],
})
export class MyApp {}
```
### Accessibility
The `MdDatepickerInput` directive adds `aria-haspopup` attribute to the native input element, and it
triggers a calendar dialog with `role="dialog"`.

`MdDatepickerIntl` includes strings that are used for `aria-label`s. The datepicker input
should have a placeholder or be given a meaningful label via `aria-label`, `aria-labelledby` or
`MdDatepickerIntl`.

#### Keyboard shortcuts
The keyboard shortcuts to handle datepicker are:

| Shortcut             | Action                              |
|----------------------|-------------------------------------|
| `ALT` + `DOWN_ARROW` | Open the calendar pop-up            |
| `ESCAPE`             | Close the calendar pop-up           |


In month view:

| Shortcut             | Action                              |
|----------------------|-------------------------------------|
| `LEFT_ARROW`         | Go to previous day                  |
| `RIGHT_ARROW`        | Go to next day                      |
| `UP_ARROW`           | Go to same day in the previous week |
| `DOWN_ARROW`         | Go to same day in the next week     |
| `HOME`               | Go to the first day of the month    |
| `END`                | Go to the last day of the month     |
| `PAGE_UP`            | Go to previous month                |
| `ALT` + `PAGE_UP`    | Go to previous year                 |
| `PAGE_DOWN`          | Go to next month                    |
| `ALT` + `PAGE_DOWN`  | Go to next year                     |
| `ENTER`              | Select current date                 |


In year view:

| Shortcut             | Action                              |
|----------------------|-------------------------------------|
| `LEFT_ARROW`         | Go to previous month                |
| `RIGHT_ARROW`        | Go to next month                    |
| `UP_ARROW`           | Go to previous 6 months             |
| `DOWN_ARROW`         | Go to next 6 months                 |
| `HOME`               | Go to the first month of the year   |
| `END`                | Go to the last month of the year    |
| `PAGE_UP`            | Go to previous year                 |
| `ALT` + `PAGE_UP`    | Go to previous 10 years             |
| `PAGE_DOWN`          | Go to next year                     |
| `ALT` + `PAGE_DOWN`  | Go to next 10 years                 |
| `ENTER`              | Select current month                |
