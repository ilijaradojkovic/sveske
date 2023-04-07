# Kafka

Message Broker

Kafka je streaming app.To je kao mysql servis koji instaliramo u pozadini da se run,zato kazem da je app.Ona nam sluzi da primi neki event u svoj queue i da taj even neko prisluskuje,ne mora biti event moze biti bilo koj objekat.

- Distributed je
- Real time
- Streaming



Kafka dolazi uz zookeeper to je isto app koja podesava kafka konfiguraciju.Kako je kafka komplikovana iznutra prikazacemo je na slici:

![image-20221207213925416](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221207213925416.png)

Vidimo da se kafka saastoji od:

-  Klastera 		---> koji se run ,on interno sadrzi brokere 
- Broker             --->koji nam sluze da primaju i prosledjuju poruke,svaki broker sadrzi odredjen broj topica na kojima se subscribe
- Topic               ---> ovo su kao kanali na kojima se mi subscribe
- Partition          ---> memorijski slotovi koji cuvaju poruke



![image-20221207214931647](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221207214931647.png)

Kada poruku posaljemo na odredjen broker,one ce se cuvati tamo ,bukvalno ce ih kafka store negde,to nemamo kod drugih message provajdera

Provider iz zookeepera-a vadi id brokera da zna gde da salje msg

Kafka i zookeeper su posebna instance servera na web-u.Njih mozemo da pokrenemo u docker-u





```java
version:"3"
services:
	zookeeper:
		image:zookeeper
         enviromnet:
				ZOOKEEPER_CLIENT_PORT:2181
                  ZOOKEEPER_TICK_PORT:2000
    kafka:
		image:bitnami/kafka
         depends_on:
				-zookeeper
         ports:
				-29092:29092
         environment:
				KAFKA_BROKER_ID:1
                  KAFKA_ADVERTISED_LISTENERS:PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
				KAFKA_LISTENER_SECURITY_PROTOKOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
                  KAFKA_INTER_BROKER_LISENER_NAME:PLAINTEXT
                  KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR:1
```

Kako je ovo docker compose mi ovo pokrenemo preko docker-compose up da bi se sve lepo build

Da bi poceli sa kafkom dodamo dependency 

- spring_kafka
- kafka_streams

## KafkaTemplate<K,V>

Ovo je osnovna klasa koju moramo da ubacimo u kontekst da bi radili tj slali dogadjaje.

Dva bitna interfejsa koja upravljaju kafkom su producer i consumer i na osnovu njih  mi radimo odredjene stvari

Mi imamo 2 opcije za konfiguraciju

- U application.property
- U bean-ovima

#### Bean config

##### Producer

```java
@Bean
public ProducerFactory<K,V> producerFactory(){
    Map<K,V> confis=new HashMap();
    configs.put(ProducerConfig.BOOTSTRAP_SERVER_CONFIG,localhost:29092);
    configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,StringSerializer.class);
    configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class);
    return new DefaultKafkaProducerFactory(configs);
}
```

Glavne stvari na sta cemo se fokusirati ovde su:

- <span style=color:#DC4C4C>ProducerFactory<K,V></span>
- <span style=color:#DC4C4C>ProducerConfig</span>
- <span style=color:#DC4C4C>DefaultKafkaProducerFactory(configs)</span>

```java
@Bean
public KafkaTemplate<K,V> produceTemplate(){
return new KafkaTemplate<K,V>(producerFactory());
}
```

​	Mi da bi napravili KafkaTemplate mi moramo da iskonfigurisemo producerfactory,jer je funkcionalnost ovog template da salje poruke,a to radi producerFactory pa je neophodno prvo uraditi ovu konfiguraciju.



```properties
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
```

Ovo znaci da ce template samo string da prima,nece moci objekat pa sam da serialize,za to bi nam bilo potrebno Serialize<myobject> da vi znao



