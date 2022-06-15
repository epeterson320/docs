> This is a modified version of the engineering documentation I wrote for an application where I was the lead developer.

# Our Monorepo

This is the monorepo that holds the code for the API server and
frontend.

Projects are managed using
[npm workspaces](https://docs.npmjs.com/cli/v8/using-npm/workspaces).
These are:

- [**api**](./api/README.md) — NestJS API
- [**sdk**](./sdk/README.md) — Typescript API client
- [**web**](./web/README.md) — ReactJS UI

Each project has its own README for more docs and guides.

## Getting started

1. Install Node.js 16+ and npm 8+.
1. Install docker.
1. Install dependencies by running `npm install` from the repository
   root directory.
1. Initialize the local dev stack by running `npm run system:init` in
   the repository root directory.
1. Go through the steps in [Running the dev server](#running-the-dev-server) to make sure the system runs locally.
1. (Optional) Install VSCode and recommended extensions for this project (the list should be visible from VSCode's extensions pane).
1. (Optional) If you're using VSCode, go through the steps in [Full-stack debugging](#full-stack-debugging) to make sure you can use breakpoints correctly.

You can also install VS Code and recommended extensions to get autoformatting, test results, and linting feedback in your editor.

## Running the dev server

Run the stack locally via `npm system:start`. This starts:

- A PostgreSQL database, via docker compose, at localhost:5432
- The NestJS app server on http://localhost:3000
- The ReactJS frontend on http://localhost:3001
- A PostgreSQL admin UI (also via docker compose) on http://localhost:3002. Use database `app` and password `localdevpass` to access it.

To stop the local system, press Ctrl+C. The local database container will keep its data. To drop and re-create the database schema, and re-seed data, run `npm run system:init`.

## Other npm tasks

- Build all projects: `npm run build --workspaces`
- Run all tests: `npm test --workspaces --if-present`

There are more npm tasks, but they vary by workspace. Run `npm run --workspaces` to see them all. **sdk**, for instance, has no `test` task because the code is all generated. **api** has a `start:prod` task while **web** doesn't because it's a static site.

## Full-stack debugging

To use breakpoints across both `api` and `web`, run the system locally via `npm run system:start`, then run the _Attach to system:start_ debug task from VSCode.

To make sure it works on your machine, follow these steps:

1. Start the system locally via `npm run system:start`.
1. Set a breakpoint in **web/src/healthCheck/StatusPage.tsx** right after the line including `useStatus()`.
1. Set a breakpoint in **api/src/health/health.controller.ts** on the line calling `this.healthService.getDBTime()`.
1. Open the debug pane in VSCode and run the debug task titled _Attach to system:start_.
1. Wait for a new Chrome window to open, and navigate to http://localhost:3001/status.
1. Verify that the UI shows the loading spinner, VSCode opens the **StatusPage.tsx** (frontend) file, and indicates that it is stopped on the breakpoint.
1. Verify that the _VARIABLES_ section in VSCode's debug pane shows an `isLoading: true` variable in scope.
1. Click the continue button (or press F5) in the debug controls to continue executing the program. Due to the way react-query loads data, you should hit the frontend breakpoint again.
1. Continue clicking the continue button a few more times until VSCode stops at your breakpoint in **health.controller.ts** (backend).
1. Click continue again and verify that VSCode stops in the frontend file again.
1. Continue until the UI shows the loaded status page.

## Debugging tests

To debug tests, open VSCode, set a breakpoint in test or production code, then navigate to the test file and click the "Debug" text above the test definition. This should run the test in question and stop at any breakpoints hit during testing.

## Deployment

### From Git (recommended)

Every PR against `main` will deploy to an ad-hoc URL which will be added as a comment on the pull request. This is a preview environment to test changes, but it will still connect to the staging database.

Every commit to `main` will trigger a deploy to stage. This will happen automatically after you merge your approved pull requests.

When we get a production environment, we plan to create a `production` branch from which to automatically deploy to prod. In that case, production deployments will happen via a PR from `main` or a stage/release branch to `production`. At that time we may also add more control around when staging deployments happen too.

### Manually (as needed)

This should only be done if the git workflow isn't possible for some reason.

To deploy manually:

- [Install the gcloud CLI](https://cloud.google.com/sdk/docs/install).
- Initialize and authenticate `gcloud`.
- Make sure you have the _App Engine Deployer_ role or higher on the _ourapp-prod_ GCP project.
- Create a file named **deploy-env.yaml** in the repository root with the following contents:
  ```
  env_variables:
    MIKRO_ORM_PASSWORD: staging_db_password
  ```
  (Replace "staging_db_password" with the actual staging DB password)
- Run `npm run build --workspaces` from the repository root.
- To deploy to an ad-hoc environment, run `gcloud app deploy --version=adhoc-myenv --no-promote`, replacing `adhoc-myenv` with a name of your choice. This will deploy to a custom URL **but** it will still connect to the staging database.
- To deploy to staging, run `gcloud app deploy`.
