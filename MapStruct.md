# MapStruct

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

