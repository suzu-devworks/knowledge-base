# License Maven Plugin

## Table of Contents <!-- omit in toc -->

- [license-maven-plugin](#license-maven-plugin-1)
  - [Show available licenses](#show-available-licenses)
  - [Reporting on third-party licenses](#reporting-on-third-party-licenses)

## license-maven-plugin

Manage your project's licenses and their dependencies.

- <https://www.mojohaus.org/license-maven-plugin/index.html>

### Show available licenses

```sh
mvn license:license-list
```

```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------< com.github.suzu_devworks.examples_java:examples-java >--------
[INFO] Building examples-java 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- license-maven-plugin:2.0.0:license-list (default-cli) @ examples-java ---
[INFO] Adding a license repository jar:file:/home/vscode/.m2/repository/org/codehaus/mojo/license-maven-plugin/2.0.0/license-maven-plugin-2.0.0.jar!/META-INF/licenses
[INFO] register GNU Free Documentation License (FDL) version 1.3
[INFO] register GNU General Lesser Public License (LGPL) version 3.0
[INFO] register GNU Affero General Public License (AGPL) version 3.0
[INFO] register GNU General Public License (GPL) version 3.0
[INFO] register GNU General Public License (GPL) version 2.0
[INFO] register GNU General Public License (GPL) version 1.0
[INFO] register Apache License version 2.0
[INFO] register Eclipse Public License - v 2.0 with Secondary License
[INFO] register Eclipse Public License - v 2.0
[INFO] register Eclipse Public License - v 1.0
[INFO] register Eclipse Public + Distribution License - v 1.0
[INFO] register COMMON DEVELOPMENT AND DISTRIBUTION LICENSE (CDDL) Version 1.0
[INFO] register GNU General Lesser Public License (LGPL) version 2.1
[INFO] register MIT-License
[INFO] register BSD 2-Clause License
[INFO] register BSD 3-Clause License
[INFO] register European Union Public License v1.1
[INFO] Available licenses :

 * agpl_v3     : GNU Affero General Public License (AGPL) version 3.0
 * apache_v2   : Apache License version 2.0
 * bsd_2       : BSD 2-Clause License
 * bsd_3       : BSD 3-Clause License
 * cddl_v1     : COMMON DEVELOPMENT AND DISTRIBUTION LICENSE (CDDL) Version 1.0
 * epl_only_v1 : Eclipse Public License - v 1.0
 * epl_only_v2 : Eclipse Public License - v 2.0
 * epl_v1      : Eclipse Public + Distribution License - v 1.0
 * epl_v2      : Eclipse Public License - v 2.0 with Secondary License
 * eupl_v1_1   : European Union Public License v1.1
 * fdl_v1_3    : GNU Free Documentation License (FDL) version 1.3
 * gpl_v1      : GNU General Public License (GPL) version 1.0
 * gpl_v2      : GNU General Public License (GPL) version 2.0
 * gpl_v3      : GNU General Public License (GPL) version 3.0
 * lgpl_v2_1   : GNU General Lesser Public License (LGPL) version 2.1
 * lgpl_v3     : GNU General Lesser Public License (LGPL) version 3.0
 * mit         : MIT-License

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.840 s
[INFO] Finished at: 2022-12-11T12:39:58Z
[INFO] ------------------------------------------------------------------------
```
<!-- spell-checker: words codehaus -->

### Reporting on third-party licenses

```sh
$ mvn license:third-party-report

# exclude `test`
$ mvn license:add-third-party -Dlicense.excludedScopes=test
```
<!-- spell-checker: words Dlicense -->

```console
[INFO] Scanning for projects...
[INFO]
[INFO] --------< com.github.suzu_devworks.examples_java:examples-java >--------
[INFO] Building examples-java 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- license-maven-plugin:2.0.0:add-third-party (default-cli) @ examples-java ---
[INFO] Writing third-party file to /workspace/examples-java/modules/examples-java/target/generated-sources/license/THIRD-PARTY.txt # ← here!
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.670 s
[INFO] Finished at: 2022-12-11T12:36:42Z
[INFO] ------------------------------------------------------------------------
```

`target/generated-sources/license/THIRD-PARTY.txt`:

<!-- spell-checker: disable -->
```console
Lists of 11 third-party dependencies.
     (Apache License, Version 2.0) Byte Buddy (without dependencies) (net.bytebuddy:byte-buddy:1.12.10 - https://bytebuddy.net/byte-buddy)
     (The Apache License, Version 2.0) org.apiguardian:apiguardian-api (org.apiguardian:apiguardian-api:1.1.2 - https://github.com/apiguardian-team/apiguardian)
     (Apache License, Version 2.0) AssertJ fluent assertions (org.assertj:assertj-core:3.23.1 - ${project.parent.url}#assertj-core)
     (Eclipse Public License v2.0) JUnit Jupiter (Aggregator) (org.junit.jupiter:junit-jupiter:5.9.1 - https://junit.org/junit5/)
     (Eclipse Public License v2.0) JUnit Jupiter API (org.junit.jupiter:junit-jupiter-api:5.9.1 - https://junit.org/junit5/)
     (Eclipse Public License v2.0) JUnit Jupiter Engine (org.junit.jupiter:junit-jupiter-engine:5.9.1 - https://junit.org/junit5/)
     (Eclipse Public License v2.0) JUnit Jupiter Params (org.junit.jupiter:junit-jupiter-params:5.9.1 - https://junit.org/junit5/)
     (Eclipse Public License v2.0) JUnit Platform Commons (org.junit.platform:junit-platform-commons:1.9.1 - https://junit.org/junit5/)
     (Eclipse Public License v2.0) JUnit Platform Engine API (org.junit.platform:junit-platform-engine:1.9.1 - https://junit.org/junit5/)
     (The Apache License, Version 2.0) org.opentest4j:opentest4j (org.opentest4j:opentest4j:1.2.0 - https://github.com/ota4j-team/opentest4j)
     (The MIT License) Project Lombok (org.projectlombok:lombok:1.18.24 - https://projectlombok.org)
```
<!-- spell-checker: enable -->
