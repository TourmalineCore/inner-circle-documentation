# Add calendar DDR

## Context

We faced the problem of choosing a calendar while designing the interface for taking a book for a specific period of time. How will it work? How will it look like? In order not to invent a new calendar, we decided to use an existing graphical component library.

We currently use the React Datepicker library, but it has a limited set of static calendar components, which doesn't meet our UI design needs.

## Decision

1. PRIMEREACT

It is a popular open source library for creating user interfaces in React applications from PrimeTek.

- MIT License to use the component library.

- Extensive documentation. The documentation for Prime React is really detailed and covers a wide range of use cases, making development easier.

- Regular library updates.

- Technical support for users and an extensive community.

**Disadvantages**

- Paid file with components in Figma. This makes the UI/UX designer's job more difficult because as you will have to create the calendar yourself for UI design.

- PRIMEREACT has one static calendar component that marks only one date, not a range of dates.

2. Material UI

It is an open source React component library that implements Google's Material Design.

**Advantages**

- MIT License to use the component library.

- Extensive documentation.

- Regular library updates.

- Technical support for users and an extensive community.

- Free UI kit in Figma. This will help the designer quickly add the component into the current UI design and style it as Inner Circle.

- A large number of static calendars with different date range selections. For example, the calendar can be a controlled calendar. If the user changes one of the two dates, the range will be changed rather than reset.


**Disadvantages**

- At the moment, we haven't found any Material UI flaws.


## Consequences

After weighing all the advantages and disadvantages, we came to the conclusion that the Material UI component library is more suitable for us than PRIMEREACT. We were attracted by the variability of the date range component for our UX requirements and the ability to quicken UI development, since all components are open source in Figma.