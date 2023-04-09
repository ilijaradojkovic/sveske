

# Resiliency Patterns



## Circuit break

Resilience4j moramo da ubacimo ako hocemo da imamo circuit breaker

Spring Cloud’s Circuit Breaker library provides an implementation of the Circuit Breaker pattern: When we wrap a method call in a circuit breaker, Spring Cloud Circuit Breaker watches for failing calls to that method and, if failures build up to a specified threshold, Spring Cloud Circuit Breaker opens the circuit so that subsequent calls automatically fail. While the circuit is open, Spring Cloud Circuit Breaker redirects calls to the method, and they are passed on to our specified fallback method.



Ukoliko imamo i `Actuator` mozmeo da pristuopimo preko localhost:port/actuator/circuitbreakers

localhost:port/actuator/circuitbreakers/name



Bitni koncepti:

- CircuiBreaker
- CircuitBreakerFactory
- RectiveCircuitBreaker
- ReactiveCircuitBreakerFactory
- @CircuitBreaker

### Blocking

Ovde koristimo `CircuitBreakerFactory` koja nam stvara CircuitBreaker,nju automatski dobijamo  kada podesimo .yml fajl. 

```xml
       <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
            <version>2.1.5</version>
        </dependency>

```

1) Sirovo koriscenje

   

   ```java
   public class Controller {
       @Autowired
       private CircuitBreakerFactory circuitBreakerFactory;
   
       @Autowired
       private Microservice2FeignClient microservice2FeignClient;
   
       @GetMapping("/test")
       public String test(){
          CircuitBreaker id = circuitBreakerFactory.create("id");
           id.create(SUPPLIER);
   
           return microservice2FeignClient.getUsers();
       }
   }
   ```
   

   Vidimo da mozemo da autowrire ovaj `CircuitBreakerFactory` uiz nje da pravimo sam CircuitBreaker.

   Ona ima jednu metodu `create("name")` i to stvara instasncu CircuitBreaker-a.

   

   <span style="color:red">circutiBreakerFactory.create("name")</span>

   i onda nad <span style="color:red">circuitBreakerInstance.create(SUPPLIER)</span>

2) Anotacije

   Ako hocemo sa anotacijama moramo da konfigurisemo u yml fajlu,ovo `resilience4j` ovo ostalo je za actuator 

   ```yml
   
   resilience4j:
     circuitbreaker:
         configs:
         default:
           register-health-indicator: true -> da bi se povezao na actuator,registruj se na actuator
       instances:
         myCircuitBreaker: -> ime instance tj ono ime u @CircuitBreaker(...)
           minimum-number-of-calls: 5 -> nadgledace 5 poziva da li je sve ok
           sliding-window-size: 10
           failure-rate-threshold: 50 -> ovo je 50% od 5 poziva gore (  minimum-number-of-calls) threshold
           wait-duration-in-open-state: 10000 -> koliko circuitbreaker mora da ceka da vidi da li je server ok
     timelimiter:
       instances:
         myCircuitBreaker: -> ime instance tj ono ime u @CircuitBreaker(...)
           timeout-duration: 2000
   management:
     health:
       circuitbreakers:   -> actuator
         enabled: true
     endpoint:
       health:
         show-details: always
     endpoints:
       web:
         exposure:
           include: health
   ```

   

   

```java
@Service
public class MyService {

    @CircuitBreaker(name = "myCircuitBreaker", fallbackMethod = "fallbackMethod")
    public String myMethod() {
        // method implementation
    }

    public String fallbackMethod(Exception e) {
        // fallback method implementation
    }

}
```

Ili cemo ovde u service da apply,ili cemo u `Controller` deo nad endpointom

i zatim moramo da ga config



![image-20230409010008422](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230409010008422.png)

Mozemo da vidimo da on ne dozvoljava da se posalju API pozivi kada se ispune oni uslovi u config-u ,to mozemo da vidimo u ovo polje `notPermitterCalls` ili da vidimo u konzoli aplikacije da se baca notpermitedexception a ne connection exception

### Non-Blocking

Ovde koristimo `ReactiveCircuitBreakerFactory` jer radimo sa reactive stvarimo,iz nje pravimo CircuitBreaker.

Prvo moramo da ubacimo dependency:

```xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
 </dependency>

i jer je deo spring cloud-a moramo 
<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```



1. Sirovo koriscenje

```java
@Service
public class BookService {


  private final WebClient webClient;
  private final ReactiveCircuitBreaker readingListCircuitBreaker;

  public BookService(ReactiveCircuitBreakerFactory circuitBreakerFactory) {
    this.webClient = WebClient.builder().baseUrl("http://localhost:8090").build();
    this.readingListCircuitBreaker = circuitBreakerFactory.create("recommended");
  }

  public Mono<String> readingList() {
    return readingListCircuitBreaker.run(webClient.get().uri("/recommended").retrieve().bodyToMono(String.class), throwable -> {
      LOG.warn("Error making request to book service", throwable);
      return Mono.just("Cloud Native Java (O'Reilly)");
    });
  }
}
```



