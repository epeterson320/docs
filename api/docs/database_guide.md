# Working with the Database

## Querying

### Using a repository (recommended)

Generally, when querying the database, follow the MikroORM recipes in
the NestJS documentation, specifically the
[Repositories](https://docs.nestjs.com/recipes/mikroorm#repositories)
section. For most uses, you should be able to leverage NestJS to create
and inject a repository into the service that needs it.

### Query builder or raw SQL (as needed)

If a repository isn't flexible enough, get NestJS to inject an instance
of `EntityManager` into your service. This supports a lot of non-ORM
use cases ([see documentation](https://mikro-orm.io/docs/query-builder)),
but if it still doesn't have what you need, it can run raw SQL, like so:

```ts
import { EntityManager } from '@mikro-orm/postgresql';

@Injectable()
export class MyService {
  constructor(private readonly em: EntityManager) {}

  async someMethod() {
    // Using it
    const [{ col1 }] = await this.em
      .getConnection()
      .execute<{ col1: number }[]>('SELECT 1 as col1;');
  }
}
```

## Migrations

### Add or modify a table

1. Edit or add model(s) defined in the **\*.entity.ts** file you're interested in.
1. If you created a new entity, import it in **src/mikro-orm.config.ts** and reference it in the array.
1. Run `npx mikro-orm migration:create` to create a migration file based on the diff between the current schema (read from **migrations/schema.json**) and the current entities (**\*.entity.ts**).
1. Review the new migration file created in **migrations/** to make sure the SQL is what you want, and adjust it if necessary.
1. Run the following commands to make sure your migration works correctly:
   ```sh
   docker compose up -V
   npx mikro-orm migration:up
   npx mikro-orm seeder:run
   npx mikro-orm migration:down
   npx mikro-orm migration:fresh --seed
   docker compose down -v
   ```
1. Continue working on the rest of your feature.
1. Commit your files and open a PR. When the PR is merged, the migration will be run in stage and then prod.

### Extensions, functions, and other database changes

If you need to change the database in ways other than adding/removing tables and columns, for instance enabling extensions or defining functions or custom types, run `npx mikro-orm migration:create --blank` as above, and then edit the `up()` and optional `down()` method in the generated file in **migrations/**.
