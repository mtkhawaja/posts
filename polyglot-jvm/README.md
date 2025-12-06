# Polyglot JVM - Adding Multiple JVM Languages

Each language section is self-contained, allowing Groovy, Scala, Kotlin, or Clojure to be enabled. The emphasis is on concise, copy-oriented snippets covering compiler configuration, source layout,
and required dependencies.

While all languages can reside within one project structure, multi-module layouts are generally regarded as more maintainable over time. Clear separation reduces the likelihood of compiler plugin
interactions, improves build diagnostics, and allows language-specific dependencies or version changes to be introduced without affecting others. This guide, however, intentionally avoids multi-module
examples and focuses on the minimal configuration surfaces needed to activate each language within a single project.

In most development contexts, additional JVM languages are adopted incrementally rather than in bulk. As with any structural decision, selection depends on requirements: isolation is beneficial when
multiple languages are expected to coexist long-term, while a single consolidated project may be enough for evaluation, experimentation, or targeted language adoption.

A working multi-module reference (Java 25, Maven 4) with all languages enabled is available here: [polyglot-jvm](https://github.com/mtkhawaja/polyglot-jvm)

## Groovy Support

```plaintext
.
├── pom.xml
└── src
    ├── main
    │   └── groovy
    │       └── com
    │           └── example
    └── test
        └── groovy
            └── com
                └── example
```

```xml

<dependencyManagement>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.groovy/groovy-all -->
        <dependency>
            <groupId>org.apache.groovy</groupId>
            <artifactId>groovy-all</artifactId>
            <version>5.0.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```xml

<dependencies>
    <!-- https://mvnrepository.com/artifact/org.apache.groovy/groovy -->
    <!-- See https://repo1.maven.org/maven2/org/apache/groovy/groovy-all/5.0.3/groovy-all-5.0.3.pom for other groovy modules defined in groovy-all -->
    <dependency>
        <groupId>org.apache.groovy</groupId>
        <artifactId>groovy</artifactId>
    </dependency>
</dependencies>
```

```xml

<build>
    <plugins>
        <!-- https://mvnrepository.com/artifact/org.codehaus.gmavenplus/gmavenplus-plugin -->
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>4.2.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>addSources</goal>
                        <goal>addTestSources</goal>
                        <goal>generateStubs</goal>
                        <goal>compile</goal>
                        <goal>generateTestStubs</goal>
                        <goal>compileTests</goal>
                        <goal>removeStubs</goal>
                        <goal>removeTestStubs</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Groovy References

1. [groovy build tool support](https://groovy.apache.org/download.html#buildtools)
2. [GMavenPlus Plugin](https://github.com/groovy/GMavenPlus/wiki/Usage)

## Scala Support

```plaintext
.
├── pom.xml
└── src
    ├── main
    │   └── scala
    │       └── com
    │           └── example
    └── test
        └── scala
            └── com
                └── example
```

```xml

<properties>
    <scala.version>3.7.4</scala.version>
    <scala-maven-plugin.version>4.9.7</scala-maven-plugin.version>
</properties>
```

```xml
<!-- https://mvnrepository.com/artifact/org.scala-lang/scala3-library -->
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala3-library_3</artifactId>
    <version>${scala.version}</version>
</dependency>
```

```xml

<build>
    <sourceDirectory>src/main/scala</sourceDirectory>
    <testSourceDirectory>src/test/scala</testSourceDirectory>
    <plugins>
        <!-- https://mvnrepository.com/artifact/net.alchim31.maven/scala-maven-plugin -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>${scala-maven-plugin.version}</version>
            <configuration>
                <scalaVersion>${scala.version}</scalaVersion>
            </configuration>
            <executions>
                <execution>
                    <id>scala-compile-first</id>
                    <phase>process-resources</phase>
                    <goals>
                        <goal>add-source</goal>
                        <goal>compile</goal>
                    </goals>
                </execution>
                <execution>
                    <id>scala-test-compile</id>
                    <phase>process-test-resources</phase>
                    <goals>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Scala References

1. [Scala Maven Docs](https://docs.scala-lang.org/tutorials/scala-with-maven.html)
2. [Scala Maven Plugin Usage](https://davidb.github.io/scala-maven-plugin/usage.html)
3. [Scala Maven Plugin Examples](https://davidb.github.io/scala-maven-plugin/example_java.html)

## Kotlin Support

```plaintext
.
├── pom.xml
└── src
    ├── main
    │   └── kotlin
    │       └── com
    │           └── example
    └── test
        └── kotlin
            └── com
                └── example
```

```xml

<properties>
    <kotlin.version>2.2.21</kotlin.version>
</properties>
```

```xml

<dependencies>
    <!-- https://mvnrepository.com/artifact/org.jetbrains.kotlin/kotlin-stdlib -->
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>
```

```xml

<build>
    <sourceDirectory>src/main/kotlin</sourceDirectory>
    <testSourceDirectory>src/test/kotlin</testSourceDirectory>
    <plugins>
        <!-- https://mvnrepository.com/artifact/org.jetbrains.kotlin/kotlin-maven-plugin -->
        <plugin>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-plugin</artifactId>
            <version>${kotlin.version}</version>
            <executions>
                <execution>
                    <id>compile</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>compile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

    </plugins>
</build>
```

### Kotlin References

1. [Kotlin Maven Docs](https://kotlinlang.org/docs/maven.html)

## Clojure Support

```plaintext
.
├── pom.xml
└── src
    ├── main
    │   └── clojure
    │       └── com
    │           └── example
    └── test
        └── clojure
            └── com
                └── example
```

```xml

<dependencies>
    <!-- https://mvnrepository.com/artifact/org.clojure/clojure -->
    <dependency>
        <groupId>org.clojure</groupId>
        <artifactId>clojure</artifactId>
        <version>1.12.3</version>
    </dependency>
</dependencies>
```

```xml

<build>
    <sourceDirectory>src/main/clojure</sourceDirectory>
    <testSourceDirectory>src/test/clojure</testSourceDirectory>
    <plugins>
        <!-- https://github.com/talios/clojure-maven-plugin -->
        <plugin>
            <groupId>com.theoryinpractise</groupId>
            <artifactId>clojure-maven-plugin</artifactId>
            <version>1.8.3</version>
            <extensions>true</extensions>
            <configuration>
                <sourceDirectories>
                    <sourceDirectory>src/main/clojure</sourceDirectory>
                </sourceDirectories>
                <testSourceDirectories>
                    <testSourceDirectory>src/test/clojure</testSourceDirectory>
                </testSourceDirectories>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Clojure References

1. [Clojure Java Interop](https://clojure.org/reference/java_interop) and [Clojure FAQ Java Iterop](https://clojure.org/guides/faq#_java_and_interop)
2. [Clojure Maven Plugin](https://github.com/talios/clojure-maven-plugin)
3. [Clojure Interop Example](https://github.com/stuarthalloway/clojure-from-java)