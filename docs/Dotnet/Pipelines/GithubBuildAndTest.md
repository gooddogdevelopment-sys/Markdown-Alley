# Example for Github Action Deploy Pipeline
This is an example of how to configure a git repo to build, restore and test a project when changes are pushed to it. Create a .github\workflows folder and place the build.yml file in it. 

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