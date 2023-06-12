# Spring WebFlux

## What is Project Reactor?

Projekat Reactor je fremework ko je napravio spring.On implementira reactive API patterne,najpoznatiji Reactive Streams.Ako smo vec upoiznati sa Java 8 Streams brzo cemo se navici na ovo,postoji velika slicnost izmedju Stream i Flux(ili njegova pojedinacna verzija Mono).Glavna razlika ovih esencijalnih klasa reaktivnog programiranja je da je mono samo 1 objekat,dok je flux vise objekata.Oba ova objekta prate publisher-subscriber,i implementiraju backpressure,dok Java Stream API ne .



**Backpressure**  je nacin da se signalizira klijentu da salje previse podataka i ukine to slanje.Dozvoljava bolje flow managmenet i onemogucava da se server kresuje napadima kao sto su DDOS.



![image-20221207103436204](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20221207103436204.png)

![spring-mvc-and-webflux-venn](C:\Users\Ilija\Downloads\spring-mvc-and-webflux-venn.png)



Spring WebFlux je potpuno non-blocking,anotation-based vec framework koji je napravljen na Project Reactoru,zato nam omogucava da pravimo reaktivne aplikacije na HTTP sloju.Rektivno programiranje je suportovano na servirima koa sto je netty ali i tomcat.

Spring WebFlux takodje koristi funkcionalno programiranje za rad,ali ne mora.

![image-20221217011114862](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221217011114862.png)

Ovako je bilo bez ovoga da je reactive,svaki request se dodeljuje na 1 thread

![image-20221217011211993](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221217011211993.png)

Nemamo 1 na 1 request i thread,vec se salje u event loop koji radi posao u pozadini,to je svrha ovoga

# Router functions

RouterFunction je drugi nacin pisanje rekativnih ruta,takodje mozemo koristiti i anotacije 

<u>Tradicionalan pristup</u>

```java
@RestController
public class ProductController {
    @RequestMapping("/product")
    public List<Product> productListing() {
        return ps.findAll();
    }
}
```

Functional pristup

```java
@Bean
public RouterFunction<ServerResponse> productListing(ProductService ps) {
    return route().GET("/product", req -> ok().body(ps.findAll()))
      .build();
}
```



You can use the `RouterFunctions.route()` to create routes instead of writing a complete router function. Routes are registered as Spring beans and therefore can be created in any configuration class.



RequestMapping` and `Controller` annotation styles are still valid in WebFlux if you are more comfortable with the old style, `RouterFunctions` is just a new option for your solutions.

# WebClient

WebClient je reaktivni web klijent WebFlux-a koji je nasledio poznat `RestTemplate`.Ovo je  interfejs koji predstavlja glavnu ulaznu tacku web requets i podrzava sinhronizovano i asinhronizovane operacije.Najvise se koristi za .reactive backend-to-backend communication

Koristimo ga da saljemo neke request preko koda,jedan backend koristi da posalje request drugom bekendu.

Kada import `spring-boot-starter-webflux` on dolazi sa tim.

```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
WebClient client = WebClient.create();
```

Pri kreiranju WebClienta mozemo upotrebiti i builer njegov

```java
WebClient.builder()
    .filter(ExchangeFilterFunction) -> Promeni request ili response 
    .apply(Consumer) -> Sluzi da apply vise konfiguracionih opcija 
    .build()
```

Apply nam slizi da configure webcient a filter da promenimo request ili response

#### ExchangeFilterFunction 

Ovo je interfejs koji ima metode za presretanje requeta i responsa i na osnovu njega se nesto apply.

Sluzi nam da menjamo reqquest ili response na neki nacin.Mozemo napraviti 

nasu klasu koja ce ovo da import

```java

public class MyFIlter implements ExchangeFilterFunction {
    @Override
    public Mono<ClientResponse> filter(ClientRequest request, ExchangeFunction next) {
        return null;
    }

    @Override
    public ExchangeFilterFunction andThen(ExchangeFilterFunction afterFilter) {
        return ExchangeFilterFunction.super.andThen(afterFilter);
    }

    @Override
    public ExchangeFunction apply(ExchangeFunction exchange) {
        return ExchangeFilterFunction.super.apply(exchange);
    }
}
```

 ili mozemo preko lambde

```
WebClient webClient = WebClient.builder()
  .filter((request, next) -> {
    request.headers().add("Authorization", "Bearer " + accessToken);
    return next.exchange(request);
  })
  .build();
