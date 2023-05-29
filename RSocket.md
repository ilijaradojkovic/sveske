# RSocket

1. Request & Response
2. HTTP
3. Serialization & Deserialization
4. Blocking Thread

problem je sto kada hocemo da saljemo REST zahteve preko HTTP-ja mi uspostavljamo uvek 3way handshake i to je sporije i nije real time jer za svak irequest imamo request response

Socket je protokol koji koristi `persistance tcp connection`

Socket ima stalnu komunikaciju o update stvarima

Kada radimo sa request response uvek moramo da prebacujemo podatke u JSON,a u Socketu koristimo Binary formatu pa je brze



Ako koristimo Spring Web mozemo da dodjemo u situaciju da je thread blokirana,to resavamo sa WebFlux-om

Tako da je RSocket `Async` i `Non Blocking` 

![WhatsApp Image 2023-05-17 at 1.04.10 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-05-17 at 1.04.10 PM.jpeg)

Jer se stalno salje klijentu RSocket podrzava `Back pressure` to je da optimizuje load koji se salje klijentu koji ne moze da podrzi bas dobro

MI kod Socketa mi ne moramo svaki put da uspostavljamo konekciju vec samo 1 i ona ce se odrzavati.

Takodje mi u HTTP modelu za Request & Response imamo tacno sta je Client i sta je Server,ovde nemamo,jer i jedan i drugi element u komunikacijji mogu da budu i klijent i server,jer oba mogu da recive i send data.

Kao telefonski poziv pricas i slusas

`Peer to peer` je 

saljemo bytes preko TCP -ja 

1. Binary protocol
2. Peer to peer
3. Supports Reactive-Streams
   1. Async
   2. Non-Blocking
   3. Backpressure 
4. Supports
   1. TCP
   2. WebSockets
   3. Aeron(UDP)

Postoje modeli komunikacije u RSocketu

## Modeli Komunikacije

### Request-Response

### 																		![Group 8](C:\Users\Ilija\Desktop\sveske\images\Group 8.png)				

### Fire-Forget

Posaljemo `request` i ne zanima nas sta se desilo sa tim,da li je neko primio poruku ili ne

![Group 9](C:\Users\Ilija\Desktop\sveske\images\Group 9.png)



### Request-Streaming

Posaljemo `request` i druga strana periodicno salje  vise `odgovora`tokom vremena



​																 ![Group 10](C:\Users\Ilija\Desktop\sveske\images\Group 10.png)

### Fequest-Channel

ovo je kao Request-Streaming samo sa obe strane,zove se i bidirectional

![Group 811](C:\Users\Ilija\Desktop\sveske\images\Group 811.png)

|      RSocket       | Request:Response | HTTP | Example                                                      | Input Type | Output Type |
| :----------------: | :--------------: | :--: | ------------------------------------------------------------ | ---------- | ----------- |
| Request & Response |       1:1        | Yes  | Any typical request response.Create User                     | Mono<T>    | Mono<R>     |
|   Fire & Forget    |       1:0        |  X   | Sign out; Delete User; Logging; Raising an event             | Mono<T>    | Mono<Void>  |
|   Request Stream   |       1:N        |  X   | Search results; Order updates; Uber Driver location updates; Periodic data emission | Mono<T>    | Flux<R>     |
|  Request Channel   |       M:N        |  X   | Interactive communication like Game, Chat                    | Flux<T>    | Flux<R>     |



Da bi radili sa RSocket-om u Spring-u moramo dodati dependency:

```xml
  <dependency>
            <groupId>io.rsocket</groupId>
            <artifactId>rsocket-core</artifactId>
            <version>1.1.0</version>
        </dependency>
        <dependency>
            <groupId>io.rsocket</groupId>
            <artifactId>rsocket-transport-netty</artifactId>
            <version>1.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.11.2</version>
        </dependency>
```

Fora je sto nama Spring radi ove stvari automatski:

1. Spring auto configuration
2. Serialization & Deserialization
3. Dependency Injection
4. Exception Handling
5. SSL/TLS
6. Routing



Ovde dodajemo `@MessageMapping ` i to se ponasa kao Rest Controller i mozemo automatski da stavimo koj tip se vraca i koji se prima i on ce automatski da serialize and deserialize

