# MapStruct



Glavni problem koja ova biblioteka resave je mapiranje request objekat u nase dto objekte,ili mapiranje entity obj u nase response,dto objekte



![image-20230407191816443](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230407191816443.png)

Nasi modeli koje vracamo frontu nikad nisu isti kao i entiteti ,vec su skracene verzije.

![image-20230407191854527](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230407191854527.png)

 

Nasi servisi ce primate druge podake ne entity,vec response ili nesto slicno,akcenat je na razlicitosti kolona tih entiteta,kako ne bi rucno radili ova biblioka mapira ovo umesto nas sve.



Dodamo dependency za mapstruct kako bi omogucili anotacije

 <dependency>
       <groupId>org.mapstruct</groupId>
       <artifactId>mapstruct</artifactId>
       <version>1.5.3.Final</version>
     </dependency>

Time omogucujemo anotacije za mapstruct.

Mapstruct radi tako sto sto **anotiramo interfejs** **@Mapper**



Definisacemo 2 objekta

1)

```java
public class Customer {
    private String firstName;
    private String lastName;
}
 
 
public class CustomerDTO {
    private String firstName;
    private String lastName;
}
```

 

Mapiranje je ovde prosto i uocljivo,mozemo

@Mapper

Interface CustomerMapper {

![image-20230407192037081](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230407192037081.png)

 

Jer se polja zovu isto nije potreno da eksplicitno pokazemo maperu koje u koje polje konvertuje

Mi da bi konvertovali moramo uzeti instance ovog interface 

****

```java
Mappers.getMapper(mapperinterface)
```

I onda mozemo koristiti metodu customerToDto koja ce sve to automatski mapirati

 Takodje da bi mogli da autowire mapper da ne pisamo ovo getMapper stalno mozemo da dodamo opcije ovako:

```java
@Mapper(
        imports = {UUID.class, AddOn.class},
        componentModel = MappingConstants.ComponentModel.SPRING,
        injectionStrategy = InjectionStrategy.CONSTRUCTOR
)
```

 

2)

```
public class Customer {
    private String firstName;
    private String lastName;
    private String myTitle;
}
 
 
public class CustomerDTO {
    private String firstName;
    private String lastName;
    private String title;
}
```

 

 

```java
@Mapper

Interface CustomerMapper {

@Mapping(target=”title”,source=”myTitle”)
CustomerDTO customerToDto(Customer customer);

}
```

 

Vidimo da su imena kolona razlicita sto znaci da mapper nece moci da odradi posao do kraja kako ne zna gde u sta da mapira.Ako ostavimo ovako on ce mapirati samo ono sto zna,ostalo ce biti null.

Zato je potrebno deifnisati tj da mu pokazemo gde sta da poveze.

Koristeci Mapping mi njemu zadajemo target tj u sta da mapira,a mapira ce source to je odakle da mapira.

@Mapping(target,source,dateFormat,numberFormat,constant,expression,defaultExpression,ignore,defaultVale)

 

 

 

 

 

 

4.a)

```java
public class Customer {
    private String firstName;
    private String lastName;
    private Title myTitle;
}
 
 
public class CustomerDTO {
    private String firstName;
    private String lastName;
    private String myTitle;
}
 
default String toString(Title title){
    return title.getName();
}
 
 
```

 

```java
@Mapper
Interface CustomerMapper {

@Mapping(target=”title”,source=”myTitle”)

CustomerDTO customerToDto(Customer customer);

}
```

 

On nece znati da prebaci Title objekat sa customer(source) u String sto je target,za to moramo mi rucno da napisemo default metodu u interfejsu koja to radi,a mapper ce sam da je uhvati i stavi na svoje mesto

 

4.b)

```java
public class Customer {
    private String firstName;
    private String lastName;
    private String myTitle;
}
 
 
public class CustomerDTO {
    private String firstName;
    private String lastName;
    private Title myTitle;
}
 
default Title toTitle(String title){
    return new Title(title);
}
```

 

U odnosu na prethodni slucaj ovde je obrnuto,pa moramo napisati default metodu koja konvertuje string u Title,

Sustina je da ce mapper da pokupi povratnu vrednost i videti kako da je konvertuje

 

5.a)

Imamo 2 objekta koja su slicna a ipak razlicita

