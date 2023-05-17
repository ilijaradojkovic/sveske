Za dodati u arhitecture

Mi event store moze da implementiramo preko :

1. EventStore -> Greg Young
2. AxonDb -> AxonIQ
3. Relational DBMS

![image-20230515123217590](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230515123217590.png)

Svaka komanda ima svoje ime i svoju data klasu(record)

Mi sa Axonom imamo anoacije koje same rade posao mapiranja za koji agregat ide

```xml
    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-spring-boot-starter</artifactId>
        <version>4.7.4</version>
    </dependency>
```

@TargetAggregateIdentifier -> ovo stavljamo na id komande

```java
public class CreateProductCommand{

    @TargetAggregateIdentifier
    private final String productId;
    
    private final String title;
    private final BigDecimal price;
    private final Integer quantity;
}
```

MI kada napravimo ove komande moramo da ih prosledjujemo u `Command Bus` to radi `Command Gateway` sve to imamo uz ovaj axion

Command bus salje commands na specificni `Command Handler` koji je deo Domain-a

AKo nemamo command handler za tu komandu baca se exception

Mi command Gateway mozemo da autowire

```java
@Autowired
private final CommandGateway commandGateway
```



On ima dve metode

send(object)  : CompleatableFuture<T>

sendAndWait(object) : T

![WhatsApp Image 2023-05-15 at 1.06.51 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-05-15 at 1.06.51 PM.jpeg)

Event handler oni citaju iz baze Event Store sve evente i replay ih da bi se dobilo stanje objekata

Mi kada napravimo aggreaget moramo da anotiramo tu klasu

```
@Agregate
```

Moramo da imamo NoArgsConstructor i AllArgsConstructor

i id te klase moramo da oznacimo sa @AggregateIdentifier 

Mora biti istog tipa kao u command sto je oznaceno sa @TargetIdentifier

Takodje da bi napravili command handlere u aggreagtu moramo tu metodu da 

```
@CommandHandler
public ProductAggregate(CreateProductCommand p){

}
@CommandHandler
public void handle(komanda){

}
```

@COmmandHandler moze na bilo koju metodu,ovicno su create komande na konstruktor metodi

EventName 

1. past tense
2. <Noun><PerformedAction>Event

Mi dobijemo komandu  i to se sve lepo obradi u COmmandHandler i onda tu pravimo Event i njega pustimo u event store

Kako pustimo event u event store?

```
AggregateLifecycle.apply(event)
```

i sada to uhvatimo u event handler



```
@EventSourcingHandler
public void on(ImeEventa){ -> imaju ime on(tipeventa)
 ovde radimo samo update polja
 this.price=event.getPrice();
 ...
}
```

Jer koristimo Axon sa sprign cloud-om moramo da dodamo google guava ,ne znam da li moara jos to je bilo pre

Kako smo imali CommandGateway tako imamo i za Query

`QueryGateway`

```
qyeryGateway.query(object,ResponseType.multipleInstanceOf(kojaklasa)); -> vraca listu jer je multipleinstance

```

Kako imamo gateway tako imamo i handler

```
`@QueryHandler`

public returnType findnesto(Objnas obj){

}
```

While it is possible to implement CQRS within the same microservice, it is more common to implement it across different microservices. This is because the write and read models often have different scalability and performance requirements, and may require different data storage and processing mechanisms.

In a typical CQRS implementation, the write model is responsible for processing commands and updating the state of the system, often using a domain-driven design approach. The write model may use a database or event store to persist the state changes as events. These events can be published to a message broker or event bus, which can then be consumed by one or more read models.

The read model is responsible for processing queries and providing a view of the system's state to the clients. The read model can be implemented using a separate microservice that subscribes to the events published by the write model, and updates its own data store to provide a denormalized view of the system's state that is optimized for query performance. The read model may use a different type of database or storage mechanism than the write model, such as a document database or a search engine.

Separating the read and write models into separate microservices can provide several benefits, including:

