# How to share resources across projects in maven
There are a few ways to share resources across multiple projects or modules:

- Cut and paste them.
- Use Assembly and Dependency plugins
- Use the maven-remote-resources-plugin

In this post I'll show how to do the second option.

I'm going to show how this is done in a multi module project, so the first thing we need is the top level parent pom:

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.sonatype</groupId>
  <artifactId>resource-sharing</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>Resource Sharing Parent</name>

  <build>
     <pluginManagement>
        <plugins>
           <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-dependency-plugin</artifactId>
             <version>2.0</version>
          </plugin>
          <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-assembly-plugin</artifactId>
             <version>2.2-beta-2</version>
          </plugin>
       </plugins>
    </pluginManagement>
  </build>
  <modules>
     <module>resource-consumer</module>
     <module>assembly</module>
  </modules>
</project>
```

In an enterprise, you will probably want this to inherit from your corporate pom. The only thing particularly interesting about the pom above is that I have declared the versions of the plugins I'm using in pluginManagement. I have also inverted the modules list to show that Maven is smart enough to reorder modules based on declared dependencies (this is another FAQ).

The next step is to create the module that will zip up the resources to be shared. I'm going to do this using the maven-assembly-plugin to zip up **src/main/resources** from my shared-resources project. The assembly descriptor in **src/main/assembly/resource.xml** to do this is pretty small:

```
<assembly>
  <id>resources</id>
  <formats>
    <format>zip</format>
  </formats>
  <includeBaseDirectory>false</includeBaseDirectory>
  <fileSets>
    <fileSet>
      <directory>src/main/resources</directory>
      <outputDirectory></outputDirectory>
    </fileSet>
  </fileSets>
</assembly>
```

We just told the assembly plugin to make a zip using a classifier of **resources** (more on this below). It will pick up the contents of **src/main/resources** and include them in the root of the zip. By default, assembly will create a subfolder with the artifactid-version as the name, so we turn this off by specifying the empty **outputDirectory** element.

The next thing we need is the pom.xml to do the work in the assembly module:

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.sonatype</groupId>
        <artifactId>resource-sharing</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>shared-resources</artifactId>
    <packaging>pom</packaging>
    <name>Shared Resources Bundle</name>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <executions>
                    <execution>
                        <id>make shared resources</id>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <phase>package</phase>
                        <configuration>
                            <descriptors>
                                <descriptor>src/main/assembly/resources.xml</descriptor>
                            </descriptors>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

Here we have used a pom packaging project to get an empty lifecycle pallet. We bind the assembly:single goal to the package phase and point it at the descriptor. Note that no version is specified here because we already locked it down in the parent pom's pluginManagment section. Be sure not to bind the assembly:assembly goal in a phase because this forks the build and can generally cause a mess of recursive builds.

So far we have managed to bundle up the resources into a zip that can be versioned and deployed to a Maven Repository Manager. So far, so good. Now we need to get those resources where we actually want them. To do this, we will use the dependency:unpack-dependencies to fetch the bundle and unpack it to the projects that need it. The pom to do it looks like this:

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>com.sonatype</groupId>
    <artifactId>resource-sharing</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>

  <artifactId>resource-consumer</artifactId>
  <packaging>jar</packaging>

  <name>Resource User A</name>

  <dependencies>
    <dependency>
      <groupId>${project.groupId}</groupId>
      <artifactId>shared-resources</artifactId>
      <version>1.0-SNAPSHOT</version>
      <classifier>resources</classifier>
      <type>zip</type>
      <!-- Make sure this isn't included on any classpath-->
      <scope>provided</scope>
    </dependency>
  </dependencies>

  <build>
    <resources>
      <resource>
        <directory>${basedir}/src/main/resources</directory>
      </resource>
      <resource>
        <directory>${project.build.directory}/generated-resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
          <execution>
            <id>unpack-shared-resources</id>
            <goals>
              <goal>unpack-dependencies</goal>
            </goals>
            <phase>generate-resources</phase>
            <configuration>
             <outputDirectory>${project.build.directory}/generated-resources</outputDirectory>
             <includeArtifacIds>shared-resources</includeArtifacIds>
             <includeGroupIds>${project.groupId}</includeGroupIds>
             <excludeTransitive>true</excludeTransitive>
             <!--use as much as needed to be specific...also scope,type,classifier etc-->
            </configuration>
          </execution>
        </executions>
      </plugin&gt
    </plugins>
  </build>
</project>
```

The first thing we did was add our resource bundle as a dependency of this project. This is done to tell Maven about the dependency so that the bundle is created before it's needed. We could have used dependency:unpack to do this without declaring a dependency but this causes other issues when used inside a multimodule build. Notice that I used a property to avoid having to hardcode the groupId of my sibling dependency. I did not do this for the version as it used to cause problems with SNAPSHOTs when deployed. I think it has been fixed but I haven't tested it yet. Also note that the classifier matches the id used in the assembly descriptor above.

The next thing we've done is add the ${project.build.directory}/generated-resources as a resource and enabled filtering (note the use of a property and not hard coding of /target/). We have to re add the default resource of src/main/resources because it will get lost otherwise.

Then we bind the dependency:unpack-dependencies goal to the generate-resources phase (again leaving the version to be picked up in pluginManagement). The unpack-dependencies goal starts with a list of all dependencies of this project. Since we are only interested in one, we can use the various filters to narrow it down. Here I have filtered on the group and artifact ids and excluded all transitive dependencies. I could have also added scope, classifier and type if I felt it was needed. I have told the plugin to extract the contents of the zip to ${project.build.directory}/generated-resources (the same location I added as a resource above).

Now, to make this all happen, I only need to go up to the root of my project and execute mvn install. (package is the minimum to make the zip that unpack will need to find but I recommend just using install when working with multi-modules...it will save you lots of grief).

Since I'm a skeptic, I'm going to go into resource-consumer/target and unpack the jar. In the root of the jar I should find a ReadMe.txt with the following content:

This Readme is included in resource-consumer-1.0-SNAPSHOT.jar
Note that my properties were replaced with the values of the project that did the filtering. This was an extreme example as you may not always want to filter the unpacked values, but that's even easier...just tell dependency where to drop the files and you're done. If you're working with wars, you can just drop the files into ${project.build.directory}/${project.build.finalName} and the war plugin will pick them up and include them in the final war.

This assembly sharing technique is also useful for sharing checkstyle and pmd rules. The only difference in those instances is that you don't actually need to unpack the file. Instead of making a zip, make a jar and then add the jar in a dependency block inside the plugin declaration. If you are using those plugins as reports, you need to add the jar as an extension in the pom (report.plugins.plugin doesn't allow a dependency). Then the plugins will look for and find your rule file on the classpath
