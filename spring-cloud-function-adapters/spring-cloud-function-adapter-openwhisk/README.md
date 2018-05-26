Implement a POF (be sure to use the `functions` package):

```
package functions;

import java.util.function.Function;

public class Uppercase implements Function<String, String> {

	public String apply(String input) {
		return input.toUpperCase();
	}
}
```

Install it into your local Maven repository:

```
./mvnw clean install
```

Create a `function.properties` file that provides its Maven coordinates. For example:

```
dependencies.function: com.example:pof:0.0.1-SNAPSHOT
```

Copy the openwhisk runner JAR to the working directory (same directory as the properties file):

```
cp spring-cloud-function-adapters/spring-cloud-function-adapter-openwhisk/target/spring-cloud-function-adapter-openwhisk-1.0.0.RC2.jar runner.jar
```

Generate a m2 repo from the `--thin.dryrun` of the runner JAR with the above properties file:

```
java -jar -Dthin.root=m2 runner.jar --thin.name=function --thin.dryrun
```

Use the following Dockerfile:

```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY m2 /m2
ADD runner.jar .
ADD function.properties .
ENV JAVA_OPTS=""
ENTRYPOINT [ "java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "runner.jar", "--thin.root=/m2", "--thin.name=function", "--function.name=uppercase"]
EXPOSE 8080
```

> NOTE: you could use a Spring Cloud Function app, instead of just a jar with a POF in it, in which case you would have to change the way the app runs in the container so that it picks up the main class as a source file. For example, you could change the `ENTRYPOINT` above and add `--spring.main.sources=com.example.SampleApplication`.

Build the Docker image:

```
docker build -t [username/appname] .
```

Push the Docker image:

```
docker push [username/appname]
```

Use the OpenWhisk CLI (e.g. after `vagrant ssh`) to create the action:

```
wsk action create example --docker [username/appname]
```

Invoke the action:

```
wsk action invoke example --result --param payload foo
{
    "result": "FOO"
}
```
# Examples

The following examples are built based on the details and explanations above, on how to deploy Spring Cloud Functions on to [OpenWhisk](https://openwhisk.apache.org/)

* [Spring Cloud Function PoF Example](https://github.com/redhat-developer-demos/ow-scf-fruiteason)

This example shows how to use Spring Cloud Functions by defining simple Plain Old Function (POF)

* [Spring Cloud Function Application Example](https://github.com/redhat-developer-demos/ow-scf-greeter)

This example shows how to use Spring Cloud Functions with a complete Spring Boot Application
that has functions defined by extending `java.util.function.Function` interfaces.

The base docker images used for above examples is available [here](https://github.com/redhat-developer-demos/openwhisk-scf-docker).