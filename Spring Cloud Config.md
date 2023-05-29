# Spring Cloud Config

Nasu konfiguraciju cemo stavljati u conifg server,necemo sve gomilati u yml fajlove na mikorservisima

Fora je sto ce svi mikroservisi da vuku informacije iz jednom config servera

![1_eqIGnnJFi4DAAGtU7l71lQ](C:\Program Files\Typora\resources\Docs\img\1_eqIGnnJFi4DAAGtU7l71lQ.png)

Ovaj server funkcionise tako sto ima git repository gde izvlaci fajlove tj to su yml fajlovi za svaki profile.

Znaci imace fajlove

```
microservice1-dev.yml
microservice2-dev.yml
microservice1-prod.yml
micorservice2-prod.yml
...
```

Tako da nam treba novi repositorijum koji ce da sadzi sve ove fajlove.Znaci odemo u intelij i napravimo nmovi projekat koji ce biti repository i u njemu cemo imati samo ove fajlove.Promena svakog fajla i njihov commit ce Spring Cloud Config Server da primeti i odmah ce promeniti.

Za nas stvarni projekat napravimo strukturu

```
SpringCloudConfig
ProductMicroservice
AccountMicroservice
```

imacemo ova 3 modula.

## Spring Cloud Config

Dodamo dependecy

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```



Takodje moramo dodati **@EnableConfigServer**

U SpringCloudCofnig aplpication yml fajl:

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/ilijaradojkovic/GitConfigServer-Demo.git
          username: ilijaradojkovic
          password: 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

```

Moramo da mu damo putanju do github repositorijuma kao i username i password(koji smo generisali kao key u developer settings) da bi mogao da pristupi njemu i tim yml fajlovima za mikroservise.

## Producrt Microservice

Dodamo dependecy

```yml
server:
  port: 8081
spring:
  cloud:
    config:
      uri: http://localhost:8080
      profile: dev
  application:
    name: Account-Microservice
```

Bitano je da kazemo gde se nalazi config server u lokalu da bi ga ciljao da povuce konfiguraciju kao i profile koji koristi(default profile je default)

mormao reci i **application.name jer ovo mora biti ime fajla na gitu**

na gitu bi imali fajl Account-Microservice.yml i on ce automatski povezati taj fajl sa yml fajlom tog mirkoservisa jer je to application name isti

Ako imamo razlicite profiles:

Imacemo odgovarajuce profile na githubu tj fajlove koje config server gleda

```
microservice1-dev.yml
microservice2-dev.yml
microservice1-prod.yml
micorservice2-prod.yml
```

a u microservice cemo imati samo ovo profile ukljuceno da on zna koji da gleda.Znaci bitno je **ApplicationName-profile.yml**



## Dynamic Config

Mi znamo da ako promenimo samo yml fajl i commit ga promene ce se odraziti odmah na config server,ali ne i na klijenta to je teze malo.

Ovo postizemo tako sto ubacimo actuator dependency

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

i kako bi se beanovi refresh moramo da dodamo anotaciju na @configuration klasi  **@RefreshScope**

Da bi radilo kada promenimo fajl rucno moramo da pozovemo actuator rutu microservicelocalhost:port/actuator/refresh i onda ce on sam da refresh

Takodje moramo da enable ovu rutu u actuatoru jer se po defatultu enalbe samo **heath**



```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



Ako imamo strukturu fajlova na gitu 

```java
account->
	account-dev.yml
	account-prod.yml
product->
	product-dev.yml
	producy-prod.yml
```



Ovo radimo tako sto dodamo search-paths u applicaiton.yml u config server-u da bi usao u fajlove i proverio te direktorijume

```
search-paths: account,product
```

Ako imamo vise foldera razdvajamo ih **,**

## Encrypting

Mi imamo sada yml file koji sadrzi nase pasword itd,i sve je to u plain textu.Moramo nekoako da **encrypt**

### Symetric

Odemo u application.yml u mikroservisu Spring-Cloud i unesemo:

```yml
encrypt:
  key: fhf73odjsjkhHld98yuH983ndksku48slfhcflfdjG
```

Ovo je neka random vrednost

Mi imamo nekoliko ruta koje spring-config server daje a to su 

1) POST localhost:8080/encrypt (U body TEXT neki) i taj text ce da encrypt koristeci kljuc koji smo definisali 

2) POST localhost:8080/decrypt (U body ENCRYPT TEXT neki) i taj text ce se decrypt

   

   I sada ostale vrednosti kao sto su password na fajlovima na gitu stavljamo:

    **{cipher} encrypted**

```yml
    user: postgres
    password: "{cipher}df38d3217ddcb52d79021a05713d38bb6b9a382b3dc7534c60a7dc49466d718c"
```

Bitno je da ima {cipher} jer ce tako prepoznati da treba da decrypt.

MORA POD "..."

Ovo je sve ok ali mi kada uradimo localhost:8080/mirsoervice/dev vratice nam sve yml podatke i ti podaci nece biti encrypt,sta se tu desava?

Sprign Cloud Config ce znati da decrypt ove podatke ,ali ce to raditi pri svakom pozivu,gde je i ovaj poziv gde mi uzimamo da vidimo sve podatke on ce nam to decrypt i vratiti plaintext sto je lose.

Hocemo da se dekripcija radi na **client strani**

znaci moramo da dodamo na **client** strani da raid dekripciju:

```yml
decrypt:
  key: fhf73odjsjkhHld98yuH983ndksku48slfhcflfdjG
```



a na server strani :

```yml
encrypt:
  enabled: false
```

I to onemogucava da config server encrypt,vec smo to prebacili na clienta