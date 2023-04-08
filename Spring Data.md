# Spring Data

Osnovna biblioteka koju koristimo za rad

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

Kako smo ukljucili spring-boot-starter ne moramo staviti anotaciju	<span style=color:#DC4C4C>@EnableJpaRepositories</span>





![Group 235](C:\Users\radoj\Downloads\Group 235.png)

Imamo base klasu koja je osnova za sve,ostale implementacije zavise od baza koje koristimo.Na primer da koristimo mongo koristili bi specificnu bazu samo za mongo.

Mi sami ne kreiramo direktno query koji se izvrsava nad bazom vez zovemo predefinisane metode.Ali ako hocemo da kreiramo nasu neku koristicemo query postoji i za to opcija.

Nacin na koji se query kreira:

- Imenama metoda
- Custom query

Jasno nam je da custom query mi pravimo i da to se  lepo prevodi jer je vec sql,ali sta je sa prvim nacinom gde mi pomocu imena metode samo uradimo.Za to je odgovoran **Query Lookup Strategies** koje se definise u **@EnableJpaRepository**.Po defaultu ako uvezemo biliboteku preko spring-starter ne moramo da definisemo ovu anotaciju,ali ako ne uvezemo preko nje moramo definisati da bi repository radio.

QueryLookupStrategy.Key.CREATE

​											.USER_DECLARED_QUERY

​											.CREATE_IF_NOT_FOUND(default one)





Kako ova strategije parsuju ime metode u sql?

Spring radi tako sto uzme celo ime metode i iz nje izdvoji dva kljucna dela:

1. Subject 
2. Predicate



Mi definisemo **Repository** neki i on sadrzi predefinisane metode koje koristimo da manupulisemo bazom,samo pre toga moramo namestiti konfiguraciju ka odredjenoj bazi kako bi se repository povezao.

![Group 235](C:\Users\radoj\Downloads\Group 235.png)



FindByAddressZipCode(ZipCode zipCode)

Recimo da imamo ovakvu metodu

```java
class Person{
Address address;
}

class Address{
    ZipCode zipCode
}
```

ZipCode varijabli pristupamo preko Person.Address.Zipcode



Algoritam radi tako sto pronalazi **delimitar** **By** i gelda posle njega,posle By imamo AddressZipCode sledeci korak je da ovo interpretira nekako da split da bi resio lepo.Ovo moze da split na 2 nacina:

- AddressZip i Code
- Address i ZipCode sto je tacno

nekad moze da parsira ovaj prvi slucaj gde bi bila greska pa se koristi delimitar i za to <span style=color:#DC4C4C>_</span>

Address_ZipCode bi bilo pravilnije jer mu ovako mi dajemo da znanja da odvoji .

## Page i Sort

​	Pored osnovih parametra mozemo dodati i objekte 	

- <span style=color:#DC4C4C>Sort</span>
  - <span style=color:#DC4C4C>Pageable</span>

### Sort

​	Sort.by(ime kolone) -> po default-u ce da sortira ASC,ali ime kolone mora biti tacno ili ce baciti exception

​			.by(Direction,imekolone)

​			sort(Class<T>)

​			.by(Order)

### Direction

<span style=color:#DC4C4C>	Direction.ASC</span>

​	<span style=color:#DC4C4C>Direction.DESC</span>

### Order

<span style=color:#DC4C4C>	Order.asc(column)</span>

<span style=color:#DC4C4C>	Order.desc(column)</span>



### Pageable

​	<span style=color:#DC4C4C>Pageable.of(int))</span>

​	<span style=color:#DC4C4C>PageRequest.of(int_page,int_size))</span>

​						<span style=color:#DC4C4C>.of(int_page,int_size,Sort))</span>

​	ovo sve vraca Page<Result>

​	Pageable kada se izvrsi radi i brojanje svih elemenata,pa to nekad moze da bude heavy,zato se koristi Slice<Result>

### Slice

​	Page nasledjuje Slice

​	Slice<Product> FindBy...

## Povrate vrednosti repository metoda

