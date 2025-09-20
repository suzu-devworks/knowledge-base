# ILSpy

ILSpy is the open-source .NET assembly browser and decompiler.

- [ILSpy](#ilspy)
  - [Install ilspycmd in Remote Container](#install-ilspycmd-in-remote-container)
    - [Clone the Repository](#clone-the-repository)
    - [Modify the Project File](#modify-the-project-file)
    - [Build the Project](#build-the-project)
    - [Add Local NuGet Source](#add-local-nuget-source)
    - [Install the Local Tool](#install-the-local-tool)
    - [Add Tool Directory to PATH (if needed)](#add-tool-directory-to-path-if-needed)
    - [Run ilspycmd](#run-ilspycmd)
  - [References](#references)

## Install ilspycmd in Remote Container
<!-- spell-checker: words ilspycmd -->

### Clone the Repository

```shell
# On container image is `mcr.microsoft.com/dotnet/sdk:5.0`

git clone https://github.com/icsharpcode/ILSpy.git

cd ILSpy
git submodule update --init --recursive
```

### Modify the Project File

- TargetFrameworks: Add .net5.0
- Version: Do not duplicate the real thing. (Add -debug ...)

```shell
cd ICSharpCode.Decompiler.Console
code ICSharpCode.Decompiler.Console.csproj
```

```xml:csproj
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <!-- TargetFrameworks>netcoreapp2.1;netcoreapp3.1</TargetFrameworks -->
    <TargetFrameworks>netcoreapp2.1;netcoreapp3.1;net5.0</TargetFrameworks>
    <ServerGarbageCollection>true</ServerGarbageCollection>
    ...
    <ToolCommandName>ilspycmd</ToolCommandName>
    <!--Version>7.1.0.6488-preview1</Version -->
    <Version>7.1.0.6488-debug</Version>
    <AssemblyVersion>7.1.0.0</AssemblyVersion>
    ...
```
<!-- spell-checker: words netcoreapp -->

### Build the Project

```shell
dotnet build
```

### Add Local NuGet Source

```shell
dotnet nuget add source %{PATH_TO_REPOS}/ILSpy/ICSharpCode.Decompiler.Console/bin/Debug/ --name ilspy-local

dotnet nuget list source
```
<!-- spell-checker: words ilspy -->

### Install the Local Tool

```shell
# specify version
dotnet tool install ilspycmd --global --version=7.1.0.6488-debug

dotnet tool list --global

```

### Add Tool Directory to PATH (if needed)

When you execute --global for the first time, an additional message to the PATH will be displayed, so take appropriate action.

```console
Tools directory '/home/vscode/.dotnet/tools' is not currently on the PATH environment variable.
If you are using bash, you can add it to your profile by running the following command:

cat << \EOF >> ~/.bash_profile
# Add .NET Core SDK tools
export PATH="$PATH:/home/vscode/.dotnet/tools"
EOF

You can add it to the current session by running the following command:

export PATH="$PATH:/home/vscode/.dotnet/tools"
```

### Run ilspycmd

```shell
ilspycmd --help

ilspycmd <assemblyfile>.dll -il -t Examples.Features.CS8.RangeAndIndicesTests | tee xxx.txt
```
<!-- spell-checker: words assemblyfile -->

## References

- <https://github.com/icsharpcode/ILSpy>
- <https://www.nuget.org/packages/ilspycmd/>
