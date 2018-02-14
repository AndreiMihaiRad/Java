# Maven Multi Module Project
This Guideline purpose is to help organising a maven multi-module project

## Intro
### What is Maven

Apache Maven is a popular tool for build automation, primarily Java projects. Maven addresses two aspects of building software. First, it describes how a software is built and, second, it describes its dependencies. It uses conventions for the build procedure. An XML file describes the software project being built, its dependencies on other external modules and components, the build order, directories, and required plugins. It comes with predefined targets to perform certain well-defined tasks, such as code compilation and its packaging. Maven dynamically downloads Java libraries and Maven plugins from one or more repositories, such as the Maven Central Repository, and stores them locally.

### Maven build Lifecycle
Maven is implemented based around the concept of a build lifecycle. Each lifecycle has a number of phases:

* clean: - removes all the files and folders created by Maven
* site: - generates the project's documentation
* default:
  * validate: validates all project information
  * process-resources: copies project resources to the destination to package
  * compile: compiles the source code
  * test: runs unit tests within a suitable framework
  * package: packages the compile code in its distribution format
  * integration-test: processes the package in the integration test envinronment
  * verify: runs checks to verify that the package is valid
  * install: installs the package in the local repository
  * deploy: installs the final package in the configured repository Each phase is made up of plugin goals. A plugin goal is a specific task that builds the project. Some goals make sense only in specific phases (for example, the compile goal of the Maven Compiler plugin makes sense in the compile phase, but the checkstyle goal of the Maven Checkstyle plugin can potentially be run in any phase). So some goals are bound to a specific phase of a lifecycle, while others are not.

### What is POM
A Project Object Model or POM is the fundamental unit of work in Maven. It is an XML file that contains information about the project and configuration details used by Maven to build the project. It contains default values for most projects. Examples for this is the build directory, which is target; the source directory, which is src/main/java; the test source directory, which is src/test/java; and so on.

The POM was renamed from project.xml in Maven 1 to pom.xml in Maven 2. Instead of having a maven.xml file that contains the goals that can be executed, the goals or plugins are now configured in the pom.xml. When executing a task or goal, Maven looks for the POM in the current directory. It reads the POM, gets the needed configuration information, then executes the goal.

Some of the configuration that can be specified in the POM are the project dependencies, the plugins or goals that can be executed, the build profiles, and so on. Other information such as the project version, description, developers, mailing lists and such can also be specified.

#### Show dependency report
```bash
$ mvn dependency:list
$ mvn dependency:tree
```

#### Detecting unused/undeclared dependencies
```bash
$ mvn dependency:analyze
```

## Multi-module projects
#### Project Inheritance vs Project Aggregation
If you have several Maven projects, and they all have similar configurations, you can refactor your projects by pulling out those similar configurations and making a parent project. Thus, all you have to do is to let your Maven projects inherit that parent project, and those configurations would then be applied to all of them.

And if you have a group of projects that are built or processed together, you can create a parent project and have that parent project declare those projects as its modules. By doing so, you'd only have to build the parent and the rest will follow.

But of course, you can have both Project Inheritance and Project Aggregation. Meaning, you can have your modules specify a parent project, and at the same time, have that parent project specify those Maven projects as its modules. You'd just have to apply all three rules:

- Specify in every child POM who their parent POM is.
- Change the parent POMs packaging to value "pom"
- Specify in the parent POM the directories of its modules (children POMs)

```bash
|-- my-module
|   `--pom.xml`
 -- parent
    `--pom.xml`
```

### Project Inheritance
There are times when you might want a project to use values from another .pom file. You may be building a large software product, so you do not want to repeat the dependency and other elements multiple times.

Maven provides a feature called project inheritance for this. Maven allows a number of elements specified in the parent pom file to be merged to the inheriting project. In fact, the super pom file is an example of project inheritance.

#### Steps:
1. Open a project that has inheritance; project-with-inheritance in our case. This has a subfolder named child, which is the project that inherits from the parent.

2. Update the parent pom file as follows:

  ```
  	<groupId>com.packt.cookbook</groupId>
  <artifactId>project-with-inheritance</artifactId>
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>
  ```

