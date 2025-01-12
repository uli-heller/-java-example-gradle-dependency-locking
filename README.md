Example Project Showing The Usage Of Gradle Dependency Locking
==============================================================

Introduction
------------

TBD

Problems
--------

### Updated Dependencies

See [discuss.gradle.org - Subproject downloads dependencies higher version than in lockfiles](https://discuss.gradle.org/t/subproject-downloads-dependencies-higher-version-than-in-lockfiles)
for more help. Please note that I do not use subprojects!

Basic idea:

- Specify version '3.+' for spring boot within the build file
- Do an initial build when 3.2.1 is active
- Create lock files based on this version
- Wait for a couple of months, now 3.4.1 has been released
- Executing the build without a lock file creates a new version using 3.4.1
- Executing the build with the lock file from 3.2.1
  - actually just creates an error message
  - should re-create the version based on 3.2.1 - how do I achieve this?

You can reproduce the error by doing this:

- `./gradlew dependencies --write-locks -PspringBootVersion='3.2.1'`
- `./gradlew build -PspringBootVersion='3.+'

For me, I do currently get this error:

```
uli@uliip5:~/git/github/uli-heller/java-example-gradle-dependency-locking$ ./gradlew build -PspringBootVersion='3.+'
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Could not resolve all files for configuration ':compileClasspath'.
   > Could not resolve org.springframework.boot:spring-boot-dependencies:3.4.1.
     Required by:
         root project :
      > Cannot find a version of 'org.springframework.boot:spring-boot-dependencies' that satisfies the version constraints:
           Dependency path 'com.example:java-example-gradle-dependency-locking:0.0.1-SNAPSHOT' --> 'org.springframework.boot:spring-boot-dependencies:3.4.1'
           Constraint path 'com.example:java-example-gradle-dependency-locking:0.0.1-SNAPSHOT' --> 'org.springframework.boot:spring-boot-dependencies:{strictly 3.2.1}' because of the following reason: dependency was locked to version '3.2.1'

> There is 1 more failure with an identical cause.

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
> Get more help at https://help.gradle.org.

BUILD FAILED in 533ms
1 actionable task: 1 executed
```
