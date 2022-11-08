+++
title = "Java 11 to 17 Upgrade"
date = "2022-11-08T19:01:49+02:00"
author = "Dimitar Tomov"
tags = ["java", "experience"]
keywords = ["java-11", "java-17", "migration"]
description = "Thoughts of migrating an application from Java 11 to Java 17"
showFullContent = false
readingTime = false
hideComments = false
+++

Today I had to migrate our application from Java 11 to Java 17. I think I should get into the habit of always googling stuff and not just thinking "Yeah I know how to do it, I am going to be fine". This time I did google what issues might come from the upgrade and thankfully not too many. I followed [this guide by Nikita Zemnitsky @ NIX](https://dev.to/nix/migrating-a-project-from-java-11-to-java-17-a-step-by-step-guide-for-developers-42n), which does a pretty good job of explaining what issues might come up, so I am not going to list them here again.

### Upgrade your libraries frequently
The migration was super easy because my team has a good dependency hygiene and we have already upgraded the troublesome dependencies which were described in the guide long before this Java 17 upgrade. This is something I will be following religiously for all my projects from here on out! I have worked on a lot of different projects, but never have we kept our libraries so up-to-date, as in this one. Yes it is sometimes annoying when a library brakes something, but in the long run the application is way more healthy this way.

### Maven tests issue
After the migration, the only issue that was left out of the article by Nikita was that my tests weren't running. We were using the `maven-surefire-plugin` to run our tests under maven when building the application. This somehow was causing all our tests to fail with the following error:

```java
Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test (default-test) on project Idlearn: Execution default-test of goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test failed: Unsupported class file major version 61 -> [Help 1]
```
Googling the error took me into a rabbit hole of getting the same response and that was "You are compiling the tests with java 17, but trying to run them under a different version". Checking my maven setup multiple times with `mvn -version`, to make sure I have the right maven version, and it is using the correct java, and my JAVA_HOME path is correct. But it seemed like it was correct, but it was not working. After some time spent googling and trying different approaches, I finally stumbled on [this stack overflow question and answer](https://stackoverflow.com/questions/71496063/maven-surefire-test-failed-unsupported-class-file-major-version-61). So it turns out the newest version (2.22.2) of `maven-surefire-plugin`, that is not a Milestone version, has not upgraded some of its dependencies related to ASM. So you have to add the dependency to it manually. *(TBH didn't even know I could do that with plugins)*

After this issue everything else went smoothly, all tests were green and the application was running!