```java
@MessageMapping("create.user")
public Mono<User> createUser(Mono<User> user){
	return ...;
}
```

Kada pravimo klasu gde su smestene ove metode obicno je anotacija @Controller a ne @RestController jer ne radimo REST,a mozemo i ceo kontroler da oznacimo sa @MessageMapping 

```java
@Controller
@MessageMapping("math")
public class MathCOntroller{

}
```

svaka ruta za message mapping ce imati ovo math ispred

Ne moraju metode striktno da primaju ovo bitno je da imaju @MessageMapping da bi se RSocket izvrsio

### Path variable

Mozemo i preko @MessageMapping-a da uradimo `path variable` i pisemo je u formatu `{ime}` i referenciramo je preko @DestinationVariable

```
@Controller
@MessageMapping("math")
public class MathCOntroller{

	@MessageMapping("pring.{input}")
	public Mono<Void> print(@DestinationVariable type name){
	
	}

}
```

### Exceptions

Bilo koji Exception da se baca u RSocketu ce se tretirati kao `ApplicationErrorException(Message)`,ovako ce po defaultu

1) Mozemo da emitujemo `error signal` i onda ce se i stream prekinuti

2) `defaultIfEmpty` `switchIfEmpty` `onErrorReturn` ovo mozemo koristiti nad Mono i Flux da ne bacamo exception

3) @MessageExceptionHandler 

   Mozemo napraviti metodu koja ce biti oznacena sa ovim i onda cee da handle 

   ```java
    @MessageExceptionHandler
    public Mono<Integer> handleException(Exception exception){
    	...
    }
   
   ```

```java
@MessageMapping("test.{input}")
    public Mono<Integer> doubleIt(@DestinationVariable int input){
        if(input<31)
            return Mono.just(2);
        else return Mono.error(new IllegalArgumentException("cant be "));
    }
   
    @Test
    void doubleIt() {
        Mono<Integer> integerMono = rSocketRequester.route("test.20")
                .retrieveMono(Integer.class)
                .doOnNext(System.out::println)
                ;



        StepVerifier.create(integerMono)
                .expectNextCount(1)
                .verifyComplete();
    }
    @Test
    void doubleItError() {
        Mono<Integer> integerMono = rSocketRequester.route("test.40")
                .retrieveMono(Integer.class)
                .doOnNext(System.out::println)
                ;



        StepVerifier.create(integerMono)
                .verifyError();
    }
```

```java
 @MessageMapping("test")
    public Mono<Integer> doubleIt(Mono<Integer> input){

     return   input.filter(
                x->x>10
        ).switchIfEmpty(Mono.just(-1));
    }
    
    
        @Test
    void doubleIt1() {
        Mono<Integer> integerMono = rSocketRequester.route("test")
                .data(Mono.just(2))
                .retrieveMono(Integer.class)
                .doOnNext(System.out::println)
                ;



        StepVerifier.create(integerMono)
                .expectNext(-1)
                .verifyComplete();
    }
```

```java
    @MessageMapping("test12")
    public Mono<Integer> doubleIt12(Mono<Integer> input){

     return    input.flatMap(x->{
            if(x<10){
                return Mono.error(new Exception());
            }
            return Mono.just(1);
        });
    }


@Test
    void doubleIt12Error() {
        Mono<Integer> integerMono = rSocketRequester.route("test12")
                .data(Mono.just(2))
                .retrieveMono(Integer.class)
                .doOnNext(System.out::println)
                ;



        StepVerifier.create(integerMono)
                .verifyError();
    }
    @Test
    void doubleIt12() {
        Mono<Integer> integerMono = rSocketRequester.route("test12")
                .data(Mono.just(12))
                .retrieveMono(Integer.class)
                .doOnNext(System.out::println)
                ;



        StepVerifier.create(integerMono)
                .expectNext(1)
                .verifyComplete();
    }
```



### Calling the client

Mi pozivamo kljenta preko `RSocketRequeser` klase koje mozemo da inject u metodu ili da je autowire

```java
rsocketRequester.route("ime rute u message mapperu") -> ovime pozivamo @MessageMapping metodu 
```

### Testing RSocket

Nije lako da se testira RSocket moramo pokrenuti `integracione` testove za ovo.