Jer je reaective mormao da koristimo webflux i da napravimo webclient

<span style="color:red">reactiveCircuitBreaker.run(FLUX)</span>

<span style="color:red">reactiveCircuitBreaker.run(MONO)</span>

<span style="color:red">reactiveCircuitBreaker.run(FLUX,Function<Throwable,Flux>)</span>

<span style="color:red">reactiveCircuitBreaker.run(MONO,Function<Throwable,Mono>)</span>





1. Anotacije 

```java
public class Controller {


    @Autowired
    private Microservice2FeignClient microservice2FeignClient;
    @GetMapping("/test")
    @CircuitBreaker(name = "myCircuitBreaker")
    public String test(){


        return microservice2FeignClient.getUsers();
    }
}
```



## Retry Pattern

Ovo radi tako sto radi vise retry attempts kada service privremeno padne.Ovo obicno radimo kada je problem u networku,on nam tu pomaze.

config values:

- maxAttempts -> Max broj pokusaja
- waitDuration -> Period cekanja izmedju retry-a
- retryExceptions -> Ovo su Throwable classes koje se dobiju kada posaljemo network zahtev,ove uzima u obzir da bi uradio retry
- ignoreExceptions ->Ove ce da ignorise kada se dese

<span style="color:red">@Retry(name,fallbackmethod)</span>

Ovo dobijamo kada uvedemo resilience4 biblioteku iz poglavnja gore.

```java
public class Controller {


    @Autowired
    private Microservice2FeignClient microservice2FeignClient;
    @GetMapping("/test")
    @Retry(name = "myretry",fallback)
    public String test(){

        System.out.println("saljem");
        return microservice2FeignClient.getUsers();
    }
}
```

```yml
resilience4j:
  retry:
    instances:
      myretry:
        max-attempts: 3
        wait-duration:
          2000
    configs:
      default:
          register-health-indicator: true


```

Namestimo config u yml fajlu

Kada ovo pokrenemo i nas tako drugi service ne radi mi ako pozovemo ovo baci ce gresku,ali ne odmah vec ce pokusati 3 puta( jer smo tako stavili u max attempts) i to mozemo videti u konzoli jer ispisujemo samu metodu,videcemo da se na ekranu pozvali 3 puta ovo "saljem"

## Rate Limiter 

Stop overloading service sa vise api poziva,imamo mnogo requsta koji dolaze na nas API .

Stiti nas od DDOS napada

config values:

- timeoutDuration -> ovo je vreme cekanjakoje ce threads da cekaju kako bi mogle da obace api poziv,npra situacija je da imamo 100 poziva i to nam je limit sve preko toga ce ovo vreme da ceka
- limitForPeriod -> koliko requesta zelimo da primamo za neki period
- limitRefreshPeriod -> 



<span style="color:red">@RateLimiter(name,fallbackMethod)</span>

```xml

resilience4j:
  ratelimiter:
    configs:
      default:
        register-health-indicator: true
    instances:
      sayHello:
        timeout-duration: 5000
        limit-refresh-period: 5000
        limit-for-period: 1
```

```java
public class Controller {


    @Autowired
    private Microservice2FeignClient microservice2FeignClient;
    @GetMapping("/test")
    @RateLimiter(name = "myretry")
    public String test(){

        System.out.println("saljem");
        return microservice2FeignClient.getUsers();
    }
}
```

## Bulkhead

![image-20230409135855812](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230409135855812.png)

Omogucava nam da ovaj broj ima delove koji ako se natope vodom nece ceo brod da potone.

Ovako bi trebalo i mikroservisi da budu izolovani,samo deo ako padne da ne padne ceo.

Ovo je nad resursima

Bulkhead pattern nam omogucava da allocate limit resursa koji koriste neki servisi,da bi se smanjila potrosnja resursa

config values:

- maxConcurentCalls -> koliko paralelnih poziva dozvoljavamo
- maxWaitDuration -> vreme koje se ceka za niti 

<span style="color:red">@Bulkhead(name,fallbackMethod)![375f5cf4-1b89-4e54-af96-b0f07a9a26ea](C:\Users\radoj\Downloads\375f5cf4-1b89-4e54-af96-b0f07a9a26ea.jpg)</span>

Vidimo da ako nemamo bulkhead mi imamo 2 endpointa i endpoints funkcionisu tako sto se izdvaja thread za svaki poziv,moze da se desi da za jedan endpoint imamo previse thread-ova i da on bukvalno uzima sve resurse i ne ostavlja ovom account nista,to resava builkkhead

```yml

resilience4j:
  bulkhead:
    configs:
      default:
        register-health-indicator: true
    instances:
      sayHello:
       max-concurrent-calls: 5
       max-wait-duration: 10
```

```java
public class Controller {


    @Autowired
    private Microservice2FeignClient microservice2FeignClient;
    @GetMapping("/test")
    @Bulkhead(name = "myretry")
    public String test(){

        System.out.println("saljem");
        return microservice2FeignClient.getUsers();
    }
}

```

