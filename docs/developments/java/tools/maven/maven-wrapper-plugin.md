# Apache Maven Wrapper Plugin

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Integrating into an existing Maven project](#integrating-into-an-existing-maven-project)
- [Using a wrapper](#using-a-wrapper)

## Overview

This is a plugin that includes the Maven Wrapper (mvnw) command itself in your project. It was inspired by the Gradle Wrapper.

This plugin allows you to use Maven without having to install it locally.

- [Apache Maven Wrapper Plugin – Apache Maven Wrapper Plugin](https://maven.apache.org/wrapper/maven-wrapper-plugin/index.html)

## Integrating into an existing Maven project

```shell
mvn wrapper:wrapper
```

```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------< com.github.suzu_devworks.examples_java:examples-java >--------
[INFO] Building examples-java 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-wrapper-plugin:3.1.1:wrapper (default-cli) @ examples-java ---
[INFO] Unpacked bin type wrapper distribution org.apache.maven.wrapper:maven-wrapper-distribution:zip:bin:3.1.1
[INFO] Configuring .mvn/wrapper/maven-wrapper.properties to use Maven 3.8.6 and download from https://repo.maven.apache.org/maven2
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.840 s
[INFO] Finished at: 2022-12-11T13:08:29Z
[INFO] ------------------------------------------------------------------------
```

The resulting file tree:

```console
$ tree -a
.
├── .mvn
│   └── wrapper
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── github
    │               └── suzu_devworks
    │                   └── examples_java
    │                       └── App.java
    └── test
        └── java
            └── com
                └── github
                    └── suzu_devworks
                        └── examples_java
                            └── AppTest.java
```

## Using a wrapper

Check version is:

```shell
./mvnw --version
```

Clean the project:

```shell
./mvnw clean
```

Compile the project:

```shell
./mvnw compile
```

Generate artifacts such as JAR and WAR:

```shell
./mvnw package
```
