# Cassandra



Ona koristi poseban nosql jezik a to je <span style="color:red"> CQL</span> (Cassandra query language) i veoma je slican SQL jeziku 

KADA PISES NA KRAJU MORA **;**

<span style="color:red">replication factor</span> -> ovo je element koji opisuje na koliko instanci klastera ce se podaci kopirati i slati.Imacemo vise noda pa ce se podaci pisati kao duplikati i na drugim nodovima,tkao da ako jedan padne ostali ce imati te podatke i vice validni.Obicno je 3 dobar broj.Ako 1 padne imamo jos 2 iz kojih mozemo da citamo,pouzdanost.

<span style="color:red">describe keyspaces</span> -> izlista nam sve keyspaces u kasandri

<span style="color:red">describe keyspacename</span> -> prikaze detalje keyspace

<span style="color:red"> use keyspacename</span> -> sada smo uski u taj keyspace i tu mozemo da vrsimo querty

<span style="color:red"> describe tables</span> -> kada udjemo u neki keyspace mi mozmeo da vidimo koje sve tabele imamo



## Povezivanje sa bazom

Da bi povezali sa bazom moramo da ubacimo cassandra depenedncy 

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-cassandra</artifactId>
        </dependency>
```

Kao i za svako povezivanje nama treba username i password i to cemo nabaciti tako sto kreiramo token na **Astri**

gde imamo clientId-> username

​					clientSecret-> password

​					token -> kopiramo ga za posle

Takodje ovo je bilo potrebnmo kada smo raidli povezivanje sa sql azom samo ovo,ali ovde nam treba i nesto sto se zove secure bundle preko koga ce se vrsiti ta konekcija,to skinemo sa **Astre** i stavimo u resource file,stavimo zip file.

```yml
spring:
  cassandra:
    keyspace-name: main
    #clientId
    username: FvsCdXBLoPokDFcdwlmHGOCU
    #clinet secret
    password: UIJXMJW+LLJWEXuA.G_f1EBZ7nJOdmPQrTsvCTDs24rFeb7+s+0.KfshDnNkIoEJ9z,2EwjoTPFmlbK1G4J4v2Hc2f.vHfwXEOgMQ5NMG5F127pkxJYd.ZicoTsnrGqa
    schema-action: create_if_not_exists
    request:
      timeout: 10s
    connection:
      connect-timeout: 10s
      init-query-timeout: 10s
datastax.astra:
    secure-connect-bundle: secure-connect.zip
```



u cassandra deo podesimo 

<span style="color:red"> keyspace-name</span> 

<span style="color:red"> username</span>  -> clientId

<span style="color:red"> password</span>  -> clientServet

<span style="color:red"> schema-action </span> -> kako ce se praviti klase,to je pri pokretanju app sta da radi,da li da kreirano nove ili ne ili da update postojece...

datastax.astra -> nas custom property gde se nalazi lokacija secure bundle fajla,jer njega izvlacimo posle u konfiguracionu klasu 

```java
@Bean
public CqlSessionBuilderCustomizer sessionBuilderCustomizer(DataAstraxProperties astraxProperties){
    Path bundle=astraxProperties.getSecureConnectBundle().toPath();
    return builder->builder.withCloudSecureConnectBundle(bundle);
}
```

Za kraj moramo da definisemo ovaj bean gde smestimo putanju do naseg secure connection fajla.