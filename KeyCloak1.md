# KeyCloak

Ovo je autorizacioni server koji mi lokalno instaliramo.Ja sam ga preko Docker-a instalirao i pokrenuo.

```
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev
    container_name: keycloak
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    networks:
      - testing
    ports:
      - "8080:8080"
    volumes:
      - keycloak:/var/lib/keycloak/data
```

* Prvo sam koristio neki drugi provider za keycloak i on je max mogao do verzije 16,pa sam na sajtu njihovom video ovo.Takodje drugaciji su env varijable pre su bila KEYCLOAK_USER i KEYCLOAK_PASSWORD ,a sad imamo admin tu 

  

Kada ga pokrenemo pristupimo portu  na kome smo definisali i onda udjemo u njegovu konzolu,da ga podesimo.

​		`Client` -> nasa app koja se registruje da koristi KeyCloak

​		`Realm` -> ovo je kao prostor za rad,imamo vise realma u zavisnosti od aplikacija(default je master)

​		`User` -> neki user koji se registruje na sistem.



![image-20230609125728195](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230609125728195.png)

![image-20230609131202413](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230609131202413.png)

Samo se login kao admin i ono ce nas odvesti u KeyCloak config konzolu.

![image-20230531103201845](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230531103201845.png)

Ovde mozemo da vidimo da imamo ovo `Master` to cemo da promeni i kreiramo neki nas deo,ja sam napravio test i podesimo

Fora je da ce ako idemo preko dokera na keycloak u locakhostu izbaci ce nam `master` realm login stranu,a mi imamo novi realm pa hocemo da se login preko njega jer su nam tu useri

isao sam prvo na 
http://localhost:8080/realms/nomadesk/account 
pa sam na sign in pa mi izbacilo ovaj link
http://localhost:8080/realms/nomadesk/protocol/openid-connect/auth?client_id=account-console&redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Frealms%2Fnomadesk%2Faccount%2F%23%2F&state=198f4552-0b55-4d0a-987d-0ac9bcb88a61&response_mode=fragment&response_type=code&scope=openid&nonce=48c8620f-6f24-4942-84fb-8080e2183dff&code_challenge=pQjFfgd2PyrHQ5rERHaEXLEvnCdeSI4aMf29S3yNKRU&code_challenge_method=S256

Dodacemo novog client-a

![image-20230531103327878](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230531103327878.png)

Takodje jako bitna stavar je da udjemo u `Realm Settings` -> `Endpoints ` i udjemo u endpoints za OpenId tu ce nam izlistati sve endpointe koji su nama bitni.

Nama keycloak kada vrati jwt mi nemamo direktno roles u njemu vec su smestene kao u vidu mape nekog polja.Mi to moramo sami da mapiramo u roles i da mu kazemo.

## Spring

Moramo prvo dodati dependency

```xml
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>
```



Mi cemo nasu app tretirati kao resource server,jer nam je KeyCloak auth server.Tako da moramo da konfigurisemo resource server

```yml
spring:
  application:
    name: springboot-keycloak
  security:
    oauth2:
      resource-server:
        jwt:
          issuer-uri: http://localhost:8080/realms/test
          jwk-set-uri: http://localhost:8080/realms/test/protocol/openid-connect/certs


logging:
  level:
    org.springframework.security: DEBUG

server:
  port: 8081
  servlet:
    context-path: /api
```

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

i naravno namestimo security config

Verzija pisanja koda je spring-boot 3.0,u drugim verzijama ce izgledati malo drugacije sintaksa

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig
{
    public static final String ADMIN = "admin";
    public static final String USER = "user";
    private final JwtAuthConverter jwtAuthConverter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(
                x->
                x.requestMatchers(HttpMethod.GET, "/anonymous", "/anonymous/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/admin", "/admin/**").hasRole(ADMIN)
                .requestMatchers(HttpMethod.GET, "/user").hasAnyRole(ADMIN, USER)
                .anyRequest().authenticated())
                .oauth2ResourceServer(x->x.jwt(y->y.jwtAuthenticationConverter(jwtAuthConverter)))
                .sessionManagement(x->x.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }


}
```



## Keycloak API

Ovo koristimo da gadjemo endpointe da se izvrse neke komanda

Svaki admin endpoint koji nesto menja na Keycloak Api mora da ima `/admin/realms/`





Moramo da ukljucimo neke biblioteke ako zelimo da radimo sa keycloak klasam i uopste nekako da komuniciramo sa Keyclaok-om.

- **Keycloak Core** -> Keycloak Core refers to the core functionality and services provided by the Keycloak server. It includes the essential components for managing users, roles, realms, client applications, and authentication/authorization mechanisms. Keycloak Core handles the core identity and access management features, including user authentication, authorization, token management, and user federation.
- **Keycloak Adapter Core** -> Keycloak Adapter Core, on the other hand, is a separate library provided by Keycloak that helps integrate Keycloak with various client applications and frameworks. It provides the necessary functionality to secure client applications and interact with the Keycloak server. The adapter core library includes client adapters for different platforms, such as Java, JavaScript, Node.js, and more. These adapters handle tasks like authentication flows, token validation, and secure communication with the Keycloak server.
- **Keycloak Common**

Postoje jos neke uproscene,ali za nas je bitno da imamo samo `keycloak core`.

![image-20230607160932474](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230607160932474.png)

Dodacu keycloak-core

```java
     <dependency>
            <groupId>org.keycloak</groupId>
            <artifactId>keycloak-core</artifactId>
            <version>21.1.1</version>
        </dependency>