```java
public class JavaSerializer implements Serializer<Object> {
    @Override
public byte[] serialize(String topic, Object data) {
    try {
        ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
        ObjectOutputStream objectStream = new ObjectOutputStream(byteStream);
        objectStream.writeObject(data);
        objectStream.flush();
        objectStream.close();
        return byteStream.toByteArray();
    }
    catch (IOException e) {
        throw new IllegalStateException("Can't serialize object: " + data, e);
    }
}

@Override
public void configure(Map<String, ?> configs, boolean isKey) {

}

@Override
public void close() {

}
```

}



##### Consumer

```java
@Bean
public ConsumerFactory<K,V> consumerFactory(){
	Map<K,V> configs =new HashMap<>();
	configs.put(ConsumerConfig.BOOTSTRAP_SERVER_CONFIG,localhost:29092);
    configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,StringSerializer.class);
    configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringSerializer.class);
    configs.put(ConsumerConfig.GROUP_ID_CONFIG,"mygroup");
    configs.put(ConsumerConfig.AUTO_OFFSET_RESET_DOC,"earliest");
    return new DefaultKafkaConsumerFactory<K,V>(configs);
}
@Bean
public ConcurrentKafkaListnerContainerFactory<K,V> kListener(){
    	ConcurrentKafkaListnerContainerFactory factory =new ConcurrentKafkaListnerContainerFactory();
    	factory.setConsumerFactory(consumerFactory());
   		return factory;
}
```

Za consumer-a nam je situacija drugacija,on nam sluzi da consume(slusa) evente (poruke) koje nam dolaze.

Kako slusamo ?

Anotacijom metode sa 

<span style=color:#DC4C4C>@KafkaListenr(topics="...",groupId="...",containerFactory="...")</span>

containerFactory ce biti ime metode kListenerd



#### Application.properties config



##### Producer

```properties
spring.kafka.produer.bootstrap-servers=localhost:29092
spring.kafka.produer.key-serializer=org.springframework.kafka.support.serializer.neki
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.neki
```



##### Consumer

```properties
spring.kafka.consumer.bootstrap-servers=localhost:29092
spring.kafka.consumer.key-deserializer=org.springframework.kafka.support.serializer.neki
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.neki
spring.kafka.consumer.group-id=mygroup
```



MI mozemo napraviti sami iz koda topic tako sto napravimo bean

```java
@Bean
public NewTopic topic1(){
return TopicBuilder.name("ime")
                    .partitions(10)
                    .replicas(3)
                    .compact()
                    .build();
}
```



## Slanje poruke

Moramo imati instancu klase KafkaTemplate<K,V>.

K,V nam definise sta mozemo da smestimo u ovaj template,tj sta saljemo



```java
@Autowired
private KafkaTemplate<String,User> template;
```

ovako cemo uzeti bean,napomena je da pre ovoga moramo da definisemo konfiguraciju za producer-a kako bi uopste spring znao da autowired ovu klasu.



<span style=color:#DC4C4C>template.send(topic,data)</span>

<span style=color:#DC4C4C>template.send(topic,partition,key,data)</span>

```java
template.send("register",myuser);
```

Saljemo objekat myuser na topic "register".



## Slusanje poruke

Moramo da definisemo consumer configuration da bi uopste ovo radilo.Moramo takodje voditi racuna o serijalizaciji ako se salju objekti,mora biti JSON.

Na metodi oznacimo anotaciju

```java
@KafkaListenr(topics="register",groupId="mygroup")

public void listenForUser(User user){

	...

}


```

Ovde u telu metode cemo dobiti objekat user kada se posalje event na nas topic

Mi takodje iz ove listen metode koja je anotirana sa @kafkaListener mozemo da uzimamo metapodatke

```java
@KafkaListener(...)
public void listenrUser(User user,@Header(KafkaHeaders.neki) string value){
...
}
```

<span style=color:#DC4C4C>@Header(KafkaHeaders.neki) </span>

@KafkaListener je takodje i class level anotation

