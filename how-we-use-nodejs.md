# How We Use Node.js for Backend Development

We use Node.js for our backend services, using the NestJS framework, TypeScript, TypeORM and PostgreSQL. We deploy the services on Google Cloud Platform using Terraform and Github Actions. The services run as Docker containers (using Google Cloud Run), and we use `docker-compose` for our local development environment.

## NestJS

We use NestJS for all our backend services, which also means that we adhere to the NestJS architecture (Model-View-Controller architecture with dependency injected providers). We adhere to the [SOLID principles](https://en.wikipedia.org/wiki/SOLID) and divide our code into small, easily testable units of code.
We abstract dependencies into services whenever possible.

> Example: A backend that depends on Axios for outbound request would have a service called RequestService that itself calls the methods that Axios might need.

### Serialization

We use [class-transformer](https://github.com/typestack/class-transformer) to serialize data returned to the client. This provide us with a method of sanitizing and transforming objects returned in network requests in a simple and declaritive way. The build-in `ClassSerializerInterceptor` is binded globally in our [template](https://github.com/Kvalifik/template-backend-nestjs) and allows us to annotate entities with decorators.

We reccomend to strip objects for null values and return them as undefined instead. This makes handling of API response in frontend simple and data tranfered to a minimum.

```typescript
@Column({ type: String, nullable: true })
@Transform(({ value }) => value ?? undefined)
middleName: string | null
```

_Note: typeorm is not able to deduct the type automatically so we need to specify it._

We can also exclude certain properties.

```typescript
@Exclude()
password: string;
```

> Note: class-transformer can also be used in dto's and when parsing query parameters (last option is good for transforming string parameters to other types).

### Gotchas

#### Multiple path parameters in endpoint

- If your endpoint path has severals params, you need to specify it in an object like so:

```typescript
@Get('entity1/:id1/entity2/:id2')
linkEntities(
@Param()
params: {
    id1: string
    id2: string
}) {
    // Some function...
}

// Doing it like this will result in unexpected behavior, making both params undefined
@Get('entity1/:id1/entity2/:id2')
linkEntities(
@Param(':id1') id1: string
@Param(':id2') id2: string
) {
    // Some function...
}
```

## TypeORM

We use TypeORM as our ORM to integrate with PostgreSQL. We use the repository pattern, and the `@nestjs/typeorm` package to dependency inject TypeORM repositories into services and controllers. We use schema synchronization in our tests and migrations in our live environments. We automatically check in our CI flow whether or not the schema created by the migrations matches the one defined by the entities.

### Migrations

#### How to

TypeORM CLI can generate migrations automatically based on the difference between defined entitites (`**/*.entity.ts`) and the current database schema.

Use `npm run db:migration -- -n <nameOfTheMigration>` in order to generate a new migration file.

In order to revert a migration run `npx typeorm migration:revert`, delete the migration file and recreate the containers.

You can check the status of migrations by running `npx typeorm migration:show` to verify that the migration in question has been reverted.

If you forget to revert the migration before deleting the migration file, you need to use `docker-compose down -V` which will wipe the db volume and the postgres container will know to recreate the db from scratch.

#### Migration Policy

- Migrations are read-only: No migration may ever be edited when it has entered the staging or production environment.
- Migrations are only to be generated using the TypeORM CLI utility: This ensures no discrepancies between the schema created by the migrations compared to the one defined by the entities.
- Migrations are run automatically on server startup after every deployment.
- Migrations are small and preferably only affect 1 table at a time.
- Migrations should always be reversible to ensure no breaking changes.

#### Migration gotcha's

- If you are working on a migration locally and you delete the `**migration.ts` file, remember that in `dist/migrations/` the compiled JS version will still exist and you also have to remove this.
- Your development database is persisted across docker-compose runs because of `- db:/var/lib/postgresql/data` and thus in order to recreate the database (and thus reapply migrations from scratch) you need to run `docker-compose down -V` which will wipe the persistent volumes

### Best practices

#### Relationships

Read the documentation [here](https://github.com/typeorm/typeorm/blob/master/docs/eager-and-lazy-relations.md)

- We do not recommend using lazy-loaded relationships as they are unintuitive to work with. Instead we recommend using either `{ eager: true }` ( bear in mind that this can only be used on one side of the relationship ) in the entity options of the `@Entity()` decorator or load the relationship explicitly using the `FindOptions` option `{ relations: ['entity2'] }` or with a `leftJoinAndSelect`if using the `QueryBuilder`.

- When making `ManyToMany` relationships, make sure you add the `inverseSide` and specify the table name in the `@JoinTable` decorator. This will ensure typeORM does not create two tables.

- Ensure that you only have the `@Jointable` decorator on one side of the relation.

```typescript
// Specify the inverseSide of relation
@ManyToMany(() => Entity2, (e: Entity2) => e.entity1s, { eager: true })
// Specify the table name to be used. Always use the same table name for both sides of the relationship
@JoinTable({ name: 'entity1_to_entity2' })
entity2s?: Entity2[]

@ManyToMany(() => Entity1, (e: Entity2) => e.entity2s)
// Do not wrap in promise since it is eager
entity1s?: Entity1[]
```

#### Working with dates

- When working with dates, specify the column with `timestamptz`(timestamp with timezone) and use JS date as the attribute type:

```typescript
@Column({ type: 'timestamptz' })
date: Date
```

### Seeding

We use `typeorm-seeding` for creating seeds. Seeds are meant to only be used on a local environment to help test frontends by creating dummy data and admin users.

## PostgreSQL

We use managed SQL servers through Google Cloud SQL. We avoid [common mistakes](https://wiki.postgresql.org/wiki/Don't_Do_This), although our ORM mostly saves us from these.

## Terraform

We use Terraform for managing all of our infrastructure declaratively, and run our changes on every push to staging or production. We store the Terraform state in the `gs://terraform-state-kvalifik` bucket in the "Terraform Remote State Storage" Google Cloud Project. For a project that uses Terraform well, check [Testamentegenerator](https://github.com/Kvalifik/testamentegenerator-backend).

## Testing

We use Jest and Supertest for testing our software. We run all tests on every step of our CI flow. Our testing strategy is to _reasonably argue for our code correctness based on our tests_. This is achieved through end-to-end testing of all of our endpoints as well as some unit tests for complex pieces of code. Our development workflow is primarily test-driven, and we only regard code as finished when our code is well-tested. Our end-to-end tests use a live database-layer, but mocks other external dependencies to keep our testing environments stable. Our unit tests do not use a real database-layer, but instead mocks it.

## File naming policies

### Entities

Each entity should have its own `entities` folder in the `src/[entity-name-plural]` folder and have the filename `[entity-name].entity.ts`.
The service for interacting with the repository of that entity should be named `[entity-name-plural].service.ts` and also stored in `src/[entity-name-plural]`.
Each entity should preferably have a generator function that generates a random record of that entity for use in testing. That is stored in `src/[entity-name-plural]/entities` as `[entity-name].generator.ts`.

### Test files

Each end-2-end test file should be stored in the `test` folder and cover one controller. It should be named `[controller-name].e2e-spec.ts`. Each unit test file should be stored in the same folder as the unit and be called `[unit-name].spec.ts`.
We create mocks for many of our services. Each mock file should be called `[service-name].service.mock.ts` and be stored alongside the service.

### Other

Guards, pipes, controllers and modules all use the naming convention `[name].[type].ts`

## Docker

We use Docker for containerizing our run-time environment. We base our images on the [node:alpine](https://hub.docker.com/_/node) images and always use the latest LTS version. We build our docker images on every push to a branch in our CD flow.
We use `docker-compose` to manage our docker containers during development.

## Environment variables

During development, we use a `.env` file that is ignored by git. Our CI system uses the `.env.ci` file for environment variables needed for end-2-end testing, but since this file is pushed to git, **it must not contain any sensitive information**. All environment variables should be defined in Terraform, and secrets should be managed by Google Secret Manager.

> Note: As of August 2021, there is no good way of adding secret values to google secret manager through Terraform. We can create the secret, but adding a version should still be done by hand. This is a great source of pain, and should be changed in the future to something more sustainable.

## Authentication

We use JWT for authenticating our frontends, passport-jwt for handling the authentication and bcrypt for hashing our passwords. We always salt our passwords, and never store them in plaintext, anywhere.

## Google Cloud Platform

Google Cloud Platform is the only hosting platform that we use for backend projects. Here is a list of services that we use:

- Container Registry for storing our images
- Cloud Run for hosting our containers
- Logging
- Cloud SQL for hosting managed SQL instances
- Cloud Scheduler for running cron-jobs
- App Engine for hosting Cloud SQL and Scheduler
- IAM for identity management and service accounts
- Secret Manager