Mi cemo testirati pozive preko klase `RSocketRequester` nju moramo da napravimo kao bean pa da je autiwire u test klasi

```java
   @Autowired
    private RSocketRequester.Builder rSocketRequesterBuilder;

    @Autowired
    private RSocketMessageHandler rSocketMessageHandler;
    @Bean
    public RSocketRequester rSocketRequester(){
      return rSocketRequesterBuilder.rsocketConnector(c->c.acceptor(rSocketMessageHandler.responder()))
                .transport(TcpClientTransport.create("localhost",8080));
    }
```

za ovaj port sam namestio u application.yml

```yml
spring:
	rsocket:
		server:
			port=8080

```

I sada mozem oda autowire

i za testiranje samo koristimo klasu `StepVerifier`



```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@SpringBootTest
class RSocketControllerTest {

    @Autowired
    private RSocketRequester rSocketRequester;


    @Test
    void doubleIt() {
        Mono<Integer> integerMono = rSocketRequester.route("test.20")
                .retrieveMono(Integer.class)
                .doOnNext(System.out::println);

        StepVerifier.create(integerMono)
                .expectNextCount(1)
                .verifyComplete();

    }
}
```



#### Testing Fire-Forget

napisemo test:

```java
   @Test
    public void callbackTest() {
        Mono<Void> voidMono = rSocketRequester.route("test-fire-forget").data("").retrieveMono(Void.class);
        StepVerifier.create(voidMono).verifyComplete();

    }
```

za metodu:

```java
    @MessageMapping("test-fire-forget")
    public Mono<Void> test(Mono<String> mono){
        return Mono.empty();
    }
```

#### Testing Request-Stream

napisemo test:

```java
    @Test
    public void callbackTest() {
        Flux<Integer> integerFlux = rSocketRequester.route("test-request-stream").data("").retrieveFlux(Integer.class);
        StepVerifier.create(integerFlux).expectNext(1,2,3,4,5,6,7,8,9).expectComplete().verify();
    }
```

za metodu:

```java
  @MessageMapping("test-request-stream")
    public Flux<Integer> test1(Mono<String> mono){
        return Flux.just(1,2,3,4,5,6,7,8,9);
    }
```

#### Testing Request-Channel

napisemo test:

```java
    @Test
    public void callbackTest() {
        Flux<Integer> integerFlux = rSocketRequester.route("test-request-channel").data(Flux.empty()).retrieveFlux(Integer.class);
        StepVerifier.create(integerFlux).expectNext(1,2,3,4,5,6,7,8,9).expectComplete().verify();
    }

```

za metodu:

```java
   @MessageMapping("test-request-channel")
    public Flux<Integer> test12(Flux<String> flux){
        return Flux.just(1,2,3,4,5,6,7,8,9);
    }
```

## Connection Stup

Kada client zeli da se konektuje na server koristimo `@ConnectionMapping(route)`,ovo uvek vraca void .

Uvek postoji @ConnectMapping koji je default,mi mozemo da redefinisemo to,ali uvek postoji neki koji je prazan i na koji svi idu osim ako ne kazemo drugacije

```java
@ConnectionMapping
public Mono<Void> handleConnection(){
	return Mono.empty();
}
```

Kada mi hocemo da gadjamo neki socket messagemapping on ce pre toga da ode u ovo ConnectionMapping 

i to ce se izvrsiti pre savog uspostavljanja konekcije 1 

znaci ako gadjamo request-stream mi cemo jedno mda ostvarimo konekciju pa ce jednom da se izvrsi ConnectionMapping samo,posle ce messageMapping normalno da radi

U slucaju da imamo rutu u ConnectionMapping moramo da koristimo  `setupRoute("...")`

```
    @Bean
    public RSocketRequester rSocketRequester(){

        
      return rSocketRequesterBuilder
      		.setupRoute("...")
          .rsocketConnector(c->c.acceptor(rSocketMessageHandler.responder()))
                .transport(TcpClientTransport.create("localhost",8080));
    }
```

Ovaj slucaj je bio kada ovo handleConnection ne prima nista od objekata,ali moze da primi

```java
@ConnectionMapping
public Mono<Void> handleConnection(ConnetionRequestObject obj){
	return Mono.empty();
}
```