```java
@KafkaListener(...)
public clas Test{
    @KafkaHandler
    public listen1(User user){
    ..
    }
    @KafkaHandler
    pulbic listen2(Product product){
    ...
    }
}
```

ako stavimo na class level moracemo da oznacimo metode koje ce "handle" dolazece evente.

<span style=color:#DC4C4C>@KafkaHandler(isDefault) </span>

ovo isDefault cemo koristiti ako imamo vise njih koje handle istu stvar,pa da znamo koja je primarna

## Kafka in microservices

![image-20221207230537012](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221207230537012.png)

Nekad ce nam mikroservis imati i provider i consumer-a ,a nekada samo jedno od tih.





## Transakcije

Koriscenje transakcija u kafka sistemu nam omogucava sigurnost da je poruka stigla do nekog tamo listener-a1

Osigurava nam da su sve poruka uspesno publish na kafka kanal,ako jedna nije ostale ce se rollback

Kada producer posalje poruku ako transakciju,javlja se  ` wo-phase commit protocol`

1. Begin transaction ->Producer zapocinje transakciju tako sto posalje `begin transaction` komandu u kafka broker
2. Send messages -> Produces salje poruke u kafku,te poruke se pisu u transaction log,i nisu odmah vidljive korisniku.Ovde dolazi do srzi transakcije ,ako transakcija ne uspe onda se ovo iz transaction log-a brise i to je rollback
3. Commit or Abort transaction -> Kada su sve poruke poslate,producer bira da commit ili abort transakciju.Ako se commit onda su poruke vidljive konsumeru,ali ako abort poruke nisu.



1. Konfigurisemo producer-a i stavimo da ima `transaction-id-prefix`.Kada ovo stavimo nasa metoda send mora da ima @Transactional inace nece da rdi
2. Konfigurisemo consumer-a

## Kafka Streams

Ovo mozemo da radimo preko 3 biblioteke koje nam kafka provajduje:

1. Consumer API -> ovo nase sto smo radili do sada,default sa listenerima

   ### Simple use case

   ### ![WhatsApp Image 2023-03-26 at 11.01.16 PM (4)](C:\Program Files\Typora\resources\Docs\img\WhatsApp Image 2023-03-26 at 11.01.16 PM (4).jpeg)

   ### Complex use case

   ![WhatsApp Image 2023-03-26 at 11.01.16 PM (3)](C:\Program Files\Typora\resources\Docs\img\WhatsApp Image 2023-03-26 at 11.01.16 PM (3).jpeg)

   moramo da pravimo sami 5 s window,i da vidimo da upadaju dobro podaci u taj window,ovo je malo neprakticno

2. Streams API

   Ovo je next level,sve sto smo raidli sa Consumer API mozemo da radimo sa ovim,ali samo brze

3. KSQL

   Ovo je interactive sql processing on kafka

Mi Streaming postizemo preko 2 biblioteke iz kafka 

1. Streams DSL
2. Processor API



### Streams DSL

Verzija kafke i verzija streams mora biti ista

Kljucne klase

1. <span style="color:#CD5C5C">Serdes</span>
2. <span style="color:#CD5C5C">Produced</span>
3. <span style="color:#CD5C5C">Consumed</span>
4. <span style="color:#CD5C5C">KStream</span>
5. <span style="color:#CD5C5C">KTable</span>
6. <span style="color:#CD5C5C">StreamBuilder</span>

Ovo je high level api za rad sa strimovima,u pozadini koristi Processor API(jer je on low level)

Kada koristimo ove stream-ove koristimo nesto sto se zove <span style="color:red">**Serdes** </span>klasa ,ne json deserializer ili json serializer jer je stream priroda da radi i serijalizaciju i deserijalizaciju u isto vreme.

SerDes(Serialization-Deserialization)

Serdes je factory klasa

Serdes.Integer() -> Serdes za integer-e

Serdes.String() -> Serdes za String 



Mi sa strimovima radimo preko 2 klase:

- KTable
- KStream

### KStream<Key,Value>

