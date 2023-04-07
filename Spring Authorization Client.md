# Spring Authorization Client

Ovo je neko ko zeli da uzima tokene od authorization server-a.Nas primer je da je ovo Gateway mikroservis koji prosledjuje zahteve ali ujedno mora da komunicira sa auth serverom zbog tokena pa je on klijent.



## 1)

Moramo dodati client dependecy

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```



## 2)

Kako smo pre koristili login za google mi smo bili klijent i morali smo da podesavamo ono secret client id...

e to sada moramo i ovde da uradimo

```yml
  security:
    oauth2:
      client:
        registration:
          gateway:
            provider: my-provider
            client-id: client
            client-secret: secret
            authorization-grant-type: authorization_code
            scope: openid
        provider:
          my-provider:
            #Authorization server 
            issuer-uri: http://localhost:8080
```

## 3)

Moramo da podesimo security

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
    return httpSecurity.oauth2Client().and().build();
}
```