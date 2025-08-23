# Learn CI/CD 
We are learning how to automate the process of building, testing and deploying
software.

## What is a Branch?
A branch is a copy of you codebase. However, it's special kind of a copy that
makes merging new code changes from one branch to another simple and easy.

So far we have worked on one branch, we can create as many branches as we need
and work on them separately it is a great way of keeping changes isolated and 
contained.

In many teams, the main branch is reflects the state of the codebase, that's
running the production. This means that the main branch should always be stable
and ready to deploy, if you want to
- Add a new feature.
- Fix a bug.
- Refactor some code.

Then you should create a new branch to add those changes to. This allows you to
work on those changes independently without affecting the main branch.

## Pull Requests
Pull requests are a way to propose changes to a codebase. They've great way to 
collaborate with your team and ensure that your code reviewed and passes tests
before it's merged into the main branch for deployment and for other developers
to work on.

## Continuous Integration
CI is where developers are regularly push code changes into a central repository,
and by doing so automated builds are tests are automatically run.

Those tests can include unit tests, styling checks, linting checks, security checks
or any other type of automated test. If any of the tests fail, the build is considered
"broken" and the developers is notified so they can fix it.

*CI is all about automating as much of the testing and review process as possible*

Here is the CI we used in the course
```yml

name: ci

on: 
  pull_request:
    branches: [main]


jobs:
  tests:
    name: Tests
    runs-on: ubuntu-latest

    steps:
      - name: Check out code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: "1.23.0"

      - name: Force Failure
        run: (exit 1)
```
By default, a step "succeeds" if it exits with a status code of 0 and "fails"
if it exits wit a status code other then 0.

Workflows:
A workflow is triggered when an event occurs in your Github repository. For example,
we'll trigger our "tests" workflow when we open a pull request into the main branch.

In our case, the `ci.yml` file contains a single workflow called `ci`, of course
we could have named it anything.

Jobs:
A **workflow** is made up of one or more jobs. A job is a set of steps that run
on the same **runner**, which is just a virtual machine that runs your job on Github's
servers.

For now, we just have one job. You would typically have multiple job if you wanted
to run tests on multiple operating systems

In our case, the `ci.yml` workflow contains a single job called "Tests".

Steps:
A job is made up of one or more steps. A step is a single task that can run
commands, a script or action, For example the steps of a job include:
- Checking out the code.
- Installing dependencies.
- Running Test.

In our case, The "Tests" job contains 3 steps:
1: Check out the code.
2: Set up Go.
3: Force failure of the CI job.


**A workflow that triggers on pull requests will re-run when the branch to be
merged is updated**


Here is the full breakdown:

#### Workflow Name
```yml

name: ci

```
The first line simply assigns a human-readable name to the workflow.

#### Triggering the workflow
```yml

on: 
	pull_request:
		branches: [main]

```
The `on` key specifies when the workflow should run. In our case, we want to run
the workflow when a pull request is opened to `main` branch.

#### Jobs
```yml
jobs:
	tests:
		name: Tests
		runs-on: ubuntu-latest
```
The `jobs` key is list of jobs that mark up the workflow. In our case, we only
have one job called `tests`.

## Formatting

**Important**
Separate jobs are run in parallel.