1. Open a stream to a source topic
2. Process stream
3. Create topology

 <span style="color:red">StreamBuilder </span> -> ovo je bitna klasa jer se sve preko njega pravi

```java
StreamBuilder s=new StreamBuilder();
```

1)Kada kreiramo builder mi otvaramo stream,u ovom slucaju KStream,preko metode  

<span style="color:red">stream(KasfkaTopic)</span>

<span style="color:red">stream(KasfkaTopic,Consumed.with(serdeskey,serdesvalue))</span>

2)KStream klasa nam daje metode kao da radimo sa kolekcijama,tj stream()....

3)Topology je klasa u kojoj se nalazi cela logika stream-a,mi podesimo stream i logiku(foreach) i to kada build napravi se topology koja sve to sadrzi

4)Kada napravimo topologi mi moramo da start stream

​	

```java
1) KStream<Integer,String> k=streamBuilderInstance.stream(topic); ->open a stream
2) k.foreach((k,v)->...)  -> process stream
3) Topology topology=streamBuilder.build();
4) KafkaStreams streams=new KafkaStreams(topology.props);
Runtime.getRuntime().addShutdownHook(new Thread(()->{
 streams.close();  
)})
	streams.start();

```

Ovo je stream koji samo cita podatke

Ovo shuddownhook je kada ugasimo app sta da se odradi,moramo da zatvorimo (cleanup) stream

Imamo stream-ove koji citaju i salju u isto vreme

to radimo sa istim ovim koracima samo sto dodamo liniju koda 

```java
5) kstreamInstance.to(topicname)
    			 .to(topicname,Produced.with(serdesKey,serdesValue))
```

#### Paralelism

Svaki kafka stream je singlethreaded tj ne kotisti paralel streams

samo treba da namestimo config proeprty NUM_STREAM_THREADS_CONFIG na broj threadova koji ce da koristi

#### Cuvanje stanja

Ovo mozemo da postignemo tako sto imamo nasu bazu,neki fajl,ili lokalno da cuvamo u kodu.

Kafka streams nudi jednu opciju za rad sa ovim i to su `states`

Postoji nesto sto se zove state store i to napravimo i dodamo u builder preko koga smo pravili i stream,sve ce to da se prebaci u Topology



```
StoreBuilder stateStore=Stores.keyValueStoreBuilder(
	Stores.inMemoryKeyValueStore(storeName),
	serdesSerialization
	,SerdesDeserialization
	)
streamBuilderInstance.addStateStore(stateStore)
```

Postoje 3 vrste Store koja kafka nudi:

1. KeyValueStore

2. SessionStore

3. WindowStore

   i sve one postoje kao inMemory ili persistent

znaci imamo 

- inMemoryKetValueStore
- inMemorySessionStore
- inMemoryWindowStore
- persistentKeyValueStore
- persistentSessionStore
- persistentWindowStore

Stejtovi su local za svaki task,moramo paizti na particije tu su states



## Kafka Consumer API

Mozemo da podelimo da imamo vise kafka consumer-a koje citaju iz drugacijih particija,time povecavamo scalability

MI mozemo svakoj particiji da dodamo 1 kafka app,tako da najvise mozemo da imamo kafka app koliko i particija,ako jedan consumer propadne dodeljuje se particija nekom drugom consumer-u



## Serdes

Bitni pojmovi

- Serde<T> (interface)
- Serdes      (class)

Serdes je kombinacija klasa Serializer i Deserializer.

Kafka nam vec provide neke default kao sto su Integer,String..

Mi hocemo nase da napravimo jer nam trebaju konkretni za neki DTO objekat.

Ovo se pravi kroz 3 koraka:

1) Write a serializer (interface Serializer<T>) (Nije neophodno)
2) Write a deserializer (interface Deserializer<T>) (Nije neophodno)
3) Write a serde (interface Serde<T>)