```java
public class Customer {
    private String firstName;
    private String lastName;
    private Address address;
}
public class Address {
 
   private String street;
   private String city;
   private String streetnum;
   private String country;
 }
public class AddressShort {
   private String street;
   private String city;
 } 
```



```java
 
 
public class CustomerDTO {
    private String firstName;
    private String lastName;
    private AddressShort myAddress;
}
```

 

```java
CustomerDTO customerToDto(Customer customer);
 
 default String toString(Title title){
   return title.getName();
 }
 
 default MyAddress toMyAddress(Address address){
   return new MyAddress(address.getStreet(),address.getCity());
 }

 
 
```

Samo napravimo default fukciju da upravljamo gde koje polje ide

5.b)

Ista situacija samo sto cemo napraviti custom mapper i ukluciti ga u nas vec

 

```java
public class Customer {
    private String firstName;
    private String lastName;
    private Address address;
}
public class Address {
 
   private String street;
   private String city;
   private String streetnum;
   private String country;
 }
public class AddressShort {
   private String street;
   private String city;
 }
public class CustomerDTO {
    private String firstName;
    private String lastName;
    private AddressShort myAddress;
}
```

Jako je bitno da imena varijabli drzi konzistentno ,jer ce mapper lakse da odradi sve

```java
@Mapper
 public interface AddressMapper {

   MyAddress AddressToMyAddress(Address address);
 }
```

 

Kako su ista imena znace da pretvori

 

```java
@Mapper(uses = AddressMapper.class)
 public interface CustomerMapper {


   CustomerDTO customerToDto(Customer customer);

}

 
```

I samo koristimo uses da ga povezemo

6)

Sta radimo kada imamo odvojeno firstName i lastName,treba spojiti sve u jedno,u costomerDTo imamo polje koje je kombinacija drugih u Customer

```java
public class Customer {
   private String firstName;
   private String lastName;
 
 
 }

 

public class CustomerDTO {
   private String firstName;
   private String lastName;
 
   private String fullName;
 }

 
```

 

```java
@Mapper
 public interface CustomerMapper {
   @Mapping(target = "fullName",expression = "java(customer.getFirstName()+customer.getLastName())")
   CustomerDTO customerToDto(Customer customer);
 }
```

On ce da mapuje tako sto ce da spoji polja,java(..) oznacava da koristi java jezik

 

6)

Moze da radi i list konverziju,ako su items u listi razliciti moramo da napravimo posebnu metodu koja ce da ih map.

List<CustomerDTO> customerToDto(List<Customer> customer);

 

7)

Mozemo da mapiramo i vise klasa odjednom u jedno kao source

```java
@Mapping(source = "user.id", target = "id")

@Mapping(source = "user.name", target = "firstName")

@Mapping(source = "education.degreeName", target = "educationalQualification")

@Mapping(source = "address.city", target = "residentialCity")

@Mapping(source = "address.country", target = "residentialCountry")

PersonDTO convert(BasicUser user, Education education, Address address);
```



 

8)

In continuation with the above example, instead of repeating the configurations for both the mappers, we can use the `@InheritConfiguration` annotation. By annotating a method with the `**@InheritConfiguration**` annotation, MapStruct will look for an already configured method whose configuration can be applied to this one as well. 

@InheritConfiguration

 

9)

 

If we want to define a bi-directional mapping like Entity to DTO and DTO to Entity and if the mapping definition for the forward method and the reverse method is the same, then we can simply inverse the configuration by defining `**@InheritInverseConfiguration**` annotation in the following pattern:

 

 

```java
@Mapper
 public interface CustomerMapper {
 
   CustomerDTO convert(Customer customer);
 
   @InheritInverseConfiguration
 
   Customer convert(CustomerDTO customerDTO);
 
 
 }
```

 

Ovo je kada imamo iste nazive varijabli u oba entiteta

 

Ako je klasa anotirana sa @Build prvo ce mapstruct pokupiti build metodu i tako napraviti obj,ali ako nemamo @        Build 

Onda ce pokupiti new konstuktor.

 

10)

Mozemo da validiramo podatke odjednom prilikom konverzije.

 

Mozemo staviti koje hocemo ime klase,bitno je da ime metode bude ValidImePolja(Polje) i da baca exception ako nije ok.On ce automatski sve uraditi u pozadini

