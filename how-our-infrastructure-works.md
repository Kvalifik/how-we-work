# How Our Infrastructure Works
This document explains the main reasoning behind our choice of infrastructure as well as give an intuition that might make our infrastructure configuration more readable. This is _not_ a manual on how to use said technology - for this, please refer to the respective documentation.

## Terraform
We use Terraform for declarative infrastructure. Terraform allows us to describe our Google Cloud configuration in text files in the `terraform/` directory, which we can store in Git.
Whenever we merge a branch into staging or production, our Continuous Delivery setup runs Terraform, which transforms the current state of the Google Cloud setup to the state described in the configuration files.
We use Terraform for a multitude of reasons:
- Infrastructure as code is self-documenting.
- Infrastructure as code can be copied between projects.
- Infrastructure as code can keep the setup between different environment identical.
- Infrastructure as code allows us to version control our infrastructure configuration.
- Infrastructure as code allows us to reason between configurations in the PR stage

Before starting to develop Terraform, please make sure that you have a solid understanding of the difference between _ressources_, _data references_  and _providers_. It will help you immensely understand the code.

For Terraform to work, we need to store a serialized representation of the state of the project somewhere. This is done in the Terraform Remote State Storage project in a Storage bucket.

When we develop Terraform code, it is common that developers use the CD environment to run the code (i.e. push Terraform code and hope that it works).
It is also possible to locally develop the Terraform code and use the Terraform CLI to apply the changes to the staging environment.
To do such, you need access credentials to two service accounts: The service account that manages the staging project you want to edit, and the service account that manages the Terraform state bucket in the Terraform Remote State Storage project.
To get the two access credentials locally, run the following commands in `terraform/` (replace values as nescessary):
`gcloud iam service-accounts keys create state-bucket-service-account.json --iam-account=[project-name]@terraform-remote-state-storage.iam.gserviceaccount.com`
`gcloud iam service-accounts keys create service-account.json --iam-account=terraform-service@[project-id].iam.gserviceaccount.com`
> Note: These commands creates access keys that will be stored locally on your machine. It is *very* important that these are *not* stored in version controlled. It is strongly recommended that these files are deleted after use.

You will also need a `.terraform.tfvars` file containing the environment variables that terraform depends on (i.e. the variables declared in `terraform/variables.tf`). You can see what variables are needed in `.github/workflows/deploy-[environment].yml`.

Sometimes, if a developer has ended a CD flow prematurely, Terraform refuses to start again. This is due to the fact that Terraform tries to secure that only one instance is making changes to the configuration at a time by creating a lock file in the state bucket. This can be fixed by simply deleting the lock file.

When developing terraform ressources, it can be good to note that all ressources supports the `depends_on = [array]` attribute. By entering references to other ressources in this array, you can help avoid race conditions. An example of this could be to make sure a secret has been created before referencing it in your Cloud Run service.

## Cloud Run

We use Cloud Run as the platform to host our NestJS backends. Cloud Run is a service that spins up a container on an incoming request, lets the container handle the request and shuts down the instance afterwards.
Cloud Run uses a docker image as a basis to create the container. This docker image should contain the NestJS backend and listen to the `$PORT` environment variable.
Cloud Run will by default supply a URL that can be used to connect to the instance, but a domain can also be supplied (look up "Domain Mappings" in the Cloud Run documentation for more info).

Our Terraform setup declares a cloud run service called `default` that represents the server. It gets supplied with a docker image, a domain name (if applicable), environment variables (either regular envs or secrets) and potential connections to an SQL instance.

