# pom.xml settings

## Table of Contents <!-- omit in toc -->

- [Sets JDK version](#sets-jdk-version)
- [Sets source encoding](#sets-source-encoding)
- [Sets packaging](#sets-packaging)
- [Skip Tests](#skip-tests)
- [Replace `${version}` in the resource folder](#replace-version-in-the-resource-folder)
- [web.xml is missing and `<failOnMissingWebXml>` is set to true](#webxml-is-missing-and-failonmissingwebxml-is-set-to-true)

## Sets JDK version

```diff
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

  <properties>
    <!-- Generic properties -->
+   <java.version>17</java.version>
+   <maven.compiler.source>${java.version}</maven.compiler.source>
+   <maven.compiler.target>${java.version}</maven.compiler.target>
  </properties>

</project>
```

## Sets source encoding

```diff
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

  <properties>
+   <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
+   <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>

</project>
```

## Sets packaging

Jar or War

```diff
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

<groupId>jp.examples</groupId>
<artifactId>app</artifactId>
<version>1.0-SNAPSHOT</version>

- <packaging>jar</packaging>
+ <packaging>war</packaging>

</project>
```

## Skip Tests

```diff
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

  <properties>
+    <maven.test.skip>false</maven.test.skip>
+    <skipTests>true</skipTests>
  </properties>

</project>
```

- `maven.test.skip = true`:  
   Skip compiling test code and running tests.

- `skipTests = true`:  
   Compile test code but skip test execution

## Replace `${version}` in the resource folder

If you enable filtering in the maven resource plugin, you can replace the ${x} you set in the properties in the resource folder.

- <https://qiita.com/kozy4324/items/9fa17a98203761012fd9>

```diff
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

  <build>
+   <resources>
+     <resource>
+       <directory>src/main/resources</directory>
+       <filtering>true</filtering>
+     </resource>
+   </resources>
    ...
  </build>

</project>
```

## web.xml is missing and `<failOnMissingWebXml>` is set to true

Although the web.xml file is optional for modern web applications, Maven web applications require a web.xml file.

```diff
  <properties>
+   <failOnMissingWebXml>false</failOnMissingWebXml>
  </properties>
```
