# Development Environment

This article serves as a reference for my development environment setup.

## Terminal and Tools

I've written some ansible playbooks to configure my terminal and install all of my tools (zsh, tmux, nvim, java, js
etc.)

See the [dev-environment](https://github.com/mtkhawaja/dev-environment) repository on GitHub for more information on
running the playbooks or trying things out in docker.

## JetBrains

### JetBrains Toolbox App

Use the [JetBrains Toolbox App](https://www.jetbrains.com/toolbox-app) to install all JetBrains tools to ensure easy
automatic updates, rollbacks etc.

### Enable Settings Sync

Enable [settings sync](https://www.jetbrains.com/help/idea/sharing-your-ide-settings.html) to synchronize plugins,
keymaps, and other settings across multiple JetBrains tools.

### Plugins

- [.ignore](https://plugins.jetbrains.com/plugin/7495--ignore): Generate .gitignore files
- [Better Highlights](https://plugins.jetbrains.com/plugin/12895-better-highlights): Improved highlighting (comments,
  gutter icons, cognitive complexity etc.)
- [Extra ToolWindow Colorful Icons](https://plugins.jetbrains.com/plugin/16604-extra-toolwindow-colorful-icons): Adds
  colorful icons to the tool windows
- [Key Promoter X](https://plugins.jetbrains.com/plugin/9792-key-promoter-x): Yells at you if you don't use shortcuts.
- [One Dark Theme](https://plugins.jetbrains.com/plugin/11938-one-dark-theme): A dark theme that I like.
- [Rainbow Brackets](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets): Makes XML somewhat pleasant to look
  at.
- [String Manipulation](): A lot of string manipulation functions (Case switching, sorting, filtering, incrementing,
  aligning to columns, grepping, escaping, encoding, etc.)

### Show whitespace characters

Go to `Settings` → `Editor` → `General` → `Appearance` and enable **Show whitespaces**

### Show method separators

Go to `Settings` → `Editor` → `General` → `Appearance` and enable **Show method separators**

### Enable Font Ligatures

Go to `Settings` → `Editor` → `Font` and enable **Enable font ligatures**

### Live Templates

Live templates can be used to quickly insert code snippets, boilerplate, or other text into your code.

You can use live templates by typing the abbreviation for the template and pressing `Tab` to expand it.

To manage live templates, navigate to:

`Settings` → `Editor` → `Live Templates`

### IntelliJ IDEA (Java) Live Templates

By default, IDEA comes pre-configured with a number of live templates. (
See [List of Java live templates](https://www.jetbrains.com/help/fleet/live-templates-list-java.html) for more
information)

In addition to the default live templates, you can create your own custom live templates. Some examples of custom live
templates that I have created include the following:

#### Measure Elapsed Time for Selection

**Abbreviation**: ntm - (nano time measure)

> Surround with nanoSeconds based timer to measure elapsed time.

| Variable      | Expression  | 
|---------------|-------------|
| $START_TIME$  | "startTime" | 
| $ELAPSED$     | "elapsed"   |
| $CHRONO_UNIT$ | "SECONDS"   |

**Template Text**:

```text
final long $START_TIME$ = System.nanoTime();
try {
    $SELECTION$
} finally {
    final java.time.Duration $ELAPSED$ = Duration.ofNanos(System.nanoTime() - $START_TIME$);
    final java.time.temporal.ChronoUnit unit = java.time.temporal.ChronoUnit.$CHRONO_UNIT$;
    System.out.printf("Elapsed time: %d %s%n", elapsed.get(unit), unit);
}
```

**Example**:

```java
package org.example;

import java.time.Duration;
import java.time.temporal.ChronoUnit;

public class Example {
    public static void main(String[] args) {
        final long startTime = System.nanoTime();
        try {
            someOperation();
        } finally {
            final Duration elapsed = Duration.ofNanos(System.nanoTime() - startTime);
            final ChronoUnit unit = ChronoUnit.SECONDS;
            System.out.printf("Elapsed time: %d %s%n", elapsed.get(unit), unit);
        }
    }

    private static void someOperation() {
        try {
            Thread.sleep(1_000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

#### Add Logger(s)

##### SLF4J Logger

**Abbreviation**: slf - (Simple Logging Facade)

> Add a private static final SLF4J LOGGER field.

| Variable     | Expression           | 
|--------------|----------------------|
| $CLASS_NAME$ | qualifiedClassName() | 

**Template Text**:

```text
private static final org.slf4j.Logger LOGGER = org.slf4j.LoggerFactory.getLogger($CLASS_NAME$.class);
```

**Example**:

```java
package org.example;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example {
    private static final Logger LOGGER = LoggerFactory.getLogger(Example.class);
}
```

##### Log4J2 Logger

**Abbreviation**: l4j - (Logging For Java)

> Add a private static final LOG4J2 LOGGER field.

| Variable     | Expression           | 
|--------------|----------------------|
| $CLASS_NAME$ | qualifiedClassName() | 

**Template Text**:

```text
private static final org.apache.logging.log4j.Logger LOGGER = org.apache.logging.log4j.LogManager.getLogger($CLASS_NAME$.class);
```

**Example**:

```java
package org.example;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Example {
    private static final Logger LOGGER = LogManager.getLogger(Example.class);
}
```

## References

- [Using Live Templates - IntelliJ IDEA Documentation](https://www.jetbrains.com/help/idea/using-live-templates.html)