```java
public class ValidatorFristName {
   public String ValidFirstName(String firstName){
 
     if(firstName.length()<4){
       throw new RuntimeException();
     }
     return firstName;
   }
 }
```

 

11)

Mi necemo uvek imati iste tipove podataka iz jedna klase u drugu,time moramo da konvertujemo jedan tip u drugi.

Razlikovace nam se source i target,npr imacemo da moramo da konvertujemo int u string ili u long taok nesto.

 

```java
public class CustomerDTO {
   private String firstName;
   private String lastName;
   private LocalDateTime localDateTime;
 
 
 }

public class Customer {
   private String firstName;
   private String lastName;
   private Date date;
 }
```

 

```java
@Mapping(target = "localDateTime",source = "date")
 CustomerDTO convert(Customer customer);

 
```

Lako mozemo da konvertujemo jednu date klasu u drugu

 

 

```java
public class Customer {
   private String firstName;
   private String lastName;
   private int broj;
 }

 

 

public class CustomerDTO {
   private String firstName;
   private String lastName;
   private String broj_string;
 
 
 }
```

 

 

```java
public interface CustomerMapper {
 
   @Mapping(target = " broj_string ",source = " broj ")
   CustomerDTO convert(Customer customer);
 
 
 }
```

 

Konvertovace sam u string

 

Ako bi pokusali obrnuto radilo bi ako unesemo samo tacan input,za netacn tipa za string 20LGE pokusamo da ubacimo u int to ce baciti gresku,a bilo sta mozemo u string jer radi ToString()_



 

12)

 

```java
public enum MyEnum1 {
   *ENUM10*,
   *ENUM11*,
   *ENUM12
\* }

 

public enum MyEnum2 {
   *ENUM20*,
   *ENUM21*,
   *ENUM22*,
   *OTHERS
\* }

 

 

public class Customer {
   private String firstName;
   private String lastName;


   private MyEnum1 enum1;
 }

 

 

public class CustomerDTO {
   private String firstName;
   private String lastName;
 
   private MyEnum2 enum2;
 
 
 }

 

 

public interface CustomerMapper {
 
 
   @Mapping(target = "enum2",source = "enum1")
   @ValueMappings({
       @ValueMapping(source = "ENUM10",target = "ENUM20"),
       @ValueMapping(source = "ENUM11",target = "ENUM21"),
       @ValueMapping(source = "ENUM12",target = "ENUM22"),
       @ValueMapping(source = MappingConstants.*ANY_REMAINING*, target = "OTHERS"),
       @ValueMapping(source = MappingConstants.*NULL*, target = "OTHERS")
   })
   CustomerDTO convert(Customer customer);
 
 
 
 }

 
```

Mi definisemo @mapping jer su imena varijabla razlicita,e sada moramo da definisemo kako se svako mapiranje enuma poredjuje sa drugim enumom.



<span style="color:#ff4d4d">@Mapping</span>(

target -> u sta mapiramo

source -> iz cega mapiramo

dateFormat -> definise format za vreme

numberFormat -> definise format za brojeve

constant -> 

expression ->ovo je java izraz koji se pise pocnje sa "java(javacode...)"

default -> ako nema source uzece ovu vrednost

defaultExpression -> ako nema source uzece ovo

ignore -> da li da ignorise polje)



<span style="color:#ff4d4d">   @ToEntity </span> -> samo oznacava da DTO klasa mapira u entity klasu

```java
@Mapper
public interface CustomerMapper {

    @ToEntity
    CustomerEntity toEntity(CustomerDTO customerDTO);
    
}
```

​    

<span style="color:#ff4d4d"> @IterableMapping(elementTargetType = OrderItemDTO.class)</span> -> Koristimo na maprianja sa kolekcijama

```java
 @IterableMapping(elementTargetType = OrderItemDTO.class)
    List<OrderItemDTO> toOrderItemDTOList(List<OrderItem> orderItems);

```



<span style="color:#ff4d4d">  @MapMapping</span>



## Updating

Nekada nam treba da ne pravimo novi objekat vec da update jedan.

<span style="color:#ff4d4d">   @MappingTarget</span> -> Sta ce se update u target objektu

```java
@Mapper
public interface CarMapper {

    void updateCarFromDto(CarDto carDto, @MappingTarget Car car);
}
```