3. Create the pom file for child as follows:
  ```
	 <parent>
	      <groupId>com.packt.cookbook</groupId>
	      <artifactId>project-with-inheritance</artifactId>
	      <version>1.0-SNAPSHOT</version>
	  </parent>
	  <modelVersion>4.0.0</modelVersion>
	  <artifactId>child</artifactId>
	  <packaging>jar</packaging>
	  <name>Child Project</name>
  ```  
4. Run the following Maven command in the child subfolder:

  ```bash
  $ mvn clean package
  ```

#### How it works

We specified a **parent** element in the pom file of **child**. Here, we added the coordinates of the parent, namely **groupId**, **artifactId**, and **version**. We did not specify the groupId and version coordinates of the child project. We also did not specify any properties and dependencies.

In the parent pom file, we specified **properties** and **dependencies**.

Due to the relationship defined, when Maven runs on the **child** project, it inherits **groupId**, **version**, **properties**, and **dependencies** defined in the parent.

How did Maven know where the parent pom is located? We did not specify a location in the pom file. This is because, by default, Maven looks for the parent pom in the parent folder of child. Otherwise, it attempts to download the parent pom from the repository.

**Note**: If pom is not in any repository or is not in parent folder of the child, you can add attribute <relativePath>../parent/pom.xml</relativePath>


### Project Aggregation
A key difference between inheritance and aggregation is that, aggregation is defined with a top-down approach, whereas inheritance is defined the other way around. In Maven, project aggregation is similar to project inheritance, except that the change is made in the parent pom instead of the child pom.

Maven uses the term module to define a child or subproject, which is part of a larger project. An aggregate project can build all the modules together. Also, a Maven command run on the parent pom or the pom file of the aggregate project will also apply to all the modules that it contains.

#### Steps

1. Open a project that has aggregation; in our case project-with-aggregation. This has a subfolder named aggregate-child, which is the module that is aggregated by the parent project.

2. Update the parent pom as follows:
  ```
    <groupId>com.packt.cookbook</groupId>
    <artifactId>project-with-aggregation</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
  ```
3. Add the module section and specify the child
  ```
  <modules>
    <module>aggregate-child</module>
  </module>
  ```

#### How it works
We specified the child project as a module in the aggregator pom. The child project is a normal Maven project, which has no information about the fact that there exists an aggregator pom.

When the aggregator project is built, it builds the child project in turn. You will notice the word **Reactor** in the Maven output. Reactor is a part of Maven, which allows it to execute a goal on a set of modules. While modules are discrete units of work; they can be gathered together using the reactor to build them simultaneously. The reactor determines the correct build order from the dependencies stated by each module.

### Performing multi-module dependency management
Dependency management is a mechanism to centralize dependency information. When there are a set of projects (or modules) that inherit a common parent, all information about the dependency can be put in the parent pom and the projects can have simpler references to them. This makes it easy to maintain the dependencies across multiple projects and reduces the issues that typically arise due to multiple versions of the same dependencies.

#### Steps
1. Open a multi-module project (simple-multi-module).
2. Add a dependency for junit in the dependencyManagement section:
  ```
	<dependencyManagement>
	    <dependencies>
	      <dependency>
	        <groupId>junit</groupId>
	        <artifactId>junit</artifactId>
	        <version>3.8.1</version>
	        <scope>test</scope>
	      </dependency>
	    </dependencies>
	</dependencyManagement>
  ```
3. Update the dependencies section of the child project as follows:
  ```
	<dependencies>
	     <dependency>
	       <groupId>junit</groupId>
	       <artifactId>junit</artifactId>
	     </dependency>
	</dependencies>
  ```
4. Run the Maven command to check the dependency:
  ```bash
	$ mvn dependency:tree
  ```

#### How it works

Dependencies that are specified within the dependencyManagement section of the parent pom are available for use to all the child projects. The child project needs to choose the dependencies by explicitly specifying the required dependencies in the dependencies section. While doing this, the child projects can omit the version and scope information so that they are inherited from the parent.

You may ask, "Why have the dependencyManagement section when child projects inherit dependencies defined in the parent pom anyway?" The reason is, the parent centralizes dependencies across several projects. A child project typically needs only some of the dependencies that the parent defines and not all of them. The dependencyManagement section allows child projects to selectively choose these.

## How to share resources across projects in Maven