Mi kada definisemo metodu u repository ona moze da vrati:

1. Optional<T>
2. Collection<T>
3. Iterable<T>
4. List<T>
5. Streamable<T>
6. Stream<T>

Streamable je prosirena verzija iterable i dodao je neke funkcionalnosti ali takodje konvertuje u stream.Sluzi da konvertujemo iterable u stream.

​	<span style=color:#DC4C4C>Streamable.of(iterable) -> Stream</span>

## Async in repository

Po default-u metode u repository nisu asinhrone,to znaci da ce biti sinhrone i da ce se cekati da se query execute.

Neke repository postoje koje cim implementujemo dozvoljavaju da sve radi async

Mi mozemo metode oznacini da budu async

<span style=color:#DC4C4C>@Async</span>

Koristicemo ovu anotaciju nad metodama kako bi spring prepoznao da se radi o async metodi.Uz ovo povratna vrednost metode ne moze biti sinhrona vec mora biti **Future<T>**  **CompatableFuture<T>**

```java
@Async
Future<T> findBy...
```



## Projections

Ovo koristimo kada zelimo da vratimo usko specifican rezultat neke klase.Postoje 2 tipa 

1. Closed
2. Dynamic

### 	Closed

Closed su specificne jer znamo povratnu vrednost pa lako mozemo predvideti.

​	Collection<Person> findAll();

nama treba samo lastName ne cela klasa

```java
interface NameOnly{

String getFirstName();

}
```

Collection<NameOnly> findAll();

Sam ce da mapuje samo moramo pogoditi ime da bude isto.

Ovo je bilo jednostavno,mi mozemo da kombinujemo vrednosti u jednu

```java
interface NameOnly{
    @Value("#target.firstname+' ' target.lastname")
    String getFullName();
}
```

Mozemo definisati da on sastavi vrednosti,**target** je vrednost objekta

### Dynamic

```
interface PersonRepo extends Repository<Person,UUID>{
<T> Collection<T> findByName(String name,Class<T> type);
}
```

Moramo da imamo parametar **Class<T>** kako bi znao sa kojom klasom radi,da bi radilo dynamic.

Sada mozemo definisati bilo koju povratnu vrednos ovoga koja se uklapa u scope person

Collection<Person> persons=personrepo.findByName("...",Person.class);

Collection<NamesOnly> names =...

Vidimo da istom metodom zadovoljavamo sve ovo jer su pod scope Person.class

## Procedures

Kreiramo u bazi procedure "Test"

postoje 2 nacina da odradimo proceduru:

1. <span style=color:#DC4C4C>@Procedure</span>
2. <span style=color:#DC4C4C>@NamedStoredProcedureQuery</span>

```java
repository{
@Procedure(imeprocedure)
Collection<Product> imeprocedure();
}
```

Da nam procedura ima parametre ubacili bi u ovo ime motode

ako imamo vise istoimenih parametra resavamo preko <span style=color:#DC4C4C>@Param(Ime)</span>

Da bi zvali proceduru moramo pokrenuti **@Transacion**

## JPA Filters

Ovo su filteri koji se definisu kako bi radili nesto pre ili posle zahteva na jpa,isto kao i spring security filteri samo za data.

Kljucne anotacije koje se koriste:

- <span style=color:#DC4C4C>@FilterDef</span>
- <span style=color:#DC4C4C>@Filter</span>

Mi filtere definisemo koristeci ove dve anotacije.Njihove razlike:

**@FilterDef** anotacija definise parametre za filter,dok **@Filter** definise ime filtera i vredosti parametra.

@FilterDef pisemo na klasi samoj,tj na entitetu sa kojim radimo

