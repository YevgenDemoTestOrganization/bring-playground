# Bring Framework Playground

## Getting Started

Demo features:
 - List injection
 - Properties injection via Value
 - ScheduledTask
 - Configuration (Bean), Service annotation usage
 - Qualifier
 - Embedded Server Tomcat 
 - Dispatcher Servlet
 - REST API
 - Static Content Serving
 - Exception handler
 - Actuator

Please take into account that this is not a full supported features it is just for demo set. 
If you need more please take to look into example of repo [bring](https://github.com/YevgenDemoTestOrganization/bring).

## Usage

To run the `bring Playground` application:

```bash
   git clone https://github.com/YevgenDemoTestOrganization/bring-playground.git
```  

and run the 

```
com.levik.bringplayground.BringPlaygroundApplication
```  

## Lets fo feature by feature

- List Injection Overview

```java
import com.bobocode.bring.core.annotation.Autowired;
import com.bobocode.bring.core.annotation.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class Barista {
    private final List<Drink> drinks;
    
    public Barista(List<Drink> drinks) {
        this.drinks = drinks;
    }

```

List injection, a feature in Bring, involves the injection of a collection of objects into a class or component. 
In our project, the Barista class is a prime example that demonstrates this functionality.
Go to class Barista. Within this example, the Barista class receives a list of Drink instances via its constructor. 
These instances might be defined elsewhere in the project, and Bring automatically handles their injection into the Barista class upon initialization.

See the result of the logs:

```
[INFO ] 23-12-01 14:43:39.838 [main] c.l.b.BringPlaygroundApplication - bring - Barista is Barista is preparing a drink: Brewing a strong espresso!, Making a delicious latte!
```

- Properties injection via Value

We have default values that come to start the tomcat for instance server.port etc. If you want you could override them and use your port of use default.

But else you could create you class to consumer you properties for instance ShortenProperties it has serverUrl

```java
import com.bobocode.bring.core.annotation.Component;
import com.bobocode.bring.core.annotation.Value;
import lombok.Data;

@Data
@Component
public class ShortenProperties {

    @Value("shortenServerUrl")
    private String serverUrl;
}
```  

application.properties

```
shortenServerUrl = http://localhost:9000/short/
```

- ScheduledTask


This code defines a class MyScheduledTasks that contains a method scheduledMethod1(), annotated as a scheduled task. 
This method is set to run periodically with a specific initial delay and time period. 
When executed, it logs information using the logger just for demo purpose.
```java
import com.bobocode.bring.core.annotation.Component;
import com.bobocode.bring.core.annotation.ScheduledTask;
import lombok.extern.slf4j.Slf4j;

import java.time.LocalDateTime;
import java.util.concurrent.TimeUnit;

@Slf4j
@Component
public class MyScheduledTasks {
    @ScheduledTask(value = "myTask", initialDelay = 1, period = 20, timeUnit = TimeUnit.SECONDS)
    public void scheduledMethod1() {
        log.info(Thread.currentThread().getName() + " scheduledMethod1 " + LocalDateTime.now());
    }
}
```

log result:

```
[INFO ] 23-12-01 15:10:21.241 [pool-1-thread-1] c.l.b.feature.di.MyScheduledTasks - bring - pool-1-thread-1 scheduledMethod1 2023-12-01T15:10:21.241206
[INFO ] 23-12-01 15:10:41.242 [pool-1-thread-1] c.l.b.feature.di.MyScheduledTasks - bring - pool-1-thread-1 scheduledMethod1 2023-12-01T15:10:41.242068
```

- Configuration (Bean), Service annotation usage

Within our project, we have the flexibility to configure classes using annotations such as @Component or @Service,
enabling them to be recognized as Bring-managed components automatically. Additionally, we can create a separate configuration class,
annotate it with @Configuration, and employ the @Bean annotation on its methods.

```java
import com.bobocode.bring.core.annotation.Service;


@Service
public class Md5HashGenerator implements HashGenerator {
    ...
}
```


```java
import com.bobocode.bring.core.annotation.Bean;
import com.bobocode.bring.core.annotation.Configuration;

@Configuration
public class MyConfiguration {

    @Bean
    public HashGenerator incrementHashGenerator() {
        return new IncrementHashGenerator();
    }
}
```

- Qualifier

We need two classes for hash generation: one for testing and another for production. To ensure functionality, we require a Qualifier or a name that can be resolved by its name.

```java
    private final HashGenerator hashGenerator;

    public ShortenService(ShortenProperties shortenProperties,
                          @Qualifier("md5HashGenerator") HashGenerator hashGenerator) {
        ...
        }

```

- Embedded Server Tomcat, Dispatcher Servlet, Rest API, Web annotations like PostMapping, ResponseEntity, RequestBody etc.


To simplify and expedite our web development process, it would be beneficial to incorporate an embedded Tomcat that encompasses all the essential functionalities for developing and resetting APIs seamlessly.

```java
public class ShortenController implements BringServlet {

    private static final String LOCATION = "location";
    private final ShortenService shortenService;

    @SneakyThrows
    @PostMapping(path = "/api/shorten")
    public ResponseEntity<UserResponse> createShortUrl(@RequestBody UserRequest userRequest) {
        String shortUrl = shortenService.createShortUrl(userRequest.getOriginalUrl());
        return new ResponseEntity<>(new UserResponse(shortUrl), HttpStatus.OK);
    }

    @SneakyThrows
    @GetMapping(path = "/short/{hash}")
    public ResponseEntity<Void> getLongUrl(@PathVariable String hash) {
        String longUrl = shortenService.resolveHash(hash);

        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.set(LOCATION, longUrl);

        return new ResponseEntity<>(httpHeaders, HttpStatus.MOVED_PERMANENTLY);
    }
}

```

- Static Content Serving

Typically, we require serving static content such as HTML, CSS, and JS files on a server. 
With Bring, you can automate this process using out of the box, allowing for seamless hosting and browsing of these files

![Bring DI diagram](https://github.com/YevgenDemoTestOrganization/bring/assets/73576438/289a774e-dce4-4057-8506-ecaefb2a2ec3)

- Exception handler
  ![Bring DI diagram](https://github.com/YevgenDemoTestOrganization/bring/assets/73576438/f18327da-bfb3-4e73-b93e-3208229462e8)

- Actuator

We offer support for health checks, info retrieval, environment details, and logging functionalities. 
Our standout feature allows changing log levels in real-time without the need for system restarts, 
facilitated through a dedicated API.

```java
    /**
     * Endpoint for dynamically changing the log level of specific loggers based on package name.
     *
     * @param packageName The name of the logger's package to change the level.
     * @param newLevel    The new log level for the specified logger's package.
     */
    @PostMapping(path = "/loggers")
    public void logger(@RequestParam String packageName, @RequestParam String newLevel) {
        LogLevelChangerUtils.changeLogLevel(packageName, newLevel);
    }
```

In addition, Bring support many more features:
- [Core](https://github.com/YevgenDemoTestOrganization/bring/blob/main/features/Core.md)
- [Web](https://github.com/YevgenDemoTestOrganization/bring/blob/main/features/Web.md)


