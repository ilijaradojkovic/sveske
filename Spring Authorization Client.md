# Spring Authorization Client

Ovo je neko ko zeli da uzima tokene od authorization server-a.Nas primer je da je ovo `Gateway` mikroservis koji prosledjuje zahteve ali ujedno mora da komunicira sa auth serverom zbog tokena pa je on klijent.



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
    return httpSecurity
        .oauth2Login() -> sa ovim ce da me redirect na login page auth server-a
        .oauth2Client().and().build();
}
```



### Configuring API Gateway as an oauth2 client

Kada namestimo da nam Gateway bude **oauth2 client** on ce da redirektuje user-a na login page autorizacionog severa

Oauth2 client,drugim recima Gateway uspostavlja **session cookie** i cuva access token tako.Tako da access token se ne cuva u **browser storage** vec u **API Gateway** sto povecava security level.

![image-20230613144215224](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230613144215224.png)

### Configuring API Gateway as an oauth2 resource

Ovde GW ocekuje bearer token u requestu i onda koristi introspection endpoint da ga validira



![image-20230613144645470](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230613144645470.png)