```java
@FilterDef(name = "active", parameters = @ParamDef(name = "isActive", type = "boolean"))
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private boolean isActive;

    // other fields and methods
}
```

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Filter(name = "active", condition = "is_active = :isActive")
    List<User> findByName(String name, @Param("isActive") boolean isActive);
}
```

@Filter koristimo da b i primenili prethodno definisan filter

## Custom Query

Mozemo praviti tako sto na metodi u repository stavimo

​	<span style=color:#DC4C4C>@Query(value="sql neki",nativeQuery=true)</span>

Mozemo koristiti query na 2 nacina.

1) Koristicemo raw sql koji unostimo kao value 

   ```java
      @Query(value = "select * from Product",nativeQuery = true)
   	public Product test();
   ```

   Kada koristimo nativQuery to znaci da ce da prevede odmah direktno na jezik

2) Koristimo JPQL

   ```java
   @Query(value = "select p from Product p")
   public Product test();
   ```

​		JPQL je jezik koji koristi java sintaksu mesanu sa sql-om.

```java

  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);
```

Mozemo koristiti <span style=color:#DC4C4C>@Param("Name")</span> da specificiramo parametar koji se prosledjuje u query preko imena,a ne preko broja kao pre.

## Null Handling

```java
interface UserRepository extends Repository<User, Long> {

 1-> User getByEmailAddress(EmailAddress emailAddress);                    

  @Nullable
 2-> User findByEmailAddress(@Nullable EmailAddress emailAdress);          

 3-> Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress); 
}
```

1) Ako ne pronadje **User** objekat baca gresku i kod nam puca
2) Ako ne nadje **User** objekat nece baciti gresku vec ce vratiti null,metodu smo oznacili sa @Nullable
3) Ako ne nadje **User** objekat optional ce biti empty

## Commeting Query

Za ovo koristimo  <span style=color:#DC4C4C>@Meta(comment)</span> anotaciju 

```java
@Meta(comment = "find roles by name")
	List<Role> findByName(String name);
```

Time znamo sta metoda radi i lako mozemo da je nadjemo i debug





## Specification



## Simple Queries

Pageable-> interface koji jpa prima i radi page<T>

PageRequest -> Konkretna implementacija Pageable interface-a

Kada pravimo custom queries moramo da prosledimo Pageable pa to mozemo jednostavno tako sto napravimo PageRequest preko staticke metode of i onda da ga cast u Pageable,jer kad imamo custom queirs moramo da imamo @Param("ime") param mora da stoji na sve varijable osim na Pageable i Sort



## Dynamic Queries

Postoje 3 najvise koriscene biblioteke koje postizu ovo:

- JPA Specifications
- Example matcher
- Query DSL

![image-20221221143553123](C:\Program Files\Typora\resources\Docs\img\image-20221221143553123.png)

![image-20221221143631605](C:\Program Files\Typora\resources\Docs\img\image-20221221143631605.png)

![image-20221221143646324](C:\Program Files\Typora\resources\Docs\img\image-20221221143646324.png)



Ali ovo nije dynamic

Pre smo pravili dynamic queries tako sto pravimo EntityManager pa dodajemo predicates,ali ovo je teze,jer code nije reusable i sve se nabija u jednu metodu

### JPA Specifications

U JDBC-u smo koristili Criteria API za complex query,Specification je API napravlljen na ovom Criteria API.

Ovo je nacin da pravimo **dynamic** query nad bazon,omogucava lako filtriranje podataka na osnovu Specification objekta.



Moramo nalediti <span style=color:#DC4C4C>JpaSpecificationExecutor<T></span>

Bitan koncept je da svaka metoda prima  <span style=color:#DC4C4C>Specification<T></span> objekat i njega pravimo preko

```java
interface Specification<T>{
 
  Predicate toPredicate(Root<T> root, 
            CriteriaQuery<?> query, 
            CriteriaBuilder criteriaBuilder);

}
```

```java
 public static Specification<Product> isLongTermCustomer() {
        return (root, query, builder) -> {
            return builder.equal(root.get(Product_.name),"ilija");
        };
    }