3. Ovaj korak mozemo da uradimo na nekoliko nacina

   1) Serde.serdeFrom(classType) Serde.serdeFrom(Serializer<T>,Deserializer<T>)
   2) Implementujemo interface Serde<T>
   3) Nasledimo klasu Serdes

   

   ```java
   public class AppSerdes extends Serdes{
   	static public final class PosInvoiceSerde extends WrapperSerde<PosInvoice>{
           public PosInvoiceSerde(){
               super(new JsonSerializer<>(),new JsonDeserializer<>());
           }
          
       }
       
       static public Serde<PostInvoice> PosInvoice(){
           PosInvoiceSerde serde=new PosInvoiceSerde();
           Map<String,Object> serdeConfigs=new HashMap<>();
           serdeConfigs.put(JsonDeserializer.VALUE_CLASS_NAME_CONFIG,PosInovice.class)
               serde.configure(serdeConfigs,false);
           return serde;
       }
   }
   ```

## KTable<K,V>

```
streamBUilder.table(topicName)

streamBUilder.table(topicName,Consumed.with(...))
```

Mi mozemo da ktable konvertujemo u kstream preko metode `toStream()`

## Time Window

Kada se salje od producer-> Kafka -> Consumer moze biti da se poruka krece i ima razlicite intervale vremena.Koje vreme uzeti?

1) Da prilikom slanja poruke iz producer-a mi dodamo polje timestamp (Event time)
2) Da broker dodeli vreme kada poruka stigne na kafku,ovime ce overide vreme koje je namesteno u prethodnom koraku,da ovo omogucimo moramo u config  message.timestamp.type=LogAppendTime (Ingestion time)
3) Kada stigne do consumer-a da uzmemo vreme (Processing tmime)

Ovo vreme nam se nalazi u metadata i skriveno je,kako da ga uzmemo?

moramo dodati u config props.put(StreamsConfig.DEFAULT_TIMESTAMP_EXTRACTOR_CLASS_CONFIG,WallclockTimestampExtractor.class)

Timestamp Extracotr:

FailOnInvalidTimestamp

LogAndSkipOnInvalidTimestamp

UsePreviousTimeOnInvalidTimestamp

Kafka api nam daje 2 tipa window-a

1. Time window
2. Session window

npr: 5 min window da grupisemo vrednosti po vrednostima koje dolaze

za ovo nam treba time extractor

```
class InvoiceTimeExtracotr implements TimestampExtractor{
	@Override
	public long extract(...){

	}
}
}
```

Ovaj timeStampExtractor setujemo na StreamBuilder

```
StreamBuiler st=new StreamBuilder();
st.stream(topicName,Consumed.with(Serdes).withTimestampExtractor(NAS));
```

i sada grupisemo pomocu metode windowBy(...)

```java
kStreamInstance.groupBy(...).windowedBy(TimeWIndow.of(Duration.ofMinutes(5)))
```



Ako nam stizu rekordi u vremenima:

10:00

10:08

on ceto prepoznati i napravice 2 prozora

10:00->10:05

10:05->10:10

Ako nam stigne neki rekord u proslom vremenu tipa da je posle 10:08 stigao 10:03 (latecommers) kako ovo da resimo jer mi vec imamo 2 prozora kako to kafka radi?

isti window ce se update kafak resava to umesto nas

#### Hopping Windows

Ovo je kada se pomeraju prozori u vremenu za neki interval

```
kStreamInstance.groupByKey(Grouped.with(serdes,serdes).windowedBy(TimeWIndows.of(Duration.ofMinutes(5).advancedBy(DURATIon))))
```

Samo dodamo <span style="color:red">advancedBy(DURATION)</span>

## Kafka cloud app

Confluent.cloud je aplikacija koja se besplatno koristi za kreiranje klastera i brokera.Napravimo broker i topic i imacemo vizuelni prikaz.

Kada napravimo mozemo se konektovati tako sto odemo u client->new client->izaberemo spring boot app

ono ce nam izbaciti novu konfiguraciju koju treba da ubacimo u spring boot app

