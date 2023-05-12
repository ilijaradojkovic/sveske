# Arhitecture

## Clean Arhitecture

Ovo je `separation of concerns` tako sto se podeli aplikacija u razlicite delove(different layers)

![image-20230423171411354](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423171411354.png)



Vidimo jasnu separaciju delova aplikacije.

Svaka boja je razliciti deo softvera.

##### Dependency rule

Dependencies pokazuju unutar softvera,tako da entites nema nikakav dependenci .On je Raw i jedinstven.

To znaci da elementi mogu znati samo unutar njih,ne i van.Npr imamo entities koje ne znaju nista sto je spolja od njih.Gateways znaju samo unutrasnji krug,ne bi videli nita spolja kao sto je ovaj db ui...

Ako pojednostavimo sve ovo,ispada ovako:

![image-20230423171834961](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423171834961.png)



Ova strelica prikazuje "depends on" znacenje.Nista iz donjeg sloja ne bi trebalo da zavisi od gornjeg

### Frameworks and drivers

- User Interface
- Database
- External Interfaces (eg: Native platform API)
- Web (eg: Network Request)
- Devices (eg: Printers and Scanners

To sve sadrzi i radi:

- `Presenters` (UI Logic, States)
- `Controllers` (Interface that holds methods needed by the application which is implemented by Web, Devices or External Interfaces)
- `Gateways` (Interface that holds every CRUD operation performed by the application, implemented by DB)

### Application Business Rules

Provajduje nam `Use Cases` aplikacije,takodje odredjuje kada ce se koji `Gateway/Controller` zvati za koji use case

Koristi se `Dependency Inversion`  i `Polymorphism` da se lako odvoje delovi aplikacije

Source kod dependencies mogu samo da pokazuju unutar(inward) domain layer-a

Ova arhitektura se jos zove i `plugin and ports` arhitektura.Jer svaki deo mora da se prikljuci nezavisno kao plugin

Nedostatak ovoga:

1. Moramo da trosimo vise vremena na pisanje koda
2. Dupli kod 



Koristimo `Use Cases` koji sadrze biznis logiku

Dosta je slicno kao DDD

Ovde imamo Entites i Use Cases a u DDD imamo Agregates i Domain Services

### Enterprise Business Rules

Ovo je sloj koji sadrzi core-businsess ,ovo se skoro nikad ne menja,to je srz.Promene drugih slojeva nece uticati na ovaj sloj





Primer

![image-20230423181410934](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181410934.png)

Ne bi islo ovako zbog dependency rula

![image-20230423181551577](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181551577.png)

vidim oda se krsi,to resavamo preko  `Dependency Inversion Principle` tako sto ubacimo interfejs izmedju 2 sloja

![image-20230423181634234](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181634234.png)

![image-20230423181637585](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181637585.png)

![image-20230423181646912](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181646912.png)

### Primer Order Service

![image-20230423182609025](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423182609025.png)

Ovako bi poceli da raidmo,i slika nam izgleda ovako.Ovo nije dobro jer nam Buisness Logic zavisi od RDBMS-a ,to ne valja jer mi hocemo da budemo fleksibilni da bi mogli da je menjamo kada hocemo.Svaka promena Data Layer-a mora da se proprati menjanjem Domain Layer-a,to ne valja,jer imamo DIREKTNU zavisnost

![image-20230423182909145](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423182909145.png)

Ovako smo to resili

Napravili smo interface koji ima expose neke operacije,i te operacije mogui samo da se pozivaju.Nas Data LAyer ce sam da implementuje te operacije a DomainLayer ce da sadrzi referencu nad njima

Sada vise nisu zavisni

Ovo je glavni cilj `Clean Arhitecture` da mi koristimo `Dependency Inversion` zbog zavisnosti,i da uradimo `Polimorfizam` kao i `Dependency Injection`

-----------------------------------

Fora je da napravimo projekat tj module koji je npr `Domain` i tu nemamo nikakve logike vec samo klase koje su nam DTO,ENTITIES...

### Bitni koncepti:

- Entities -> objekti
- Use Cases -> ovde sadrze implementaciju biznis logike
- Infrastructure -> API,Gateway,Repositories,Configurations



Fora je da mi koristimo Use Cases kao elemente biznis logike,svaki use case ce raditi nesto ,kao one Sistemske Operacije iz PS-a

Flow:

![image-20230424004755644](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230424004755644.png)

1. An external system performs a request (An HTTP request, a JMS message is available, etc.)
2. The Delivery creates various Model Entities from the request data
3. The Delivery calls an Use Case execute
4. The Use Case operates on Model Entities
5. The Use Case makes a request to read / write on the Repository
6. The Repository consumes some Model Entity from the Use Case request
7. The Repository interacts with the External Persistence (DB,Cache,etc)
8. The Repository creates Model Entities from the persisted data
9. The Use Case requests collaboration from a Gateway
10. The Gateway consumes the Model Entities provided by the Use Case request
11. The Gateway interacts with External Services (other API, microservices, put messages in queues, etc.)

Bukvalno ovo `Repositories` je neka impl,npr ProductRepository koji sadrzi neke operacije koje radimo nad bazom,nisu samo CRUD operacije ciste vec ima neke logike i error handling,a core biznis logika je u `Use Cases`

npr metoda u repository

```
return categoryRepository.findAll().stream().map(category -> categoryRepositoryConverter.mapToEntity(category))
                .collect(Collectors.toList());
```

Tako da bi repositoryimpl imala autowired dependencty na npr ProductRepository



## Hexagonal Arhitecture

Ovo je bazicna arhitektura na kojoj su se razvili ostali.

primary adapter- > input 

secondary adapter -> output



![image-20230427102724480](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230427102724480.png)

Na onovu ove arhitekture nastale su ove ostale,imaju slican dizajn bar bazicno

## Domain Driven Design

Ovde je domain u centru,i srz aplikacije

Svrha ovoga je da se indentifikuju odnosi domena biznisa

Postoje 2 vrste 

- Strategic DDD
- Tactical DDD

#### Strategic DDD

Fokus je na `high-level` dizajn sistema,veza razlicitih domena aplikacije.

fokusira se na boundaries u domen modelu.`Single Bounded context per each domain`

Kljucni koncepti:

- Domain -> ovo je deo nase aplikacije npr Food Ordering,moze imati nekoliko subdomena
- Bounded Context -> definise granice domen-a ,single bounded context per each domain,ovo je kao podsistem sa funkcijama
- Ubiquitous Language -> zajednicki jezik izmedju eksperta i programera,imenovanje klasa mora da bude na zajednickom jeizku

#### Tactical DDD

Fokusira se na implementacione detalje u samom bounded contet-u.Dizajniranje interne strukture kako sta radi

- Entities -> Ovo je domain objekat sa jedinstvenim id-jem,sadrze biznis logiku,ako je isti id za dva objekta onisu isti,nebitno sta sadrze od ostalih polja,znaci samo id mora da se uporedjuje
- Aggregates -> ovo je grupa objekata koji su logicki grupisani -> slicno kao Bounded Context,imamo neki kao sistem mali
- Aggregate Root -> sve biznis operacije moraju da prodju kroz aggregate root,menjaju aggregates,samo iz aggregate root mozemo da dodjemo do aggregate
- Value Objects -> ovo su objekti umesto primitivnih tipova
- Domain Events -> komunikacija izmedju vise bounded context-a,message queue(Kafka)
- Domain Services -> Ovo je biznis logika koja ne moze da stane u agregat,isto kada koristimo vise agregata istovremeno
- Application Services -> kominikacija izmedju razlicitih domena,kreiranje transakcija,securiy requires,saving datastore

Anti-Corruption Layer -> ovo je  kao layer kada radimo integraciju sa nekom old aplikacijom i zelimo da promenimo termine njihova da ne koristimo jer su razumni,one language does not corrupt the language of the other



Shared Kernel -> zadednicki deo 

![image-20230424165032580](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230424165032580.png)

Kako da radimo ovo

![fewfgq](C:\Program Files\Typora\resources\Docs\img\fewfgq.png)

Kada pravimo sada te klase moramo da definisemo common klase :

Napravimo module koji je common gde ce se ovo cuvati sve

common-domain module

```java
@Data
public abstract class BaseEntity<ID> {

    private ID id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BaseEntity<?> that = (BaseEntity<?>) o;
        return Objects.equals(id, that.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

```

Ova klasa nam je base za sve,vidimo da sadrzi id polje koje je tip ID,genericka je i uporedjuje na osnovu tog id-ja

uz to definisacemo i :

```java
public abstract class AggregateRoot<ID> extends  BaseEntity<ID> {
}

```

Ovo je samo mark klasa da cisto vidimo sta je AggregateRoot

U valueobject paketu common-domain -a kreiramo value objects koji su common na svim,ili bar na 2 

```java
@AllArgsConstructor(access = AccessLevel.PROTECTED)
@Data
public abstract class BaseId<T> {
    private final T value;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BaseId<?> baseId = (BaseId<?>) o;
        return Objects.equals(value, baseId.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

Ovo je base za id te entity klase

isti princip kao i kod one BaseEntity klase u entity paketu

```java
public class OrderId extends  BaseId<UUID> {
    protected OrderId(UUID value) {
        super(value);
    }
}
```

Napravili smo konkretnu instancu entity klase,baseid je samo za klase koje su tipa ID

Napravicemo drugi `value object` koji nije id 

```java
@AllArgsConstructor
@Getter
public class Money {
    private  final BigDecimal amount;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return Objects.equals(amount, money.amount);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount);
    }
}
```

Ovakva klasa ima `immutable polja` to je bitno i u nju mozemo da ubacuje logiku to je ok vode

```java
@AllArgsConstructor
@Getter
public class Money {
    private  final BigDecimal amount;

