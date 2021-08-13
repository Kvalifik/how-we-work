# How We Use Node.js for Backend Development
We use Node.js for our backend services, using the NestJS framework, TypeScript, TypeORM and PostgreSQL. We deploy the services on Google Cloud Platform using Terraform and Github Actions. The services are run as Docker containers, and we use docker-compose for our local development environment.

## NestJS
We use NestJS for all our backend services, which also means that we adhere to the NestJS architecture (Model-View-Controller architecture with dependency injected providers). We adhere to the [SOLID principles](https://en.wikipedia.org/wiki/SOLID) and divide our code into small, easily testable units of code. 
We abstract dependencies into services whenever possible.
> Example: A backend that depends on Axios for outbound request would have a service called RequestService that itself calls the methods that Axios might need.

## TypeORM
We use TypeORM as our ORM to integrate with PostgreSQL. We use the repository pattern, and the `@nestjs/typeorm` package to dependency inject TypeORM repositories into services and controllers. We use schema syncronization in our tests and migrations in our live environments. We check in our CI flow whether or not the schema created by the migrations matches the one defined by the entities.

### Migration Policy
- Migrations are read-only: No migration may every be edited when it has entered the staging or production environment.
- Migrations are only to be generated using the TypeORM CLI utility: This ensures no discrepancies between the schema created by the migrations compared to the one defined by the entities.
- Migrations are run automatically on server startup after every deployment.
- Migrations are small and preferably only affects 1 table at a time.
- Migrations should always be reversable to ensure no breaking changes.

## PostgreSQL
We use managed SQL servers through Google Cloud SQL. We avoid [common mistakes](https://wiki.postgresql.org/wiki/Don't_Do_This), although our ORM mostly saves us from these.

## Terraform
We use Terraform for managing all of our infrastructure declaratively, and run our changes on every push to staging or production. We store the Terraform state in the `gs://terraform-state-kvalifik` bucket in the "Terraform Remote State Storage" Google Cloud Project. For a project that uses Terraform well, check [Testamentegenerator](https://github.com/Kvalifik/testamentegenerator-backend). We seperate our environments using `terraform workspace`. All terraform configurations should be able to easily be run in multiple environments.

## Testing
We use Jest and Supertest for testing our software. We run all tests on every step of our CI flow. Our testing strategy is to _reasonably argue for our code correctness on the background of our tests_. This is achieved through end-to-end testing of all of our endpoints as well as some unit tests for complex pieces of code. Our development workflow is primarily test-driven, and we only regard code as finished when our code is well-tested. Our end-to-end tests use a live database-layer, but mocks other external dependencies to keep our testing environments stable. Our unit tests do not use a real database-layer, but instead mocks it. 

## File naming policies
### Entities
Each entity should have its own `entities` folder in the `src/[entity-name-plural]` folder and have the filename `[entity-name].entity.ts`. 
The service for interacting with the repository of that entity should be named `[entity-name-plural].service.ts` and also stored in `src/[entity-name-plural]`.
Each entity might have a generator-function that generates a random record of that entity for use in testing. That is stored in `src/[entity-name-plural]/entities` as `[entity-name].generator.ts`.

### Test files
Each end-2-end test file should be stored in the `test` folder and cover one controller. It should be named `[controller-name].e2e-spec.ts`. Each unit test file should be stored in the same folder as the unit and be called `[unit-name].spec.ts`.
We create mocks for many of our services. Each mock file should be called `[service-name].service.mock.ts` and be stored alongside the service.

### Other
Guards, pipes, controllers and modules all use the naming convention `[name].[type].ts`

## Docker
We use Docker for containerizing our run-time environment. We base our images on the [node:alpine](https://hub.docker.com/_/node) images and always use the latest LTS version. We build our docker images on every push to a branch in our CD flow.
We use `docker-compose` to manage our docker containers during development. 

## Environment variables
During development, we use a `.env` file that is ignored by git. Our CI system uses the `.env.ci` file for environment variables needed to end-2-end test, but since this file is pushed to git, **it should never contain any sensitive values**. All environment variables should be defined in Terraform, and secrets should be managed by Google Secret Manager.
> Note: As of August 2021, there is no good way of adding secret values to google secret manager through terraform. We can create the secret, but the adding a version should still be done by hand. This is a great source of pain, and should be changed in the future to something more sustainable.

## Authentication
We use JWT for authenticating our frontends, passport-jwt for handling the authentication and bcrypt for hashing our passwords. We always salt our passwords, and never store them in plaintext, anywhere.

## Google Cloud Platform
Google Cloud Platform is the only hosting platform that we use for backend projects. Here is list of services that we use:
- Container Registry for storing our images
- Cloud Run for hosting our containers
- Logging
- Cloud SQL for hosting managed SQL instances
- Cloud Scheduler for running cron-jobs
- App Engine for hosting Cloud SQL and Scheduler
- IAM for identity management and service accounts
- Secret Manager
