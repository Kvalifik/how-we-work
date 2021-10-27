# How we use Forecast when developing

All of our projects are managed through [Forecast](https://app.forecast.it). In here, all tasks related to a project are managed via Kanban boards.

When developing, we strive to have 1 Task = 1 Feature = 1 Branch on a project. So if you have a task in Forecast called 'Add Sticky navigation', you should ideally be working on a branch and pull-request that addresses this concern.

To aid this, we use Forecasts GitHub integration and [Kvalifik-CLI](https://github.com/Kvalifik/Kvalifik-CLI) to automatically fetch tasks from Forecast and generate Branches/PRs based on these.

## Setup and requirements

Forecast does not automatically integrate with GitHub, so there's a bit of setup needed.

- You must have Kvalifik-CLI installed via: `npm install -g kvalifik-cli`
  For details, see [Kvalifik-CLI repository](https://github.com/Kvalifik/Kvalifik-CLI) or `kvalifik-cli help`<br><br>

- The project should be integrated with a GitHub Repository
  See 'Setup: GitHub Project integration'.<br><br>

- Your Forecast profile should be linked to GitHub
  see 'Setup: GitHub Linked Profile'

### Setup: GitHub Project integration

All Forecast projects that involve code, needs to have a linked GitHub Repository.

The GitHub integration is a 2-way sync of Forecast Task with GitHub Issues. This means that every task in Forecast, exists as an issue on the linked repository. Assignees\*, labels, description, comments and milestones/phases are also syncronised.

The integration is set up per project.

Go to 'Settings' -> 'Integrations' and scroll down to select a repository.

![](src/how-we-use-forecast/github_settings.png)

Please only select 1 repository, even if Forecast let's you select multiple. We _**can't**_ synchronise 1 project with multiple repositories. For Forecast projects that require multiple repositories, refer to the Kvalifik-CLI documentation for our workaround.

> \*Requires a linked GitHub profile. See below for how to set this up.

### Setup: GitHub Linked profile

Like our projects, all Forecast profiles should be integrated with GitHub. This is especially helpful when assigning tasks, as this is synchronised between Forecast & GitHub.

To link your GitHub account to Forecast, go to ['My Profile'](https://app.forecast.it/my-profile/profile), and scroll down to the bottom. When successfully linked, it should look like this:

![](src/how-we-use-forecast/github_linkedaccount.png)

Now, everytime you're assigned to a task in Forecast, you'll also be assigned to the Issue in GitHub. This let's you easily filter between issues when working in larger dev teams. Kvalifik-CLI has a handy '-a' flag on the 'work-on' command that only shows issues assigned to you: `kvalifik w -a`

## Usage

In practice, the Forecast/Github Integration handles most of the work. Every time a task is added to a project it automatically beocmes an Issue in the linked GitHub repository. If changes are made to the task, the Issue is updated.

To begin working on a task, clone the repository, write `kvalifik w` in your terminal and select the task from the list. This sets up a branch and draft pull request that's linked to the issue.

> At this point, you might need to manually move the task in Forecast into 'In Progress' to let your teammates know that you've picked up the task.

When your code is ready for review, you can write `kvalifik rr` and the CLI marks the pull request as 'Ready for Review'. Make sure to request a review!

> If your project has a column called 'Ready for Review', 'Code Review', 'Awaiting Approval' or similar, you should move the task into it; so everyone _not_ on GitHub is aware of what's going on.

Once your code has been reviewed, approved hopefully, and merged, the issue is closed and the task in Forecast is moved to the closest 'Done' column.
