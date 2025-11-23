# Java Logging

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [The three parts of logging](#the-three-parts-of-logging)
  - [Log Facade](#log-facade)
  - [Logging Framework (Implementations)](#logging-framework-implementations)
  - [Logging Adapter (Bridge)](#logging-adapter-bridge)
- [References](#references)

## Overview

Java logging varies according to different social situations and preferences.

- Older: Commons-logging + Log4j
- A little old: SLF4J + Logback
- Recent: ?????
- Future: ?????

## The three parts of logging

### Log Facade

A facade gives your application a single logging API so you can bind any logging implementation at runtime.

- [commons-logging](https://commons.apache.org/proper/commons-logging/)
- [SLF4J](https://www.slf4j.org/)

### Logging Framework (Implementations)

The actual logging backends and implementations.

- [java.util.logging](https://docs.oracle.com/javase/jp/8/docs/api/java/util/logging/package-summary.html)
- [Logback](https://logback.qos.ch/)
- [Log4j2](https://logging.apache.org/log4j/2.x/)
- [JEP 264: Platform Logging API (Java 9)](https://docs.oracle.com/javase/jp/9/docs/api/java/lang/System.Logger.html)

### Logging Adapter (Bridge)

Adapters translate between facades and implementations. Use them when you cannot change a third-party JAR that depends on an older logging API.

- `jcl-over-slf4j.jar` = Apache Commons Logging -> SLF4J
- `jul-to-slf4j.jar` = java.util.logging -> SLF4J
- `log4j-over-slf4j.jar` = Log4j -> SLF4J

## References

- [Java Logging Frameworks: Summary and Best Practices](https://www.alibabacloud.com/blog/java-logging-frameworks-summary-and-best-practices_598223)
- [Organizing SLF4J, Logback, and Log4J with behavior examples](https://qiita.com/NagaokaKenichi/items/9febd2e559331152fcf8)
- [About Java 9 logging (future-looking)](https://blog1.mammb.com/entry/2022/08/15/000000)
- [(personal blog link)](https://blog.kengo-toda.jp/entry/2021/05/31/200807)
