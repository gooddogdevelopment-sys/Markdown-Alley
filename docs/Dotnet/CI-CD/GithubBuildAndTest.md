# GitHub Actions Build & Test Workflow

A GitHub Actions workflow that restores, builds, and tests a .NET solution on every push and pull request to `main`.

## Prerequisites

- A .NET solution (`.sln`) with at least one test project, hosted in a GitHub repository.
- Push access to the repository so you can add workflow files and trigger runs.

## Steps

1. In the repository root, create a `.github/workflows` folder if it doesn't already exist.
2. Add a `build.yml` file to that folder with the workflow below. Replace `your-project.sln` with your own solution file name, and set `dotnet-version` to the SDK version your project targets.
3. Commit and push the file (or open a pull request) against `main`.
4. Verify the run: open the repository's **Actions** tab, confirm the **Build & Test** workflow succeeds, and download the `test-results` artifact from the run summary to inspect the `.trx` output.

## Step Overview

| Step | What it does |
|------|-------------|
| **Checkout** | Clones the repository into the runner so subsequent steps have access to the source code. |
| **Setup .NET** | Installs the specified .NET SDK version on the runner. |
| **Restore dependencies** | Runs `dotnet restore` to download all NuGet packages defined in the solution. |
| **Build** | Compiles the solution in Release configuration, skipping restore since it already ran. |
| **Test** | Runs all test projects in Release mode without rebuilding, and writes results to a `.trx` file. |
| **Upload test results** | Uploads the `.trx` file as a workflow artifact so results are accessible in the GitHub Actions UI. Runs even if tests fail (`if: always()`). |

```yml
name: Build & Test

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    name: Build, Test & Restore
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "10.x"

      - name: Restore dependencies
        run: dotnet restore your-project.sln

      - name: Build
        run: dotnet build your-project.sln --no-restore --configuration Release

      - name: Test
        run: dotnet test your-project.sln --no-build --configuration Release --logger "trx;LogFileName=test-results.trx"

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: "**/*.trx"
```

## See Also

- [.NET 10 Core API with PostgreSQL Template](../Templates/Core10WPostgres.md) — ships with a GitHub Actions CI pipeline out of the box.