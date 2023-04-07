# Spring Resources

Spring nam omogucava da upravljamo resursima.

Spring **ApplicationContext** je container koji upravlja spring bean-ovima i drugim resursima.Predstavlja nacin da konfigurisemo bean-ove i resurse.

Glavne komponente:

1. <span style="color:red">Resource</span>
2. <span style="color:red">ResourceLoader</span>
3. <span style="color:red">ResourcePatternResolver </span>

U Spring-u imamo Resource interface koji je generalan za eksterne resurse kao sto su fajlovi itd...

Spring provajduje dosta implementacija ovog interfejsa kao sto su

- FileSystemResource
- UrlResource
- ClassPathResource

ResourceLoaer je interfejs koi ucitava resurse ,koristi ga **ApplicationContext** i druge komponente da ucita resurse.

Njegova ekstenzija ResourcePatternResolver koji ucitava resurse bez wildcard patterns.Ucitava resurse samo po paternu kao npr file extensions.

Ovde spadaju i anotacije @Autowired i @Value koj

## Loading File 

Necemo se baviti citranjem fajla klasicnim putem preko biblioteka io ili nio.Fokusiramo se samo nma spring

Svi fajlovi ce se nalazi u **resource direktorijumu**

```java
FileSystemResource f=new FileSystemResource("myFile.txt");
```

```java
ClassPathResource resource = new ClassPathResource("myFile.txt");
```

In ClassPathResource spring searches for the spring configuration file at the ClassPath.



```java
@Value("classpath:data/resource-data.txt") Resource resourceFile;
```

Mi mozemo ucitati fajl iz classpath-a samo pomocu ove anotacije bitno je da u putanji imamo classpath

```java
@Autowired
ResourceLoader resourceLoader;

public Resource loadEmployeesWithResourceLoader() {
    return resourceLoader.getResource(
      "classpath:data/employees.dat");
}
```

Mi mozemo preko klase ResourceLoader da ucitamo fajl

```java
ResourceUtils.getFile(
      "classpath:data/employees.dat");
```

Imamo i ResourceUtils