```properties
spring.kafka.properties.sasl.mechanism=PLAIN
spring.kafka.properties.bootstrap.servers=pkc-lzvrd.us-west4.gcp.confluent.cloud:9092
spring.kafka.properties.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username='6YFTD66TUAFWKE4V' password='KXjgflmaaSTiLpavqs6gEBZOAB3OmDFriAvpWroDZ/G8z4kjb1jI4gW0APZgBKpe';
spring.kafka.properties.security.protocol=SASL_SSL
```

Ovo je samo konekcija sa tim klasterom,moramo namestiti kafka producer i consumer

```properties
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.consumer.valuedeserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.group-id=mygroup
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

ostalo je vise poznato,napravimo kafkatemplate koji ce da salje objekte ,i tako funkcionisemo,imacemo pregled na web app.

![image-20221210165804185](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221210165804185.png)

Poruke koje budemo slali ce se cuvati ovde,dok ih neko ne priocita,kada se procitaju izbrisace se

## KSQL

KSQL je streaming sql engine za apache kafka.Licni na sql

Ovo daje developeru mogucnost da jednostavno i brzo kontrolise messages koje prolaze kroz kafku.

Mi KSQL mozemo da run na ovom confuluent-u kao na cloud-u ,a mozemo i da start u docker-u.

Da ubacimo podatke u KSQL mozemo koristi:

- Ui Editor na Confluenc-u

- Java client

- CLI koji smo lokalno instalirali pomocu dokera

  Na ovaj nacin cemo direktno da ubacuju podatke u KSQL,posstoje i drugi nacini npr:

  Ako imamo podatke u postojacem kafka kanalu napravicemo stream koji ce da ih cita i ubacuje u KSQL

  ```sql
  CREATE STREAM people WITH(KAFKA_TOPIC='topic1',VALUE_FORMAT='AVRO')
  CREATE TABLE people WITH(KAFKA_TOPIC='topic1',VALUE_FORMAT='AVRO',PARTITIONS=3)
  ```

  ovde topic ne mora da postoji ksql ce sam da napravi topic.

​	KSQL ima odlicnu podrsku za integracijom sa drugim sistemima

![image-20221211171329535](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211171329535.png)

Tako da podatke iz mysql baze mozemo da konektujemo sa nasim ksql.

![image-20221211171417401](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211171417401.png)

Mi mozemo snapshot db da unesemo u ksql.Ovo je pravac gde mi podatke iz mysql-a uvodimo u ksql

![image-20221211171534795](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211171534795.png)

Takodje mozemo da iz ksql-a dovodimo podatke u mysql itd

![image-20221211173857222](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211173857222.png)

Jednostavno u KSQL ubacimo da filtriramo topic po necemu .Pravi stream tako  da sadrzi smao NY

Ovako bi filtrirali jedan stream,orders su neki stream kroz koje podaci prolaze

![image-20221211175531210](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211175531210.png)

Ovde su orders i items u razlicitm topicima,moramo koristiti ksql da bi spojili te informacije i napravili stream koji sadrzi info o svima kao detaljima.

![image-20221211192952202](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211192952202.png)

Takodje moze da menjamo podatke iz jednog stream-a,radimo transformaciju podataka.Samo napravimo novi stream koji je odgovarajuci

![image-20221211193415675](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211193415675.png)



Ako imamo neki nested obj mozemo da flat-ujemo da se nalaze svi elementi na istom nivou

![image-20221211193706971](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211193706971.png)

Mozemo da menjamo nacin prezentacije podataka,tj format njihov.

Default je avro 

![image-20221211194313898](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211194313898.png)

Merging 2 streams .2 streama,i jos jedan koji ih join 

![image-20221211194540109](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211194540109.png)

Mozemo i da podelimo stream na delove(source je polje)

Postoje 2 tipa query-ja u ksql-u:

- push ->ima emit changes kao keyword
- pull   ->nema emit changes kao keyword

jer ce emit changes da nastavi da se run za svaku promenu koja se desi update ce rez,never exits

pull exits



```sql
SELECT status,bytes
FROM clickstream
WHERE user_agent="Mozila"
```



Ono sto ga cini razlicitim od sql-a je to sto ovo clickstream nije tabela vec stream.Ksql takodje  radi sa tabelama ,ima support,ali ovo je prirodnije.

ksql radi sa streamovima koji su apstrakcije kafka topica

ove streams se zovu KStream u kafki to su key value pair

Takodje postoji i koncept tabela koja se zovu KTable,ovo je objekat koji ima kljuc i value koji je json vrednost neke promene

KTable znaci da ce najnoviju(posledju verziju promene objekta) da cuva u nekoj virtualnom prostoru tj u table.Tako da mozemo imati vise zahjteva koji nam stizu za promenom stanja,ali mi cemo da pamtimo samo zadnje

 Kafka nije samo velika cevka kroz koju prolaze podaci.Vec je neki log sistem,tj cuva kao stas se salje ne samo da nesto dodje i prodje kroz nju.Cuvace sve evente koje joj saljemo

Skuplja summarize statistics in windows time

## Kafka Streams

Spring ce sam da manage lifecycle of kafka streams

Java api

Mi pisemo nesto slicno listenerima ,koji slusaju sekvence podataka i one su deo nase aplikacije kada ih napravimo.

Postoji bilioketa kafka stremas koja ima svoje metode za stvaranje strimova,a takodje posotjji integracija sa spring cloud-om koja nam omogucava mnogo lakse da radimo 

## Spring boot kafka cloud streams

Moramo da dodamo dependencies

```xml
 <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kafka-streams</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