For a Cloud Run instance to connect to other server instances, networking needs to be set up. See [Cloud Networking](#Cloud-Networking). The only exception to this is Cloud SQL where Cloud Run supports a special connector to avoid handling this.

When developing for Cloud Run, it is important to note that Cloud Run instances are ephemeral. This means that the instance should not at any point carry state that lasts across different requests. 
This is due to the fact that the instance can be shut down at any time, so no state is guaranteed to last. If state is important to the application, consider using [Cloud MemoryStore](#Cloud-MemoryStore).
This is in general also just good code practice.

An update to a Cloud Run service is called a revision. A revision will only be used if Cloud Run is able to start a container from the supplied image, and the container listens to a given port.
All updates to the Cloud Run service through Terraform do not actually change the service, but just creates a new revision (which will be accepted if it can start).

## Compute Engine

We use Compute Engine as the platform to host all services that are required to be stateful or always be available (with the exception of SQL servers or Redis instances).
An example could be an FTP server or a video processing service.
Compute Engine can be quite expensive to run, so talk to your project manager before starting a compute engine instance.
Compute Engine needs a VPC connector to be able to communicate with other cloud ressources, just like Cloud Run does.
Compute Engine also do not have the Domain Mapping feature that Cloud Run does, so extra network configuration is required to do so.

Compute Engine can take a docker image as input, but requires more configuration to get to work (such as setting up volumes and such).

## Secrets

We use Google Secret Manager to manage our secrets (can be found in the GCP panel under the "Security" tab).
Google Secret manager creates a cloud ressource to represent each secret. A secret then have multiple versions, where each version contains a secret value. Often we just use the most recent secret version.

While we can (and do) declare a Secret ressource in Terraform, we cannot (due to the nature of secrets being secret) declare the actual value of the first secret version when it is created. 
This is solved by creating the secret version manually before declaring it in the code and running Terraform.

## Container Registry

We use container registry to store the docker images that Cloud Run depends on. Under-the-hood, this is done by storing the image in a storage bucket. You can also push or pull from the container registry yourself if you authorize your machine using the `gcloud auth configure-docker` command.

## Docker

We use Docker for running our backends in the same environment on different machines. Docker allows us to build a specific environment once and deploy an entire image where all dependencies are included.
We use node images with Debian as the base operating system for these images, and we store our code in `/usr/src/app` of the image.
We use `docker-compose` locally to manage our development environment and handling the different services that are required to run the application.

## Github Actions

We use Github Actions for our CD and CI flow. The configuration files are found in `.github/workflows`
Our CI flows usually handles building, linting, schema checking, unit testing and end-2-end testing. A CI flow will often have one or more service containers running to help end-2-end test the application. 
An example could be running a PostgreSQL server when testing a NestJS server, such that the server has something to connect to.
CI always uses the `.env.ci` file as an env file. If CI fails, you should probably not merge your branch into staging before fixing the issue.

Our CD flows usually handles building the application, pushing images to Google Container Registry and performing changes to the infrastructure by running terraform.
Regular environment variables are usually declared in the CD file, since they often are dependant on the environment on which they are running in. Application secrets are still declared in Google Secret Manager and need not to be declared in the CD file.
Github actions usually depends on [Github Secrets](#Github-Secrets) to run.

## Github Secrets

We use Github Secrets to supply secrets to our Github Actions. These secrets are only used for _CI_ and _CD_ and are not used by the actual application. 
This is because we do not model Github Secrets declaratively, so it is hard to version control and get the benefits of declarative infrastructure using Github Secrets.
An example of a usecase of Github Secrets is to store the authentication credentials required to perform changes using Terraform.

## Cloud SQL

We use Google Cloud SQL with PostgreSQL as our SQL service. This is because Cloud SQL is managed, so we do not need to perform maintenance ourselves, and because they have a reliable backup service.
We use Terraform to model the database instance as well as the initial configuration (i.e. the initial database to be created as well as the user to create).

## Cloud MemoryStore

We use Cloud MemoryStore with Redis as our memory storage instance. This is used for task/job queues, key-value storage and other miscellaneous tasks where an ephemeral Cloud Run instance might need to store state.
MemoryStore is by default accessible by all CloudRun instances without a username or a password. 

## Cloud Networking
> Note: This section should be expanded when we get more experience working with networking

For Cloud Run or Compute Engine to talk to eachother or to let the outside world connect to these instances through non-trivial ports, additional network configuration is needed.
To let different services connect to eachother, a VPC connector can be needed. An example of this can be seen in the Mikkeller Backend. Consult the google documentation for more information about this.

## Cloud Storage

We use Cloud Storage to store files used by our applications. The files include static files in our application, terraform state, docker images and such.

## Cloud Scheduler 

We use Cloud Scheduler to call our Cloud Run instances at a fixed time interval (like cron-jobs). 

## Expo

We use Expo for our React Native projects. Expo allows us to develop, build and deliver React Native projects easily and will soon also be used to submit the app to the different app stores.

## Vercel

We use Vercel to host our frontend sites. Vercel also manages Continuous Deployment. We do not manage Vercel through Terraform due to lack of supported providers.

## Firebase

Some (legacy) projects use Firestore to store data, Firebase Cloud Functions for cloud computing and Firestore Rules for security management. These projects are not managed using Terraform.

## Netlify

Some (legacy) projects use Netlify to deploy frontends. These are setup much alike Vercel, but we prefer Vercel because it can do Server-Side-Rendering at the edge.
