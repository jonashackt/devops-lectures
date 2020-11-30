# devops-lectures
Notes for my lecture about DevOps at Leipzig University (https://sws.informatik.uni-leipzig.de/team/)

### Development Process

* Create a new repository „devopsdemo“ & clone it locally 
* Create Issue in Repo: "New Webapplication should react with "Hello World" to a GET request.“
* Create Spring skeleton with Reactive Web
* Extract and copy to cloned location
* Import into Intellij
* Switch branch to new „new-webapplication“


__HelloRouterTest.java__ 
```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class HelloRouterTest {

	@Test void
	should_call_reactive_rest_resource(@Autowired WebTestClient webTestClient) {
		webTestClient.get().uri("/hello")
				.accept(MediaType.TEXT_PLAIN)
				.exchange()
				.expectBody(String.class).isEqualTo("Hello World");
	}
}
```

__HelloRouter.java__ 
```java
import org.springframework.context.annotation.Bean;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.server.*;
import reactor.core.publisher.Mono;

@Component
public class HelloRouter {

    public Mono<ServerResponse> hello(ServerRequest serverRequest) {
        return ServerResponse
                .ok()
                .contentType(MediaType.TEXT_PLAIN)
                .body(BodyInserters.fromValue("Hello World"));
    }

    @Bean
    public RouterFunction<ServerResponse> route() {
        return RouterFunctions.route(
                RequestPredicates.GET("/hello").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)),
                serverRequest -> hello(serverRequest)
        );
    }
}
```

* Insert Implementation code
* Run test again – should be green now
* Now commit code (use Issue id!) und push


### Continuous Integration  

* Create new .github/workflows directory & maven.yml inside

__github-actions-maven.yml__ 
```yaml
name: devopsdemo

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 12
      uses: actions/setup-java@v1
      with:
        java-version: 12
    - name: Test with Maven
      run: mvn -B test --no-transfer-progress
```

* Add badge to README.md

```markdown
[![Build Status](https://github.com/jonashackt/devopsdemo/workflows/devopsdemo/badge.svg)](https://github.com/jonashackt/devopsdemo/actions)
```

* Commit & push


### Continuous Deployment

We need JDK 11, so let's configure it (or we'll end up with `Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile (default-compile) on project devopsdemo: Fatal error compiling: invalid target release: 11 ` errors):

__system.properties__

```properties
java.runtime.version=11
```

Now create the app at the console

```
heroku apps:create devopsdemo-deployment-staging
```

* Heroku app > Deploy > GitHub: Connect to GitHub > search repo
* Enable automatic deploys + wait for CI to pass before deploy
* Add Heroku badge

```markdown
[![Deployed on Heroku](https://img.shields.io/badge/heroku-deployed-blueviolet.svg?logo=heroku&)](https://devopsdemo-deployment-staging.herokuapp.com/hello)
```

* Change some code & push
* Why didn't the automatic deploy work?
* Pull Request!
* Watch the Heroku Apps' Activity -> Deployment

Access the App:

https://devopsdemo-deployment-staging.herokuapp.com/hello

### Stages

* Add a new Pipeline to the App (at Deploy)
* Connect Pipeline to GitHub repository & enable Review Apps
* Create new Issue „cool new feature“
* Change some code, create new branch, push with #2

* PullRequest for the new feature
* Create new Review app in Heroku
* Access App in Browser
* Merge PullRequest

* Watch staging environment get updated

__How do we deploy to production?__

* Create production environment

```
heroku apps:create devopsdemo-deployment
```

* Click on "Deploy a branch..." to __manually__ issue a production release


### Docker

Install Docker (Mac: `brew cask install docker`)

Create __Dockerfile__

```dockerfile
FROM adoptopenjdk:11-jre-hotspot

VOLUME /tmp

# Add Spring Boot app.jar to Container
COPY target/*.jar app.jar

ENV JAVA_OPTS=""

# Fire up our Spring Boot app by default
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```

Build the app with Maven

```shell script
mvn clean package
```

Build Docker image with:

```shell script
docker build . --tag devopsdemo:latest
```

Now run our Docker container (with port binding):

```shell script
docker run -p 8080:8080 devopsdemo:latest
```

Access our App at

http://localhost:8080/hello