- Scalability: The read and write models can be scaled independently based on their respective performance requirements.
- Decoupling: The read and write models can be developed and deployed independently, which can help to reduce dependencies and improve agility.
- Performance: The read model can be optimized for query performance, while the write model can focus on consistency and durability.
- Resilience: The read model can continue to provide a view of the system's state even if the write model is down or experiencing issues.

In summary, while it is possible to implement CQRS within the same microservice, it is more common to implement it across different microservices to achieve better scalability, performance, and decoupling.

Mi validaciju mozemo raditi na restu,ali i u CommandHandler-u tu bi trebalo da vidimo da sama komanda moze da se procesuira,domain validation

MessageDispatchInterceptor<CommandMessage<?>> implementiramo ovo u nasoj komandi (custom) i ovo ce biti interceptor

```
@Component
publi cclass CreateProductCommandInterceptoon implements MessageDispatchInterceptor<CommandMessage<?>>
```

samo da jos registrujemo na command bus

```

@Autowired
public void registerCreateProductCommandInterceptor(ApplicationContext context,CommandBus commandBus){
	
	commandBus.registerDispachInterceptor(context.getBean(CreateProductCommandInterceptor.class));

}
```

AGGREGATES IMAJU SAMO STVARI KOJE VALIDIRAMO,ne dodajemo ono sto ne validiramo.Sva polja idu u entity klasama ,ovde samo domain provere

![image-20230515155527126](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230515155527126.png)

Axon nam nudi 

@StartSaga

@EndSaga

@SagaEventHandler(associationProperty="propertyFieldName") -> Za taj field je saga vezana

Kako imamo odvojeno Query Api i Command Api ,prilikom rada na Command nekoj mi moramo da pogledamo da li entitet vec postoji npr

Kako da brzo proverimo da li rekord postoji vec.Ovo je deo CQRS dizajna.

Resernje:

Napravicemo novu lookup tabelu koja se nalazi samo na Command Api i to ce samo Command side da koristi ne Query side.

Mi na command side imamo samo EventStore gde cuvamo evente,dok na Query side mi tu imamo sve o entitetima,i sada na Command side dodamo Lookup table za te provere kako ne bi isao do query zbog nacina na koji se komunicira izmedju njih(asinhron samo preko kafke)

i onda za update moramo da update i lookup table i Db tabelu u query mikroservisu

Za provere te nad lookup table koristimo Message Dispatch Interceptor ,a za cuvanje u toj tabeli radimo sa event handlerom

![WhatsApp Image 2023-05-15 at 4.09.00 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-05-15 at 4.09.00 PM.jpeg)

@ProccessingGroup("ime") -> ovo je samo logicka grupacija na eventHandleru,package name se koristi ako ne stavimo ovo,ovo stavljamo jer axon pravi novi tarcking event processor,ako stavimo evente u tu istu grupu dajemo mu mogocnost da se oni samo JEDNOM obrade 

```
axon.eventhandling.processors.imegrupe.mode=subscribing
```

ovo smo stavili da axon obradjuje tu grupu(imegrupe) samo na subscribing processor

 mi nikad ne radimo validaciju u eventhandleru jer su to eventi koji su se VEC DESILi

## Exceptions

@ControllerAdvice pokriva RestController,MessageInterceptor,CommandHandler

ne pokriva Event Handlere,po defaultu u event handleru ce samo da se log error i nstaviti izvrsavanje

`@ExceptionHandler `-> za handlovanje event hendlera,ali ovo ce da je uhvati odmah ovde,da bi je pustili da propagira do @ControllerAdvice-a koristimo `ListenerInvocationErrorHandler`



POstoje 2 vrste Event procesora 

1. Tracking event processor -> pull messages from a source using thread thata it manages itslef.**A diferent thread**

   ​	will retry processing event usin incremental period

2. Subscribing event processor -> Subscribe themselves to a source of eventas and are invoked by the thread managed by the publishing mechanism.**Same thread**

​			will have the exception bubble up to the publising component of the event

