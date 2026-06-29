# Example for Github Action Deploy Pipeline
This is an example of how to configure a git repo to build, restore and test a project when changes are pushed to it. Create a .github\workflows folder and place the build.yml file in it. 

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
        run: dotnet restore newsletter-api.sln

      - name: Build
        run: dotnet build newsletter-api.sln --no-restore --configuration Release

      - name: Test
        run: dotnet test newsletter-api.sln --no-build --configuration Release --logger "trx;LogFileName=test-results.trx"

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: "**/*.trx"
```