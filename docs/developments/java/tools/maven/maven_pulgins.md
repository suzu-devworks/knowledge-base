# Apache Maven Plugins

## Table of Contents <!-- omit in toc -->

- [maven-archetype-plugin](#maven-archetype-plugin)
  - [Create a project](#create-a-project)
- [maven-help-plugin](#maven-help-plugin)
  - [Print valid values, including default settings, from `settings.xml`](#print-valid-values-including-default-settings-from-settingsxml)
  - [Print valid values, including default settings from `pom.xml`](#print-valid-values-including-default-settings-from-pomxml)
- [maven-dependency-plugin](#maven-dependency-plugin)
  - [Project dependency tree output](#project-dependency-tree-output)
  - [Copy dependent libraries to a specific directory](#copy-dependent-libraries-to-a-specific-directory)

## maven-archetype-plugin

You can create a project from a Maven template called an `archetype`.

- <https://maven.apache.org/archetype/maven-archetype-plugin/index.html>

### Create a project

In most cases, `maven-archetype-quickstart` is sufficient.

```shell
$ mvn archetype:generate \
    -DgroupId=com.github.suzu_devworks.examples_java \
    -DartifactId=examples-java \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DarchetypeVersion=1.4
```

## maven-help-plugin

You can output various project information.

- <https://maven.apache.org/plugins/maven-help-plugin/index.html>

### Print valid values, including default settings, from `settings.xml`

```shell
mvn help:effective-settings -Doutput=effective-settings.xml
```

```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.example:restful-endpoint >--------------------
[INFO] Building Thorntail Example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- maven-help-plugin:3.2.0:effective-settings (default-cli) @ restful-endpoint ---
[INFO] Effective-settings written to: /home/akira/workspaces/repos/myApp/effective-settings.xml  # ← here!
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.825 s
[INFO] Finished at: 2020-08-14T17:11:11+09:00
[INFO] ------------------------------------------------------------------------
```

### Print valid values, including default settings from `pom.xml`

```shell
mvn help:effective-pom -Doutput=effective-pom.xml
```

```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.example:restful-endpoint >--------------------
[INFO] Building Thorntail Example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- maven-help-plugin:3.2.0:effective-pom (default-cli) @ restful-endpoint ---
[INFO] Effective-POM written to: /home/akira/workspaces/repos/myApp/effective-pom.xml # ← here!
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.756 s
[INFO] Finished at: 2020-08-14T17:12:36+09:00
[INFO] ------------------------------------------------------------------------
```

## maven-dependency-plugin

プロジェクトの依存関係の情報を収集、リモートのリポジトリから指定された場所にコピーおよびまたはアンパックできます。

- <https://maven.apache.org/plugins/maven-dependency-plugin/index.html>

### Project dependency tree output

```shell
mvn dependency:tree -DoutputFile=dependency.txt
```

```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.example:restful-endpoint >--------------------
[INFO] Building Thorntail Example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ restful-endpoint ---
[INFO] Wrote dependency tree to: /home/akira/workspaces/repos/myApp/dependency.txt # ← here!
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.468 s
[INFO] Finished at: 2020-08-14T17:22:35+09:00
[INFO] ------------------------------------------------------------------------
```

`dependency.txt`:

<!-- spell-checker: disable -->
```console
com.example:restful-endpoint:war:1.0.0-SNAPSHOT
+- io.thorntail:jaxrs:jar:2.7.0.Final:compile
|  +- io.thorntail:container:jar:2.7.0.Final:compile
|  |  +- io.thorntail:spi:jar:2.7.0.Final:compile
|  |  |  \- org.jboss:jandex:jar:2.1.2.Final:compile
|  |  +- io.thorntail:bootstrap:jar:2.7.0.Final:compile
|  |  +- io.thorntail:config-api:jar:2.7.0:compile
|  |  |  +- io.thorntail:config-api-runtime:jar:2.7.0:compile
|  |  |  \- org.wildfly.common:wildfly-common:jar:1.5.1.Final:compile
|  |  \- io.thorntail.jdk-specific:thorntail-jdk-specific:jar:2:compile
```
<!-- spell-checker: enable -->

### Copy dependent libraries to a specific directory

```shell
mvn dependency:copy-dependencies -DoutputDirectory=lib
```

<!-- spell-checker: disable -->
```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.example:restful-endpoint >--------------------
[INFO] Building Thorntail Example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:copy-dependencies (default-cli) @ restful-endpoint ---
[INFO] Copying staxmapper-1.3.0.Final.jar to /home/akira/workspaces/repos/myApp/lib/staxmapper-1.3.0.Final.jar
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.606 s
[INFO] Finished at: 2020-08-14T17:26:39+09:00
[INFO] ------------------------------------------------------------------------
```
<!-- spell-checker: enable -->