i jer se radi u spring lcoud-u moramo da dodamo dependency managment

```xml
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



Spring boot je java based framework koji nam pomaze da lako koristimo i pravimo app.Kafka ima posebnu biblioketu koja se bavi stream-ovima,ali postoji cloud streams koji radi i koristi tu kafka biblioteku i time nam jos vise olaksava rad sa stream-ovima.

Ovo je primer kako da koristimo spring boot cloud kafak stream da napravimo message-driven microservice.

Osnovne vrednosti koje kafak stream pa i sam cloud stream kafak ima su 

KStream<K,V> -> ovo je obicna stream koji sadrzi kljuc i vrednost vezanu za njega

KTable<

![image-20221211194915780](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221211194915780.png)

Da bi poceli da radimo sa streams u ovom pogledu mi definisemo bean koji vraca neki funkcionalni interfejs

```java
@Bean
public Consumer<KStream<Object,String>>> process(){
    ...
}
```

U zavisnosti od funkcionalnog interfejsa sta ce stream da radi

Consumer -> kako je ovo interfejs koji samo prima vrednost,ovaj stream ce samo da prima tj listen

Producer -> kako je ovo interfejs koji samo salje vrednost,ovaj stream ce samo da salje 

Function -> ovo prima i slusa tako da ce da primi neki stream i njega mozemo da modifikujemo i da vratimo na neki drugi kanal

Function<Kstream1,Kstream2> ->ovo prima jedan stream sa jednog topica,i vraca drugi stream na drugi topic

ovo sve definisemo u properties fajlovima.

```yml
spring:
	cloud:
		stream:
			function:
				definition: imenasefunkcije
			bindings:
				imefunkcije-in-broj:
					destiantion: topic
				imefunkcije-out-broj:
					destination: topic
			kafka:
				bindings:
					imefunkcije-in-br:
						consumer:
							configuration:
								value:
									deserializer: org.springframerowrk.kafka.support...
					imefunkcije-out-br:
                            producer:
                                configuration:
                                    value:
                                        serializer: ...
				binder:
					brokers:
						localhost:29092
				
```

imefunkcije-in-broj-> ovo kaze da ta funkcije prima neki stream(zato je in) sa topica destination

imefunkcije-out-broj->ovo kaze da ta funkcije salje neki stream(zato je out) na topica destination

a posle u bindings definisemo koji su serializeri i deserializeri za taj stream.