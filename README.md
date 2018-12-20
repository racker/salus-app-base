This is a Maven "pom" module that provides a simple, unified configuration for building
Spring Boot application modules. This module references Spring Boot Starter Parent, so
all application modules using this salus-app-base will transitively gain the same benefits
of the Spring Boot parent.

# Application POM Configuration

## Parent reference

The typical way to reference this parent module is to add this to the application's `pom.xml`:

```xml
<parent>
    <groupId>com.rackspace.salus</groupId>
    <artifactId>salus-app-base</artifactId>
    <version>0.1.0-SNAPSHOT</version>
    <relativePath>../salus-app-base</relativePath>
</parent>
```

The `relativePath` is optional; however, the convention is to specify the relative path to
the `apps/salus-app-base` directory of the [Salus Telemetry Bundle repo](https://github.com/racker/salus-telemetry-bundle/)

## Main plugin config

The Spring Boot parent configures the 
[spring-boot-maven-plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/index.html);
however, each application still needs to "activate" the plugin with at least the following
plugin specification:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## Docker build config

This app base module preconfigures the [jib maven plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin);
however, each application needs to "activate" the plugin to enable Docker image building by 
specifying the following. 

> **NOTE** as you can see, the convention is to place the plugin within
a `docker` profile block. This will ensure a "default" maven build will avoid the Docker
build behavior and keep things simple. The profile can be activated by adding `-P docker` to
the maven command-line.

```xml
<profiles>
    <profile>
        <id>docker</id>

        <build>
            <plugins>
                <plugin>
                    <groupId>com.google.cloud.tools</groupId>
                    <artifactId>jib-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

By default, the jib plugin is configured to use an OpenJDK JRE Alpine base image; however, the
base image (and other configuration) can be overridden by adding this type of `<configuration>`
section within the `<plugin>`:

```xml
<configuration>
    <from>
        <!-- SHA can be obtained from https://console.cloud.google.com/gcr/images/distroless/GLOBAL/java?gcrImageListsize=50 -->
        <image>gcr.io/distroless/java@sha256:b430543bea1d8326e767058bdab3a2482ea45f59d7af5c5c61334cd29ede88a1</image>
    </from>
</configuration>
```

# Usage

The included Maven plugins enable some standardized build/run behaviors for the applications
that reference this parent.

## Running applications via Maven

An application can be built and started from the command-line using:

```bash
mvn spring-boot:run
```

Application command-line arguments can be passed via `spring-boot.run.arguments`, such as
```bash
mvn spring-boot:run -Dspring-boot.run.arguments=--server.port=9999
```

Spring profiles can be activated via `spring-boot.run.profiles`, such as
```bash
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

More information about the configuration of [the run goal can be found here](https://docs.spring.io/spring-boot/docs/current/maven-plugin/run-mojo.html).

## Building local Docker images

When the Maven profile "docker" is activated, the application module will produce
a Docker image as part of the `package` goal.

> **NOTE** this specific goal requires access to a Docker daemon, such as via Docker for Desktop
or a mounted Docker socket.

## Publishing Docker images to GCR

Use [the preparation part of these docs](https://cloud.google.com/container-registry/docs/pushing-and-pulling) 
to install the Cloud SDK tools and configure Docker for authentication. You can disregard the 
details about `docker push` since the [Maven jib plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin)
will take of the equivalent operations.

**Tip:** on MacOS you can install the Cloud SDK using brew:
 
```bash
brew cask install google-cloud-sdk
```

In the application module, run the following replacing `$PROJECT_ID` with the Google
Cloud project's ID, which is of the form of an identifer and number separated by a dash:

```
mvn -P docker -Ddocker.image.prefix=gcr.io/myapp-12345 deploy
```

If you would like to avoid pushing the Maven artifact during this build, then add

```
-Dmaven.deploy.skip=true
```

If the local system doesn't have Docker installed, you can still perform the remote publish and
skip the local Docker build by adding:

```
-DskipLocalDockerBuild=true
```

