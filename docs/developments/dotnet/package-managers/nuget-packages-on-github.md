# NuGet packages on GitHub

- [NuGet packages on GitHub](#nuget-packages-on-github)
  - [Creating a NuGet Package](#creating-a-nuget-package)
  - [Configuring GitHub Packages as a NuGet Source](#configuring-github-packages-as-a-nuget-source)
  - [Publishing a NuGet Package to GitHub (Manual)](#publishing-a-nuget-package-to-github-manual)
  - [Automatic Publishing with GitHub Actions](#automatic-publishing-with-github-actions)
  - [References](#references)

## Creating a NuGet Package

Add nuget property in `*.csproj`

```xml
  <PropertyGroup>
    <!-- Properties related to NuGet packaging: -->
    <IsPackable>True</IsPackable>
    <PackageId>SleighBells.Examples.Shared</PackageId>
    <Authors>akira suzuki</Authors>
    <Company>suzu-devworks</Company>
    <Version>1.0.0-alpha</Version>
    <Product>SleighBells.Examples</Product>
    <RepositoryUrl>https://github.com/suzu-devworks/examples-dotnet</RepositoryUrl>
    <PackageProjectUrl>https://github.com/suzu-devworks/examples-dotnet</PackageProjectUrl>
    <PackageTags>local;learning</PackageTags>
    <Description>
      Libraries for learning dotnet programming.
    </Description>
  </PropertyGroup>
```

Execute package generation (`\*.nupkg`)

```shell
dotnet pack --configuration Release
```

The package file will be output to the `bin/Release/` directory.

## Configuring GitHub Packages as a NuGet Source

Creating Github personal access token (classic)

- [Creating a personal access token](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

Edit `nuget.config`

If you already have nuget.config, just edit it. Otherwise, create a new one.

```shell
dotnet new nugetconfig
```
<!-- spell-checker: words nugetconfig -->

```diff
 <?xml version="1.0" encoding="utf-8"?>
 <configuration>
   <packageSources>
     <!--To inherit the global NuGet package sources remove the <clear/> line below -->
     <clear />
+    <add key="github" value="https://nuget.pkg.github.com/suzu-devworks/index.json" />
     <add key="nuget" value="https://api.nuget.org/v3/index.json" />
   </packageSources>
 </configuration>
```

## Publishing a NuGet Package to GitHub (Manual)

```shell
dotnet nuget push "src/Examples.Core/bin/Release/SW.Examples.Core.1.0.0-alpha.nupkg"  --api-key YOUR_PERSONAL_ACCESS_TOKEN --source "github"
```

## Automatic Publishing with GitHub Actions

Edit `.github/workflows/dotnet-package.yml`

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

permissions: {}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-dotnet@v5
        with:
           dotnet-version: '8.0.x'

      - name: Auth Github
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

Setting `GITHUB_TOKEN`.
Set 'Read and write permissions' for workflows:

- Repository settings > Actions > Workflow permissions
- Select "Read and write permissions"

## References

- [Create a NuGet package with the dotnet CLI | Microsoft Learn](https://learn.microsoft.com/ja-jp/nuget/create-packages/creating-a-package-dotnet-cli)
- [Package authoring best practices | Microsoft Learn](https://learn.microsoft.com/ja-jp/nuget/create-packages/package-authoring-best-practices)
- [Working with the NuGet registry - GitHub Docs](https://docs.github.com/ja/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
