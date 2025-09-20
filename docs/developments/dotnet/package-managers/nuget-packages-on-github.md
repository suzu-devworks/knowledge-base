# NuGet packages on GitHub

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Creating a NuGet Package](#creating-a-nuget-package)
- [Configuring GitHub Packages as a NuGet Source](#configuring-github-packages-as-a-nuget-source)
- [Publishing a NuGet Package to GitHub (Manual)](#publishing-a-nuget-package-to-github-manual)
- [Automatic Publishing with GitHub Actions](#automatic-publishing-with-github-actions)
  - [Setting GITHUB\_TOKEN Permissions](#setting-github_token-permissions)
- [References](#references)

## Overview

This guide explains how to create, configure, and publish NuGet packages to GitHub Packages, both manually and automatically using GitHub Actions.

## Creating a NuGet Package

Add NuGet properties to your `*.csproj` file:

```xml
  <PropertyGroup>
    <IsPackable>True</IsPackable>
    <PackageId>Examples.Shared</PackageId>
    <Authors>akira suzuki</Authors>
    <Company>suzu-devworks</Company>
    <Version>1.0.0-alpha</Version>
    <Product>Examples</Product>
    <RepositoryUrl>https://github.com/suzu-devworks/examples-dotnet</RepositoryUrl>
    <PackageProjectUrl>https://github.com/suzu-devworks/examples-dotnet</PackageProjectUrl>
    <PackageTags>local;learning</PackageTags>
    <Description>
      Libraries for learning dotnet programming.
    </Description>
  </PropertyGroup>
```

Generate the package (`*.nupkg`):

```shell
dotnet pack --configuration Release
```

The package file will be output to the `bin/Release/` directory.

## Configuring GitHub Packages as a NuGet Source

You need a GitHub personal access token to publish packages.  
See: [Creating a personal access token](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

If you do not have `nuget.config`, create it:

```shell
dotnet new nugetconfig
```
<!-- spell-checker: words nugetconfig -->

Edit `nuget.config` to add the GitHub source:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="github" value="https://nuget.pkg.github.com/suzu-devworks/index.json" />
    <add key="nuget" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

## Publishing a NuGet Package to GitHub (Manual)

Use the personal access token as the API key:

```shell
dotnet nuget push "src/Examples.Shared/bin/Release/Examples.Shared.1.0.0-alpha.nupkg" --api-key YOUR_PERSONAL_ACCESS_TOKEN --source "github"
```

## Automatic Publishing with GitHub Actions

Create or edit `.github/workflows/dotnet-package.yml`:

```yaml
name: packaging

on:
  push:
    branches: [ "main" ]
    paths:
      - "src/**"
  pull_request:
    branches: [ "main" ]
    paths:
      - "src/**"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-dotnet@v5
        with:
          dotnet-version: '8.0.x'

      - name: Authenticate to GitHub Packages
        run: dotnet nuget update source github --username suzu-devworks --password ${{ secrets.GITHUB_TOKEN }} --store-password-in-clear-text

      - name: Install Tools
        run: dotnet tool restore

      - name: Restore dependencies
        run: dotnet restore --locked-mode

      - name: Build
        run: dotnet build --configuration Release --no-restore

      - name: Test
        run: dotnet test --configuration Release --no-build --verbosity normal

      - name: Pack
        run: dotnet pack --configuration Release --no-build -o artifacts/

      - name: Push
        run: |
          dotnet nuget update source github --username suzu-devworks --password ${{ secrets.GITHUB_TOKEN }} --store-password-in-clear-text
          dotnet nuget push artifacts/*.nupkg --source "github" --skip-duplicate
```

### Setting GITHUB_TOKEN Permissions

Set 'Read and write permissions' for workflows:

- Go to Repository settings > Actions > Workflow permissions
- Select "Read and write permissions"

## References

- [Create a NuGet package with the dotnet CLI | Microsoft Learn](https://learn.microsoft.com/ja-jp/nuget/create-packages/creating-a-package-dotnet-cli)
- [Package authoring best practices | Microsoft Learn](https://learn.microsoft.com/ja-jp/nuget/create-packages/package-authoring-best-practices)
- [Working with the NuGet registry - GitHub Docs](https://docs.github.com/ja/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