    public Boolean isGreaterThenZero(){

        return  amount!=null && this.amount.compareTo(BigDecimal.ZERO)>0;
    }
    public Boolean isGreaterThen(Money money){
        return money!=null && this.amount.compareTo(money.getAmount())>0;
    }

    public Money add(Money money){
        return new Money(setScale(this.amount.add(money.getAmount())));
    }
    public Money substract(Money money){
        return new Money(setScale(this.amount.subtract(money.getAmount())));
    }
    public Money multiply(Integer multiplier){
        return new Money(setScale(this.amount.multiply(new BigDecimal(multiplier))));
    }

    public BigDecimal setScale(BigDecimal input){
        //2 mesta zareza
        return input.setScale(2, RoundingMode.HALF_EVEN);
    }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return Objects.equals(amount, money.amount);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount);
    }
}


```

#### Where to fire the event?

U Application service.Domain layer should not know about how to fire the event,domain layer sluzi samo za rad sa domenim objektima u smislu njihove promene 

Domain Service vraca event a application service opali event

#### Where to create events?

Domain service or entities,application nikad nece da prica direktno sa entities jer ima sloj izmedju koji je service pa cemo tu

Nama bukvalno domain sluzi da proverimo da li taj objekat moze dalj da se obradjuje nema tu nikakvih repository klasa itd

application service je prvi koji ima kontakt sa spoljnim svetom

### Struktura projekta

infrastructure -> ovde drzimo ostale biblioteke tj njihove modele konfiguracije itd

​	kafka 

​	docker-compose

common -> zajednice klase

​	comon-application

​	common-dataaccess

​	common-domain

Order-Service -> ovo je root module za Order domain 

​	order-application -> ovde drzimo rest kontrolere,otvor sa spoljnim svetom

​	order-container -> ovde drzimo main app

​	order-domain -> ovde drzimo entity i domain objekte i servisnu logiku,ovo je srce svega

​			order-application-service -> ovde drzimo dto,helpere mappere i inpute outputs

​			order-domain-core -> ovde drzimo entity i domain objekte i servisne metode,eventi

​	order-dataaccess -> ovo je konekcija sa bazon

​	order-messaging -> komunikazica sa eventima(input i output ports)





#### Kako ide flow:



(Order-Application)OrderController.java  -> (Order-Application) OrderApplicationServiceImpl ->(Order-Application) OrderCreateCommandHandler -> (Order-Application) OrderCreateHelper -> (Order-domain-core)OrderDomainServiceImpl

![image-20230428104745067](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230428104745067.png)

## KORACI:

1.Prvo moramo da uradimo analizu projekta sa high-levela,to je Strategic DDD
	1.1 Moramo da nadjemo `Domain`
	1.2 Moramo da nadjemo `Boundede Context`
	1.3 Moramo da nadjemo `Ubiquitous Language`
2.Posle ove analize nalazimo detalje prethodne analize koja je low-level,to je Tactical DDD
	2.1 Moramo da odredimo sta su `Aggregates`
	2.1 Moramo da odredimo sta su `Aggregate Root`
	2.1 Moramo da odredimo sta su `Entities`
	2.1 Moramo da odredimo sta su `Value Objects`
	2.1 Moramo da odredimo sta su `Domain Events`
	2.1 Moramo da odredimo sta su `Application Service`

3.Posle moramo da definisemo ostale komponente	
	3.1 Moramo da definisemo `Mappers`
	3.2 Moramo da definisemo `Config`
	3.3 Moramo da definisemo `Command Handlers`
	3.3 Moramo da definisemo `Command Helpers`
	3.3 Moramo da definisemo `Commands`
	3.3 Moramo da definisemo `Responses`





Domenske objekte ubacujemo u `order-service` modul,u njegov `order-domain-core`  tu cemo imati pakete 

entity ->  ovo su domain objektu (`Aggregate Root`,`Entities`)

event -> eventi (`Domain Events`)

exception -> greske

valueobject -> value objects for entities(`Value Objects`)

service -> interfejs i impl njegova,ovde nam se nalazi logika koja komunicira sa domain objektima(`Domain Service`)

----



`order-application-service` 

​	config -> konfig klasa koja sadrzi value objects iz .property fajla (`Config`)

​	dto ->

​		create -> response objekat iz biznis logike,ovde cuvamo i command\ (`Command`,`Responses`)

​	mapper -> (`Mapper`)

​	handlers ->(`Command Handlers`)

​	helpers -> ovo koristi handler (`Command Helpers`)

​	ports -> interfaces

 		input -> ono sto dolazi kod nas

​				service -> ovde drzimo servisnu interfejs klasu koju rest api poziva (`Application Service`)

​		 output -> ono sto odlazi od nas,mi saljemo ovo

​				message->

​						publisher 

​				repository -> interfejsi repository sa kojima radimo,oni ce se impl u `order-dataaccess` modulu,ovo je kao 					samo repositry koji kaze imaces te metode,ovo je kao logika sta sve moze da izvrsi repository ovo nije onaj 					interface koji nasledjuje jpa vec sta sve moze da uradi ovaj layer,kao service i service impl ovo je service 					samo za repository

-----



`order-dataaccess` ovaj paket je podeljen po imenima domena i svaki domen ima ove 4 podpaketa

​	adapter -> implementacija repository interfejsa iz oreder-application-service,sto smo definisali iznad

​	repository -> ovo je impl interfejsa konkrenta JPA baze,Jpa impl,adapter ce da koristi ovu impl

​	entity -> klasa koju cuvamo,ovo nije isto sto i domain klasa,tj mogu imati istoimene info ,ali ova sadzi primitivne tipove 	podataka,ima anotacije @Entity itd

​	mapper -> mapiranje 

---

`order-messaging` -> ovde imamo konkretne implementacije kafka listenera i publisher-a za order module

​	listener.kafka ->kafka listener(sadrze @KafkaListener),implementacija kafka publisher-a iz portova u order-		     application-service modula

​	 mapper ->

​	 publisher.kafka-> implementacija kafka publisher-a iz portova u order-application-service modula

Cela fora je da kontroler koristi  orderApplicationService koji ne radi logiku direktrno vec sadrzi `commands handlers` koje to rade ,a command handleri sadrzi `heleper-e` koji rade dublje ,mi u tim helperima radimo poziv ka bazama i pozivamo zadnji sloj a to je `domain service` koji poziva domen objekte i njihovu biznis logiku

 

## Saga

Moramo da imamo neki event bus to ce nam biti `Kafka` ,moramo odrediti koordinatora saga pattern-a to ce biti jedan mikroservice (oreder service)

![saga-2](C:\Users\Ilija\Downloads\saga-2.png)

Oreder service je  `SAGA Coordinator` on pocinje saga pattern tako sto salje vent ORDER CREATED

imacemo interface koji ce da radi saga korake

```java
public interface SagaStep<T,S extends DomainEvent,U exdents DomainEvent>{
     S process(T data);
     U rollback(T data);
}
```



Bitni su nam ovi topici za response iz mikroservisa,oni ce da urade saga pattern.

Mi posaljemo 2 eventa ,jedan na Payment a drugi Storage i onda ko od njih padne menja state na CANCELLED,i to je to.Provera koja se radi u order mikroservisu u domenu uvek proveraca da li je cancelled jer ako je taj cancelled onda ne radi nista dalje.

Nama dodje event sa payment-a koji je prosao,i on stavi u state PAID,onda dodje sa storage mikroservisa i to stavi u state COMPLEATED.

Ako se desi slucaj da prodje stavi u state PAID onda u storage padne i on ga stavi u state CANCELLED

Ako se desi da payment padne onda ga stavi u state CANCELLED i sta god storage da posalje nazad kao rezultat ne menja taj state

Sada pattern se deli na dva dela:

- Choreography 
- Orchestration

### Choreography

Ovde imamo `eventbus` i tu saljemo evente,imacemo 1 glavni miksroservis na koji ce se vracati ti eventi,i u zavisnosti kakvi su enventi to odredjuje state.

Mikroservis ce obaviti neku akciju(transakciju) u zavisnosti koji event dodje 

mikroservici salju evente jedan za drugim i kada zavrse svoj deo obaveste order service

![image-20230503160846213](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503160846213.png)

Order service update status of order u zavisnosti sta se desi

Deadlock possible ako imamo vise eventa,ovo je bolje kada ih imamo manje

![image-20230503162503903](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503162503903.png)

![image-20230503162616251](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503162616251.png)

kADA se payment otkazao nista nemaom da menjamo i ne utice na ostale korake,ali ako restaurant fails onda moramo da uradimo refund na payment service ,to moramo jer su ti koraci zavisni i deluju jedan na drugog

![image-20230503162733265](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503162733265.png)

### Orchestration

Ovde cemo imati jednu odvojenu mikroservisnu komponentu koja upravlja saga transakcijom

 ![image-20230503161134851](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503161134851.png)

![image-20230503161147482](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503161147482.png)

![image-20230503161241973](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230503161241973.png)

Ovde mikroservisi ne zovu ostale vec imamo jednu komponentu koja salje te evente drugde,ne sami mikroservisi ne salju na event bus

Centralizovano je i sav load je nabijen.Promena u jednom mikroservisu moze dovesti da se menja i ovaj Orchestrator service



MyApp

![image-20230509160352723](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230509160352723.png)

Znaci imacemo posebne kafka kanale za svaki dogadjaj

https://excalidraw.com/#json=US5PzV2yBfQOB4nkPL--D,aE-ReYbNuXDS5Z0BsEYy0A

## Outbox Pattern

Ovo je kao nadogradnja sage,ako publish operacija padne stanje sistema nije konzistentno,ili ako db padne.

Najbolje je da kombinujemo ova dva

Koristicemo Outbox table zajedno sa scheduler-om ,u outbox pattern ne publish direktno event-e vec ih cuvamo u outbox table,ova table pripada istoj DB na mikroservisu.Bukvalno cuvamo evente i uz pomoc scheduler-a ako se nisu desili opet ih saljemo.

Postoje 2 pristupa:

1. Pulling Outbox Table: Pull the events with a scheduler
2. Change Data Capture: Listen transaction logs

Kako nam je koordinator Order microserice mi cemo u njemu da napravimo 2 tabele jedna za Payment events(Payment_Outbox) i za Storage(Storage_Outbox)

Naravno kada napravimo tabele moramo da napravimo i entity modele,to radimo u order-application-service u novi paket outbox

fora je da smo mi pre koristili OrderCancelledPaymentRequestMessagePublisher ili tako neke sto salju event koji direktno radi sa sagom,sada kada uvodimo Outbox pattern mi te publishere brisemo,

jer imamo outbox publisher

2 scheduler-a svaki za jedan mikroservis

![image-20230510125205653](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230510125205653.png)

Mi cemo cuvati i DomainEventState i OutboxState

Outbox je log od domain events i njih cuva,one koje tek treba da  publish.Kako mi da izvadimo te evente koji su kao log sacuvani i da ih dalje process?

Napravimo dodatan service ili job koji ih vadi i radi sa njima.

Ovu tabelu stavljamo u onu bazu gde se inicijalizuje sama saga,jer tu ako baza i padne nece ni otpoceti saga tako da je ok 

Ovaj patern koristimo kada `2Phase commit` nije opcija