```





RestTemplate je depricated,pa se za sve koristi WebClient.



webClientInstance.get() ->Poziva get metodu

webClientInstance.post() ->Poziva postmetodu

webClientInstance.delete() ->Poziva deletemetodu

webClientInstance.put() ->Poziva put metodu

webClientInstance.patch() ->Poziva patch metodu

​											.uri("uri") ->Dajemo uri kako bi gadjao taj backend deo koda,tj taj 																																	endpoing	

.bodyValue(object) -> ovo postavlja neki obj u body koji saljemo,ovo je kao body za post,payload

.attribute(key,val)

.header(key,value)

.contentyType(MediaType)

.contentLength(long)

.accept(MediaType)

.retrive() -> ovo izvrsava request 

.bodyToMono(class) ->Moze i Void.class

.bodyToFlux(class)  ->Moze i Void.class

.toEntity(class)





.exchangeToMono(Function<Request,Mono>) 

.exchangeToFlux(Function<Request,Flux>)

 `exchangeToMono()` and `exchangeToFlux()` methods  are useful for more advanced cases that require more control, such as to decode the response differently depending on the response status



.block() -> ova metoda blokira poziv,nikako ne bi trebalo da radimo ovo ako zelimo reaktivno ili ako je reaktivni tip povratna vrednost

```java
public List<Student> getStudents() {
       try {
           ResponseEntity<Student[]> response = restTemplate.getForEntity("https://decode.agency:8080/students", Student[].class);
           return Arrays.asList(response.getBody());
       } catch (RestClientException ex) {
           throw new StudentException(ex);
       }
   }
```

Ovakav nacin pisanja koji je blokirajuci je bio pre,vidimo da se koristi restTemplate koji je sada depricated.

```java
public Flux<Student> getStudentsWithClient() {
       return webClient
               .get()
               .uri("https://decode.agency:8080/students")
               .retrieve()
               .bodyToFlux(Student.class)
               .onErrorMap(StudentException::new);
   }
```

Instead of a list, the term Flux appears this time. Spring WebFlux uses Reactive streams and introduces composable API types for communication, Mono, and Flux. The difference between the two types is in the length of the data sequence. Mono stores a sequence of 0..1, while Flux may contain a range of 0..N.

Keep in mind that a blocking method should never be called within a method that returns a reactive type. That would block one of the few threads of the application running on Netty and cause bad consequences.



Spring MVC assumes threads will be blocked and uses a large thread pool to keep moving during instances of blocking. This larger thread pool makes MVC more resource-intensive as the computer hardware must keep more threads spun up at once.

WebFlux instead uses a **small thread pool** because it assumes you’ll never need to pass off work to avoid a blocker. These threads, called **event loop workers**, are fixed in number and cycle through incoming requests quicker than MVC threads. This means WebFlux uses computer resources more efficiently because active threads are always working.



Kako WebFlux koristi po defaultu Netty server,mi mozemo da izbrisemo spring-starter-web jer on koristi Tomcat,prirodnije je da koristimo Netty.



I mono i flux implementuju interfejse 

```java
public interface Publisher<T> {  public void subscribe(Subscriber<? super T> s); }

public interface Subscriber<T> {   public void onSubscribe(Subscription s);  public void onNext(T t);  public void onError(Throwable t);  public void onComplete(); }

