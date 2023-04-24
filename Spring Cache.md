# Spring Cache

Da bi radili sa kesom u Spring-u moramo da dodamo dependency

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
```

Dodamo spring-boot-starter-cache koji nam omogucava interno da imamo kes kao key-value pair unutar nase spring aplikacije.

Postoje neke vrste kesova koji radimo

Da bi radili cache moramo da dodamo anotacije ,on tako radi

Kad god radimo kes moramo da dodamo  <span style="color:red">@EnableCaching</span>

Bitne anotacije:

- <span style="color:red">@Cacheable</span>

Ova anotacija nam sluzi da mi sacuvamo kes u bazi,samo za cuvanje.Prilikom cuvanja ako posaljemo oept taj request on ce ga brze izvrsiti jer ga nalazi u kesu.Ovo stavljamo na get metode

```java
    @GetMapping("/all")
    @Cacheable(value = "property",key = "'2'")
    public  List<Property> getList(){
        System.out.println("List");
        if(property==null) return List.of();
        return List.of(property);
    }

   @GetMapping("/{id}")
    @Cacheable(value = "property",key = "#id")
    public Property getProperty(@PathVariable("id") String id){
        System.out.println("Fetching");

        return property;
    }
```

@Cacheable(

<span style="color:red">value</span> -> alijas za cacheName

<span style="color:red">cacheNames </span>-> Ime/imena kesa gde se storuje

<span style="color:red">key</span> -> SPEL expression za izracunavanje kljuca,po ovome ce da pretrazuje gde je cache

<span style="color:red">KeyGenerator</span> -> Bean name od custom KeyGenerator-a

<span style="color:red">cacheManager</span> -> Bean name od custom CacheManager-a

<span style="color:red">condition</span> -> SPEL expression da li se kresuje ili ne 

<span style="color:red">unless</span> -> SPEL za expression,ovo se evaluira posle izvrsenja metode

<span style="color:red">sync</span> -> kada imamo vise niit koje citaju iz istog kesa

)

Fora je da Cacheable gleda return type metode i za nek id(key) ce da veze tu vrednost

cacheName je mesto gde je kes(gde je hashmap),a pomocu key-a mi nalazimo tacnu vrednost u toj mapi

Ako metoda vraca `Optional` on ce ga sam unwrap i cuva konkretnu vrednost

Postoji `#result` ako to stavimo na key on ce staviti key kao result metode

- <span style="color:red">@CachePut</span>

Ovo nam sluzi da update kes,i on ce se uvek izvrsiti,tj uvek ce se metoda izvrsiti.On ne dodaje kes vec update ako nadje key i name

```java
@PutMapping("/{id}")
@CachePut(value = "property",key = "#id")
public Property updateProperty(@PathVariable("id") String id){
    System.out.println("Update");

    property.setName("updated");
    property.setPrice(100.0);
    return property;
}
```



- <span style="color:red">@CacheEvict</span>

  On brise kes i baze ako nadje key i name da se poklapaju

  postoji param <span style="color:red">allEntries</span> -> brise sve,celu mapu,nece da gleda key pa da trazi u mapi 

  Ova metoda obicno ide sa void

## Redis

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```

Ovde smo dodali i Redis kako bi mogli da raidmo redis cache sa tim

Moramo dodati jedis kao adapter i starter-data-redis kako bi on cuvao to lepo u redis bazu,takodje moramo da pokrenemo redis server