```

Bitno nam je da znamo sta je Product_.name.To je klasa koja je staticki ucitana sa svojim atributima tako da se oni znaju kada se pise kod,to omogucavamo preko anotacije <span style=color:#DC4C4C>@StaticMetamodel(class)</span> na klasi koja zelimo da se staticki ucita.Nama treba da ucitamo staticki da bi znali koja polja ona poseduje kako bi napravili specification objekat

```
@StaticMetamodel(Product.class)
```



Na primer hocemo da napravimo Specification koji ce porediti ime proizvoda

```java
public static Specification<Product> isNameEqual() {
    return (root, query, builder) -> {
        return builder.equal(root.get(Product_.name),"ilija");
    };
}
```

Na primer ime sadrzi i 

```java
public static Specification<Product> isNameEqual() {
    return (root, query, builder) -> {
        return builder.like(root.get(Product_.name),"%i%");
    };
}
```



Ako imamo nested objects pristupamo ovako

```java
public static Specification<Product> isNameEqual() {
    return (root, query, builder) -> {
        return builder.equal(root.get(Product_.address).get(Address_.city),"Jagodina");
    };
}
```

root.get(Product_.address) ce vratiti open neki root pa cemo to moci sa get da idemo dalje u taj objekat. 

Kljucni pojmovi:

- <span style="color:#ff6347">JpaSpecificationExecutor<T></span>
- <span style="color:#ff6347">Specification<T></span>
- <span style="color:#ff6347">@StaticMetamodel(class)</span>



JPA 2 introduces a criteria API that you can use to build queries programmatically. By writing a `criteria`, you define the where clause of a query for a domain class.

To support specifications, you can extend your repository interface with the <span style="color:#ff6347">`JpaSpecificationExecutor`</span> interface, as follows:

```java
public interface CustomerRepository extends CrudRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {
 …
}
```

Kada dodam ovaj interfejs u nas *repository* dobijamo nove metode,i te metode su iste kao i CRUD samo sto mozemo da ubacujemo *specification<T>* objekat koji radi dinamicko filtriranje.

The additional interface has methods that let you run specifications in a variety of ways. For example, the `findAll` method returns all entities that match the specification, as shown in the following example

```java
List<T> findAll(Specification<T> spec);
```

Ovako cemo raditi filtriranje

The <span style="color:#ff6347">`Specification`</span> interface is defined as follows:

```java
public interface Specification<T> { 
    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query,CriteriaBuilder builder); 
}
```

Specifications can easily be used to build an extensible set of predicates on top of an entity that then can be combined and used with `JpaRepository` without the need to declare a query (method) for every needed combination, as shown in the following example:

```java
public class CustomerSpecs {


  public static Specification<Customer> isLongTermCustomer() {
    return (root, query, builder) -> {
      LocalDate date = LocalDate.now().minusYears(2);
      return builder.lessThan(root.get(Customer_.createdAt), date);
    };
  }

  public static Specification<Customer> hasSalesOfMoreThan(MonetaryAmount value) {
    return (root, query, builder) -> {
      // build query here
    };
  }
}
```

The `Customer_` type is a metamodel type generated using the JPA Metamodel generator (see the [Hibernate implementation’s documentation for an example](https://docs.jboss.org/hibernate/jpamodelgen/1.0/reference/en-US/html_single/#whatisit)). So the expression, `Customer_.createdAt`, assumes the `Customer` has a `createdAt` attribute of type `Date`. Besides that, we have expressed some criteria on a business requirement abstraction level and created executable `Specifications`. So a client might use a `Specification` as follows:

The JPA (Java Persistence API) Metamodel Generator is a tool that generates metamodel classes for entities in a Java application. These metamodel classes provide a programmatic representation of the entity model, which can be used in Java code to refer to entity attributes and relationships.

The JPA (Java Persistence API) Metamodel Generator is a tool that generates metamodel classes for entities in a Java application. These metamodel classes provide a programmatic representation of the entity model, which can be used in Java code to refer to entity attributes and relationships.

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <version>5.3.7.Final</version>
</dependency>
```

<span style="color:#ff6347">@StaticMetamodel(class)</span>

```java
@Entity
@Table(name = "EMPLOYEE")
@StaticMetamodel(Employee.class)
public class Employee {
    @Id
    @Column(name = "ID")
    private Long id;

    @Column(name = "FIRST_NAME")
    private String firstName;

    @Column(name = "LAST_NAME")
    private String lastName;

    // Getters and setters omitted for brevity
}
```

