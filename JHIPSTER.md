# JHIPSTER

Ovo je bibiloteka koja nam autogenerise ceo projekat.

Pokrecemo je preko komandne linije(cmd),i da bi je pokrenuli moramo da instaliramo:

1) Node  (**nvm install verzija**)
2) Yoman (**npm install -g yo**)

i onda mozemo da instaliramo jhipster preko **npm install -g generator-jhipster**



i sada mozemo pokrenuti jhipster generator preko :

 **jhipster** -> ovo je komanda





## JDL Studio

MI ne definisemo pk vec to on sam uradi,bice to long sa nazivom id

Moramo da pazimo na pisanje varijabli ,mora biti camel-case

Imena klasa tj entity moraju da pocinju velikim slovom

Ovde dodajemo nove entitete,moramo napraviti jdl fajl koji sadrzi medjusobne veze entiteta i samo ga import,jhipster ce sve sam da izgenerise.

Import jdl file

```
jhipster jdl ./my-jdl-file.jdl
```



```cmd
jhipster jdl my_file1.jdl my_file2.jdl
```

Ako radimo sa vise jdl fajlova moramo ovako da ih spajamo

```cmd
jhipster jdl ./my-jdl-file.jdl --json-only
```

Ako ne zelimo da generisemo entities nego samo da ih dodamo u jhipster json file

```cmd
jhipster jdl ./my-jdl-file.jdl --force
```

Ovo ce isforsirati da se svi generisu opet





```jdl
entity A
entity A {}
ekvivalentno je jer nema varijabli
```

Ovako generisemo entity ,ali ovako ako ostavimo entity A je prazna

```jdl
entity MyEntityName{
  variableName variableType
}
```

Ovime imamo entity koji ima varijable

```jdl
entity MyEntityName{
 variableName variableType validations...
}
```

Ovime dodajemo validacije koje su build in u sam jdl

npr:

```jdl
entity A {
  name String required
  age Integer min(42) max(42)
}
```

#### Types:

- String
- Integer
- Long
- BigDecimal
- Float
- Double
- Enum
- Boolean
- LocalDate
- ZonedDateTime
- Instant
- Duration
- UUID
- Blob
- ImageBlob
- TextBlob



#### Validacije:

- required
- min(...) ->za brojeve
- max(...) ->za brojeve
- pattern("regex") ->for strings
- unique
- minlenght(...) -> for strings
- maxlenght(...) ->for strings



## Enums

```jdl
enum Naziv{
   ENUMA1,
   ENUM2,
   ENUM3
}
```



## Veze

```jdl
relationship TIP_VEZE{
  DEFINICIJA_VEZE
}

relationship OneToOne {
  A to B,
  B to C,
  C to D,
  D to A
}
```



```
relationship (OneToMany | ManyToOne | OneToOne | ManyToMany) {
  <from entity>[{<relationship name>[(<display field>)]}] to <to entity>[{<relationship name>[(<display field>)]}]+
}
```

- `(OneToMany | ManyToOne| OneToOne | ManyToMany)` is the type of your relationship,
- `<from entity>` is the name of the entity owner of the relationship: the source,
- `<to entity>` is the name of the entity where the relationship goes to: the destination,
- `<relationship name>` is the name of the field having the other end as type,
- `<display field>` is the name of the field that should show up in select boxes (default: `id`),
- `required` whether the injected field is required.

```jdl

relationship OneToOne {
  A to B
}
relationship OneToOne {
  A{b} to B{a}
}

```

Ovo bi bila biderikciona veza,i obe su ekvivalentne

```jdl
relationship OneToOne {
  A{b} to B
}
```

Ovo je unidirekciona veza.Iz A mozemo da dodjemo do B,ali ne iz B do A



```
relationship ManyToMany {
  A{b required} to B{a}
}
relationship ManyToMany {
  A{b} to B{a required}
}

```

Ovo znaci da je jedna od njih obavezna



#### Parent-Child

```
relationship ManyToMany {
  A{parent} to A{child}
}
```

## Options

Opcije mozemo da koristimo kroz anotacije u jdl-u ili preko njihovih naziva.



Anotacije stavljamo na entity

```jdl
<option name> <option entity list> with <option value>
@<option name>(<option value>)
```

Imamo **Options**  i **Options Values**.Vidimo da option values idu u ovim zagradama ili uz ovo with

#### Option names:

- readOnly -> entity nema setere,nema ***options value***

  ```
  readOnly A
  
  
  @readOnly
  entity A 
  
  ```

- dto ->kreira dto objekte,ima mapstruct kao ***option value***

  ```
  dto A with mapstruct
  
  @dto(mapstruct)
  entity A
  
  ```

  Ako ih imamo vise  (**all** je isto sto i *****)

  ```
  dto all with mapstruct
  
  @dto(mapstruct)
  entity A
  
  @dto(mapstruct)
  entity B
  ```

- skipClient -> ne generisi client code,nema ***options value***

  ```
  skipClient * except A
  
  @skipClient
  entity B
  ```

- skipServer -> ne generisi server code,nema ***options value***

```

@dto(mapstruct)
entity A 
```

