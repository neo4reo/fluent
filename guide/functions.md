Functions in FTL
================

Functions provide additional functionality available to the localizers. They
can be either used to format data according to the current language's rules or
can provide additional data that the localizer may use (like, the platform, or
time of the day) to fine tune the translation.

FTL implementations should ship with a number of built-in functions that can be
used from within localization messages.

The list of available functions is extensible and environments may want to add
additional functions designated to aid localizers writing translations targeted
for a given environment.

Using Functions
---------------

FTL Functions can only be called inside of placeables. Use them to return
a value to be interpolated in the message or as selectors in select
expressions.

Example:

    today-is = Today is { DATETIME($date) }

Function parameters
-------------------

Functions may (but don't have to) accept positional and keyword arguments. Some
keyword parameters are only available to developers and cannot be used inside
of FTL translations. Instead they can be set on external arguments passed into
translations by the developer.

See the reference below for more information about the arguments accepted for
each built-in function.

Built-in Functions
------------------

Built-in functions are very generic and should be applicable to any translation
environment.

### `NUMBER`

Formats a number to a string in a given locale.

Example:

    dpi-ratio = Your DPI ratio is { NUMBER($ratio, minimumFractionDigits: 2) }

Parameters:

    currencyDisplay
    useGrouping
    minimumIntegerDigits
    minimumFractionDigits
    maximumFractionDigits
    minimumSignificantDigits
    maximumSignificantDigits

Developer parameters:

    style
    currency

See the
[Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat)
for the description of the parameters.

### `DATETIME`

Formats a date to a string in a given locale.

Example:

    today-is = Today is { DATETIME($date, month: "long", year: "numeric", day: "numeric") }

Parameters:

    hour12
    weekday
    era
    year
    month
    day
    hour
    minute
    second
    timeZoneName

Developer parameters:

    timeZone

See the [Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat) for the description of the parameters.

Implicit use
------------

In order to simplify most common scenarios, FTL will run some default functions
while resolving placeables.

### `NUMBER`

If the variable passed from the developer is a number and is used in
a placeable, FTL will implicitly call a NUMBER function on it.

Example:

    emails = Number of unread emails { $unreadEmails }

    emails2 = Number of unread emails { NUMBER($undeadEmails) }

Numbers used as selectors in select expressions will match the number exactly
of they will match the current language's [CLDR plural
category](http://www.unicode.org/cldr/charts/30/supplemental/language_plural_rules.html)
for the number.

The following examples are equivalent and will both work. The second example
may be used to pass additional formatting options to the `NUMBER` formatter for
the purpose of choosing the correct plural category:

    liked-count = { $num ->
      [0]     No likes yet.
      [one]   One person liked your message
     *[other] { $num } people liked your message
    }

    liked-count2 = { NUMBER($num) ->
      [0]     No likes yet.
      [one]   One person liked your message
     *[other] { $num } people liked your message
    }

### `DATETIME`  

If the variable passed from the developer is a date and is used in a placeable,
FTL will implicitly call a DATE function on it.

Example:

    log-time = Entry time: { $date }

    log-time2 = Entry time: { DATETIME($date) }

Partial arguments
-----------------

In most cases users will not have to call out Function explicitly, thanks to
the implicit calls.

The cases where implicit doesn't work will often come when the Function has to
be called with additional parameters, but even then, majority of scenarios will
require the parameters to be set by the developer and only in rare cases
localizer will have to touch them.

Developers can provide the variable already wrapped in Function as an argument.

Example:

    main.js:

    let date = new Date();
    let s = ctx.format('key1', {
      day: Intl.MessageDateTimeArgument(date, {
        weekday: 'long'
      })
    })

    main.ftl:

    key1 = Today is { $day }

If the localizer decide that they have to modify the parameters, for example
because the string doesn't fit in the UI, they can pass the variable to the
same Function and overload parameters. Example:

    main.ftl:

    key1 = Today is { DATETIME($day, weekday: "short") }