iz ovoga ce se generisati 

```java
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetamodel(Employee_.class)
public abstract class Employee_ {
    public static volatile SingularAttribute<Employee, Long> id;
    public static volatile SingularAttribute<Employee, String> firstName;
    public static volatile SingularAttribute<Employee, String> lastName;
}
```



In this example, the `Employee_` class is the generated metamodel class for the `Employee` entity. The `SingularAttribute` fields in the `Employee_` class represent the attributes of the `Employee` entity, and can be used in criteria queries to specify which attributes to include in the query result or to specify conditions for the query.

Mi kada oznacimo tu klasu sa metamodel onda ce ona pretvoriti sve njene atribute da budu staticko dostupni svuda pa cemo tako praviti query jer su u celom programu staticko dostupni atributi

Employee_.lastName i znamo da ima lastName



```java
List<Customer> customers = customerRepository.findAll(isLongTermCustomer());
```

Why not create a query for this kind of data access? Using a single `Specification` does not gain a lot of benefit over a plain query declaration. The power of specifications really shines when you combine them to create new `Specification` objects. 

`Specification` offers some “glue-code” default methods to chain and combine `Specification` instances. These methods let you extend your data access layer by creating new `Specification` implementations and combining them with already existing implementations.

to su metode na specification or and...

If you have to do a join (for example if you want to retrieve `Vehicle.city.id`) then you can use:

```java
return (root, query, cb) -> cb.equal(root.join("city").get("id"), city);
```