```



Kljucne klase:

### AccessToken

Mi kada imamo jwt token mozemo da odemo na onaj sajt JWT.io i tu da ga vidimo sta sve ima,ova klasa u stvari predstavlja to,taj sadrzaj tokena.

Ova klasa je reprezentacija jwt tokena.

AccessToken.NESTO -> daje nam ime polja koje ima

takodje mozmeo je napraviti posto ima prazan konstruktor

```
AccessToken token=new AccessToken();

```

Ovde je problem sto nemamo info o poljima ,mozemo da setujemo sami jer ova klasa ima set metode,ali mi opet imamo problem jer sta cemo sa jwt-jem koji smo dobili tu su info moramo to nekako da `pretvorimo u ovu klasu`

### TokenVerifier

Ovo je klasa koja pretvara cist JWT token (BEZ BEARER DELA) u AccessToken.class

```java
        AccessToken token = TokenVerifier.create(s, AccessToken.class).getToken();
```



## Keycloak Flow Types

Kao sto znamo koji flow types postoje ovde mozmeo njih podesiti

Namestimo realm -> pa idemo na Client -> i tu mozemo da podesimo



#### Authorization code type 

Moramo da stikliramo `Standard Flow`.



Koristimo callback uri: http://localhost:8080/authorization-code/callback i takodje da namestimo polje u nasem klijentu za vlid redirect url

za ostale info uzimamo iz onih well known endoints.

#### Password grant type

Moramo da stikliramo `Direct access grants` 

Ovde samo posaljemo password i username

#### Implicit grant type

Moramo a stikliramo  `Implicit flow`



## Keycloak Email Sender

Idemo u *Authentication* i tu imamo tab *Required actions*  i tu podesimo na *set as default action* `Verify Email`.

Ovo ce da omoguci slanje email-a kada se user register.

Sada moramo da namestimo email sender,tj taj servis.Idemo u *realm settings* pa u *email*.

Bitno je da namestimo host i port tj deo za *Connection & Authentication*

Namestili smo test mailing server koji se zove `MailHog`,instalirali smo ga kao docker container.

Kako su svi na istom networkku u dockeru host je ime tog container-a.MailHog ima 2 porta 1025 koji je za mail i 8025 koji je UI

Host: mailhog

Port: 1025

![image-20230609122039765](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230609122039765.png)

za Mailhog je ovo dovoljno,ako podesavamo google moramo da ukljucimo SSL i TLS i Authentication

## 2FA

Ovo mozemo da omogucimo i keycloak ima default procese koji radie dveofaktorsku

Authentication->Requeired Actions -> Configure OTP

ovo ce da konfigurese da imamo 2fa ali samo preko nekih provajdera,ako hocemo email base ili sms based moramo da ubacimo neke jar fajlove koji ce to raditi,tj moramo sami da napravimo taj rad i da ubacimo u keyloak

https://github.com/mesutpiskin/keycloak-2fa-email-authenticator -> Email Authentication

https://github.com/dasniko/keycloak-2fa-sms-authenticator -> SMS Authentication,moramo da podesimo da postoji polje u useru `mobile_number` mozemo to rucno ali je bolje da imamo custom akciju 

ljudi su vec ovo napravili pa mozemo da skinemo,ali i mi mozemo da napravimo nase pomocu samo 2 klase koje ce svaka da impl po jedan interface

1. <span style="color:tomato">Authenticator</span>
2. <span style="color:tomato">AuthenticatorFactory</span>

i tu podesimo sami ako hocemo



Kada napravimo app pomocu ovoga ili skinemo vec postojace samo treba da build i install u mavenu i onda dobijemo .jar file

moramo da kopiramo taj jar fajl  u specijalan folder u keycloaku `/opt/keycloak/providers/`

```
docker cp buildovanJarFile ContainerId:/opt/keycloak/providers/
```

Kada dodamo ovaj fajl moramo da ga ubacimo da se koristi pri `autentikaciji`.To mozemo da odradimo tako sto odemo u Authentication tab i kreiramo kopiju browser flow-a

![image-20230622100757853](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230622100757853.png)

i sada taj nas flow mozemo da kostumizujemo i da tu ubacimo dodati jar file da izvrsava posao

## Custom Action

Moramo da implementiramo kao gore neke klase:

- <span style="color:tomato">RequiredActionProvider</span>
- <span style="color:tomato">RequiredActionFactory</span>

i ovo ubacimo u istu potanju na keycloaku on ce sam da registruje

## Recovery Code

Ovo je ako se npr telefon izgubi pa ne moze 2fa da uradi,pa za to postoji na keycloaku recovery code

Postoje neke akcije u Keycloaku koje nisu enabled by default,pa moramo mi sami da ih enable.To sam uradio tako sto sam dodao cmd komandu prilikom podizanja keycloak servera u compose file

```
  command: start-dev --features="preview"
```

https://www.keycloak.org/server/features#_enabling_features vise o tome

i sada imamo u Authentication/Required Actions nove akcije 

Samo ga enable

FOra je sto moramo da ga dodamo u login browser flow da bi radilo pri loginu.Ako stavimo da je sve alternative onda  ce se prikazati na jedoj formi

![image-20230622133022612](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230622133022612.png)

## User Storage

Gde cemo cuvati usere,ovako se cuvaju interno.Ovo mozemo promeniti da se cuvaju u nasoj bazi

https://github.com/dasniko/keycloak-extensions-demo/tree/main skinemo ovo i build peanuts-userprovider i to oept iskopiramo u /opt/keycloak/providers