public interface Subscription<T> {
  public void request(long n);
  public void cancel();
}
```

Subscription je veza izmedju subscriber-a i publisher-a



First of all, you should never block within a reactive pipeline. You should **subscribe** instead. In that particular case, the Spring Webflux framework will subscribe for you as long as you provide your publisher. To achieve this, the controller method has to return your `Mono` publisher like this:

Mono<Value>





If you use `spring-boot-starter-webflux` the configuration is automatically done via `ReactiveWebServerFactoryAutoConfiguration` and `WebFluxAutoConfiguration`, so you don't need `@EnableWebFlux`

When you use `spring-webflux` without `spring-boot` you need to add `@EnableWebFlux` on a `@Configuration` class to import the Spring WebFlux configuration from `WebFluxConfigurationSupport`





When we return a Flux from a service endpoint many things can happen. But I assume you want to know what is happening when Flux observed as stream of events from client of this endpoint.

**Scenario One**: By adding 'application/json' as the content type of the endpoint Spring will communicate to the client to expect JSON body.

```
@GetMapping(value = "/async-flux", produces = MediaType.APPLICATION_JSON_VALUE)
public Flux<String> handleReqDefResult1(Model model) {
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 10000; i++) {
        list.add(String.valueOf(i));
    }
    return Flux.fromIterable(list);
}
```

The output at the client will be the whole set of numbers in one go. And once the response delivered the connection will be closed. Even though you have used Flux as the response type, you are still bound the laws of how HTTP over TCP/IP works. The endpoint got a HTTP request, execute the logic and respond with HTTP response containing final result. [![Result delivered as single response](https://i.stack.imgur.com/L78T2.png)](https://i.stack.imgur.com/L78T2.png)

As a result, you do not see the real value of a reactive api.

**Scenario Two**: By adding 'application/stream+json' as the content type of the endpoint, Spring starts to treat the resulting events of the Flux stream as individual JSON items. When an item is emitted is gets serialised, the HTTP response buffer is flushed, and the connection from the server to client keep open up until the event sequence get completed.

To get that working we can slightly modify your original code as follows.

```
@GetMapping(value = "/async-flux",produces =  MediaType.APPLICATION_STREAM_JSON_VALUE)
public Flux<String> handleReqDefResult1(Model model) {
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 10000   ; i++) {
        list.add(String.valueOf(i));
    }
    return Flux.fromIterable(list)
            // we have 1 sec delay to demonstrate the difference of behaviour. 
            .delayElements(Duration.ofSeconds(1));
}
```

This time we can see the real value of reactive api endpoint where it is able to deliver results to it's client as date get available.



*Reactive programming does not make application faster, but it allows the application to serve more clients with less resource.*



The database operations are blocking I/O calls, JPA, JDBC both behave this way.

You might wonder why MongoDB, why not MySQL of any other relational data bases. This is because in February 2019, at the time I am writing this article JDBC implementation of Java, still use the ***blocking manner\*** of communicating with relational databases. We can not get the benefit of using reactive programming if our data source does not support non-blocking way of data communication.

Ovo gore sto smo koristili produces je u stvari logika koju koriste aplikacije kao sto su taxi app,ili glovo za slanje lokacije gde je vozilo npr gde ce server slati podatke u nekom intervalu to je SSE(Server Sent Events) gde server u toku vremena salje podatke

```
GetMapping(value = "/async-flux",produces = MediaType.APPLICATION_STREAM_JSON_VALUE)
public Flux<String> handleReqDefResult1(Model model) {
return Flux.interval(Duration.ofSeconds(1));
}
```

## Sinks

Kreiramo je pomocu klase <span style="color:red">Sinks</span> ona ima staticke metode za rad sa njima

```
Sinks.Empty -> ovo kreira empty sink sto znaci da nemao next value u njemu vec samo signale za end i kraj
Sinks.One -> Stvara sink koji moze samo 1 value da proizvede,mada moze i 0,ali max je 1
Sinks.Many -> Proizvodi vise value
Sinks.many().replay() -> subscriberima ce poslati sve podatke ili mi ako ogranicimo
                    .multicast() -> samo nove podakle ce da posalje subscriberima
sinkObject.emitError(Throwable,EmitFailureHandler)
sinkObject.tryEmitError(Throwable)
Throwable -> exception koji emit
 <span style="color:red">EmitFailureHandler </span>-> interfjes koji handle ako sink fail da emituje event,zato ovo tryEmitError nema taj EmitFailureHandler
sinkObject.emitNext(object,EmitFailureHandler)
sinkObject.tryEmitNext(object)
sinkObject.emitComplete(EmitFailureHandler)

