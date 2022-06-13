# Creating new HTTP Endpoints

If you need to create a number of related HTTP endpoints, first create a new controller in a **\*.controller.ts** file so the endpoints can be grouped together easily. If you just need to create one endpoint, see if there's an existing controller where it can be added reasonably, or create a controller for it. If you need to create a set of endpoints for a related resource, you may try running `npx nest generate resource my-feature-name` from inside the **api/** directory.

To create a controller and/or add route handlers:

1. Create the controller and methods via the docs at [Controllers | NestJS](https://docs.nestjs.com/controllers).
2. Write tests for the controller's endpoints via the docs at [Testing | NestJS](https://docs.nestjs.com/fundamentals/testing).
3. Document the endpoint(s) via the OpenAPI annotations described in [Operations – OpenAPI | NestJS](https://docs.nestjs.com/openapi/operations).
   - Add an `@ApiOperation()` annotation with a summary and description.
   - Add one or more [`@ApiReponse()`](https://docs.nestjs.com/openapi/operations#responses) annotations (or related shorthand decorators, per the docs) with descriptions.
4. Update the OpenAPI schema by running `npm run build -w api -w sdk` from the repo root.
5. Lint the OpenAPI schema by running `npm run lint -w sdk` from the repo root.
   - Fix any errors by modifying or adding more to the controllers' annotations.

> **Q:** OpenAPI seems like a lot of work! Why do we have to use it do document the API?
>
> **A:** tl;dr High-impact communication.
>
> OpenAPI makes communication possible not just between developers but also between their tools. When developing a new feature or endpoint, the API dev has to tell the web (or mobile app) dev, "Here's the endpoint path, method, request body, response body, the fields of those payloads, their types, and what each one means." _Then_ they have to tell the web dev every time those API endpoints are updated.
>
> With OpenAPI, the schema can be used to generate client SDKs, so your request/response body fields and types are automatically enforced in the client apps. All the frontend dev has to do is update to the new version of the generated SDK. Even the comment blocks _you write_ for the endpoints, payloads, and fields will end up in the frontend developers' intellisense. You're still communicating, but you're doing the communication when _you_ code to every dev consuming your API when _they_ code. Neither of you have to stop coding to have a meeting to explain and learn the API.
