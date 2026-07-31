# CI/CD Pipeline Fundamentals

A CI/CD pipeline is an automated sequence that takes a code change from a commit through building, testing, and releasing it, so that integration and delivery happen consistently rather than by hand.

This doc explains the vocabulary and building blocks that apply to any pipeline, whatever tool runs it (GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, and so on). The concepts are portable even though the YAML syntax differs.

For a concrete implementation, see the [GitHub Actions Build & Test](../../Dotnet/CI-CD/GithubBuildAndTest.md) workflow. For the official model this terminology follows, see the [GitHub Actions documentation](https://docs.github.com/en/actions/using-workflows/about-workflows).

---

## CI vs. CD

Continuous Integration (CI) is the practice of merging every change into a shared mainline frequently and verifying each merge automatically by building the code and running tests. Its goal is to catch integration problems early, while they are small.

Continuous Delivery (CD) extends CI so that every change that passes CI is automatically prepared for release and can be deployed to production with a manual approval. Continuous Deployment goes one step further and releases every passing change to production automatically, with no human gate. The two share the initials "CD"; which one a team means depends on whether a human approves the final push to production.

```text
Continuous Integration:  commit -> build -> test
Continuous Delivery:     commit -> build -> test -> deploy to staging -> (manual approval) -> production
Continuous Deployment:   commit -> build -> test -> deploy to staging -> production   (no manual gate)
```

---

## Triggers

A trigger is the event that starts a pipeline run. The most common trigger is a push to a branch or a pull request, but runs can also start on a schedule (cron), on a published tag or release, or manually on demand. Choosing the right trigger keeps pipelines relevant: run the full test suite on pull requests, run deployments only on the main branch or on a tag.

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:   # manual "Run workflow" button
```

---

## Jobs, Steps, and Stages

A pipeline is composed of jobs, and each job is an ordered list of steps. A step runs a single command or a prepackaged action; a job is a group of steps that run together on the same runner and share a filesystem. Steps within a job run sequentially and stop on the first failure.

Jobs run in parallel by default. A stage (sometimes called a phase) is a logical grouping of jobs — such as "build", "test", "deploy" — where one stage completes before the next begins. In tools without an explicit stage keyword, stages are expressed by declaring dependencies between jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # a prepackaged action
      - run: dotnet build                   # a shell command

  test:
    needs: build        # "test" waits for "build" -> forms a stage boundary
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: dotnet test
```

---

## Runners

A runner is the machine or container that executes a job. Hosted runners are provided and maintained by the CI service (a fresh, clean virtual machine per job); self-hosted runners are machines you own and register, used when a job needs special hardware, private network access, or licensed software. Each job starts on a clean runner, which is why any state a job needs — dependencies, build output — must be installed or restored during the run.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest     # hosted runner image
  deploy:
    runs-on: [self-hosted, production]   # matches a runner you registered with these labels
```

---

## Artifacts

An artifact is a file or set of files produced by one job and preserved so a later job — or a person — can use it. Because each job runs on its own clean runner, passing a compiled binary or a test report from a build job to a deploy job requires uploading it as an artifact in the first job and downloading it in the second. Artifacts are also how you retain build outputs, coverage reports, or logs after a run finishes.

```yaml
  build:
    steps:
      - run: dotnet publish -o ./out
      - uses: actions/upload-artifact@v4
        with:
          name: app
          path: ./out

  deploy:
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app
```

---

## Environments and Gates

An environment represents a deployment target such as staging or production. Environments let you attach protection rules — required reviewer approval, a wait timer, or a restriction on which branches may deploy — so that reaching production is a controlled step rather than an automatic side effect. This is the mechanism that distinguishes Continuous Delivery (a gate before production) from Continuous Deployment (no gate).

```yaml
  deploy-production:
    needs: test
    environment:
      name: production        # approval rules configured on this environment apply here
      url: https://example.com
    steps:
      - run: ./deploy.sh
```

---

## Caching

Caching stores dependencies or intermediate output between runs so a pipeline does not redownload or rebuild everything each time. A cache is keyed on something that changes when the cached content should change — typically a hash of a lock file — so the cache is reused while dependencies are stable and rebuilt when they change. Caching is an optimization and must never hold anything a run depends on for correctness; treat it as disposable.

```yaml
      - uses: actions/cache@v4
        with:
          path: ~/.nuget/packages
          key: nuget-${{ hashFiles('**/packages.lock.json') }}
```

---

## Anatomy of a Typical Pipeline

Putting the pieces together, a mainstream pipeline triggers on pull requests and pushes to main, restores cached dependencies, builds once and shares the output as an artifact, runs tests in parallel, and — only on the main branch — deploys through a gated production environment.

```text
trigger (PR / push to main)
   -> build job      (restore cache, compile, upload artifact)
   -> test job       (download nothing new; run unit + integration tests)   [depends on build]
   -> deploy job     (download artifact, deploy to gated environment)        [main branch only]
```

---

## See Also

- [GitHub Actions Build & Test](../../Dotnet/CI-CD/GithubBuildAndTest.md) — a working .NET implementation of the build-and-test stages described here.