ali moracemo ovde da ubacimo taj objekat i da ga namestimo kada se pravi `RSocketRequester`

```java
    @Bean
    public RSocketRequester rSocketRequester(){
        ConnetionRequestObject obj=new ConnetionRequestObject("test","test");
        
      return rSocketRequesterBuilder
          .setupData(obj)
          .rsocketConnector(c->c.acceptor(rSocketMessageHandler.responder()))
                .transport(TcpClientTransport.create("localhost",8080));
    }
```

Imamo foru da hocemo da se konekcija prekine odmah ako se objekat koji nije poslao nije dobar.

Znaci hocemo da ako je password empty mi zelimo odmah da zatvorimo kanal

```java
@ConnectionMapping
public Mono<Void> handleConnection(ConnetionRequestObject obj,RSocketRequester rsocketRequester){
return obj.getPassword().isEmpty? Mono.fromRunnable(()-> rSocketRequester.rsocketClient().dispose()) : Mono.empty();
}
```

## ConnectionManager

```java
@Service
public class MathClientManager{
	private Set<RSocketRequester> set=Collections.synchronizedSet(new HashSet<>());
	
	public void add(RSocketRequester requester){
		rsocketRequester.rsocket()
            .onClose()
            .doFirst(()->this.set.add(requester))
            .doFinally(s->this.set.remove(requester))
            .subscribe();
       	}
    
    public void print(){
        System.out.println(set);
    }
    public void notify(int i){
        Flux.fromIterable(set)
            .flatMap(r->r.route("...").data().send())
            .subscribe();
    }
}
```

i sada cemo ovog menadjera da autowire u ConnectionMapping da bi dodavali konekcije

```java
@Autowire
private MathClientManager clientManager;

@ConnectionMapping
public Mono<Void> handleConnection(RSocketRequester rsocketRequester){
	return Mono.fromRunnable(()->this.clientManager.add(rsocketRequester));
    
}
```

i sada kada se svaki klijent konektuje mi cemo da cuvamo tu konekciju

I sada cemo u testu da napravimo 2 konekcije i gadjamo 2 messageMapping-a da bi se te konekcije registrovale i dodale 

## Connection Retry

Sta se desavan kada nam je server down pa ne moze da se konektuje

```java
    return rSocketRequesterBuilder.rsocketConnector(c->c.acceptor(rSocketMessageHandler.responder()))
        .rsocketConnectior(c->c.reconnect(Retry
                                          .fixedDelay(10,Duration.ofSeconds(2))
                                          .doBeforeRetry(s->...)
                                         )
                          )       
        .transport(TcpClientTransport.create("localhost",8080));
```

Ovo reconnect nam sluzi kada se salju novi responsovi,npr pokrenemo da saljemo nesto a server nije up tako da ce on da retry jer je taj novi request koji pokusava da se posalje,ali ako imammo da smo mi vec slali i server se ugasi onda ce ovo da pukne jer ovo reconnect radi samo ako je nov request a mi smo neke vec poslali

za to koristimo `resume` to je nesto sto backend mora da support

```java
@Bean
public RSocketServerCustomizer customizer(){
	return c->c.resume( resumeStrategy());
}
private Resume resumeStrategy(){
	return new Resume()
				.sessionDuration(Duration)
}
```

Ako se klijent diskonektuje onda ce server da ceka i drzace session otvorenim

Takodje mi moramo da provide Resume strategy za klijenta takodje

```java
    @Bean
    public RSocketRequester rSocketRequester(){
      return rSocketRequesterBuilder
          .rsocketConnector(c->c
                            .resume(resumeStrategy()))
                .transport(TcpClientTransport.create("localhost",8080));
    }
    
    private Resume resumeStrategy(){
    	return new Resume().retry(...);
    }
```



## Session Resumption

Ovo je slucaj kada internet nestane sta se onda desi

## Load Balancing

Da balansiramo pozive ka soketira 

## RSocket Secutiry

### SSL/TLS

Treba nam sertifikat  koji moramo da kupimo,ali cemo mi da genersimo nas za test deo

keytool dolazi sa javom

1. Generisemo key pair 

