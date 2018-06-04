# Publishing of a modular library to a nexus repository using Gradle


## Project Structure
```
ROOT
│   build.gradle
│   settings.gradle
├───subA
│       build.gradle
│
├───subB
│       build.gradle
│
└───subC
        build.gradle
```

## Configure **root build.gradle**
```groovy
apply plugin: 'maven-publish'
subprojects {
    publishing {
        publications {
            "$project.name"(MavenPublication) {
                groupId project.group
                artifactId project.name
                version project.version
                from components.java
            }
        }
        repositories {
            maven {
                name 'deploy'
                url "http://my_nexus_repo_url"
                credentials {
                    username = "admin"
                    password = "admin123"
                }
            }
        }
    }
}
```

## Each subproject defines its own groupid and version like so:

```
group = 'org.test.sample.A'
version = '1.0'
```

## The artifactId is picked up from the subproject name. Running gradle publish results in a repo of this structure:

```bash
./gradlew publish
```

```
org
└───test
    └───sample
        ├───A
        │   └───subA
        │       └───1.0
        │               subA-1.0.jar
        │               subA-1.0.pom
        ├───B
        │   └───subB
        │       └───1.0
        │               subB-1.0.jar
        │               subB-1.0.pom
        └───C
            └───subC
                └───1.0
                        subC-1.0.jar
                        subC-1.0.pom

```