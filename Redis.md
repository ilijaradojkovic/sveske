# Redis

Redis je baza koja ima karakteristike:

- no-sql
- key-value
- koristi se za cache dosta

Imamo 2 mogucnosti kako da koristimo redis kao no-sql bazu:

1. Intaliramo lokalno
2. Run preko dockera

Takodje imamo i 2 mogucnosti kako cemo pristupiti toj bazi:

1. CLI
2. GUI (Another redis desktop manager)

Redis je **binary safe** sto znaci da je key ili value koji se cuvaju mogu biti bilo sta

Redis radi sa nekim predefinisam kolekcijama:

- List
- Set
- Sorted set
- Map



###  CLI komande:



<span style=color:#DC4C4C>SET KEY VALUE</span>

O(1) operacija koja za neki key namesta vrednost

npr: SET  1 Iiija



<span style=color:#DC4C4C>GET KEY</span>

O(1) operacija koja na osnovu kljuca vraca vrednost

npr: GET 1



<span style=color:#DC4C4C>DEL KEY</span>

O(1) operacija koja na osnovu kljuca brise vrednost

npr: DEL 1



<span style=color:#DC4C4C>BGSAVE</span>

Ovo znaci da sacuva vrednosti,jer je redis in-memory db moramo joj reci da sacuva vrednosti ili ce se ugasiti ako iskljucimo redis server



<span style=color:#DC4C4C>INFO MEMORY</span>

Daje informacije o memoriji



<span style=color:#DC4C4C>INCR KEY</span>

inkrementuje zadat kljuc ali samo ako je on integer

npr: INCR 1 -->2



<span style=color:#DC4C4C>INCRBY  KEY VALUE</span>

inkrementuje zadat ljuc za value puta

npr: INCRBY 1 2 -->3



<span style=color:#DC4C4C>DECR KEY </span>





<span style=color:#DC4C4C>DECRBY KEY VALUE</span>



<span style=color:#DC4C4C>FLUSH ALL</span>

Brise sve kljuceve



SET KEY VALUE <span style=color:#DC4C4C>EX/PX TIME</span>

EX->Seconds

PX->Miliseconds

Postavlja ovaj key da vazi samo za odredjeno vreme

npr:SET 1 ilija EX 2

ovo ce vaziti samo 2 sekunde

<span style=color:#DC4C4C>TTL KEY</span>

time to live,vidi koliko jos kljuc zivi 



SET KEY VALUE <span style=color:#DC4C4C>XX/HX</span>

XX->ovo je update value ali samo ako key posotji

HX->isto kao XX samo sto ako ne postoji napravice ga



#### Liste

<span style=color:#DC4C4C>LPUSH KEY VALUES</span>

<span style=color:#DC4C4C>RPUSH KEY VALUES</span>

Dodaje vrednosti na levoj ili desnoj strani liste

Ovo mozemo gledati kao da za svaki key imamo listu vrednosti

npr: LPUSH 1 1 2 3 4 5

Dodacemo key 1 sa listom [1,2,3,4,5]



<span style=color:#DC4C4C>LRANGE KEY START_INDEX END_INDEX</span>

Vraca range elemenata,ovo je get

LGET 1 0 2 ->[1,2,3]

LGET 1 0 -1-> ako stavimo da je zadnji element -1 znaci idi do kraja ->[1,2,3,4,5]



<span style=color:#DC4C4C>LPOP KEY</span>

<span style=color:#DC4C4C>RPOP KEY</span>

Izbacuje listu



### Set

<span style=color:#DC4C4C>SADD KEY VALUES</span>

Dodaje vrednosti u set 

npr: SADD k1 1 2 3 4 5 -> napravice se [1,2,3,4,5]



<span style=color:#DC4C4C>SMEMBERS KEY </span>

Vraca set



<span style=color:#DC4C4C>SISMEMEBER KEY VALUE</span>

Da li u setu sa KEY postoji VALUE



<span style=color:#DC4C4C>SMOVE KEY_SET1 KEY_SET2 VALUE</span>

Premesta vrednost iz set 1 u set 2



