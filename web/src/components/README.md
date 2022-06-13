# UI Components

This directory is not, nor should it ever become a UI library!

React components can be divided into two categories: _**presentational**_
and _**connected**_.

[MUI](https://mui.com) should be flexible enough to handle 99% of the
_presentational_ component usages we need. If you need a nice text
field, card, or multi-select, check their docs first.

_Connected_ components need to be aware of application logic and data,
and are not reusable across the entire project, so they should be
co-located next to the features they are concerned about, not placed in
a general **components/** directory.

Components may live in this directory if:

- They're wrappers for third-party libraries that add our theme
- They don't have equivalents in MUI or elsewhere on npm
- They have JSDocs so good that other developers will use them without having to read the code.

### What about theme and branding?

> Surely we'll eventually need to build our own UI library to express
> our product's brand, right?

No, React components are much more than JSX+CSS. A UI library
also needs to handle:

- Accessibility
- Grid/layout utilities for developers
- Popover positioning (this one is super crazy)
- Mobile breakpoints
- Implementing an alternative to the basic HTML `<select>` because it
  can't do half the stuff users expect from a select.
- Anticipating developer use cases to design a flexible props API, and
  incorporating developer feedback as use cases are discovered, while
  maintaining backward compatibility and/or communicating API
  deprecations (this is an ongoing cost)
- Documenting components, props, and layout utilities, both with English
  descriptions and images (this is an ongoing cost)

Developers shouldn't need to re-solve all those engineering problems
just so they can express their product's brand. MUI has anticipated
this, which is why they have a very flexible
[design system](https://mui.com/material-ui/customization/theming/)
which lets developers customize everything in their components. So if
you need to start adding or modifying components, look at their docs
first.
