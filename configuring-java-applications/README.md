# Configuring Java Applications

A quick reference for configuring Java applications.

**Note**: maven & Java 17 or higher are assumed. 

## Operating System Environment Variables

You can name your environment variables however you want, but they are typically in [screaming snake case](https://en.wiktionary.org/wiki/screaming_snake_case) by convention (and in [certain style guides](https://google.github.io/styleguide/shellguide.html#constants-and-environment-variable-names)).

### Using export

````bash
#!/usr/bin/env bash

export SERVER_PORT=8080
java -jar app.jar
````

**Note**: Using the [export](https://www.man7.org/linux/man-pages/man1/export.1p.html) keyword makes environment variables available for **all** child processes.

### Using env

````bash
#!/usr/bin/env bash

env SERVER_PORT=8080 \ 
    APPLICATION_NAME=sample \ 
    FOO=BAR \
    java -jar app.jar
````

The benefit of using the [env](https://man7.org/linux/man-pages/man1/env.1.html) command is that it will set the environment only for the duration of the "java" command. This means that any other processes running in the same shell will not see these variables.

### Accessing Environment Variables using Java

From the Java side of things, you can access environment variables using [System.getenv()](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html#getenv()) or [getenv(java.lang.String)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html#getenv(java.lang.String))

```java
public class EnvironmentVariablesExample {
    public static void main(String... ignore) {
        Map<String, String> environmentVariables = System.getenv();
        // Do something with environmentVariables
    }
}
```

**Note**: While it is trivial to read environment variables, there isn't a straightforward way to update or add environment variables in Java unless you [spawn a subprocess](https://docs.oracle.com/javase/tutorial/essential/environment/env.html)

## System Properties

| Prefix       | Description                                                                                    | Documentation                                                                                                                                                       | Example                            |
|--------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------|
| -<No Prefix> | Standard JVM properties supported by most JVM implementations.                                 | [Standard options for Java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#standard-options-for-java)                                           | `-verbose`                         |
| -D           | User-defined system properties.                                                                | Hopefully you're using something like Spring's [Configuration Metadata](https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html) | `-Dserver.port`                    | 
| -X           | General purpose options that are specific to the Java HotSpot Virtual Machine.                 | [Extra Options for Java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#extra-options-for-java)                                                 | `-Xcheck:jni`                      |
| -XX          | Advanced runtime options, JIT options, Serviceability options, and garbage collection options. | [Advanced Options for Java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#advanced-options-for-java)                                           | `-XX:+UnlockExperimentalVMOptions` |

**Note**: Refer to the [Java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html) command documentation for your version of Java and your JVM vendor documentation (e.g. [Azul](https://docs.azul.com/prime/Command-Line-Options)) for more information about what options are available.

Considering user-defined properties system properties, you'd supply them as follows:

```bash
java -Dserver.port=8080 -jar app.jar
```

### Setting system properties via Environment Variables

| Environment Variable | Documentation Reference                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `JDK_JAVA_OPTIONS`   | [Using the JDK_JAVA_OPTIONS Launcher Environment Variable](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#using-the-jdk_java_options-launcher-environment-variable)                                                                                                                                                                                                                                                                                                                                                                                         |
| `JAVA_TOOL_OPTIONS`  | [JAVA_TOOL_OPTIONS](https://docs.oracle.com/en/java/javase/17/docs/specs/jvmti.html#tooloptions)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `JAVA_OPTS`          | Not a standard environment variable and won't take effect automatically. Despite that, you'll find it used all over the place e.g. [Jenkins](https://docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-masters/how-to-add-java-arguments-to-jenkins#_running_jenkins_inside_docker), various application servers like [Tomcat](https://github.com/apache/tomcat/blob/main/bin/catalina.sh#L65), [JBoss](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html/configuration_guide/configuring_jvm_settings) etc. |

For example, instead of passing system properties directly, you can configure them via the `JDK_JAVA_OPTIONS` environment variable:

```bash
export JDK_JAVA_OPTIONS="-Dserver.port=value"
java -jar app.jar
```

### Accessing System Properties using Java

From the Java side of things, system properties can be accessed via [System.getProperties()](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html#getProperties()) or [System.getProperty(java.lang.String)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html#getProperty(java.lang.String)):

```java
public class SystemPropertiesExample {
    public static void main(String... ignore) {
        Properties systemProperties = System.getProperties();
        // Do something with systemProperties
    }
}
```

## Command Line Arguments

Passed at runtime to your application e.g.

```bash
#!/usr/bin/env bash

java -jar app.jar "server.port=8080" "application.name=foo" "service.apiKey=bar"

```

### Accessing System Properties using Java

From the Java side of things, the CLI arguments can be accessed from the main method:

```java
public class CommandLineArgumentsExample {
    public static void main(String... commandLineArguments) {
        // Do something with commandLineArguments
    }
}
```

## File Based Configuration

Configuration files can be:

1. Bundled with the application as a [classpath resource](https://docs.oracle.com/javase/8/docs/technotes/guides/lang/resources.html).
2. Looked up from a well-known location e.g. in the user's home directory.
3. Specified at runtime using CLI Args, System Properties, or Environment Variables.

[Properties](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Properties.html) files are a standard way to define configuration for Java applications with built-in support. 


### Parsing other configuration formats

Including properties files, the [jackson-dataformats-text](https://github.com/FasterXML/jackson-dataformats-text) libraries can be used to parse other formats such as XML, JSON, YAML/YML, INI/TOML, CSV etc. 

Include the dependency for your desired format in your POM:

```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-annotations -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-yaml -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-yaml</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-toml -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-toml</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-csv -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-csv</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-properties -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-properties</artifactId>
        </dependency>   
</dependencies>
```

Assuming that you have the following Java class modelling your configuration:

```java
class ApplicationConfiguration {
    public String environment;
    public Metadata metadata;
    public DatabaseConfiguration databaseConfiguration;

    record Metadata(String name, String version) {
    }

    record DatabaseConfiguration(String host,
                                 String port,
                                 String database,
                                 String username,
                                 String password,
                                 String jdbcUrl) {
    }

}
```

And that the following YAML file represents your configuration:

```yaml
environment: "learning"
metadata:
  name: "My Application"
  version: "1.0.0"
databaseConfiguration:
  host: "localhost"
  port: "3306"
  database: "my_database"
  username: "service-account"
  password: "something-hopefully-strong"
  jdbcUrl: "jdbc:mysql://localhost:3306/my_database"
```

Then you can read it from the file system as follows:

```java
import com.fasterxml.jackson.dataformat.yaml.YAMLMapper;

import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Objects;

class ReadFromFileSystem {


    public static void main(String... commandLineArguments) throws Exception {
        final var configurationFilePath = Paths.get(System.getProperty("user.home"), "config.yaml");
        // or: ObjectMapper, JavaPropsMapper, TomlMapper, XmlMapper, CsvMapper
        final var mapper = new YAMLMapper();
        try (var inputStream = Files.newInputStream(configurationFilePath)) {
            Objects.requireNonNull(inputStream, configurationFilePath + " not found!");
            var configuration = mapper.readValue(inputStream, ApplicationConfiguration.class);
            // do something with configuration
        }
    }

}

```

Alternatively, if it is available on the classpath, you can read it as follows:

```java
import com.fasterxml.jackson.dataformat.yaml.YAMLMapper;

import java.util.Objects;

class ReadFromClasspath {


    public static void main(String... commandLineArguments) throws Exception {
        final var classpathResource = "config.yaml";
        // or: ObjectMapper, JavaPropsMapper, TomlMapper, XmlMapper, CsvMapper
        final var mapper = new YAMLMapper();
        try (var inputStream = ReadFromClasspath.class.getClassLoader().getResourceAsStream(classpathResource)) {
            Objects.requireNonNull(inputStream, classpathResource + " not found!");
            var configuration = mapper.readValue(inputStream, ApplicationConfiguration.class);
            // do something with configuration
        }
    }
}
```


## References / Additional Reading

- "Screaming Snake Case", Wiktionary, [en.wiktionary.org/wiki/screaming_snake_case](https://en.wiktionary.org/wiki/screaming_snake_case)
- “Shell Style Guide.”, Google, [google.github.io/styleguide/shellguide.html#constants-and-environment-variable-names](https://google.github.io/styleguide/shellguide.html#constants-and-environment-variable-names)
- "Export(1P)", Linux Manual Page, [www.man7.org/linux/man-pages/man1/export.1p.html](https://www.man7.org/linux/man-pages/man1/export.1p.html)
- "Env(1)", Linux Manual Page, [man7.org/linux/man-pages/man1/env.1.html](https://man7.org/linux/man-pages/man1/env.1.html)
- “Java Development Kit Version 17 API Specification.”, Oracle, [docs.oracle.com/en/java/javase/17/docs/api/index.html](https://docs.oracle.com/en/java/javase/17/docs/api/index.html)
- “Environment Variables (The Java&trade; Tutorials &gt; Essential Java Classes &gt; The Platform Environment)”, Oracle, [docs.oracle.com/javase/tutorial/essential/environment/env.html](https://docs.oracle.com/javase/tutorial/essential/environment/env.html)
- "Standard options for Java", Oracle, [docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#standard-options-for-java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#standard-options-for-java)
- "Configuration Metadata", SpringBoot, [docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html](https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html)
- "Extra Options for Java", Oracle, [docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#extra-options-for-java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#extra-options-for-java)
- "Advanced Options for Java", Oracle, [/docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#advanced-options-for-java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#advanced-options-for-java)
- "java command", Oracle, [java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html)
- "Command-Line Options", Azul, [docs.azul.com/prime/Command-Line-Options](https://docs.azul.com/prime/Command-Line-Options)
- "Using the JDK_JAVA_OPTIONS Launcher Environment Variable", Oracle, [docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#using-the-jdk_java_options-launcher-environment-variable](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#using-the-jdk_java_options-launcher-environment-variable)
- "JAVA_TOOL_OPTIONS", Oracle, [docs.oracle.com/en/java/javase/17/docs/specs/jvmti.html#tooloptions](https://docs.oracle.com/en/java/javase/17/docs/specs/jvmti.html#tooloptions)
- "How to add java arguments to Jenkins", Cloudbees, [docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-masters/how-to-add-java-arguments-to-jenkins#_running_jenkins_inside_docker](https://docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-masters/how-to-add-java-arguments-to-jenkins#_running_jenkins_inside_docker)
- "catalina.sh", Tomcat, [github.com/apache/tomcat/blob/main/bin/catalina.sh#L65](https://github.com/apache/tomcat/blob/main/bin/catalina.sh#L65)
- "Configuring JVM Settings", JBoss, [access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html/configuration_guide/configuring_jvm_settings](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html/configuration_guide/configuring_jvm_settings)
- “Jackson Dataformats Text”, FasterXML, [github.com/FasterXML/jackson-dataformats-text](https://github.com/FasterXML/jackson-dataformats-text)
- "Why do JVM arguments start with "-D"?", StackOverflow, [stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d](https://stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d)
- "How do I use the JAVA_OPTS environment variable?", StackOverflow, [stackoverflow.com/questions/5241743/how-do-i-use-the-java-opts-environment-variable](https://stackoverflow.com/questions/5241743/how-do-i-use-the-java-opts-environment-variable)
- "That's a lot of YAML", ghuntley, [noyaml.com](https://noyaml.com)
- "Using the JDK_JAVA_OPTIONS Launcher Environment Variable", Oracle, [docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#using-the-jdk_java_options-launcher-environment-variable](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#using-the-jdk_java_options-launcher-environment-variable)
- "Properties", Oracle, [docs.oracle.com/javase/tutorial/essential/environment/properties.html](https://docs.oracle.com/javase/tutorial/essential/environment/properties.html)
- "System Properties", Oracle, [docs.oracle.com/javase/tutorial/essential/environment/sysprop.html](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)
- "A Complete Guide To The Bash Environment Variables", [www.shell-tips.com/bash/environment-variables/#gsc.tab=0](https://www.shell-tips.com/bash/environment-variables/#gsc.tab=0)