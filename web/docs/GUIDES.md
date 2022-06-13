# Guides

Task-oriented how-to guides

## Overview

Because the frontend is built and tested with [Create React App](https://create-react-app.dev/docs/getting-started), it comes with strong but well-tested and widely-used defaults. If you want to do typical stuff like [run tests](https://create-react-app.dev/docs/running-tests), [configure build variables](https://create-react-app.dev/docs/adding-custom-environment-variables), set breakpoints and [debug](https://create-react-app.dev/docs/setting-up-your-editor#debugging-in-the-editor), dynamically import code at runtime, or use CSS modules, etc., their documentation is usually the best place to go first.

## Developing locally

Run `npm start` to start a dev server and open a browser to `localhost:3000` (this is [configurable](https://create-react-app.dev/docs/advanced-configuration) if you prefer). The dev server will include hot-reloading, so you should see UI changes as you save your code. To stop the dev server, press `Ctrl+C`.

The frontend makes API requests relative to its own origin. This keeps the frontend codebase minimally aware of backend infrastructure, and avoids the work of enabling CORS securely. When running locally, the local dev server will proxy API requests from `localhost:3000` to our dev API at `dev-api.ourcompany.com`. To change this, in case you're running the backend locally too, create a file named **.env.local** in the project root and set the `API_ORIGIN` variable.

## Adding a page

1. Create a **.tsx** component file with "Page" as the name suffix in a directory for the page appropriate to the feature the page is about. For example, **src/users/ListUsersPage.tsx** or **src/organizations/OrgContactInfoPage.tsx**. DON'T create a **src/pages/** directory as this starts a pattern of decoupling page component locations from their children, resulting in a lot of cross-directory imports.
2. Pick a layout for the page. Options include:
   - [**`<AppBarPage>`**](../src/layouts/AppBarPage/index.tsx)
   - [**`<SmallCenteredPage>`**](../src/layouts/SmallCenteredPage/index.tsx)
3. Implement the page component by rendering the page content inside its layout. For example:

   **src/users/ListUsersPage.tsx**

   ```tsx
   export default function ListUsersPage() {
     return (
       <AppBarPage>
         <List>
           <ListItem>Alice</ListItem>
           <ListItem>Bob</ListItem>
         </List>
       </AppBarPage>
     );
   }
   ```

4. Add a route to **<AppRoutes />** with the appropriate authorization definition. Use `React.lazy()` to import the page only when the route matches.

   **src/app/AppRoutes.tsx**

   ```tsx
   const ListUsersPage = lazy(() => import('../users/ListUsersPage'));
   // ...other pages declared
   export default function AppRouteSwitch() {
     return (
       <Routes>
         {/* other routes above */}
         {/* add this */}
         <Route path="users" element={<ListUsersPage />} />
         {/* or add this */}
         <AuthorizedRoute
           path="users"
           permission={Permissions.LIST_USERS}
           element={<ListUsersPage />}
         />
         {/* other routes below */}
       </Routes>
     );
   }
   ```

## Calling the API

1. Add a custom hook to **src/api/hooks.ts**, following existing patterns.
2. Use the hook in components that need the returned data or call the API's mutations.

The resulting component will look something like this:

**ExampleUsersListPage.tsx** (Simplified for brevity)

```tsx
import { useUsersQuery } from 'api/hooks';
// other imports omitted

export default function UsersListPage() {
  const { isLoading, error, data: users, refetch } = useUsersQuery();

  if (isLoading) {
    return <FullScreenLoadingPage />;
  }

  if (error || !data) {
    return <FullScreenErrorPage error={error} onRetry={refetch} />;
  }

  return (
    <FullScreenPage>
      <Button icon={<RefetchIcon />} onClick={() => refetch()} />
      <UsersList users={users} />
    </FullScreenPage>
  );
}
```

## Deployment

To deploy to stage, open a pull request against the `main` branch and merge it when you're confident it works as expected.

To deploy to production, first create a production environemnt because it doesn't exist. Then deploy there, and tell everyone else how to deploy to prod. Don't forget to update these docs.

## Running tests

`npm test` will launch the tests in watch mode. `CI=true npm test -- --coverage` will run everything and print results.

If you use the VSCode Jest extension, you should see red/green icons next to test code in your editor.

## Build, Release & Config

To use config variables in the code, import config from **src/common/config.ts**.

```ts
import { Build, Release } from './common/config.ts';
```

The `Build` and `Release` variables have different types, and their values can be configured at different times as suggested by "The Twelve-Factor App".

[The Twelve-Factor App — Build, release, run](https://12factor.net/build-release-run)

![Release depends on build and config. Build depends on code.](./release.png)

`npm run build` (takes a while) will create a built, optimized, minified bundle in **build/**. Build-time config variables will be built into the app [(docs)](https://create-react-app.dev/docs/adding-custom-environment-variables).

`npm run release` (optional, works quickly) will modify the _release_ configuration of the app after an optimized bundle has been built, based on environment variables.

This allows us to build _once_, test, and deploy the same build to different environments. This allows us to do stuff in CI like:

```zsh
# build (just once)
npm run build

# release for e2e testing and test
API_ORIGIN=localhost:3001 npm run release -- "e2etest"
./my-e2e-test-tool test

# release for stage and deploy
API_ORIGIN=stage-api.example.com npm run release -- "stage"
./my-deploy-tool deploy "https://stage.example.com"

# release for prod and deploy
API_ORIGIN=api.example.com npm run release -- "prod"
./my-deploy-tool deploy "https://app.example.com"
```

## Big Changes

These are some other tasks that will happen rarely, if ever, and would require a team discussion of pros and cons before doing it, as there are alternatives to the strategies described here. But if this project grows very large and needs big changes, we should be able to make those big changes without a full rewrite.

### Switching dev/build tooling

Create React App is great, but other frameworks exist with better performance and features, such as Webpack (more configurable, plugins), esbuild (100x faster compilation), Next.js (server rendering), Snowpack (faster dev server), Redwood.js (serverless full-stack with "rails-like" big feature set). All except Webpack and Next.js are fairly immature projects, but it may eventually make sense to adopt them. Also, [Deno](https://deno.land/) is gaining popularity as an alternative JavaScript/TypeScript runtime instead of Node.js. Deno is at v1 but its ecosystem is immature.

So it's not unthinkable that we may want to adopt something else in the future. It's hard to know what the specifics of switching will require, but one reason to choose Create React App _today_ is that it supports a conservative feature set and very minimal configuration. Switching from our limited, zero-config tool to a more robust, more configurable tool should feel like running downhill, not crossing a mountain.

### Adding another app

Say we want to deploy a second app, with separate features, and targeting separate users, but re-using code from this repo. npm 7 has support for _workspaces_, which enables multiple npm projects in the same repository, with the ability to import sibling code. So a second app could be supported by adding additional workspaces for app 2 and common components. Project layout would change as follows:

**Before**

```
/
  package.json
  web/
    package.json
    src/
```

**After**

```
/
  package.json
  web-common/ # Common code extracted here
    package.json # Set name field to @local/common or something
    src/
  web/ # Existing stuff stays here
    package.json # Add dependency on @local/common
    src/
  app2/ # New app
    package.json # Declare dependency on @local/common
    src/
```

### Gradually switching component libraries

[MUI](https://mui.com) should have everything we need, but if for some reason we decide it doesn't fit our needs, or if this project is maintained for many years and we decide it's a good time to take advantage of a newer component framework without a rewrite, we may want to switch over gradually.

To do so:

1. Create a **src/common/layouts/MUIPage.tsx** component that renders its children inside a `<ScopedCssBaseline>`.
2. Remove `<CssBaseline>` from the main `<App>` component. Move any global CSS into `<MUIPage>` and scope it to that component using CSS modules or another strategy.
3. Prefix the page layout components in **src/common/layouts** with "MUI", and have them render their contents inside an `<MUIPage>`, e.g. `<FullScreenPage>` becomes `<MUIFullScreenPage>`.
4. Re-create the page layout components in the new framework, and prefix them with the frameworks name, e.g. `<GoodUIKitCardPage>`. Use these for new development.
5. As priority allows, switch legacy pages from MUI layouts to new framework layouts, swapping old components for their newer equivalents.

> Q: Can't we just build an abstraction layer in e.g. **src/compat/mui** that wraps and re-exports the old framework, then swap the implementation for the new one?

That's not incompatable with the approach described above, and may be helpful. But probably, if we're doing something as drastic as switching UI frameworks, it'll be because we _want_ to make big changes, and have likely chosen a new UI framework so different from MUI (if we aren't, why are we undertaking the effort of switching?) that there won't really be a good abstraction layer above it and MUI.