```

### EmitFailureHandler

Imamo predefinisane ove handler-e

```
Sinks.Many<Integer> moviesInfoSink = Sinks.many().replay().all();
```

```java
@PostMapping("/movieinfos")
public Mono<MoiveInfo> addMovieInfo(@ReqeustBody @Valid MoviewInfo movieInfo){
    return moviesInfoService.addMovieInfo(movieInfo) -> service ce nam vratiti Mono pa mozemo primeniti mono operacije
                .doOnNext(savedInfo-> moviesInfoSink.tryEmitNext(savedInfo));
                
                
                
@GetMapping("/movieinfos/stream",MediaType.APPLICATION_NDJSON_VALUE)
public Flux<MovieInfo> getMoveInfo(){
    return moviesInfoSink.asFlux();
}
```

Da bi radili sa sinkom jer on salje neprestano update podakte kada mu stigne event mi moramo da raidmo sa njim kao da vracamo stream zato imamo mediatype

## Reactive Mongo

Da bi ucinili bazu reaktivnom samo dodamo da repository bude reactive,ovo je moguce kod nosql baza u trenutku jer je jdbc blokirajuci.

```java
public interface BookRepository extends ReactiveMongoRepository<Book, String> {}
```

<span style="color:red">ReactiveMongoRepository<...></span>  -> vidimo da imamo reaktivan repository

Note that this repository extends the **ReactiveMongoRepository** it provides reactive support for MongoDB.

 **MongoRepositry** which return java.util.List<T>, reactive equivalent return Mono and Flux where applicable.

## Mono

Ovo je osnovna klasa za asinhroni rad u springu.Ova klasa oznacava da ce se 0 ili 1 objekat poslati nazad asinhrono.

```java
Mono.just(data) 					-> kreira Mono objekat na osnovu data
    .empty()    					-> kreira Mono objekat da bude prazan						
    .exception(Throwable)             -> baca exception
    .fromFuture(Future)				 -> kreira Mono na osnovu Future objekta
    .fromRunnable(Runnable)           -> kreira Mono na osnovu Runnable objekta
    .justOrEmpty(Data)                -> ako ne kreira objekat na osnovu Data kreirace prazan
    .never()                          -> nikad ne vrati,ide u beskonacno
    .delay(Duration)                  -> stvara delay
    
    
MonoInstance.doFinaly(Consumer)       -> uradice na kraju ovaj consumer funkcionalni 					                                     interfejs,izvrsice se
  		    .doFirst(Runnable)
            .doOnError(Consumer)
            .doOnCancle(Runnable)
            .doOnSuccess(Consumer)
            .doOnTerminate(Runnable)
 Redosled izvrsavanja je doFrist->doOnSuccess->doOnFinally
            
    
             .cache()                      ->Stvara kes instancu u pozadini,pa svaki put kada hocemo 										  taj objekat koji ima taj data u Mono on ce da vrati kes 									        instancu,time ce ubrzati proces	
             .cache(Duration)
    		.cacheInvalidateIf(Predicate)
    
    		.thenReturn(R)                 ->Vraca ovu vrednost na kraju,override ostale
             .cast(Class)                   ->Kastuje u klasu
             .flux()                        ->Kastuje u Flux
             .defaultIfEmpty()              ->
    		.timeout(Duration)
     		.retry(integer)
   			.subscribe(Consumer<ValueOfMonoOrFlux>)
    
    
```



## Flux

Ovo je objekat koji prima 0..M drugih,on time emituje te druge objekte kroz vreme.

```java
Flux.just(data...) -> Kreira flux na osnovu podataka ...
	.fromIterable(Iterable) -> Kreira Flux na osnovu itreable neke
    .range(start,end) -> Kreira Flux na sonovu int-a redom od start do end inkrementalno
    .from(Publisher<T>) -> Kreira Flux na osnovu nekkog publishera to moze biti Flux ili Mono isto
    .generate(initialState,generator)
   
```



```
 
    		generate(
    			()->0, 							-> initial state 
    			(state,sink) ->{				-> State nam je kao sta generisemo,taj tip,i 														predstavlja prethodnu vrednost
    											 Sink nam omogucava da se pomeramo izmedju state
                    sink.next(state);			-> Moramo imati sink.next kako bi se kretali kor 														sink
                    if(state==9){
                        sink.complete();
                    }
                    return state+1;
                		}
                    )
                    
                    
=====================================================================================================
                     Flux<Game> generate = Flux.generate(
                            () -> new Game(),
                            (integer, synchronousSink) -> {
                                synchronousSink.next(integer);
                                Game game = new Game();
                                game.setId(UUID.randomUUID().toString());
                                return game;
                            }
        );
        generate.concatMap(x->Flux.just(new GameModel(x.getId(),Math.random(),Math.random()),new 								GameModel(x.getId(),Math.random(),Math.random())))

				.delayElements(Duration.ofSeconds(1)).subscribe(System.out::println);
				
	Na svaku jednu generisanu igru generisace 2 GameModel-a i tako u beskonacno

```

```java
Flux.interval(Duration) -> izvrsava Flux na svakih Duration vremena
    empty() -> kreira prazan
    combineLatest(FluxSource1,FluxSource2,Function for combining values) -> Ovo radi tako sto mi prosledimo flux-ove i onda on uzima zadnju vrednost za fluxove i kombinuje ih pomocu ove funkcije
	
```

```
  		Flux<Integer> source1 = Flux.just(1, 2, 3);
        Flux<String> source2 = Flux.just("A", "B", "C");
        Flux<Double> source3 = Flux.just(0.1, 0.2, 0.3);

        Flux.combineLatest(source1, source2, source3,
                (num, str, dbl) -> num + "-" + str + "-" + dbl)
                .subscribe(System.out::println);
```

Mozemo proslediti vise fluxova,birno je da os source1 i souce2 uzima zadnju vrednost a ovaj zadnji source uzima sve redom



```
Flux.concat(publishers...) -> spaja vise publishera zajedno
```

```
   Flux<Integer> source1 = Flux.just(1, 2, 3);
        Flux<Integer> source2 = Flux.just(4, 5, 6);
        Flux<Integer> source3 = Flux.just(7, 8, 9);

        Flux.concat(source1, source2, source3)
                .subscribe(System.out::println);
         
        1,2,3,4,5,6,7,8,9
        
```

```java
Flux.create(custom defined producer)-> kreiramo i imamo vecu kontrolu sta se salje ,jer mi definisemo 									  sta se salje
```

```
  Flux<Integer> customFlux = Flux.create(emitter -> {
            emitter.next(1);
            emitter.next(2);
            emitter.next(3);
            emitter.complete();
        });
        ali moramo subscribe da bi radilo
```

```
Flux.defer(Supplier)-> Ovo lazily kreira Flux,odlaze kreiranje fluxa.
```

```
   Flux<Integer> deferredFlux = Flux.defer(() -> {
            int randomNumber = (int) (Math.random() * 100);
            return Flux.just(randomNumber);
        });
```

```
Flux.fromStream(Stream)
```

```
  List<String> fruits = Arrays.asList("Apple", "Banana", "Orange");

        Stream<String> fruitStream = fruits.stream();

        Flux<String> fruitFlux = Flux.fromStream(fruitStream);
```

```
Flux.merge(fluxes...)
```

```
 	    Flux<Integer> flux1 = Flux.just(1, 2, 3);
        Flux<Integer> flux2 = Flux.just(4, 5, 6);

        Flux<Integer> mergedFlux = Flux.merge(flux1, flux2);  
```

Merge radi tako sto spaja flux i salje elemente cim budu dostupni,dok concat salje cim svi budu dostupni



Kada napravimo instancu fluxa mi mozemo da dodajemo neke funckije:

```
	FluxIntance.delayElements(Duration) -> Svaki element delay
		  .map(...)
		  .delayUntil(Funtion<FluxElementType,Publisher>) -> delay emisiju elementa sve dok se 																provajdovan mono ne emituje singal
```

```

  	Flux<Integer> flux = Flux.range(1, 5);
        Mono<Void> delayMono = Mono.delay(Duration.ofSeconds(2)).then();

        Flux<Integer> delayedFlux = flux.delayUntil(signal -> delayMono);

        delayedFlux.subscribe(System.out::println);
```



```
FluxInstance.disticnt()
		   .distinct(pocemu)
```

```
FluxInstance.flatMap(function<fluxelement,Publisher>) ->Emituje za svaki element Publisher
```

```
 .flatMap(ignore ->  {
     return redisTemplate.opsForZSet().rangeWithScores(Configuration.currentPoints,Range.unbounded());

               })
               
               
  Flux.just(1,2,3).flatMap(x->Flux.just(x,-1)) emitovace 
  1 -1 2 -1 3 -1 vidimo da sve emituje iz ovo just dela 
  ova funkcija nam omogucava da emitujemo vrednosti `paralelno` i redosled nije zagarantovan,takodje ovo je parallel.Po defaultu nije parallel ali moze da se uvede to nije moguce da uradimo na concatmap
  
   Flux<Integer> flux = Flux.range(1, 5);

        Flux<String> result = flux.flatMap(number -> {
            return Flux.just(number)
                    .map(n -> "Processed: " + n)
                    .subscribeOn(Schedulers.parallel());
        });
```

On ce da raspakuje taja Mono ili Flux koji vraca 

```
FluxInstance.repeat()
			.repeat(BooleanInstance); ponavlja izvrsavanje fluxa
```

```
repeat(()->boolean)
```

```
FluxInstance.take(number_of_elements)
			.takeWhile(Predicate)
			.takeLast(number)
			.takeUntill(Predicate)
```

```
FluxInstance.subscribe() -> ovo pocinje flux
			.subscribe(
			consumer -> uradi na svaki element emitovan
			,errorConsumer -> uradi ako se desi error
			,completeConsumer
			); -> uradi ako se zavrsi
			
			
```

```
 flux.subscribe(
            element -> System.out.println("Element: " + element),
            error -> System.err.println("Error: " + error),
            () -> System.out.println("Completed")
        );
```



```
FluxInstance.concatMap(Function<FluxElementType,Flux>) -> transformisi svaki element koji se emituje u flux
```

```
Flux.just(1,2,3).concatMap(x->Flux.just(x+20));

ispisace 21 22 23 
flatMap bi ispisala i brojeve u just(...) delu
ova funkcija nam omogucava da emitujemo vrednosti `sekvencijalno` i redosled je zagarantovan,takodje ovo nije parallel
```



```
FluxInstance.materialize() -> ovo ceo flux pretvara u FLux<Singal<nasTip>> sto znaci da ce se svaki event u fluxu koji su bili razliciti sada pretvoriti u jedan signal
```

```
Flux<Signal<Integer>> materializedFlux = flux.materialize();

        materializedFlux.subscribe(signal -> {
            if (signal.getType() == SignalType.ON_NEXT) {
                System.out.println("Value: " + signal.get());
            } else if (signal.getType() == SignalType.ON_ERROR) {
                System.err.println("Error: " + signal.getThrowable());
            } else if (signal.getType() == SignalType.ON_COMPLETE) {
                System.out.println("Completed");
            }
        });
```

```
FluxInstance.dematerialize()

Flux<Signal<Integer>> flux = Flux.just(
                Signal.next(1),
                Signal.next(2),
                Signal.next(3),
                Signal.complete()
        );

        Flux<Integer> dematerializedFlux = flux.dematerialize();

        dematerializedFlux.subscribe(
                value -> System.out.println("Value: " + value),
                error -> System.err.println("Error: " + error),
                () -> System.out.println("Completed")
        );
```

```
FluxInstance... then(mono) -> kad se sve zavrsi vrati ovaj Mono

     Flux<Integer> flux = Flux.just(1, 2, 3);

        flux
            .doOnComplete(() -> System.out.println("Flux completed"))
            .then(Mono.fromRunnable(() -> System.out.println("Action executed")))
            .subscribe();
    }
    
    Flux completed
