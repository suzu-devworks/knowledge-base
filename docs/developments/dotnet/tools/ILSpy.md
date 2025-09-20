# ILSpy

ILSpy is an open-source .NET assembly browser and decompiler.

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Install ilspycmd in Remote Container](#install-ilspycmd-in-remote-container)
  - [Clone the Repository](#clone-the-repository)
  - [Modify the Project File](#modify-the-project-file)
  - [Build the Project](#build-the-project)
  - [Add Local NuGet Source](#add-local-nuget-source)
  - [Install the Local Tool](#install-the-local-tool)
  - [Add Tool Directory to PATH (if needed)](#add-tool-directory-to-path-if-needed)
  - [Run ilspycmd](#run-ilspycmd)
- [References](#references)

<!-- spell-checker: words ilspycmd -->

## Overview

This guide explains how to build and use `ilspycmd` in a remote container environment.

## Install ilspycmd in Remote Container

### Clone the Repository

```shell
# The container image should be `mcr.microsoft.com/dotnet/sdk:5.0`
git clone https://github.com/icsharpcode/ILSpy.git
cd ILSpy
git submodule update --init --recursive
```

### Modify the Project File

- Add `net5.0` to `TargetFrameworks`.
- Change `Version` to avoid conflicts with official releases (e.g., add `-debug`).

```shell
cd ICSharpCode.Decompiler.Console
code ICSharpCode.Decompiler.Console.csproj
```

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFrameworks>netcoreapp2.1;netcoreapp3.1;net5.0</TargetFrameworks>
    <ServerGarbageCollection>true</ServerGarbageCollection>
    <ToolCommandName>ilspycmd</ToolCommandName>
    <Version>7.1.0.6488-debug</Version>
    <AssemblyVersion>7.1.0.0</AssemblyVersion>
    ...
  </PropertyGroup>
</Project>
```
<!-- spell-checker: words netcoreapp -->

### Build the Project

```shell
dotnet build
```

### Add Local NuGet Source

```shell
dotnet nuget add source /path/to/ILSpy/ICSharpCode.Decompiler.Console/bin/Debug/ --name ilspy-local
dotnet nuget list source
```
<!-- spell-checker: words ilspy -->

### Install the Local Tool

```shell
dotnet tool install ilspycmd --global --version 7.1.0.6488-debug
dotnet tool list --global
```

### Add Tool Directory to PATH (if needed)

If you install a global tool for the first time, you may need to add the tools directory to your PATH.

```shell
# For bash, add to your profile:
cat << 'EOF' >> ~/.bash_profile
# Add .NET Core SDK tools
export PATH="$PATH:/home/vscode/.dotnet/tools"
EOF

# For the current session:
export PATH="$PATH:/home/vscode/.dotnet/tools"
```

### Run ilspycmd

```shell
ilspycmd --help
ilspycmd <assemblyFile>.dll -il -t Examples.Features.CS8.RangeAndIndicesTests | tee output.txt
```

## References

- [ILSpy GitHub Repository](https://github.com/icsharpcode/ILSpy)
- [ilspycmd NuGet Package](https://www.nuget.org/packages/ilspycmd/)
