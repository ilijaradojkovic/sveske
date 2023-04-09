# Eureka

Eureka nam je 3rd party koja nam omogucava service discovery,load banacing i jednostavna je za test okruzenje da se postavi.



Imamo 2 dependency-ja  kao kod kafke,jedan za client,i jedan za server.

Napravimo posebnu app gde ce se nalaziti eureka i samo to,i ostale app koje su mikroservisi i gateway.Samo ce eureka da sadrzi server dependency,a ostale ce client.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

Na severskoj strani dodamo @EnableEurekaServer

## Server Side

Dodamo  server dependency.

Dodamo @EnableEurekaServer

```yaml
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

Dodamo ovu konfiguraciju gde smo promenili port,i konfig za erueku da ne registruje sam sebe kao servis.Ako odemo na localhost:8761 otvorice nam se mali UI za eureku.

44:21

Koriscenje load balancera je veoma mocno,iza scene to radi tako sto koristi spring cloud load balancer apstrakciju da u  client memory odluci kojoj instanci da da request da obradi,mi mozemo da promenimo ovaj algoritam rada koji je most recently used,na round robina npr itd.

Mi kada hocemo da gadjamo rute radimo sa 

lb://imeapp...

bitno nam je ovo imeapp to je application.name ,takodje ime koje vidimo u eureka UI

## Client Side

Dovoljno je da imamo samo dependency za eureka client i da se eureka server run da se oni sami regitruju.

I idemo @EnableEurekaClient 

eureka.client.serviceUrl.defaultZone=http://localhost:9999/eureka/ ako ne nadje odmah eura server moramo definisati putanju do njega



## Feign Client

- <span style="color:red">@FeignClient</span>
- <span style="color:red">@EnableFeignClients</span>

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

i naravno jer je cloud dependency moramo:

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2022.0.1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

@FeignClient nam omogucava da ovaj interface poziva druge endpointe samo treba da ga autowire

```
@Component
@FeignClient(name = "otherClient",url = "http://localhost:8081")
public interface ExternalController {

    @GetMapping("/test")
    String getUsers();

}
```

i posle ovoga moramo da dodamo @EnableFeignClients

On je po deafultu blocking a mozmeo da ga konfigurisemo da ne bude blocking

```
@FeignClient(name = "example", url = "http://example.com", configuration = ExampleConfiguration.class)
public interface ExampleClient {
    @GetMapping("/")
    Mono<String> get();
}

@Configuration
public class ExampleConfiguration {
    @Bean
    public WebClient webClient() {
        return WebClient.builder()
            .baseUrl("http://example.com")
            .clientConnector(new ReactorClientHttpConnector(HttpClient.create().wiretap(true)))
            .build();
    }

    @Bean
    public Client feignClient(WebClient webClient) {
        return new ReactorClientHttpConnectorAdapter(new ReactorClientHttpConnector(HttpClient.create().wiretap(true)));
    }
}
```

Ovo radi load balacing na eurka serveru

ILI 

jer radimo sa webclient-om mozemo ovako

```
 @Bean
    @LoadBalanced
    WebClient.Builder builder() {
        return WebClient.builder();
    }
    
    @Bean
    WebClient webClient(WebClient.Builder builder) {
        return builder.build();
    }
```