```
keytool -genkeypair -alias rsocket -keyalg RSA -keysize 2048 -storetype PKCS12 -validity 3650 -keystore rsocket-server.p12 -storepass password
```

organizatioal unit -> localhost

2.

```
keytool -exportcert -alias rsocket -keystore rsocket-server.p12 -storepass password -file cert.pem
```

3.

```
keytool -importcert -alias rsocket -keystore client.truststore -storepass password -file cert.pem
```

ovo je za klejnta ,browser 

sada u Spring-u moramo da omogucimo `SSL`

```
spring:
	rsocket:
		server:
			ssl:
				enabled: true
				key-store-type: PKCS12
				key-store: location of keystore location(rsocket-server.p12) prvo sto smo generisali
				key-store-password: password
				key-alias: rsocket(iz koraka 3)
```

Njemu nesto nije htelo lokalno pa je morao da podesi da mu veruje 

```
static{
	System.setProperty("javax.net.ssl.trustStore","path")
		System.setProperty("javax.net.ssl.trustStorePassword","password")
}
```

Na klijentskom delu pre smo slali direktno na adresu i port,ali sda jer imamo SSL koristimo nesto TcpClientTransport

```
RsocketBuilderInstance.transport(TcpClientTransport.create(TcpClient.create.host("..").port().secure()))
```

## RSocket Auth

```
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-messaging</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-rsocket</artifactId>
		</dependency>
```

treba nam ovo 

Kada radimo sa RSOcketom isto pravimo UserDeatils i UserService gde imamo metodu findByUsername samo se razlikuje config samo sto mora da implementira `ReactiveUserDetailsService`

U user service koji mi treba da napravimo nisam to kopirao od njega,samo iz neke lokalne liste tipa itd

tu su nam i roles

```java
@COnfiguration
@EnableRSocketSecutiry
public class RSocketSecurity{
	@Bean
	public PayloadSocketAcceptorInterceptor interceptor(RSocketSecurity secutiry){
		return security
            .simpleAuthentication()
            .authorizePayload(authorize->{
                              //kada se radi onaj setup deo da se 												radi auth
                             authorize.setup().permitAll();
                              authorize.anyRequest.permitAll();
             return authorize;                    
            }).build();
    }
    
    //Spring dolazi po defaultu sa nekim encoders i decoders pa ne moarmo stalno da menjamo ovo,a ovo radimo jer mi saljemo UsernamePassword metadata i on ne zna da decode to
    @Bean
    public RSocketStrategiesCustomizer st(){
        return c-> c.encoder(new SimpleAuthenticationEncoder());
    }
}
```

i client mora da predamo u metadata,jer u restu je header,a mi ovde u Metadata stavljamo

MimeType->What type of object it is

Sve mimeType mozemo da vidimo preko  enuma `WellKnownMimeType` i tu udjemo i vidimo

`MimeTypeUtils.parseMimeType`



```java
UsernamePasswordMetadata credentials =new UsernamePasswordMetadata("username","password");
RsocketBuilderInstance
.setupMetadata(credentials,MimeTypeUtils.parseMimeType(WellKnownMimeType.MESSAGE_RSOCKET_AUTHENTICATION))
.transport(TcpClientTransport.create(TcpClient.create.host("..").port().secure()))
```

Kada uradimo Setup level auth mi cemo pri konekciji da uradimo auth i to bi bilo samo jednom,takodje mozemo da uradimo auth svaki poziv na messagemapping



 ![WhatsApp Image 2023-05-24 at 1.24.53 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-05-24 at 1.24.53 PM.jpeg)

npr ako se povezuje direktno klijent onda bi bilo ok da radimo samo setup auth,ali ako imamo drugi mikroservis koji se povezuje onda cemo morati pri svakom requestu da radimo,jer mikroservis ima vise klijenata i ne mozemo mu dati sve da ispuni jer su klijenti sa razlicitim dozvolama

Da bi pristupili metadata koji se salje koristimo `MessageHeaders`

```java
   @MessageMapping("your.mapping")
    public Mono<Response> handleRequest(@Payload Request request, MessageHeaders messageHeaders) {
        // Access the metadata from the messageHeaders
        Object metadata = messageHeaders.get("your-metadata-key");
        // ...
    }
```