1. **![WhatsApp Image 2023-05-16 at 9.48.55 AM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-05-16 at 9.48.55 AM.jpeg)**

u zavisnosti sta se desilo (Command,Querry) imamo CommandExecutionException.class i QueryExecutionException.class

Fora je ako se desi exception u command handler event nece biti objavljen ,jer ne objavi odmah event i ako se exception desi on se brise kao rollback za event

Kako raidmo sa eventima u @EventHandler delu moramo napraviti novu metodu koja ce da handle to



```
@ExceptionHandler(ExceptionClass.class)
public void handle(Exception ex){

}
```

ovo je kao lokalna metoda koja ce da uhvati sve exceptione za ovu KLASU,razlicite klase nece da vidi

Kada dodje do greske u event handler metodi imamo 2 opcije za rollback

1. Nista da ne uradimo time ce se baciti exception sam i rollback ce se
2. Ako zelimo da presretnemo exception i da ga bacimo dalje da bi ga ControllerAdvice uhvatio

Ovo 2. je moguce samo ako je processing group `subscribe`

```
@ExceptionHandler(...)
public void handle(...){
	throw new Exception() -> retrow exception,ali ovo nije dovoljno 
}
```

Moramo napraviti novu klasu

```java

public class ProductServiceEventErrorHandler implements ListenerInvocationErrorHandler{
 ovde imammo metodu i u njoj opet throw exception da bi ga controller advice pokupio i rollback da se desi
}
```

Kao sto smo interceptor registrovali moramo i ovo.

```java
@Autowired
public void configure(EventProcessingConfigurer config){
	config.registerListenerInvocationErrorHandler("Proccesing Group",config-> new ProductServiceEventErrorHandler());
}
```

Axon nam nudi default klasu koja impl ListenerInvocationErrorHandler 

`PropagatedErrorHandler` 

## Saga

Saga klase moramo da anotiramo sa `@Saga` anotacijom to napravi klasu na component i postane saga

`@StartSage` -> ovime se pocinje saga

`@EndSaga`-> ovime se zavssava saga

`@SagaEventHandler(associationProperty)`



```java
@Saga
public class OrderSaga{
	@Autowired
	private CommandGateway commandGateway;
	
	@StartSaga
	@SagaEventHandler(associationProperty="orderId")
	public void handle(OrderCreatedEvent orderCreatedEvent){
	}
	
	@SagaEventHandler(associationProperty="productId")
	public void handle(ProductReservedEvent productReservedEvent){
	}
	
	@SagaEventHandler(associationProperty="paymentId")
	public void handle(PaymentProcessedEvent paymentProccesedEvent){
	}
	
	@EndSaga
	@SagaEventHandler(associationProperty="orderId")
	public void handle(OrderApprovedEvent orderApprovedEvent){
	}
	
}
```

Svaka @Saga mora da ima @StartSaga i @EndSaga

### Saga Deadlines

Ako prodje neko vreme i saga se nije zavrsila

Deadline je `event` koji se podize kada neki event nije zavrsen a ceka se dugo

Deadline is an Event that takes place of an absence of an event.

Koristimo klasu DeadlineManager

Moramo napraviti bean

SimpleDeadlineManager.class

On cuva scheduler taskove u memoriji,pa kada se ugasi app on brise sve

```java
@Bean
public DeadlineManager deadlinemanager(Configuration configuration,SpringTransactionManager transactionManager){
	return SimpleDeadlineManager.builder()
	.scopeAwareProvider(new ConfigurationScopeAwareProvider(configuration))
	.transactionManager(transactionManager)
	.build();
}


```

QuartzDeadlineManager.class

Moramo da dodamo novi dependency Quartz Scheduler 

Koristi persistance storage da cuva scheduler taskove,ovaj se koristi za druze periodew vremena kada imamo rizik od gasenja app

