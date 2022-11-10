+++
title = "Migrating from Java 11 to 17"
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

## Upgrade your libraries frequently
The migration was super easy because my team has a good dependency hygiene and we have already upgraded the troublesome dependencies which were described in the guide long before this Java 17 upgrade. This is something I will be following religiously for all my projects from here on out! I have worked on a lot of different projects, but never have we kept our libraries so up-to-date, as in this one. Yes it is sometimes annoying when a library brakes something, but in the long run the application is way more healthy this way.

## Maven tests issue
After the migration, the only issue that was left out of the article by Nikita was that my tests weren't running. We were using the `maven-surefire-plugin` to run our tests under maven when building the application. This somehow was causing all our tests to fail with the following error:

```java
Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test (default-test) on project Idlearn: Execution default-test of goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test failed: Unsupported class file major version 61 -> [Help 1]
```
Googling the error took me into a rabbit hole of getting the same response and that was "You are compiling the tests with java 17, but trying to run them under a different version". Checking my maven setup multiple times with `mvn -version`, to make sure I have the right maven version, and it is using the correct java, and my JAVA_HOME path is correct. But it seemed like it was correct, but it was not working. After some time spent googling and trying different approaches, I finally stumbled on [this stack overflow question and answer](https://stackoverflow.com/questions/71496063/maven-surefire-test-failed-unsupported-class-file-major-version-61). So it turns out the newest version (2.22.2) of `maven-surefire-plugin`, that is not a Milestone version, has not upgraded some of its dependencies related to ASM. So you have to add the dependency to it manually. *(TBH didn't even know I could do that with plugins)*

After this issue everything else went smoothly, all tests were green and the application was running!

## Honorable mentions

After the migration I decided to once again read [the article about what is new in Java 17 from 11](https://mydeveloperplanet.com/2021/09/28/whats-new-between-java-11-and-java-17/), but this time without skimming through it. Besides the new features there is also two very interesting things in the introduction.

### OracleJDK 17 is free?
> The Oracle licensing model has changed with the introduction of Java 17. Java 17 is issued under the new NFTC (Oracle No-Fee Terms and Conditions) license. It is therefore again allowed to use the Oracle JDK version for free for production and commercial use. 

So it turns out Oracle found out that hiding their JDK behind a paywall for production and commercial use, was such a huge mistake on their part, and they switched back to it being free. 
It was only natural for most companies to start using a free alternative, when there is one available. Whenever a library or software becomes paid, the first question from my manager would be "Is there a free alternative that does the same thing, and can be switched easily", and for JDK it was such an easy switch to OpenJDK.

### dev.java
I don't know how recent this is,but Oracle has created a cool little (lets hope only for now) [website](https://dev.java/). It has **tutorials** with topics ranging from basic java syntax to new java features. Also there is information about **how you can contribute to Java**, list of **communities you can participate in**, and information about **future java projects**.