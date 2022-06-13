# Concepts

Explanations of architecture, patterns, and best practices

## Folder Layout

> tl;dr package by feature, not by layer

Beyond what Create React App scaffolds for [folder layout](https://create-react-app.dev/docs/folder-structure), we add the following:

```
web/
  src/
    api/ # React hook wrappers to the code in api-sdk
    app/ # App-global code
    auth/ # Auth utilities like useUser()
    common-pages/ # Error page, loading page, no-data page
    layouts/ # Page layout components
    config.ts # Project config
    logger.ts # Base logger
```

Everything else in src/ should be packaged by feature or entity, like:

```
web/
  src/
    organizations/
    users/
    regions/
    map-layers/
    simulations/
    uploads/
```

We do NOT package by type or layer, which means DON'T use folder names like these, as they will become disorganized and unintuitively coupled as the codebase grows large.

```
web/
  src/
    actions/
    reducers/
    models/
    types/
    components/ # Exceptions apply, see src/components/README.md
    pages/
    utils/
```

For further reasoning, see the following articles:

- [Screaming Architecture](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html) — Robert C. Martin
- [Package by feature](http://www.javapractices.com/topic/TopicAction.do?Id=205) — Java Practices

## Global `<App />` Component Hierarchy

The app entry point is [**src/index.tsx**](../src/index.tsx), which renders the [**`<App />`**](../src/app/App.tsx) component and global context providers. The app component takes care of global concerns and renders the router. The global component hierarchy is:

```
<StrictMode>                   # Extra quality checks, only in dev
  <BrowserRouter>              # react-router-dom context
    <ReactQueryConfigProvider> # data fetching and caching config
      <AuthProvider>           # auth context
        <AppErrorBoundary>     # graceful crash fallback
          <Suspense>           # graceful loading fallback
            <AppRouteSwitch /> # The page routes themselves
          </Suspense>
        </AppErrorBoundary>
      </AuthProvider>
    </ReactQueryConfigProvider>
  </BrowserRouter>
</StrictMode>
```

**`<AppRouteSwitch>`** renders a react-router-dom [`<Routes>`](https://reactrouter.com/docs/en/v6/api#routes-and-route) that matches routes and loads the matched page on demand using [`React.lazy()`](https://reactjs.org/docs/code-splitting.html#reactlazy).

Pages are implemented by rendering their content inside a reusable page layout component. So if the app route switch contained a `<Route path="/about" element={<AboutPage />} />`, the `<AboutPage>` would in turn render `<PublicBrandingPage><h1>About us!</h1></PublicBrandingPage>`.

## Component Development

> tl;dr It's 2022 and all CSS is a code smell now.

Try to decompose larger UI features into sub-components with clear definitions, and compose them together to implement the main pages and features. Developers have a lot of discretion about how they want to break up and solve a problem.

There are a number of different levels of abstraction you can use to implement a component. Prefer the highest level that makes sense, in this order:

1. Compose other components in this project.
2. Render general-purpose presentational components from MUI (our UI library), e.g. implementing an `<OrgCard>` that renders its content in an MUI `<Card>`.
3. Render a specialized component from a library installed from npm, such as with Mapbox.
4. Implement a presentational component using MUI's [`<Box>`](https://mui.com/components/box/) component and [`sx` prop](https://mui.com/system/basics/#the-sx-prop).
5. Implement a presentational component rendering primitive HTML tags (`<div>` and such), styled with [CSS modules](https://create-react-app.dev/docs/adding-a-css-modules-stylesheet).

See also: [Composition vs Inheritance — React](https://reactjs.org/docs/composition-vs-inheritance.html)

## State Management

> tl;dr Just say no to Redux.

Use your judgment along with the following heuristic to decide where to manage state:

1. Store state in the component itself via `React.useState()` or `React.useReducer()`.
2. If parent or sibling components need access to the state too:

   If it's form data, use [react-hook-form](https://react-hook-form.com/). Otherwise, store state up the component hierarchy by having the child component expose a `myObject` and `onChangeMyObject`.

   See also: [Lifting State Up — React](https://reactjs.org/docs/lifting-state-up.html)

3. If there are more than 4 components in the component sub-tree that contains the state, and it's too much boilerplate to lift state up and pass it down through the components:

   Store the state in the URL, like `/users/4`, and access it with React Router's hooks like [`useMatch()`](https://reactrouter.com/docs/en/v6/api#usematch).

4. If the state is too complex to put in the URL:

   Put it in top page component for the route, and pass it down via props or [context](https://reactjs.org/docs/context.html).

5. If multiple pages need access to the data:

   If it's remote data (or related to it like loading/saving/error status), use [React Query's](https://react-query.tanstack.com/) `useQuery()` and `useMutation()` hooks specifically designed for this.

   If it's authentication data, use use the dedicated Context to manage it.

6. If multiple pages need access to it, and it's not auth state or remote data:

   Use the [`useLocalStorageValue()`](https://mantine.dev/hooks/use-local-storage-value/) hook.

7. If it's too complicated to handle with `useLocalStorageValue()`:

   Install an npm library that handles the service logic and provides React hooks to access it.

8. If no npm library exists because the service logic deals with application-specific conerns:

   Write a module (use `class` or don't) to encapsulate a stateful feature, and provide react bindings to it via `React.createContext()` and custom hooks built upon `useContext()`.

9. If the actions and state for the application-specific logic are complex enough that you want to use the reducer pattern, and you really want to use Redux:

   Install Redux. If state really needs to be persisted across pages AND component lifecycles AND it's not just a copy of (or derived from) remote data AND it's too complicated to persist with [`useLocalStorageValue()`](https://mantine.dev/hooks/use-local-storage-value/), then consider using a state management library like redux-toolkit. But keep in mind that global variables tend to accumulate and should be proactively re-organized as features are added.

   If we rush out a page component with messy state management, its mess will be limited to that page. If we rush out messy state management in Redux, that affects the state and actions available everywhere.

   DO feel free to use the reducer pattern in more limited scopes, via [`useReducer()`](https://reactjs.org/docs/hooks-reference.html#usereducer), as it's a great tool to encapsulate and describe state with more than 5 fields tightly connected to more than 5 types of actions.

## Layout, Margins and Padding

Don't use CSS to set margins or padding to specific pixels or rems. Use the `m` and `p` props of the `sx` prop as described here: https://mui.com/system/spacing/.

Components shouldn't set margins on their top-level children and should be agnostic about where they are laid out by accepting the standard MUI `sx` prop if necessary. That way a parent component can take care of making sure its children are laid out sensibly.

## Linting & Code Style

ESLint is enabled only to detect probable bugs, not to enforce a code style. Prettier is an autoformatter that will reformat the code to a consistent code style. If you use VSCode with the recommended extensions for this project, it will format your code every time you save.

Both ESLint and Prettier are enabled to run as git hooks, so any code style inconsistencies will be automatically fixed when you commit, and you won't be able to commit probable bugs detected by eslint. If you need to disable these commit hooks, use `git commit --no-verify` at your own risk.

If this project ever gets more than 100 lines of CSS, we should consider adding [StyleLint](https://stylelint.io/) to these checks too.

## Authentication and Authorization

To restrict parts of the UI to certain roles, use the `<Authorized roles={someRoles}>` component. It only renders its children if the current user has the specified roles. Otherwise, it renders a fallback UI (by default nothing). For more complicated authorization checks, use the `useUser()` hook to get the current user object and make checks accordingly.

Always remember API authorization is for security, but frontend authorization is for UX only. We don't control users' browsers, and should assume they can reverse-engineer our frontend. So even if they can get the "fire missiles" button to display, clicking it should result in "403 Forbidden".

Users are authenticated by being redirected to **https://login.ourcompany.com** and redirected back to their site, following the standard OAuth 2.0 "Authorization Code with PKCE" grant type. Auth0 takes care of hosting and implementation.

## Naming conventions

### Component naming

Avoid vague words like "View" or "Display" in component names. Literally every React component is viewing or displaying _something_. Instead of `<OrgView>`, which could mean many things, prefer `<OrgSummaryCard>` or `<OrgInfoPage>` as appropriate.

### Prop naming

- If a prop accepts a callback function, prefix it with "on", e.g. `onChangeRegion`, not `setRegion` or `handleRegionChange`.
- Don't overload the meaning of HTML event props. Basically, if you make an `onChange` prop, it should pass an `HTMLEvent` to its callback (because that's what `<input>` and friends' `onChange` callbacks do). If it passes something different, give the prop a more specific name, like `onChangeRegion()`.

### File naming

Name files after their default export, if they have one. Otherwise, use your judgment.

Don't name anything **utils.ts**. Because "utils" can mean just about anything, a file with that name will collect all sorts of unrelated code and get messy. Instead of "utils," use a more specific name based on what functions a file contains, such as **assertions.ts**, **format-fns.ts**, or **timing-fns.ts**.

## Testing

Create React App comes with Jest, so we use that for unit tests. Code coverage thresholds are set at 80%. See the guide for running and debugging tests.

Locate test files next to the prod files they test. Use the `.test.ts` or `.test.tsx` file extension.

The definition of a "Unit test" can be pretty broad, so if you write feature tests for a page, and its components all end up with good coverage, you may choose not to write tests for all its child components.

**Synthetic Tests** are run by Datadog against our prod and stage environments as a sanity check to make sure the app is running and a user can log in.

## Real User Monitoring (RUM)

We can use Datadog's RUM library to collect session events and errors. It's installed in the project but currently disabled for cost reasons. If you want to see flamegraphs of network requests, look up sessions for a given user's email, or see stacktraces of app crashes, they have dashboards for it.
