# pom.xml

POM stands for `Project Object Model`.

## Table of Contents <!-- omit in toc -->

- [Artifact Version](#artifact-version)
  - [DefaultArtifactVersion](#defaultartifactversion)
  - [Qualifiers](#qualifiers)
- [Super pom](#super-pom)

## Artifact Version

### DefaultArtifactVersion

Basically, you specify it as follows:

```console
<MajorVersion [> . <MinorVersion [> . <IncrementalVersion ] ] [> - <BuildNumber | Qualifier ]>
```

It is recommended to follow the [Semantic Version](https://semver.org/lang/ja/) in this regard.

The points to note here are:

> [!NOTE]
> Where MajorVersion, MinorVersion, IncrementalVersion, and BuildNumber are all numbers, and qualifiers are strings. If the version number does not match this format, the entire version number is treated as the qualifier.

In other words, it seems that 1.2.9.1 > 1.2.10.1.
It's just letters.

But I don't know if this has been improved in Maven 3...

### Qualifiers

Modifiers are appended to the version number using a dash and can be in upper or lower case, although the common convention is to use upper case.

| #   | Qualifier      | Examples                        | Description                                     |
| --- | -------------- | ------------------------------- | ----------------------------------------------- |
| 1   | alpha or a     | 1.2.3-alpha1, 1.2.3-alpha2, ... | Alpha version                                   |
| 2   | beta or b      | 1.2.3-beta1, 1.2.3-beta2, ...   | Beta version                                    |
| 3   | milestone or m | 1.2.3-m1, 1.2.3-m2, ...         | Milestone                                       |
| 4   | rc or cr       | 1.2.3-rc1, 1.2.3-rc2, ...       | Release Candidate                               |
| 5   | snapshot       | 1.2.3-SNAPSHOT                  | Informs users and Maven that this is not a stable release |
| 6   | ga or final    | 1.2.3                           | Same as without a modifier.                     |
| 7   | sp             | 1.2.3-sp1, 1.2.3-sp2, ...       | Bug fixes                                       |

The order seems to be as follows:

```console
1.0-beta1-SNAPSHOT
1.0-beta1
1.0-beta2-SNAPSHOT
1.0-rc1-SNAPSHOT
1.0-rc1
1.0-SNAPSHOT
1.0 or 1 (These are equal)
1.0-sp
1-whatever
1.0.1
```

This can be confirmed with code like this:

```java
import org.apache.maven.artifact.versioning.ComparableVersion;
import org.junit.Assert;
import org.junit.Test;

Assert.assertTrue(new ComparableVersion("1.0-SNAPSHOT").compareTo(new ComparableVersion("1.0")) < 0);
```

## Super pom

A POM that describes the default configuration that applies to all projects.

- <https://maven.apache.org/pom.html#The_Super_POM>
