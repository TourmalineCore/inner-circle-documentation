# Add calendar DDR

## Context

We faced the problem of choosing a calendar while designing the interface for taking a book for a specific selection period. How will it work? How will it look like? In order not to invent a new calendar, we decided to use an existing graphical component library.

## Decision

1. PRIMEREACT

It is a popular open source library for creating user interfaces in React applications from PrimeTek. This library is attractive because it has different types of calendar: pop-up with a field to enter the date, static, with multiple month selection, with date range selection and touch interface.

**Advantages**

- MIT License to use the component library.

- Extensive documentation. The documentation for Prime React is really detailed and covers a wide range of use cases, making development easier.

- Regular library updates.

- Technical support for users and an extensive community.

**Disadvantages**

- Paid file with components in Figma. This makes the UI/UX designer's job a bit more difficult because as you will have to create the calendar yourself for UI design.

2. Material UI

It is an open source React component library that implements Google's Material Design. Material UI attracts attention with a large number of components for choosing date and time. We were most interested in the following options of a single component: with an input field and a pop-up calendar for several months to select a date range, mobile variant, adaptive, static, with shortcuts for quick range selection.  

**Advantages**

- MIT License to use the component library.

- Extensive documentation.

- Regular library updates.

- Technical support for users and an extensive community. 

- Free UI kit in Figma. This will help the designer quickly add the component into the current UI design and style it as Inner Circle.


**Disadvantages**

- At the moment, we haven't found any Material UI flaws.


## Consequences

After weighing all the advantages and disadvantages, we came to the conclusion that the Material UI component library is more suitable for us than PRIMEREACT. We were attracted by the variability of the date range component for our UX requirements and the ability to quickly UI development, since all components are open source in Figma.