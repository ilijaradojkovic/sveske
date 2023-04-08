# Eureka

Eureka nam je 3rd party koja nam omogucava service discovery,load banacing i jednostavna je za test okruzenje da se postavi.



Imamo 2 dependency-ja  kao kod kafke,jedan za client,i jedan za server.

Napravimo posebnu app gde ce se nalaziti eureka i samo to,i ostale app koje su mikroservisi i gateway.Samo ce eureka da sadrzi server dependency,a ostale ce client.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

Na severskoj strani dodamo @EnableEurekaServer

## Server Side

Dodamo  server dependency.

Dodamo @EnableEurekaServer

```yaml
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

Dodamo ovu konfiguraciju gde smo promenili port,i konfig za erueku da ne registruje sam sebe kao servis.Ako odemo na localhost:8761 otvorice nam se mali UI za eureku.

44:21

Koriscenje load balancera je veoma mocno,iza scene to radi tako sto koristi spring cloud load balancer apstrakciju da u  client memory odluci kojoj instanci da da request da obradi,mi mozemo da promenimo ovaj algoritam rada koji je most recently used,na round robina npr itd.

Mi kada hocemo da gadjamo rute radimo sa 

lb://imeapp...

bitno nam je ovo imeapp to je application.name ,takodje ime koje vidimo u eureka UI

## Client Side

Dovoljno je da imamo samo dependency za eureka client i da se eureka server run da se oni sami regitruju.

I idemo @EnableEurekaClient 

eureka.client.serviceUrl.defaultZone=http://localhost:9999/eureka/ ako ne nadje odmah eura server moramo definisati putanju do njega