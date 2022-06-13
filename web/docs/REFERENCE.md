# Reference

Links to documentation for the most important parts of the app.

## Major Libraries (External)

- **React** renders the UI.
  - [**Tutorial**](https://reactjs.org/tutorial/tutorial.html) (if you don't know anything about React)
  - [**Documentation**](https://reactjs.org/docs)
  - In particular, make sure you're familiar with [**hooks**](https://reactjs.org/docs/hooks-intro.html) for managing state and effects and [**`props.children`**](https://reactjs.org/docs/jsx-in-depth.html#children-in-jsx) for decoupling container UIs and their content.
- [**Create React App**](https://create-react-app.dev/docs/getting-started) is our build toolchain. This has docs on building, testing, deployment, dev server, environment configuration, and a lot of good guides on typical web application concerns.
- [**Lodash**](https://lodash.com/) has utility functions. Look at [`keyBy()`](https://lodash.com/docs/#keyBy) and [`compact()`](https://lodash.com/docs/4.17.15#compact) for an example of the kind of stuff it does.
- [**Luxon**](https://moment.github.io/luxon) provides the immutable `DateTime` object (and functions) which doesn't have the problems that Javascript's [`Date`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) has, namely, it works in more than just the system time zone.
- [**MUI**](https://mui.com/getting-started/usage/) is our UI component library.
- [**React-map-gl**](https://visgl.github.io/react-map-gl/) renders the maps.
- [**Deck.gl**](https://deck.gl/) handles geospatial visualizations and layers on top of the maps.
- [**Mantine Hooks**](https://mantine.dev/hooks/getting-started/) has useful React hooks for common use cases. Check here before you start writing your own reusable hooks. Some of theirs include `useDebouncedValue()`, `useClickOutside()`, `useClipboard()`, and `useLocalStorageValue()`.
- [**React Query**](https://react-query.tanstack.com/) takes care of data fetching, caching, and refetching when stale.
- [**React Hook Form**](https://react-hook-form.com/) takes care of form logic and validation.
- [**React Router**](https://v5.reactrouter.com/web/guides/quick-start) handles SPA routing.

## Major Modules (Internal)

Major modules intended for reuse in this project are below. Click the links to see their definition and JSDocs.

- Page layout components
  - [**`<AppBarPage>`**](../src/layouts/AppBarPage.tsx)
  - [**`<SmallCenteredPage>`**](../src/layouts/SmallCenteredPage.tsx)
- Reusable pages
  - [**`<LoadingPage>`**](../src/commonPages/LoadingPage.tsx)
  - [**`<FullScreenErrorPage>`**](../src/commonPages/FullScreenErrorPage.tsx)
  - [**`<NotFoundPage>`**](../src/commonPages/NotFoundPage.tsx)
- Authentication & authorization
  - [**`useUser()`**](../src/auth/useUser.ts)
  - **`<Authorize>`** (Not implemented)
  - **`<InternalOnly>`** (Not implemented)
- [**`logger.ts`**](../src/logger.ts)
- [**`api/hooks.ts`**](../src/api/hooks.ts)
- Add your own to this list! If you intend for your code to be reused by other developers, make sure it has a _really good description_ in a JSDoc block (starts with `/**`) so VSCode intellisense will let the rest of the team read the docs as they use it. Ideally give a brief description of the function parameters or React props, which should be kept as minimal as possible. Don't worry about describing the types in JSDoc, but make sure the Typescript type signature is bulletproof.
