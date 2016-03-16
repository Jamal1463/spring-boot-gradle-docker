# Spring Boot with Docker

### Gradle minimal dependencies

We will use gradle as dependency manager. To use spring boot with docker we must set up the *Spring Boot Plugin*

Gradle file:

```gradle
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.3.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
    baseName = 'gs-spring-boot-docker'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}
```

### Adding a simple REST endpoint

Now we will create a simple REST endpoint to test if our application is working.

```java
@SpringBootApplication
@RestController
public class HelloApplication {

	@RequestMapping("/hello-docker")
	public String home() {
		return "Hello Docker World";
	}

	public static void main(String[] args) {
		SpringApplication.run(HelloApplication.class, args);
	}

}
```

You could access the REST endpoint in the following address:

```bash
http://localhost:8080/hello-docker
```

### Generating a jar by using Gradle

To generate a new jar we must use the **gradle build** command. 

Execute the following command:

```bash
$ docker build
```

This will create a jar file named **gs-spring-boot-docker.jar** in the **build/libs** directory. Note that this name is from build.gradle file.

```bash
$ build/libs/spring-boot-gradle-docker.jar
```

To execute the Application, you just need to execute the jar file and access the address:

```bash
$ java -jar build/libs/spring-boot-gradle-docker.jar 
```

### Dockerfile

```docker
FROM frolvlad/alpine-oraclejdk8:slim

VOLUME /tmp

ADD spring-boot-gradle-docker.jar spring-boot-app.jar

RUN bash -c 'touch /spring-boot-app.jar'

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/spring-boot-app.jar"]

```

### Build a Docker Image with Gradle

```gradle
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.3.RELEASE")
        classpath('se.transmode.gradle:gradle-docker:1.2')
    }
}

group = 'springio'

apply plugin: 'docker'
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
    baseName = 'gs-spring-boot-docker'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}

task buildDocker(type: Docker, dependsOn: build) {
  push = false
  applicationName = jar.baseName
  dockerfile = file('src/main/docker/Dockerfile')
  doFirst {
    copy {
      from jar
      into stageDir
    }
  }
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}
```





