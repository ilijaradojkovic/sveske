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
    //nove verzije idu ovako
     .oauth2ResourceServer(x->x.jwt(y->y.jwtAuthenticationConverter(jwtAuthConverter)))

            return http.build();
}
```

jako je bitno da kazemo da je ovo

 oauth2ResourceServer()  -> Tretirace celu app kao resource server

 jwt() -> autoriacija se vrsi preko jwt tokena



Ovo je potrebno samo da se pokrene jednostavan Spring Authorization Resource Server

## JwtAuthenticationConverter 

Ovo je klasa koja konvertuje `raw jwt` u AbstractAuthenticationToken.Ovaj konverter bi trebalo da ekstraktuje rolove iz njega i da ih lepo sredi

primer:

```java
@Component
public class JwtAuthConverter implements Converter<Jwt, AbstractAuthenticationToken> {

    private final JwtGrantedAuthoritiesConverter jwtGrantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();

    private final JwtAuthConverterProperties properties;

    public JwtAuthConverter(JwtAuthConverterProperties properties) {
        this.properties = properties;
    }

    @Override
    public AbstractAuthenticationToken convert(Jwt jwt) {
        Collection<GrantedAuthority> authorities = Stream.concat(
                jwtGrantedAuthoritiesConverter.convert(jwt).stream(),
                extractResourceRoles(jwt).stream()).collect(Collectors.toSet());
        return new JwtAuthenticationToken(jwt, authorities, getPrincipalClaimName(jwt));
    }
    private String getPrincipalClaimName(Jwt jwt) {
        String claimName = JwtClaimNames.SUB;
        if (properties.getPrincipalAttribute() != null) {
            claimName = properties.getPrincipalAttribute();
        }
        return jwt.getClaim(claimName);
    }
    private Collection<? extends GrantedAuthority> extractResourceRoles(Jwt jwt) {
        Map<String, Object> resourceAccess = jwt.getClaim("realm_access");
        List<String> roles = (List<String>) resourceAccess.get("roles");
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toSet());
    }
}
```

Bitno je da nam metoda conert vrati tip ove `AbstactAuthenticationToken` to mozemo da napravimo preko <span style="color:red">JwtAuthenticationToken</span>

### AbstractAuthenticationToken 

​	on nasledjuje `Authentication` pa je springu potreban da uradi auth.