<span style=color:#DC4C4C>SPOP KEY COUNT</span>

Brise 2 elementa u setu sa KEY

npr: SPOP k1 2 ->brise 2 elementa iz k1



<span style=color:#DC4C4C>SREM KEY VALUE</span>

Brise specificnu vrednost u setu



### Map

![Group 234](C:\Users\radoj\Downloads\Group 234.png)



Map je malo kompleksniji da se razume,pa ova slika pomaze,imamo key koji je vezan za radis i objekat map koji je vezan za taj key,a taj objekat key ima svoje keys i values



<span style=color:#DC4C4C>HSET KEY_REDIS (MAP_KEY,MAP_VALUE)</span>

npr:

HSET k1 age 27 name ilija

napravice map koji sadrzi k1-> 

​													  age -> 27

​										             name -> ilija

<span style=color:#DC4C4C>HGETALL REDIS_KEY</span>



<span style=color:#DC4C4C>HGET KEY FIELD</span>



<span style=color:#DC4C4C>HLEN</span>

Vraca length 



<span style=color:#DC4C4C>HEXISTS KEY MAP_KEY</span>

1->Postoji

0->Ne postoji





Sorted Set

Ovo radi na osnovu SCORE vrednosti,po njoj soritra vrednosti,ne automatski



<span style=color:#DC4C4C>ZADD KEY SCORE VALUE</span>

Dodaje u sorted set

 

<span style=color:#DC4C4C>ZRANGE KEY START_INDEX END_INDEX </span>

Vraca elemente



<span style=color:#DC4C4C>ZPOPMIN KEY NUMBER_OF_ELEMENTS</span>

Izbacuje min elemente n puta



<span style=color:#DC4C4C>ZPOPMAX KEY NUMBER_OF_ELEMENTS</span>

Izbacuje maxelemente n puta



## Spring

Kako bi radili Redis moramo da imamo server koji se ranuje,to cemo sve da dokerizujemo.

Ako hocemo da imamo lep UI za radis koristimo `Another Desktop Redis Manager`

Kljucni pojmovi:

-  <span style=color:#DC4C4C>RedisConnection</span> ->ovo je core building block za komunikaciju sa Redis serverom 
- <span style=color:#DC4C4C>RedisConnectionFactory</span> ->ovo je fabrika koja stvara RedisConnection
- 

Zavisi kako konfigurisemo u spring-u on ce da vraca ili svaki put novu konekciju ili ce da koristi postojace preko connection pool-a

### Sync

Postoje 2 tipa driver-a koje mozemo da koristimo da se povezemo sa radis serverom

<span style=color:#DC4C4C>Lettuce</span>

```xml
<dependency>
  <groupId>io.lettuce</groupId>
  <artifactId>lettuce-core</artifactId>
  <version>6.1.1.RELEASE</version>
</dependency>
```

<span style=color:#DC4C4C>Jedis</span>

```xml
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>3.7.0</version>
</dependency>
```

Podklase `RedisConnectionFactory` su:

1. <span style=color:#DC4C4C>LettuceConnectionFactory </span>-> <span style=color:#DC4C4C>LettuceConnection</span>
2. <span style=color:#DC4C4C>JedisConnectionFactory</span>-> <span style=color:#DC4C4C>JedisConnection</span>

Da bi radili sa redisom moramo da dodamo dependency

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

####  Raw

Sami cemo da definisemo bean-ove za `RedisTemplate<K,V>`

Fora je sto ovde nemamo yml kada hocemo sami i moramo da definisemo core komopnentu za redis,a to je RedisConnectionFactory 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```
@Bean
public FactoryTypeKojKoristimo getFactory(){
	RedisStandaloneConfiguration config=new RedisStandaloneConfiguration(host,port...);
	config.setPort(...);
	config.setHost(...);
	config.setDatabase(...);
	config.setPassword(...);
	//create connection pool,sve zavisi koj je driver 
	LettuceClientConfiguration poolConfig=LettuceClientConfiguration.Buidler()...
	poolConfig.nesto...
	return new FactoryTypeKojKoristimo(config,poolConfig);
}
```

