# 001 Choose Time tracker

## Status
Accepted

## Context
We need to find a ready-made calendar with timeline solution that will allow us to implement our own time tracker faster.

## Decision
It was decided to use the [react-big-calendar](https://github.com/jquense/react-big-calendar) package.
This package will allow us to realize most of our needs much faster.

Some Advantages of this package:
1. MIT license.
2. There is a functionality for switching between day and week.
3. Support for working with time zones.
4. Flexible customization of components.
5. There is a function for applying cards to an event.

## Consequences
This solution will allow us to implement the UI part of the time tracker much faster than if we wrote everything from scratch.

## Other solutions considered
[react-calendar-timeline](https://www.npmjs.com/package/react-calendar-timeline) - The package also tracks time, but not in the format we need.

[fullCalendar](https://fullcalendar.io/pricing) - The functionality we need is paid.