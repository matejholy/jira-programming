---
layout: post
title:  "How to keep JIRA settings after cleaning a project"
description: "How to keep JIRA settings and log4j configuration after cleaning a project."
categories: main
tags: jira, server, amps, settings, keep, clean, maven, logging, log4j
comments: true
---

Normally, when you run `atlas-clean` command, you loose JIRA configuration and have to start over when you run your development instance again.
Luckily there's a way to keep both your JIRA configuration and log4j settings after cleaning the target directory.

*These instructions have been tested in JIRA 7.13.9.*

Even though it's a [documented feature](https://developer.atlassian.com/server/framework/atlassian-sdk/atlas-create-home-zip/) of Atlassian SDK, I learned about it in the [JIRA Development Cookbook](https://www.amazon.com/JIRA-Development-Cookbook-Jobin-Kuruvilla/dp/1785885618) by Jobin Kuruvilla. It's a great book to learn the basics when starting JIRA development.

Let's see how to keep JIRA settings first and then the log4j settings.

## Keeping JIRA settings

Here's what you need to do:
1. stop your JIRA instance, if it's running.
2. run `atlas-create-home-zip` in the root directory of your project. This creates a file named `generated-test-resources.zip` in `\target\jira\` subdirectory.
3. copy the `generated-test-resources.zip` file to `\src\test\resources`.
4. edit project's `pom.xml` and add the following line to the `jira-maven-plugin` (or `maven-jira-plugin` depending on your AMPS version) configuration section:
      `<productDataPath>${project.basedir}/src/test/resources/generated-test-resources.zip</productDataPath>`

Here's how the plugin configuration looks like in `pom.xml`:
{% highlight xml %}
<build>
    <plugins>
        <plugin>
            <groupId>com.atlassian.maven.plugins</groupId>
            <artifactId>jira-maven-plugin</artifactId>
            <version>${amps.version}</version>
            <extensions>true</extensions>
            <configuration>
                <productVersion>${jira.version}</productVersion>
                <productDataVersion>${jira.version}</productDataVersion>
                <productDataPath>${project.basedir}/src/test/resources/generated-test-resources.zip</productDataPath>
                <!-- ... -->
            </configuration>
        </plugin>
    </plugins>
</build>
{% endhighlight %}

Any time you do any configuration changes to JIRA, just generate a new `generated-test-resources.zip` file and copy it to the `\src\test\resources` directory.

## Keeping log4j settings

Keeping log4j settings is similar:
1. run your JIRA development instance at least once. A `log4j.properties` file will be generated in `\target\jira\webapp\WEB-INF\classes\` directory.
2. copy the file into `\src\test\resources` directory.
3. make necessary changes to the `log4j.properties` in your `test\resources` directory. For example, to enable *DEBUG* level logging to both console and log file for `com.test.jira.plugins` package, add the following lines at the end of the file:

      `log4j.logger.com.test.jira.plugins = DEBUG, console, filelog`
      `log4j.additivity.com.test.jira.plugins = false`
4. edit project's `pom.xml` and add the following line to the `jira-maven-plugin` (or `maven-jira-plugin` depending on your AMPS version) configuration section:
      `<log4jProperties>${project.basedir}/src/test/resources/log4j.properties</log4jProperties>`

Here's how the plugin configuration looks like in `pom.xml`, including the `productDataPath` configuration from previous section:
{% highlight xml %}
<build>
    <plugins>
        <plugin>
            <groupId>com.atlassian.maven.plugins</groupId>
            <artifactId>jira-maven-plugin</artifactId>
            <version>${amps.version}</version>
            <extensions>true</extensions>
            <configuration>
                <productVersion>${jira.version}</productVersion>
                <productDataVersion>${jira.version}</productDataVersion>
                <productDataPath>${project.basedir}/src/test/resources/generated-test-resources.zip</productDataPath>
                <log4jProperties>${project.basedir}/src/test/resources/log4j.properties</log4jProperties>
                <!-- ... -->
            </configuration>
        </plugin>
    </plugins>
</build>
{% endhighlight %}

And that's it. This is one of the first things I do when creating a new project and setting up JIRA after the first execution.
