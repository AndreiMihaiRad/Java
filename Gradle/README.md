# Gradle

In this article class'll discuss how to use Gradle to manage building, testing and deploying your Java application

## Intro
- Gradle is written in Java, but the build language is a Domain Specific Language which is written in Groovy and this language makes it easy to configure our builds. So we're not writing a XML; we're are writing in a language that's Specific to our build.
- Like Maven It supports dependencies; supports both Ivy and Maven dependencies
- Supports multi-project builds
- Gradle allows to manage builds other than Java projects (ex: JavaScript, C++)
- Gradle has declarative Build Language
  - Express Intent (what we'd like to happen, not how we would like it to happen)

### Installing Gradle

- We can first of all go to the website (http://gradle.org)
- Use a tool named **gvm** (Groovy enVinronment Manager) (http://gvmtool.net)

##### MacOS
```bash
brew install groovy
brew install gradle
```
##### Windows

##### Linux

### Running Gradle
Gradle consists of a build file called build.gradle

- build.gradle:
  - tasks
  - plugins
  - dependencies

Example_1
```groovy
task hello {
   doLast {
     println "Hello, Gradle"
   }
}
```
- we've defined a task (hello)
- every task have a lifecycle
  - configuration phase
  - initialisation phase
  - method
    - doFirst
    - doLast
```bash
$ gradle hello
```
Example_2

```gradle
apply plugin: 'java'
```
Check what tasks are available within plugin
```bash
$ gradle tasks
```
Build a java app
```bash
$ gradle build
```

### Gradle Wrapper
This may be useful when you want to wrap your gradle installed version on your PC with another version. This way all developers can use the same version for Gradle
 Example build.gradle file
 ```gradle
apply plugin: 'java'
task wrapper (type: Wrapper){
  gradleVersion = '2.6'
}
```
Execute
```bash
$ gradle wrapper
$ gradlew build
```

## Basic Gradle tasks

### Task
= code that gradle executes
- has a lifecycle - different parts of task will execute at different times.
  - initialisation phase
    - used to configure multi project build
  - configuration phase
    - Executes code in the task that's not the actions
    - ex
      ```
      Task myTask {
              description "myTask configuration"
            }
       ```
  - execution phase - when the task actions are executed
    - doFirst
    - doLast
- has properties - any information that tasks needs
- has actions - code that's going to execute
  - fist action
  - last action
- has dependencies
 - Task6.dependsOn Task5

#### Other Dependencies
- mustRunAfter - If two execute one **must** run after the other
- shouldRunAfter
  - If two tasks execute one **should** run after the other
  - This ignores circular dependencies
- finalizedBy
  - inverted dependencies

### Running tasks

```bash
$ gradle task_name
```

### Typed Tasks

Abstract the code away so that we can reuse that code
Ex: Copying files

https://docs.gradle.org/current/dsl/

## Building a Java Project

Add the Java Plugin into our build gradle files
  ```
  apply plugin 'java'
  ```
This add many tasks to our project Like:
    - build
    - clean
    - javadoc
    - compile test
    - run test

Like Maven, Gradle supports a standard layout, supports a convention as to where we want to put our sources and where it's going to build the outputs
  - src/main/java
  - src/main/resources
  - src/test/java
  - src/test/resources

So if you don't like standard Gradle layouts, or if your company supports a different convention, then you can very easy to change in Gradle. Gradle has a concept, well known as SourceSets and we use the SourceSets to define where we want out sources.

If you have a different conventio, then you can configure the layout using a SourceSet.

EX:
```
sourceSets {
  main {
    java {
      srcDir 'src/java'
    }
    resources {
      srcDir 'src/resources'
    }
  }
}
```

### Writing a Multi-project Build

- Add top level settings.gradle
  - List of all the projects in the build
- Add top level build.gradle (configures all of the projects in this multi-project build)
  - Add information that's common to all projects and we can add dependencies between these projects, as well

## Dependencies
- Other projects
- External libraries
- Internal libraries

### Dependencies Can Be Satisfied From ...
- Other projects
- File System
- Maven repositories : local, remote

#### List Dependencies

```bash
gradle -q dependencies
gradle -q dependencies -configuration compile
```

### Using Repositories
In Gradle we can use different repositories and the usage of these things is all very similar so we can use Maven and we can use our remote Maven repository.

We can use mavenLocal and that looks in the .m2 directory for the repository

## testing

- Defines
  - a source set
  - task to compile the tests
  - task to run the tests