Prosledjujemo objekat koji zelimo da update (CarDto) i anotiramo sta zelimo da update u njemu (Car)

## Using Constructors

1. Gleda da li postoji konstruktor koji je oznacen sa <span style="color:#ff4d4d"> @Default</span>

2. Ignorise non public konstruktore

3. Prvo uzima prazne konstruktore

4. Ako ih ima vise moramo da oznacimo sa @Default

   ```java
   public class Vehicle {
   
       protected Vehicle() { }
   
       // MapStruct will use this constructor, because it is a single public constructor
       public Vehicle(String color) { }
   }
   
   public class Car {
   
       // MapStruct will use this constructor, because it is a parameterless empty constructor
       public Car() { }
   
       public Car(String make, String color) { }
   }
   
   public class Truck {
   
       public Truck() { }
   
       // MapStruct will use this constructor, because it is annotated with @Default
       @Default
       public Truck(String make, String color) { }
   }
   
   public class Van {
   
       // There will be a compilation error when using this class because MapStruct cannot pick a constructor
   
       public Van(String make) { }
   
       public Van(String make, String color) { }
   
   }
   ```

   Moze se desiti da nas konstruktor ima parametre koji nemaju ista imena sa objektom iz kojeg pokusavamo da map,pa onda nece znati kako da poveze to

   za takvu situaciju moramo da kazemo kako gde da gleda pomocu <span style="color:#ff4d4d">@ConstructorProperties</span>

## Enum

##### Custom name transformation

Kada nemamo @ValueMapping definisano svaka konstanta enuma ce se napirati u `istoimenu` u drugom enumu.

Imacemo i slucajeve gde enum moramo da transformisemo dodavanjem neki prefiksa ili sufiksa za to sluzi ovo.

```java
public enum CheeseType {
    BRIE,
    ROQUEFORT
}

public enum CheeseTypeSuffixed {
    BRIE_TYPE,
    ROQUEFORT_TYPE
}
```

```java
   @EnumMapping(nameTransformationStrategy = "suffix", configuration = "_TYPE")
   CheeseTypeSuffixed map(CheeseType cheese);
```

Za ovo koristimo <span style="color:#ff4d4d">@EnumMapping(nameTransformationStrategy,configuration)</span>

vec postoje predefinisane strategije kao sto je suffix 

configuration je sta radimo sa tim ovde kazmeo da dodajemo _TYPE

Predefinisane strategije:

- ​	suffix ->Dodaje sufiks

- stripSuffix -> Skida sufiks

- prefix -> Dodaje prefiks

- stripPrefix -> Skida prefiks

- case(upper,lower,capital) -> transformise slova

  

Mi mozemo da definisemo i nasu strategiju sami tako sto implrementiramo interfejs <span style="color:#ff4d4d">EnumTransformationStrategy</span>

```
public class CustomEnumTransformationStrategy implements EnumTransformationStrategy {

    @Override
    public String getStrategyName() {
        return "custom";
    }

    @Override
    public String transform(String value, String configuration) {
        return value.toLowerCase() + configuration;
    }
}
```

## Subclass Mapping

Apple.java -> subclass of Fruit.class

Banana.java-> sublcass of Fruit.java

Koristimo anotaciju  <span style="color:#ff4d4d">@SubclassMapping</span>

```java
@Mapper
public interface FruitMapper {

    @SubclassMapping( source = AppleDto.class, target = Apple.class )
    @SubclassMapping( source = BananaDto.class, target = Banana.class )
    Fruit map( FruitDto source );

}
```

Ovo je metoda koja vraca Fruit a mi hocemo da vraca specificnu klasu pa zbog toga stavljamo subclassmapping jer bi onda vratio Fruit a ne Apple.class npr

source -> podtip klase iz koje uzimamo da mapuje mo u nesto

target -> podtip klase u koju mapiramo



## Conditions

 <span style="color:#ff4d4d">@Condition</span> Ovime anotiramo default metodu koja vraca boolean i to ce se pri svakom mapiranju primeniti da proveri,neke koje postoje ce override.

```java
@Mapper
public interface CarMapper {

    CarDto carToCarDto(Car car);

    @Condition
    default boolean isNotEmpty(String value) {
        return value != null && !value.isEmpty();
    }
}
```

