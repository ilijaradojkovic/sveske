# Authorization Resource Server

Ovo je svaki mikroservice koji ima konekciju sa bazon,tj u nasem slucaju svaki mikroservis je ujedno i Resource Server.

Ovo ovodimo tako sto unesemo dependency za Resource Server

## 1)

```xml
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>
```

## 2)

Onda moramo namestiti konfiguraciju za ovo u application.yml fajlu

```yml
security:
  oauth2:
    resourceserver:
      jwt:
      #authorization server
        issuer-uri: http://localhost:8080
```

Namestimo url da zna odakle dolaze zahtevi,tj mora sa Authorization Server-a.

## 3)

Zatim u Security Config moramo:

```java
@Bean
public SecurityFilterChain configure(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(x->x.requestMatchers("/**").hasAuthority("SCOPE_message.read"))
            .oauth2ResourceServer()
            .jwt();

            return http.build();
}
```

jako je bitno da kazemo da je ovo

 oauth2ResourceServer()  -> Tretirace celu app kao resource server

 jwt() -> autoriacija se vrsi preko jwt tokena



Ovo je potrebno samo da se pokrene jednostavan Spring Authorization Resource Server