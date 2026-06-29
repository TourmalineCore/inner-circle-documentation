# 007: Choice of Design System for Inner Circle Redesign

## Status
Proposed (2026-06-29)

## Context
We need to redesign Inner Circle to unify the user experience across its multiple services. 

Currently, the codebase consists of several independent services built with:
- React + Vite
- SCSS for styling
- custom component with inconsistent design patterns

**Key requirements:**
- unified visual language across all services
- high accessibility standards
- centralized distribution of design system
- minimal disruption to existing codebase
- support for existing Sass infrastructure
- ability to gradually migrate services

## Decision
Create a custom UI kit with unified visual language using [React Aria](https://react-aria.adobe.com/) unstyled components distributed via npm package.

### Rationale
1. **Freedom from Tailwind constraint**: React Aria Components are completely unstyled, allowing us to retain and extend our existing Sass infrastructure without conflicts. *shadcn/ui* would force a migration to Tailwind CSS, creating significant overhead during development to make it co-exist with our current architecture.

2. **Top-class accessibility**: React Aria components are built by Adobe with superior accessibility across all platforms, including mobile screen readers. 

3. **Distribution via npm**: npm package approach gives us predictable versioning. Unlike Module Federation, npm packages avoid runtime version conflicts (singleton challenges) and are simpler to maintain.

4. React Aria ecosystem is **actively maintained** by Adobe, so it is a more reliable option.

5. React Aria documentation contains **examples of styles**, which we can copy to spend less time on styling from scatch. 

## Consequences
**Positive:**
- consistent design language across all services
- improved accessibility out of the box
- gradual component-by-component migration
- full control over styling without specific CSS framework (Tailwind)
- predictable npm-based distribution with clear versioning

**Negative:**
- significant effort required to style all components from scratch
- need to build a styling layer (tokens, variables, mixins) on top of React Aria
- longer time spent on development compared to using a pre-styled solution

## Alternatives
### 1. Headless design systems (TanStack)
TanStack provides headless logic for tables, forms, routing, and data management but does not provide UI components. 
TanStack complements rather than replaces a UI library. It should be considered as a separate decision for data-heavy features, not as the primary design system.

#### Advantages
- powerful data management
- excellent performance 

#### Disadvantages
- not a UI component library, only provides logic for specific use cases
- no styling primitives, all UI must be built from scratch
- accessibility support is inconsistent across libraries (Router has none, Table is good, Select is experimental)
- doesn't solve the core UI unification problem

### 2. Styled components (shadcn/ui)
shadcn/ui is a popular copy-paste component library built on Radix UI/Base UI with Tailwind CSS.
Excellent for new projects without existing styles, but creates significant friction in a React + Sass environment like ours.

#### Advantages
- ready-to-use styled components
- minimal bundle size - only components you copy into your project
- full code ownership - you can modify any component's logic

#### Disadvantages
- Tailwind conflicts with existing Sass infrastructure
- manual updates required, each component upgrade requires manual diff and merge
- risk of losing customizations when overwriting components

### 3. Ready Design Systems (MUI)
Fully-featured UI libraries with complete components and styling.
Suitable if speed is the only priority, but too opinionated and rigid from the long-term perspective.

#### Advantages
- thousands of ready components
- comprehensive documentation and large communities
- theming systems for customization
- great accessibility support

#### Disadvantages
- heavy bundle size
- difficult to replace later
- limited customization, styling often requires fighting the framework

## Styled vs Unstyled Component Libraries

| Criterion | Unstyled Components: React Aria by Adobe | Styled Starters: shadcn/ui based on Base UI from Material UI creators |
| :--- | :--- | :--- |
| **License** | Apache | MIT |
| **Installation** | npm package | Copy-paste component code into your project |
| **Development Speed** | Lower at start. Requires significant time to style each component (though style examples are available in docs). | Higher at start. Styles are ready, but no vanilla CSS/SCSS support – only Tailwind. |
| **Flexibility & Customisation** | Very high. Can write styles completely from scratch. | Very high. All code lives in your project, you can change anything. |
| **Updates** | Easy – via npm version update. | Complex – updates must be merged manually, conflicts possible. |
| **Dependencies** | Fewer – only basic primitives. | More – installs all dependencies for each component (Radix, Tailwind, utilities). |
| **Developer Experience** | Better – can reuse existing classes and write styles in SCSS files, easier to tweak. | Higher entry curve – new tool (Tailwind), no SCSS support. More setup at start to support both new and legacy components. |
| **a11y** | Top‑tier accessibility. Designed with accessibility as a primary priority. Wins on mobile a11y. | Excellent accessibility. |
| **Figma** | None | Basic black‑and‑white styles available, but limited number of components. |
| **Community Support** | More activity from developers and the community. | Low activity on GitHub, many open issues and unmerged PRs. |
| **Distribution** | npm package | npm package, Module Federation |


## Distributing a Design System

<table>
  <thead>
    <tr>
      <th>Strategy</th>
      <th>Advantages</th>
      <th>Disadvantages</th>
      <th>Comments</th>
      <th>Suitability for Design System</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td valign="top"><strong>npm package</strong></td>
      <td valign="top">
        Predictability and control.<br>
        No runtime dependency: all components are bundled into each service at build time.<br>
        No network delays or risk of remote module unavailability.
      </td>
      <td valign="top">
        To update the design system in one service, you need to update the package version.<br>
        Code duplication: if the design system is large and there are many services, its code will be duplicated in each service's bundle, increasing overall size.
      </td>
      <td valign="top">
        Good starting point. Suitable for basic settings like colors, fonts, tokens, spacing, and reusable components.
      </td>
      <td valign="top">
        ✅ shadcn/ui<br>
        ✅ react-aria
      </td>
    </tr>
    <tr>
      <td valign="top"><strong>Module Federation</strong></td>
      <td valign="top">
        Instant updates. You can update a component in the design system, deploy only it, and all consuming services will instantly receive the update.
      </td>
      <td valign="top">
        Risks related to singleton versioning: when MF tries to resolve dependency version conflicts at runtime, the application may work incorrectly or even crash.
      </td>
      <td valign="top">
        Risks related to singleton versioning: when MF tries to resolve dependency version conflicts at runtime, the application may work incorrectly or even crash.
      </td>
      <td valign="top">
        ✅ shadcn/ui: public examples and libraries (<a href="https://www.npmjs.com/package/@tumaet/prompt-ui-components">prompt-ui-components</a>, <a href="https://github.com/stefan-moraru/micropods">micropods</a>)<br>
        Since shadcn/ui is your own code after copying, there are no issues with dependency resolution on the consumer side.<br>
        ❓ react-aria: bugs with react-aria dependencies (e.g., clsx) can cause runtime errors in the MF environment.
      </td>
    </tr>
  </tbody>
</table>