Mi moramo da defisnisemo `RedisTemplate<K,V>` da bi mogli da radimo redis operacije.

```java
@Bean
public RedisTemplate<String,Object> getTemplate(){
	RedisTemplate<String,Object> template=new RedisTemplate();
	template.setKeySerializer(...)
			.setHashKeySerializer(...)
			.setValueSerializer(...)
			.setHashValueSerializer(...)
			.setEnableTransactionSupport(true)
			.setConnectionFactory(nasa)
}
```



Ove serializer-e mi ne pravimo,mozemo,ali ima vec gotovi:

- GenericJackson2JsonRedisSerializer
- GenericToStringSerializer
- Jackson2JsonRedisSerializer
- JdkSerializationRedisSerializer

#### Yml

```yml
spring:
	redis:
		database: ...
		port: ...
		host: ...
		password: ...
		timeout: ...
		factoryType: -> lettuce / jedis
			pool:
				enabled: ...
				max-active: ...
				max_idle: ...
				min_idle: ...
```

Kada imamo ovo mozemo da autowire direktno `RedisTempate` ali ne moramo jer hocemo da radimo sa repositories

Kao sa ostalim stvarimo ovde entiteti imaju anotacije :

- <span style=color:#DC4C4C>@RedisHash(name,ttl)</span> -> ovo ide na nivo klase,kao @Table je samo u redisu
- <span style=color:#DC4C4C>@Id</span>
- <span style=color:#DC4C4C>@Indexed</span>
- <span style=color:#DC4C4C>@TimeToLive(milis)</span> -> stavljamo na nivo klase
- <span style=color:#DC4C4C>@Reference</span> -> stavljamo na referencu neku
- <span style=color:#DC4C4C>@GeoIndex</span> -> ovo je geo,moze se staviti samo na Point,Circle...
- 

Bitna stvar je da klasa implementira `Serializable`

```java
@RedisHash("users")
public class User implements Serializable {
  @Id
  private String id;
  
  @Reference
  private List<Profile> profiles;
  // ...
}

@RedisHash("profiles")
public class Profile {
  @Id
  private String id;
  // ...
}
```

### Transactions

Redis nam omogucava transakcije

To mozmeo da radimo 

1. @Transactional
2. Koriscenje RedisTemplate

Koriscenjem RedisTemplate imamo veci nivo kontrole



```java
redisTemplate.execute(RedisCallback<T>)
			.multi()
			.exec()
			.discard()
			
```

npr:

```
template.execute(new RedisCallback<List<Object>>(){
	...
});
```

### Reactive

Samo koristimo

- <span style=color:#DC4C4C>ReactiveRedisConnectionFactory</span>
- <span style=color:#DC4C4C>ReactiveRedisTemplate</span>

```java
@Bean
public ReactiveRedisConnection getConnection(){
	return new LettuceConnectionFactory(RedisStandaloneCOnfig);
}
```



Mi kada pravimo objekat i cuvamo ga u Redis ,ako pristupimo tom rekordu u  Redis-u vidimo da on ima key-value(takva je redis baza).

`Value` ce biti ono sto smo sacuvali,a `key` ce biti sama klasa tj ono `class.getName()`

To zelimo da promenimo <span style=color:#DC4C4C>@TypeAlias("name")</span>

Pored toga sto uvek moramo da pisemo RedisTemplate<T,V> mozmeo da pisemo StringRedisTempalte,ObjectRedisTemplate...

Kada radimo geoIndex moramo da radimo samo sa klasama kao sto su 

##### Point

```java
Point p=new Point(x,y)
```

##### Circle

```
Circle c =new Circle(center_x,center_y,radius)
```

##### Box

```java
Box b=new Box(Point first,Point second)
```

##### Distance

```java
Distance d=new Distance(value,Metric)
```

##### Metric

Metrics.KILOMETERS

​			MILES

​			.NEUTRAL -> default

##### Polygon

```java
Polygon p=new Polygon(List<Point>)
```

##### GeoResult
