# CSV

prevod je comma separated values,tj to su vrednosti koje su razdvojene **,** i **/n**

, -> razdvajamo vrednosti istog objekta,kao varijable

/n -> razdvajamo razlicite objekte

```csv
name,age,email
ilija,23,radojkovicika@gmai.com
test,23,test@gmail.com
```



Svaki csv fajl mora da ima prvi red koji objasnjava koja su koja polja

## Parsing

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>4.1</version>
</dependency>
```

Moramo da dodamo ovu biblioteku



```java
@AllArgsConstructor()
@NoArgsConstructor()
@Data
public class Person {
    @CsvBindByName(column = "name")
    private String name;
    @CsvBindByName(column = "age")
    private Integer age;
    @CsvBindByName(column = "email")
    private String email;
}
```



sada oznacimo odgovarajuca polja sa @CsvBindByName(columnName) to mora da se match sa prvim redom csv fajla,taj prvi red objasnjava koja su koja polja



```java
     List<Person> people = new CsvToBeanBuilder(new FileReader("persons.csv"))
                .withType(Person.class)
         		.withSeparator(';') -> menja separator da ne bude , vise
         		.skipLines(2) -> ako imamo neki text pre csv headera
                .build()
                .parse();
```

sa ovim procitamo csv file.

<span style="color:red">CsvBindByName</span>(name) -> bind polje klase po imenu koje je definisano u prvom redu csv fajla

<span style="color:red">CsvBindByPosition</span>(position)-> bind polje klase po poziciji informacije u csv fajlu 0,1...

<span style="color:red">CsvDate</span>(dateformat)

<span style="color:red">@CsvBindAndJoinByName(column = "name", elementType = String.class)</span> ->spaja vrednosti

```csv
id,name
1,John,Doe
2,Jane,Smith
```

vidimo da imamo vise vrednosti koje se ponavljaju,ovo name.Kako izdvajamo ovo? pa preko ovoga join da bi ih skupilo u listu

```java
public class Person {
    @CsvBindByName(column = "id")
    private int id;

    @CsvBindAndJoinByName(column = "name", elementType = String.class)
    private List<String> names;

    // getters and setters
}
```

Imamo ovaj fajl i koriscenjem 

<span style="color:red">@CsvBindAndSplitByName(elementType,splitOn)</span>

```java
name,tags
Alice,tag1;tag2;tag3
Bob,tag2;tag4
Charlie,tag1;tag3;tag5
```

Imamo vrednosti koje pripadaju istoj varijabli ali su spojene(tj one su radvojene nekim znakom pa ce ovde ove tagove posmatrati kao 1 vrednost,da smo stavili , onda bi posmatrao odvojeno primer gore)

```java
public class Person {
    @CsvBindByName
    private String name;

    @CsvBindAndSplitByName(elementType = String.class, splitOn = ";")
    private List<String> tags;

    // constructor, getters, setters
}
```



## Map values in csv

```csv
id,name,properties
1,John,"age:30|gender:male"
2,Jane,"age:25|gender:female"
```



Mozemo predstaviti map vrednosti u csv fajlu,npr ovde su smestene u "..." i razdvojene sa | delimitrom

1) Mozemo sami da pisemo custom to bean 

   koristimo da implementiramo interfejs <span style="color:red">CsvToBeanProcessor<T></span> i onda cemo sami da izdvojimo po | i tako to

   



## Primeri

```csv
Username,Password,Datum Rodjenja,Starost
ilija,1234,20.04.2020,20
```



```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class MyUser {


    @CsvBindByName(column = "Username")
    private String username;
    @CsvBindByName(column = "Password")

    private String password;
    @CsvDate(value = "dd.MM.yyyy")
    @CsvBindByName(column = "Datum Rodjenja")
    private Date datumRodjenja;
    @CsvBindByName(column = "Starost")

    private int starost;

}
```