Action executed
```

```
FluxInstance.subscribeWith(Subscriber) -> na postojeci subscriber se suba
```

```
FluxInstance.buffer(max) -> ovo ce uzimati max elemenata koji se emituju i ujedinice ih u listu i to 												ce da vraca sada 

Flux.range(1, 10)
    .buffer(3)
    .subscribe(System.out::println);
    
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
[10]
FluxInstance.buffer(Duration) -> emitovace na osnovu vremena,duration time window

FluxInstance.buffer(Publisher) -> Svaki put kada publisher emituje signal on ce da pokupi elemente u listu

```

```
FluxInstance.cache() -> kesira da se ne vraca sve opet
			.cache(duration)
			.cache(int history)
```

```
FluxInstance.checkpoint(String) -> debug
```

```
FluxInstance.collectList() -> vraca listu
			.collect(Collector)
			.collectSortedList()
			.collectSortedList(Comparator)
			.collectMap(Function<FluxElementType,Key>,Function<FluxElementType,Value) -> imamo value 																			i key mapper-e
```

```
FluxInstance.delaySequance(Duration) -> emituje sve posle DUration sekundi,ne svaki pojedinacno
			.delaySequance(Publisher) -> na osonovu vracanje vrednosti publishera radi delay
```

```
FluxInstance.window(size) -> Ovo ce automatski da prepakuje Flux<Element> u Flux<Flux<Element>> jer ce da uzme window size i da ga obradi tako pojedinacno
			.window(Duration)
			.windowUntill(predicate)
