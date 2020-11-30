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


### Continuous...  

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