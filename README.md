Example Project Showing The Usage Of Gradle Dependency Locking
==============================================================

Introduction
------------

TBD

Problems
--------

### Updated Dependencies

#### Problem Description

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

Note: I compared the lock files created by

- `./gradlew dependencies --write-locks -PspringBootVersion='3.4.1' (3.4.1 is the most current version at the moment)
- `./gradlew dependencies --write-locks -PspringBootVersion='3.+' (3.+ is my pattern to use the most current version)

The lock files are identical. So I assume that

- `./gradlew dependencies --write-locks -PspringBootVersion='3.+' (executed a couple of months ago when 3.2.1 was the most current version)
- `./gradlew dependencies --write-locks -PspringBootVersion='3.2.1' (executed now)

produce identical lock files, too!

#### Solution

Thanks to [discuss.gradle.org - Subproject downloads dependencies higher version than in lockfiles](https://discuss.gradle.org/t/subproject-downloads-dependencies-higher-version-than-in-lockfiles)
and Björn Kautler, I finally got a good understanding of the problem and two
possible solutions.

The problem is the plugin version. It is not locked,
so when a new version of the plugin is available, I do get
new BOM_COORDINATES and they do not match the locked versions
any more.

Possible solutions:

- Avoid using "SpringBootPlugin.BOM_COORDINATES"

  ```diff
  --------------------------------- build.gradle ---------------------------------
  index ddc7b75..bdb8d78 100644
  @@ -20,6 +20,6 @@ repositories {
   }
 
   dependencies {
  -    implementation platform(org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES)
  +    implementation platform("org.springframework.boot:spring-boot-dependencies:${springBootVersion}")
       implementation 'org.springframework.boot:spring-boot-starter-web'
   }
  ```

- Lock plugin versions

  ```diff
  --------------------------------- build.gradle ---------------------------------
  index ddc7b75..fea4dd4 100644
  @@ -1,3 +1,9 @@
  +buildscript {
  +    configurations.classpath {
  +        resolutionStrategy.activateDependencyLocking()
  +    }
  +}
  +
   plugins {
       id 'java'
       id 'org.springframework.boot' version "${springBootVersion}"
  ```

Both solutions do work.
Thanks a lot to Björn Kautler and the gradle forum!
