# How we use GitHub (and Git in general)

We use Git for version controlling our code and GitHub for hosting our repositories, running CI/CD tasks and managing pull requests, issues and code reviews. All code must be stored on GitHub.

### Sections:

- [Getting started](#getting-started)
- [Workflow and best pratices](#workflow-and-best-practices)
- [Project setup](#project-setup)

---

## Getting Started

If you are not familiar with git, start from learning the basics: [Here is a guide for learning how to use git](https://rogerdudler.github.io/git-guide/).

We use 2-factor authentication for our GitHub accounts. We recommend [connecting SSH keys to your Github account](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh).

If you are familiar with git, you should know that we automate some of the git workflow using [kvalifik-cli](https://github.com/Kvalifik/Kvalifik-CLI), an internal tool we've built. There are 2 commands you should know about: `kvalifik-cli work-on` (use to start working on a feature/bug fix/anything else) and `kvalifik-cli ready-for-review` (use when the PR created using `work-on` is ready for review).

- `work-on` starts a new branch and a draft PR based on a previously-created [issue](#issues) (i.e. a task in forecast and a GitHub issue). Use the `-b` paramater to provide the name of a branch if you want to branch out from another than the default branch. See [kvalifik-cli documentation](https://github.com/Kvalifik/Kvalifik-CLI) and [Branches (below)](branches) for more info.
- `ready-for-review` will change the status of the PR that `work-on` created from draft to a normal PR - you can then open it and request a review from your team members.

#### Tip for working on a new task that depends on a PR that isn't merged yet

Often you will have just finished work on 1 task and set the PR ready for review. While you await review, you will start work on the "next" task, that needs the code from your un-merged PR. Let's call the PR await for review the "first PR" and the new, draft PR the "second PR"

_What to do in that case:_

1. When starting the next task, make sure to be on the _un-merged-branch-name_ (the one from the first PR) [`kvalifik-cli work-on -c`](https://github.com/Kvalifik/Kvalifik-CLI#kvalifik-cli-work-on) to branch out of the _un-merged-branch-name_ instead of the default branch
2. Edit the base branch in the PR created by `work-on` (second PR) from the default branch to the _un-merged-branch-name_
3. Make a comment that the second PR should be preceeded by the first PR
4. After merging the first PR, **delete the branch** (the option appears in the GitHub UI after merging the branch) which will automatically change the base of the second PR to the default branch.

_That way you gain 2 advantages:_

- The reviewer of this PR doesn't accidentally review the code from the earlier PR
- You don't have merge conflicts when either of these are merged to the trunk

The only thing you need to be careful about is, that the earlier PR is merged _before_ this new one. Otherwise this PRs code diff will show up as diff in the earlier one.
Merging the earlier one first and deleting the branch will cause this PR to automatically resolve it's base branch to staging, meaning everthing will work smoothly :-)

## Workflow and best practices

The patterns we use and best practices we would like to follow. Some of it is automated by the _kvalifik-cli_.

- **Branches**
  _(Handled by kvalifik-cli)_ We use Gitflow as our branching strategy. Read more about it [here](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).
  Each repository has a default branch. This should be set to a branch named `staging`, and should represent the state of the staging environment of the application (called `develop` in the Gitflow documentation).
  Each repository should also have a branch called `production`, which stores our production code (called `main` in the Gitflow documentation).
  Each feature branch should have the name `feature/[issue-id]-[title]` corresponding to the Github issue ID and the title of the issue. _You may only push to a feature branch, never directly to staging nor production_.

      > Note: Some Shopify projects (notably shop.mikkeller.dk) use a three-tier infrastructure of development-staging-production

- **Issues**
  _(Handled by kvalifik-cli_) We use Github issues to connect tasks to pull requests and to easily work on tasks through the [Kvalifik CLI](https://github.com/Kvalifik/Kvalifik-CLI). The issues are populated through Forecast by Forecast's Github integration.
- **Commits**
  We aspire to keep commits small and plenty. They should only contain relevant hunks and their commit message should accurately describe the change that the hunk implements. We do not squash our commits due to potential merge conflicts.
- **Pull requests**
  _(Mostly handled by kvalifik-cli)_ We use Github pull requests to manage pending code changes. Each pull request should be linked to the issue that it solves, and should always go from a feature branch to the `staging` branch. It is also possible to make a pull request from the `staging` branch to the `production` branch to launch new features. _We never merge a feature branch directly into the production branch._ Pull requests are automatically generated by the Kvalifik CLI, and should be created as a draft pull request as soon as one starts working on an issue. It shall stay as a draft until it is ready for review. All developers should aspire to keep pull requests small, but plenty.
  When a PR has been merged, the PR branch should be deleted to keep the repository free of stale branches.
- **Merge Conflicts**
  We resolve merge conflicts and test our code again before requesting code review. We recommend resolving merge conflicts locally and not using GitHub's web interface.
- **Code review**
  We use Code Reviews to review and approve code changes to the `staging` branch. Every pull request needs a code review to be merged, and everyone can perform a code review.
- **CI/CD**
  We use _Continuous Integration_ through Github Actions to perform lint checks, run tests, build software and perform static validations on the code before we merge into `staging` or `production`. A branch shall only be merged when all checks have passed. We use _Continous Deployment_ through Github Actions to deploy our code to its respective platforms. This happens whenever code is merged into `staging` or `production`.

## Project setup

### Forecast Integration

All projects should have the Forecast integration enabled. It can be enabled on the forecast project under "Settings".

### Teams

By default, members of our GitHub organization can't access any repositories. Instead, access is limited to teams within the organization. Each team relates to a project or client. For example, all repositories related to Mikkeller are part of the Mikkeller team.

To gain access to these repositories, you can ask to join the team, via the team overview found [here](https://github.com/orgs/Kvalifik/teams).

If you work more than 15 hours a week as a developer at Kvalifik, then you should be an owner of the organization, which gives you access to all repositories.

All interns should be members of the 'Interns' team granting them access to a limited subset of repositories.

### Branch rules

Each repository should have the following branch rules:
For branches matching `/staging/`:

- Require pull request reviews before merging
- Require status checks to pass before merging
- Require branches to be up to date before merging (note that this requires a status check to be enabled)

For branches matching `/production/`:

- Require pull request reviews before merging
- Require status checks to pass before merging
- Require branches to be up to date before merging (note that this requires a status check to be enabled)
