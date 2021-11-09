# How we setup a new backend environment for a project using Terraform

First, create a GitHub repository by copying our template repository [template-backend-nestjs](https://github.com/Kvalifik/template-backend-nestjs).

<aside>
☝ Please change the directory to your local repository corresponding to the project. This will enable the upcoming gh commands to add secrets to the right repository in GitHub.

</aside>

### Setting up Terraform state management

Let's generate a service account in the GCP project Terraform Remote State Storage and use it to store terraform state for the project.

In the following we have one parameter (with example):

- **[project-name]** = running26 (alphanumeric, using lower-kebab-case)

Note that [project-name] is independent of the environments (staging, production, etc.) which we will consider in the following section.

Create a service account for the project Terraform Remote State Storage:

```bash
gcloud iam service-accounts create [project-name] --description="Allows Terraform to store state for [project-name]" --display-name="[project-name]-service" --project=terraform-remote-state-storage
```

Assign Roles "Storage Legacy Bucket Owner" and "Storage Legacy Object Owner" for the bucket "terraform-state-kvalifik" to this service account:

```bash
gsutil iam ch serviceAccount:[project-name]@terraform-remote-state-storage.iam.gserviceaccount.com:roles/storage.legacyObjectOwner gs://terraform-state-kvalifik
gsutil iam ch serviceAccount:[project-name]@terraform-remote-state-storage.iam.gserviceaccount.com:roles/storage.legacyBucketOwner gs://terraform-state-kvalifik
```

Create a private key for the service account.

<aside>
🚨 This will output a file at the specified path. Delete it as soon as you have finished this tutorial and do not check it into version control.

</aside>

```bash
gcloud iam service-accounts keys create terraform-state-key.json --iam-account=[project-name]@terraform-remote-state-storage.iam.gserviceaccount.com
```

The contents of this JSON file should be added to the GitHub repo for a secret called "GCP_SA_KEY_TERRAFORM_STATE"

```bash
gh secret set GCP_SA_KEY_TERRAFORM_STATE < terraform-state-key.json
```

After making sure the above command succeeded, remember to delete the service account key:

```bash
rm terraform-state-key.json
```

Next, we create a GCP project for each environment.

### Setting up an environment

In the following we have several parameters (with examples):

- **[environment]** = staging (alphanumeric, using lower-kebab-case)
- **[environment_uppercased]** = STAGING (alphanumeric, using UPPER_SNAKE_CASE)
- **[project-id]** = [project-name]-[environment] = running26-staging

Do the following for each environment:

Create a GCP project. 

```bash
gcloud projects create [project-id] --name="[project-id]" --organization=4077818049
```

Enable billing by running the following command. **This will set "Kvalifik" as the default billing account. Please contact Nikolaj and have a unique billing account for this project created ASAP.**

```bash
gcloud beta billing projects link [project-id] --billing-account 0177D2-4E78D4-844B8B
```

Creating Service Account in Project [project-id]
Grant Access to Dev Team:

```bash
gcloud projects add-iam-policy-binding [project-id] [--member="group:developer-team@kvalifik.dk](mailto:--member=%22group:developer-team@kvalifik.dk)" --role="roles/owner"
```

Create Service Acount:

```bash
gcloud iam service-accounts create terraform-service --description="Allows Terraform to operate" --display-name="Terraform Service" --project=[project-id]
```

Grant Access to project:

```bash
gcloud projects add-iam-policy-binding [project-id] --member="serviceAccount:terraform-service@[project-id].iam.gserviceaccount.com" --role="roles/owner"
```

Enable Cloud Resource Manager API (might need special privileges, ask Nikolaj):

```bash
gcloud services enable [cloudresourcemanager.googleapis.com](http://cloudresourcemanager.googleapis.com/) --project=[project-id]
```

Create a private key for the service account.

<aside>
🚨 This will output a file at the specified path. Delete it as soon as you have finished this tutorial and do not check it into version control.

</aside>

```bash
gcloud iam service-accounts keys create [project-id]-key.json --iam-account=terraform-service@[project-id].iam.gserviceaccount.com
```

The contents of this JSON file should be added to the GitHub repo for a secret called "GCP_SA_KEY_[environment_uppercased]"

```bash
gh secret set GCP_SA_KEY_[environment_uppercased] < [project-id]-key.json
```

After making sure the above command succeeded, remember to delete the service account key:

```bash
rm [project-id]-key.json
```

Make another secret in GitHub setting GCP_PROJECT_ID_[environment_uppercased] to [project-id].

```bash
gh secret set GCP_PROJECT_ID_[environment_uppercased] 
```

<aside>
☝ Note, please let the CD workflow (GitHub Action) run without deploying to Cloud Run the first time (these lines are commented out in the template repository) so that the container registry can be set up correctly. Afterwards, reintroduce these lines of code to also deploy Cloud Run.

</aside>

<aside>
☝ Note, the CD workflow (GitHub Action) might fail the time because a lot of GCP APIs are activated and the script might timeout. In that case, just run the script again on GitHub by clicking "Re-run all jobs" on the failed workflow.

</aside>