```

```
FluxInstance.timed()-> Ovo vraca Flux<Timed<Objectnas>> ovaj timed objekat ima informacije o vremenu
```

```
FluxInstance.groupBy()
```



## Sending Elements Reactive

Ovo je nacin kada mi saljemo elemente pomocu flux ili mono,tj kada imamo povratnu vrednost Flux ili Mono kako se ponasa.

Po defaultu saljem mono vrednosti ili flux sacekace se da se obradi rezultat i poslace se gotovo,ovaj nacin koji je default nije bas reaktivan.Postoji nacin da mi definisemo da se vrati ok vrednost i da se onda osluskuje producs(Flux ili Mono) kada on vrati vrednost.

U postmanu ovo ne mozemo da testiramo,vec na google chrome ili neki pretrazivac.

```java
@GetMapping(value = "/")
public Flux<String> getProducts(){
  return Flux.just("cao1","cao2","cao3","cao4","cao5","cao6");
}
```

ovo ce vratiti Flux kada se sve ove vrednosti ucitaju,nece onako jednu po jednu u vremenu.Prva stvar je sto nemamo delay izmedju elemenata pa ce se brzo vratiti,druga je to sto on getmapping proizvodi text return type nije stream,mi moramo da definisemo da je stream neki kako bi vracalo podatke krzo vreme neko

```java
@GetMapping(value = "/")
public Flux<String> getProducts(){
  return Flux.just("cao1","cao2","cao3","cao4","cao5","cao6").delayElements(Duration.ofSeconds(1));
}
```

Dodali smo delay,ali ovaj delay ce delay elemente a mi  necemo dobiti rezultat kao jedna po jedna vrednost vec ce prvo sacekati delay za svaki element da prodje pa ce na kraju da ispise sve odjednom.Imamo 6 elementa i svaki je delay 1S znaci cekace 6 sekunde pa ce da posalje sve.Mi hocemo da svake sekunde posalje po 1 element,za to se koristi stream.

Da bi definisali da nasa get metoda vraca stream vrednosti,a ne celu tj da ne ceka ,u okviru getmappinga imamo poslje produces cija je default vrednost namestena na APPLICATION_JSON_VALUE tj uvek salje jedan obejkat zato i ceka da zavrsi ceo flux pa salje,mi hocemo ovo da set da salje stream vrednosti.



application/stream+json  -> ovo cemo namestiti u produces kako bi spring znao da ta get metoda vraca json stream value ,i da ne ceka ceo objekat da zavrsi sa radom vec salje jedan po jedan

```java
@GetMapping(value = "/",produces = "application/stream+json")
public Flux<String> getProducts(){
  return Flux.just("cao1","cao2","cao3","cao4","cao5","cao6").delayElements(Duration.ofSeconds(1));
}
```

Mi mozemo da imamo instancu mono ili flux i nad njima da pozovemo subscribe koja prima consumer 

```java
Flux<String> stringFlux = Flux.just("cao1", "cao2", "cao3", "cao4", "cao5", "cao6").delayElements(Duration.ofSeconds(1));
stringFlux.subscribe(x->System.out.println(x));
```

ovo x cemo dobijati kao elemente svake sekunde

## Reciving Reactive Elements

Mi mozemo Mono ili Flux da primamo kao parametar funkcije,to znaci da mi dobijamo stream a ne vrednosti direktno,vec moramo cekati na vrednosti



Kada koristimo ovo kao parametar znaci da moramo da ga tretiramo kao steam,sto znaci da ga trenutno jos nemamo,i nikad ne smemo da koristimo bloc() jer time unitavamo reaktivan pristup



- When to use a Mono in general ?
  - Well, when you still don't have the value.
- When to use Flux in general ?
  - Well, when you have a stream of data coming or not.