```java
@Bean
public DeadlineManager deadlineManager(Scheduler scheduler,AxonConfiguration configuration,SpringTransactionManager transactionManager,Serializer serializer){
	return QuartzDeadlineManager.builder()
	.scopeAwareProvider(new ConfigurationScopeAwareProvider(configuration))
	.serializer(serializer)
	.transactionManager(transactionManager)
	.scheduler(scheduler)
	.build();
}
```

#### Schedule a ne deadline

```
@Autowired
private DeadlineManager deadlineManager;
deadlineManager.schedule(Duration,name,event);
```

#### Handle deadline

```java
@DeadlineHandler(deadlineName="name")
funkcija
```

deadlineManager.cancel("")

​							.cancelAll("")

## Subscription Queries

Ovo je slucaj kada mi uradimo neku komandu i uradimo query i taj query nam daje podatke koje nisu jos update od strane te komande  jer se komunikacija rsi preko eventa i nije uvek u istom trenutku ista

Ovo je da se on subscibe i da dobije informacije kada se promeni

![WhatsApp Image 2023-05-16 at 1.45.10 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-05-16 at 1.45.10 PM.jpeg)

QueryGateway ima `subscriptionQuery` metodu

```java
@Autowired
private QueryGateway queryGateway;

SubscriptionQueryResult<class,class> result =queryGateway.subscriptionQuery(queryObject,initial response type,full update object)
queryGateway.subscriptionQuery(new FindOrderQuery(),ReponseTypes.instanceOf(class),
FindOrderQuery(),ReponseTypes.instanceOf(class),
)
i sada nad tim objektom pozovemo
    result.updates()-> vraca flux
```

Da bi obavestili subscription query o update koji se desio kako bi se opet izvrsio i uzeo nove vrednosti koristimo klasu Emmiter

`QueryUpdateEmmiter`

```
queryUpdateEmiter.emit(QueryClass,predicate,updateObject)

npr
queryUpdateEmiter.emit(QueryClass,query->true,updateObject)
```

Ovo emit moramo baciti u create metode update i delete

## Snapsoting

Kada kreiramo,menjamo objekat axon cuva sve te evente u db  tako da ako napravimo create on ce sacuvati taj evetn,i posle njega 500 puta update on ce imati 501 event u bazi.

Fora je ta sto kada uradimo svaki put update ili neki event axon ce da procita sve ostale evente iz baze i replay ce ih ,prvo ce da napravi empty constructor i onda ce da replay sve evente to nije bas efikasno jer replay sve.

mi kazemo kada da napravi snapshot i onda nece da cita sve evente nego samo snapshot

npr po defailut axon pravi to kada dodje do broja od 4 eventa 

```
@Bean(name="myname")
public SnapshopTriggerDefinition snap(Snapshotter snapshotter){
	return new EventCountSnapshotTriggerDefinition(snapshotter,3);
}
```

Ovo je jedna od impl,svaki put kada event count dodje do 3 eventa u bazi

i sada moramo da ga ukljucimo u aggregate jer on gleda moramo da ih linkujemo

@Agreggate(snapshotTriggerDefinition="myname")

## Replay Events

Mi replay events i tako restore db

replay event moze biti od 0,a moze biti od nekog trenutka 

Event Replay je suportovan od Tracking Event Processor-a

`@DisallowReplay `ovo stavljamo na @EventHandler da bi exclude ponovno izvrsavanje preko replay eventa,npr ako se preko tih eventa salju mejlovi nema potrebe da se svima salju opet



```java
@ResetHandler
public void reset(){
	productRepository.deleteAll();
}
```

`@Reset|Handler` pre nego sto se desi reset svih eventa ova funkcija se odigra,moramo da izbrisemo sve iz baze kako ne bi doslo do duplikata i gresaka



```java
@Autowired
private EventProcessingConfiguration eventProcessingConfiguration;

Optional<TrackingEventProcessor> processor=eventProcessingConfiguration.eventProcessor(processorName,TrackingEventProcessor.class);
processor.get().shutDown();
processor.get().resetTokens();
processor.get().start();
```