- service -> kreira service ,ima serviceClass i serviceImpl kao ***option values***
- paginate -> kreira paginaciju ,ima pagination,infinite-scroll kao ***option values***



#### Option values:

- mapstruct ->Whether to create DTOs for your entities, if an entity has a DTO but no service, then 'serviceClass will be used'
- serviceClass -> imace service klasu koja ce da zove repo
- serviceImpl -> imace service klasu koja implementira interface service koja zove repo(bolje)

```
entity A
entity B
entity C

// no service for A -> kako nismo definisali service za a ona to nema
service B with serviceClass 
service C with serviceImpl

```

- pagination ->Pagination as an option is forbidden when the application uses Cassandra
- infinite-scroll ->Pagination as an option is forbidden when the application uses Cassandra
- elasticsearch ->Requires the application to have the searchEngine option enabled

Ako imamo vise njih mormao za svaku da pisemo anotacije,ali ako idemo dugim principom lakse je jer mozemo da koristimo **all** ili *****

## Pitanja:

Sta je ovo u zagradi

- sta znaci kada dodamo opcije,da li uvek moraju da se dodaju?

- Sta se desi ako ne defisnisem neke opcije,npr service?

- Jdl automatski definise pk?Da

- kako se radi manytomany?Nemamo kontrolu

- kako definisati slabe objekte?

- kako deifnisati komplikovani pk? ili da definisem samo surogatni pk pa ove da stasvim na fk

- sta znaci ovo sa metodom?   A to B with jpaDerivedIdentifier

  
  
  ## Filtering
  
  ovako specificiramo param koje prima Criteria object `tipFakture.equals`

## Microservices

Kada definisemo mikroservise koristimo 

```jdl
application{
	config{
		baseName NAME
		applicationType microservice
		servicePort BROJ
	}
	entities A,B

}
```

Moramo da definisemo application kao i konfiguraciju u njeumu.

Ako ne spomenemo da hocemo ove entities (koje smo definisali vec u istom jdl fajlu) on ce ukljuciti sve,ovako samo selektivno birmao

mozemo koristiti entitties * za sve

entities * except A ->sve osim A

### applicationType

- monolith
- microservice
- gateway

### databaseType

- sql
- mongodb
- cassandra
- couchbase
- no



### authenticationType

- jwt
- session
- oauth2

### buildTool

- maven
- gradle

### cacheProvider

- ehcache
- caffeine

### messageBroker

- kafka
- no

### searchEngine

- elasticsearch
- couchbase
- no

### serviceDiscoveryType

- eureka
- consul
- no

### enableHibernateCache

### enableSwaggerCodegen

### enableTranslation

### enableRancherLoadBalancing

### reactive

- true
- false

```
application {
  config {
    baseName gateway1 
    reactive true 
    packageName com.test.gateway
    applicationType gateway
    buildTool maven 
    serverPort 9000
    clientFramework react
    serviceDiscoveryType no
  }
  entities Blog,Post,Tag,Product
}



application {
  config {
	baseName store 
    reactive true 
    packageName com.test.store
    applicationType microservice
    buildTool maven 
    serverPort 9000
    serviceDiscoveryType no
  }
    paginate Post, Tag with infinite-scroll
	paginate Product with pagination
	service all with serviceImpl
	dto * with mapstruct
  entities Blog,Post,Tag,Product
}

entity Blog {
  name String required minlength(3)
  handle String required minlength(2)
}

entity Post {
  title String required
  content TextBlob required
  date Instant required
}

entity Tag {
  name String required minlength(2)
}

entity Product {
  title String required
  price BigDecimal required min(0)
  image ImageBlob
}

relationship ManyToOne {
  Post{blog(name)} to Blog
}

relationship ManyToMany {
  Post{tag(name)} to Tag{post}
}



microservice Blog,Post,Tag,Product with store
```

Kada pravimo mikroservise moramo na kraju da kazemo gde ce se entiteti definisati,ovde imamo gateway u kom ne generisemo entitete vec to radimo u mikroservisu store

to je ova linija microservice entities... with microservicename

Dok u application{..} mi stavljamo entities ... koje ce da koristi tj da zna taj mikroservis

Kada napravimo app moramo da clean install

Mi ako hocemo da menjamo user klasu to ne mozemo,moramo da napravimo neku klasu koja ce da sadrzi user klasu kao polje i onda u toj novokreiranoj klasi mi dodajemo polja koja zelimo.

Takodje postoje ogranicenja kada radimo sa user-om,ako je user pojavi u jdl-u ne smemo imati dto objekte kao i jwt auth

 better to use an userId. samo

Kada radimo sa oauth2 moramo da pokrenemo keycloak,u docker fajlu koji nam vec jhipster provide mozemo da pokrenemo docker compose up da se generise keycloak docker container i tu da se run,to moramo da uradimo jer je app defaul namestena da gadja keycloak

Po defaultu JSHIPER ne generise eureka server to moramo sami

## Entity generation

Mi mozemo da napravimo jhispter app sami,i onda da uradimo navigaciju u taj folder i samo uradimo jhipster entity ime i on ce da nam otvori prozor za generisanje tog entiteta