[`Root`](https://docs.oracle.com/javaee/6/api/javax/persistence/criteria/Root.html) is the root of your query, basically *What* you are querying for. In a `Specification`, you might use it to react dynamically on this. This would allow you, for example, to build one `OlderThanSpecification` to handle `Car`s with a `modelYear` and `Drivers` with a `dateOfBirth` by detecting the type and using the appropriate property.

Similiar [`CriteriaQuery`](https://docs.oracle.com/javaee/6/api/javax/persistence/criteria/CriteriaQuery.html) is the complete query which you again might use to inspect it and adapt the Predicate you are constructing based on it.

```java
 Specification.where(
                (root, query, criteriaBuilder) ->{
                    Predicate datesPredicate=criteriaBuilder.between(root.get(Faktura_.datumIzdvanja.getName()), dateFromInstant, dateToInstant);
                    Predicate komitentIdPredicate= (komitentId!=null)? criteriaBuilder.equal(root.join(Faktura_.komitent.getName()).get(Komitent_.id.getName()),komitentId): null;
                    Predicate grupacijaIdPredicate= (grupacijaId!=null)? criteriaBuilder.equal(root.join(Faktura_.komitent.getName()).join(Komitent_.grupacija.getName()).get(Grupacija_.id.getName()),grupacijaId): null;
                   ArrayList<Predicate> predicates = new ArrayList();


                    if(datesPredicate!=null){
                        predicates.add(datesPredicate);
                    }
                    if(komitentIdPredicate!=null){
                        predicates.add(komitentIdPredicate);
                    }
                    if(grupacijaIdPredicate!=null){
                        predicates.add(komitentIdPredicate);
                    }

                    return criteriaBuilder.and(predicates.toArray(new Predicate[0]));

                }

                );
```



Ovde imamo vise and slucaja tj vise predicates kkoje moramo da zadovoljimo,napravimo ArrayList i preko nje  ubacujemo.Bitno je da vodimo racuna da predicate ne bude null,jer ako je null nece da raid .

### Example Query

Kljucni koncepti:

- <span style="color:#ff6347">ExampleMatcher</span>
- <span style="color:#ff6347">Example</span>
- <span style="color:#ff6347">QueryByExampleExecutory<T></span>

Query by Example (QBE) is a user-friendly querying technique with a simple interface. It allows dynamic query creation and does not require you to write queries that contain field names. In fact, Query by Example does not require you to write queries by using store-specific query languages at all.

The Query by Example API consists of four parts:

- Probe: The actual example of a domain object with populated fields.

- `ExampleMatcher`: The `ExampleMatcher` carries details on how to match particular fields. It can be reused across multiple Examples.

- `Example`: An `Example` consists of the probe and the `ExampleMatcher`. It is used to create the query.

- `FetchableFluentQuery`: A `FetchableFluentQuery` offers a fluent API, that allows further customization of a query derived from an `Example`. Using the fluent API lets you to specify ordering projection and result processing for your query.

  Before getting started with Query by Example, you need to have a domain object. To get started, create an interface for your repository, as shown in the following example:

  Example 109. Sample Person object

  ```java
  public class Person {
  
    @Id
    private String id;
    private String firstname;
    private String lastname;
    private Address address;
  
    // … getters and setters omitted
  }
  ```

  The preceding example shows a simple domain object. You can use it to create an `Example`. By default, fields having `null` values are ignored, and strings are matched by using the store specific defaults.

  Examples can be built by either using the `of` factory method or by using [`ExampleMatcher`](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#query-by-example.matchers). `Example` is immutable. The following listing shows a simple Example:

  ```java
  public interface Repo extends JpaRepository< MUser,UUID>, QueryByExampleExecutor<MUser> {
  
  ```

  <span style="color:#ff6347">`QueryByExampleExecutor`</span>

```java
Person person = new Person();                         
person.setFirstname("Dave");                          

Example<Person> example = Example.of(person);
```

Sva polja person klase osim firstName su null,pa ce ih ExampleMatcher ignorisati

You can run the example queries by using repositories. To do so, let your repository interface extend `QueryByExampleExecutor<T>`

```java
Person person = new Person();                          
person.setFirstname("Dave");                           

ExampleMatcher matcher = ExampleMatcher.matching()     
  .withIgnorePaths("lastname")                         
  .withIncludeNullValues()                             
  .withStringMatcher(StringMatcher.ENDING);            

Example<Person> example = Example.of(person, matcher);
```

Po defaultu kada napravimo Example.of(person) on ce korstiti default matcher ,ali mi mozemo sami da definisemo pravila

By default, the `ExampleMatcher` expects all values set on the probe to match. If you want to get results matching any of the predicates defined implicitly, use `ExampleMatcher.matchingAny()`.

You can specify behavior for individual properties (such as "firstname" and "lastname" or, for nested properties, "address.city"). You can tune it with matching options and case sensitivity, as shown in the following example:

Example 113. Configuring matcher options

```java
ExampleMatcher matcher = ExampleMatcher.matching()
  .withMatcher("firstname", endsWith())
  .withMatcher("lastname", startsWith().ignoreCase());
}
```

ili koriscenjem lambda izraza

```java
ExampleMatcher matcher = ExampleMatcher.matching()
  .withMatcher("firstname", match -> match.endsWith())
  .withMatcher("firstname", match -> match.startsWith());
}
```



```java
Optional<Person> match = repository.findBy(example,
    q -> q
        .sortBy(Sort.by("lastname").descending())
        .first()
);
```

```java
MUser user=new MUser();
user.setName("nini");

Example<MUser> of = Example.of(user);

return repo.findAll(of);
```

Ovo ce da match sve MUser objekte u bazi koji imaju Name ="nini"

Po defaultu Example se pravi kao ExampleMatcher.All() pa ako uradimo ovo

<span style="color:#ff6347"> ExampleMatcher.All()</span>

<span style="color:#ff6347"> ExampleMatcher.Any()</span>

```java
MUser user=new MUser();
user.setName("nini");
user.setActive(true);

Example<MUser> of = Example.of(user,ExampleMatcher.matchingAny());

return repo.findAll(of);
```

on ce traziti Name i Active da budu zadovoljeni,zato smo ovde ubacili matchingAny()





<span style="color:#ff6347">withIgnorePaths</span>

```java
MUser user=new MUser();
        user.setName("nini");
        user.setActive(true);
        ExampleMatcher matcher=ExampleMatcher.matchingAny().withIgnorePaths("name");

        Example<MUser> of = Example.of(user,matcher);

        return repo.findAll(of);
```

namestili smo Name i Active i napravili ExampleMatcher Any ali ignorise name sa 



<span style="color:#ff6347">withIgnoreCase()</span>

<span style="color:#ff6347">withIgnoreCase(Field)</span>

```java
     MUser user=new MUser();
        user.setName("Nini");
        user.setActive(true);
        ExampleMatcher matcher=ExampleMatcher
                .matchingAny()
                .withIgnoreCase("name");

        Example<MUser> of = Example.of(user,matcher);

        return repo.findAll(of);
```



Imamo withIgnoreCase koji ignorise case ,ako ostavimo samo withIgnoreCase() onda ce ignorisati case za sve,a ne za specificno polje

<span style="color:#ff6347">withStringMatcher(StringMatcher)</span>

```java
 MUser user=new MUser();
        user.setName("i");
        ExampleMatcher matcher=ExampleMatcher
                .matching()
                .withStringMatcher(ExampleMatcher.StringMatcher.ENDING);

        Example<MUser> of = Example.of(user,matcher);

        return repo.findAll(of);
```

Setujemo ime na neki string i uradimo ovaj matchin,u ovom primeru je ENDING sto znaci da ce geldati rekorde u bazi i videti da li je ENDING na "i"

postoje jos StringMatcher-i:

- ENDING
- STARTING
- EXACT
- DEFAULT
- CONTAINING

<span style="color:#ff6347">withTransformer(field,Transofrmer)</span>

```java
MUser user=new MUser();
user.setName("NINI");
ExampleMatcher matcher=ExampleMatcher
        .matching().withTransformer("name",x->{
            System.out.println(x.get());

            return Optional.of(x.get().toString().toLowerCase());
        });

Example<MUser> of = Example.of(user,matcher);

return repo.findAll(of);
```

Za ovaj transformer koristimo lambda jer je funkcionalan interfejs.Ovo nam sluzi da mi ako imamo vrednosti u klasi koje nisu dobre ili nisu odgovarajuce transformisemo u dobre kako bi on lepo poredio sa bazom,zato mi ovde NINI transofmisemo u lowerCase jer u bazi imamo nini pa da poredi tacno



<span style="color:#ff6347">.withIncludeNullValues()</span>

Ukljuci ce i null da poredi u bazi,ovako bi ignorisao null vrednosti za poredjenje sa poljima ,ali ovako bi poredio,pazi jer moze porediti null sa 23 npr sto nece biti tacno

### Query DSL

Ovo je jos jedan biblioteka koju mozemo da koristimo da filter vrednosti dynamic,za sada meni najbolja.

Kljucni koncepti:

- <span style="color:#ff6347">QClass</span>
- <span style="color:#ff6347">Expression</span>
- <span style="color:#ff6347">BooleanExpression</span>
- <span style="color:#ff6347">QuerydslPredicateExecutor</span>

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
</dependency>
```

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.mysema.maven</groupId>
            <artifactId>apt-maven-plugin</artifactId>
            <version>1.1.3</version>
            <executions>
                <execution>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/generated-sources/java</outputDirectory>
                        <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



Moramo dodati dependencies i annotaion process plugin kako bi kada build sve izgenerisao posebne klase sa kojima radimo query.

Te klase su u formatu QImeKlase,ako imamo klasu

```java
...
class Product{
@Id
private int id;
private String name;
...
}
```

on ce izgenerisati klasu QProduct preko koje radimo query metode,slicno generisanje kao kod specifications.

```java
QProduct product=QProduct.product
```



Ovako smo napravili product query objekat preko koga radimo query.

```java
private BooleanExpression filterByStatus(PropertyStatus propertyStatus,QProperty property){
    if(propertyStatus==null) return null;
    return property.status.eq(propertyStatus);
}
```

Sve sto radimo sa ovim nacinom vraca BooleanExpression tako da mozemo nadovezivati itd.

za svaki filter napravimo posebnu metodu kako bi mogli kasnije da je reuse.

property.polje_koje_query.kojametoda

eq->equasl

Ovo mi je bolje jer sam otkerio kako se radi podjenje u listi

```java
property.intervals.any().pk.interval.eq(interval)
```

intervals je lista,sa any kazemo da bilo koj element zadovoljava ovo



i u repository moramo naslediti QuerydslPredicateExecutor koji nam daje mogucnost da u metodama prosledimo BooleanExpression i na osvnovu toga filter sve.

filterByName(name).and(...).and(..)-> ako nam je name null ova metoda vraca null i necemo moci da radimo and pa to resavamo ovako dole

```
Expressions.allOf(BooleanExpression) -> ovako cemo da unosimo filter metode jer ako je jedna null ono and nece da radi pa sve naguramo ovde ,a on preskace ako je neka null
```

ili

```java
BooleanBuilder where = new BooleanBuilder()
    .and(ProductQuery.nameEqualTo(name));
```

@NoBeanRepository

The annotation is used to avoid creating repository proxies for interfaces that actually match the criteria of a repo interface but are not intended to be one. It's only required once you start going into extending all repositories with functionality. Let me give you an example:

Assume you'd like to add a method foo() to all of your repositories. You would start by adding a repo interface like this

```java
public interface com.foobar.MyBaseInterface<…,…> extends CrudRepository<…,…> {

  void foo();
}
```

You would also add the according implementation class, factory and so on. You concrete repository interfaces would now extend that intermediate interface:

```java
public interface com.foobar.CustomerRepository extends MyBaseInterface<Customer, Long> {

}
```

Sluzi da oznacimo repository koji zelimo da  se nasledi u stvarni repositroy

## Spring FIlter Data

Bitne anotacije koje su kljucne:

- <span style="color:#ff6347">@FilterDef</span> ->Daje definiciju filtera,ne mora biti na klasi samo moze i na paketu.
- <span style="color:#ff6347">@Filter</span> ->Ovo povezuje entity klasu sa filter definicijom

<span style="color:#ff6347">`@FilterDef(name,defaultCondition,parameters)`</span>

​	name-> ime filtera

​	defaultCondition -> sta filter izvrsava

​	parameters -> koji su parametri nad kojima izrsava



<span style="color:#ff6347">`@Filter(name,condiiton)`</span>

```java
@FilterDef(name="activeFilter",defaultCondition = "active = :active", parameters={@ParamDef(name="active", type="boolean")})
@Filter(name ="activeFilter")
public class MUser {

    @Id
    private UUID id;
    private String name;
    private boolean active;
}
```

Definisemo filter i povezemo ga sa ovom entity klasom,ovime on jos nije enabled vec samo povezan.

Da ga aktiviramo koristimo **EntityManager**

```java
Session unwrap = entityManager.unwrap(Session.class);
Filter activeFilter = unwrap.enableFilter("activeFilter");
activeFilter.setParameter("active",true);
List<MUser> select_m_from_mUser_m = unwrap.createQuery("select m from MUser m", MUser.class).getResultList();
System.out.println(select_m_from_mUser_m);
```

Ovo su bili filteri koji imaju default condition,imacemo vise filtera koji se povezuju na definiciju koja ima default condition,ako hocemo dinamici condition to onde:

```java
@FilterDef(name="activeFilter", parameters={@ParamDef(name="active", type="boolean")})
@Filter(name ="activeFilter",condition="...")
public class MUser {

    @Id
    private UUID id;
    private String name;
    private boolean active;
}
```

Sklonimo default condition sa @FilterDef i ubacimo @Filter(condition)

sa @Filter mozemo oznaciti i kolone



### Kako ovo da koristimo sa JPA?

isto sve ,stavimo filterdef na MUser-a i FIlter ,a jer repository radi sa klasom tog tima on ce da primeni filter nad tim entitetmo,samo pre poziva repository moramo 

```java
Session unwrap = entityManager.unwrap(Session.class);
unwrap.enableFilter("activeFilter");
List<MUser> test = repo.findByName("test");
System.out.println(test);
```

