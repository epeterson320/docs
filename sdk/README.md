# API SDK

This is a small project workspace to get type-safety between the browser
and API. Under the hood, it leverages OpenAPI, but you should never have
to write YAML yourself.

## Usage

To call the API from another project in this monorepo, add this library
to the project:

```
npm install @local/sdk -w web # or other project
```

Then import its services and call the functions to get data

```ts
import { StatusService } from '@local/sdk';

const apiStatus = await StatusService.healthControllerGetHealth();
```

## How it works

This project has the sibling project in `api` installed as a dev
dependency. At build-time, it imports the API server, inspects its
controllers and DTOs, and generates an OpenAPI schema in
**dist/openapi.yaml**. Then it uses
https://www.npmjs.com/package/openapi-typescript-codegen to generate a
Typescript client from the schema. Then it compiles the typescript. It
generates definition files alongside it, so type intellisense in the
IDE should still work, and it generates sourcemaps too, so debugging
in the browser and VSCode should also continue to work.
