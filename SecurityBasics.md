# SecurityBasics

## Encoding

Encoding je proces  pretvaranja ulaza u izlaz preko opstepoznatog formata.

To znaci da imamo neki format koji upotrebljavamo da transformisemo ulaz u izlaz.Taj format je dostupan svima i svi mogu da ga provale

### Base64

Base64 is a widely used encoding scheme for data. Here are some of the popular Base64 encoders:

1. MIME Base64: It is the most commonly used encoding scheme for email attachments and is defined in RFC 2045.
2. Base64url: It is a variant of Base64 encoding used in URL-safe contexts. It differs from MIME Base64 in that it uses different characters for the last two values of the encoding table.
3. Base32: It is another variant of Base64 encoding that uses a 32-character set instead of the 64 characters used in Base64.
4. Base85: It is similar to Base64 but uses a larger character set (85 characters) to represent data.
5. Ascii85: It is a variant of Base85 that uses ASCII characters for encoding.

Ovo su najpoznatiji:

```
Base64.getEncoder();
Base64.getMimeEncoder();
Base64.getUrlEncoder();
```

Proces transformisanja  inputa

```
Base64.getDecoder();
Base64.getMimeDecoder();
Base64.getUrlDecoder();
```

Svaki Encoder ima odgovarajuci obrnuti proces koji se zove Decoder

```
NekiEncoder.encode(bytes[])
NekiDecoder.decode(bytes[])
```

## Hasing

Ovo je proces pretvaranja inputa u output koji se ne moze vratiti u input,znaci irreversable je.

Tipovi:

1. MD-5(128 bits) hacked

2. SHA-1(160 bits)

3. SHA-256(256 bits) ovo je najpouzdanije

Klasa koja se koristi ovde je  <span style="color:red"> MessageDigest</span>

```java
MessageDigest.getInstance("type");

MessageDigestInstance.diagest(bytes[]);
```

## Encoding

Ovo je proces da mi stvaramo samostalne ili parove kljuceva koji ce da desifruju poruku,to je jedini nacin da se poruka desifruje

<span style="color:red"> KeyGenerator</span>

<span style="color:red"> Chiper</span>

### Single key

<span style="color:red"> SecretKey</span>

Ovo je kljuc koji ce se koristiti i za sifrovanje i desifrovanje poruke.

Postoje 2 tipa ovoga

1. DES -> hakovan

2. AES ->128 bits

Chiper sadrzi dva mode to su Ciper.ENCRYPT_MODE i ciper.DECRYPT_MODE zavisi ta radimo 

```java
//generate key 
KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
 keyGenerator.init(256);
 SecretKey secretKey = keyGenerator.generateKey();
//encode key
Ciper chiper=Chiper.getInstance("AES");
ciper.init(Ciper.Mode,secretKey);
byte[] encripted=chiper.doFinla(message_bytes);
```

### DoubleKey

Kreiramo 2 kljuca:

<span style="color:red">PublicKey</span>

<span style="color:red">PrivateKey</span>

<span style="color:red">KeyPairGenerator</span>

<span style="color:red">KeyPair</span>

```java
KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
keyGen.initialize(2048);
KeyPair keyPair = keyGen.generateKeyPair();

RSAPrivateKey aPrivate =(RSAPrivateKey) keyPair.getPrivate();
RSAPublicKey aPublic = (RSAPublicKey) keyPair.getPublic();
RSAKey finalKey= new RSAKey.Builder(aPublic).privateKey(aPrivate).keyID(UUID.randomUUID().toString()).build();
```



## SecureRandom

`SecureRandom` is a class in Java that provides a cryptographically strong random number generator. It is part of the Java Security API and is commonly used to generate random numbers for security-sensitive operations, such as generating secure keys or tokens.

`SecureRandom` is more secure than the commonly used `Random` class, as it generates numbers using a source of randomness that is provided by the underlying operating system. This makes it less predictable and more suitable for use in cryptography applications.

It is also possible to specify the algorithm used by `SecureRandom` by calling the `getInstance` method and passing in the name of the algorithm, for example:

```JAVA
SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
```

## MessageDigest

`MessageDigest` is a class in Java that provides a secure way to compute a hash value or message digest of data. It is part of the Java Security API and is commonly used to generate a fixed-size digital signature for a message or to check the integrity of data.

`MessageDigest` supports several popular hash algorithms, such as SHA-256, MD5, and SHA-1. To use `MessageDigest`, you create an instance of the class using one of the supported algorithms and then use its `update` and `digest` methods to compute the hash value. For example:

```
javaCopy codeMessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest("hello".getBytes());
```

`MessageDigest` is also useful for generating secure passwords, as it can be used to hash a password along with a salt value to make it more difficult to crack. The `update` method can be used to add data to the digest, and the `digest` method can be used to generate the final hash value.



## PKCE (Proof Key for Code Exchange)

 is a security extension to the authorization code flow in OAuth 2.0, used to secure authorization code grants for native apps and single-page applications (SPAs). It helps prevent authorization code interception attacks by verifying that the authorization code was bound to the client and the redirect URI used in the original authorization request. Preventing malicious code or attackers from exchanging the authorization code for tokens.

Kljucni pojmovi:

<span style="color:#ff4d4d">code_challenge</span>

<span style="color:#ff4d4d">code_challenge_method</span>

<span style="color:#ff4d4d">code_verifier</span>

To implement PKCE (Proof Key for Code Exchange) in Java, you can use a library like Spring Security OAuth or implement it manually using the following steps:

### Spring

Prvo moramo dodati biblioteku

Ovo nije klasicna stater biblioteka

```xml
<dependency>
   <groupId>org.springframework.security.oauth</groupId>
   <artifactId>spring-security-oauth2</artifactId>
   <version>2.4.0.RELEASE</version>
</dependency>
```

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

   @Autowired
   private AuthenticationManager authenticationManager;

   @Override
   public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
      endpoints.authenticationManager(authenticationManager).addInterceptor(new PkceChallengeInterceptor());
   }

   @Override
   public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
      clients.inMemory()
            .withClient("clientId")
            .secret("clientSecret")
            .authorizedGrantTypes("authorization_code", "refresh_token")
            .scopes("user_info")
            .autoApprove(true);
   }
}

```



### Custom

```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

public class Pkce {
  private String codeVerifier;
  private String codeChallenge;

  public Pkce() throws NoSuchAlgorithmException {
    codeVerifier = generateCodeVerifier();
    codeChallenge = generateCodeChallenge(codeVerifier);
  }

  private String generateCodeVerifier() {
    // Generate random string
    // ...
    return codeVerifier;
  }

    //Generate code challenge from verifier
  private String generateCodeChallenge(String codeVerifier)
      throws NoSuchAlgorithmException {
    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    byte[] hash = digest.digest(codeVerifier.getBytes());
    return Base64.getUrlEncoder().withoutPadding().encodeToString(hash);
  }

  public String getCodeVerifier() {
    return codeVerifier;
  }

  public String getCodeChallenge() {
    return codeChallenge;
  }
}

```

1. Generate a **code verifier** and **challenge**: On the client side, generate a cryptographically random string (code verifier) and use it to create a hash value (code challenge).
2. Request authorization: Send a request to the authorization server with the **code challenge**.
3. Validate the **challenge**: On the authorization server, validate the **code challenge** provided by the client.
4. Send the authorization code: The authorization server returns an authorization code after successful validation.
5. Exchange code for tokens: The client sends the authorization code and **code verifier** to the token endpoint to exchange them for tokens.
6. Validate the code verifier: On the token endpoint, validate the **code verifier** by re-computing the **code challenge** from the **code verifier** and comparing it with the challenge sent in the authorization request.



Fora je da sa gateway-ja stize PKCE,znaci svaki zahtev na auth server cemo presretati sa filterom i ubaciti PCKE,bitno nam je da imamo generator za PCKE u API Gateway-u



PKCE support lives in the *spring-security-oauth2-client* module. For a Spring Boot application, the easiest way to bring this dependency is using the corresponding starter module:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
    <version>2.7.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
    <version>2.7.2</version>
</dependency>
```

```java
@Bean
public SecurityWebFilterChain pkceFilterChain(ServerHttpSecurity http,
  ServerOAuth2AuthorizationRequestResolver resolver) {
    http.authorizeExchange(r -> r.anyExchange().authenticated());
    http.oauth2Login(auth -> auth.authorizationRequestResolver(resolver));
    return http.build();
}

@Bean
public ServerOAuth2AuthorizationRequestResolver pkceResolver(ReactiveClientRegistrationRepository repo) {
    var resolver = new DefaultServerOAuth2AuthorizationRequestResolver(repo);
    resolver.setAuthorizationRequestCustomizer(OAuth2AuthorizationRequestCustomizers.withPkce());
    return resolver;
}
```





### Flow

Client je ovde Api Gateway ili neko ko direktno komunicira sa Authorization Serverom.

1. The client creates and records a secret named the "code_verifier" and derives a transformed version "t(code_verifier)"  (referred toas the "code_challenge"), which is sent in the OAuth 2.  Authorization Request along with the transformation method "t_m".The Authorization Endpoint responds as usual but recordst(code_verifier)" and the transformation method.

2. The client then sends the authorization code in the Access Token
         Request as usual but includes the "code_verifier" secret generated
         at (A).

3. The authorization server transforms "code_verifier" and compares
         it to "t(code_verifier)" from (2).  Access is denied if they are
         not equal.The client then creates a code challenge derived from the code verifier by using    one of the following transformations on the code verifier:

4. When the server issues the authorization code in the authorization
      response, it MUST associate the "code_challenge" and
      "code_challenge_method" values with the authorization code so it can
      be verified later.Typically, the "code_challenge" and "code_challenge_method" values
      are stored in encrypted form in the "code" itself but could
      alternatively be stored on the server associated with the code.  The
      server MUST NOT include the "code_challenge" value in client requests
      in a form that other entities can extract.



5. Upon receipt of the Authorization Code, the client sends the Access
      Token Request to the token endpoint.  



6. Upon receipt of the request at the token endpoint, the server
      verifies it by calculating the code challenge from the received
      "code_verifier" and comparing it with the previously associated
      "code_challenge", after first transforming it according to the
      "code_challenge_method" method specified by the client.



The client first creates a code verifier, "code_verifier", for each
   OAuth 2.0 [RFC6749] Authorization Request, in the following manner:

   code_verifier = high-entropy cryptographic random STRING using the
   unreserved characters

   The client then creates a code challenge derived from the code
   verifier by using one of the following transformations on the code
   verifier:













 In addition to the terms defined in OAuth 2.0 [RFC6749], this
   specification defines the following terms:

   code verifier
      A cryptographically random string that is used to correlate the
      authorization request to the token request.

   code challenge
      A challenge derived from the code verifier that is sent in the
      authorization request, to be verified against later.

   code challenge method
      A method that was used to derive code challenge.





Mozemo da idemo u tools->JShellConsole i tu da pisemo console java code koji mozemo da run 

code challenge:

```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;
    MessageDigest messageDigest; {
    try {
        messageDigest = MessageDigest.getInstance("SHA-256");
       byte[] arr= messageDigest.digest("MUQt-CeLhkHyFpVDF9oFbtIFhvQ7fLhr7gXDgm-t68M".getBytes());
        String codeChallenge= Base64.getEncoder().withoutPadding().encodeToString(arr);
        System.out.println(codeChallenge);
    } catch (NoSuchAlgorithmException e) {
        throw new RuntimeException(e);
    }
}
```



code verifier:

```java
import java.security.SecureRandom;
import java.util.Base64;

    SecureRandom secureRandom=new SecureRandom();
byte[] arr=new byte[32];
secureRandom.nextBytes(arr);

String codeVerified= Base64.getUrlEncoder().withoutPadding().encodeToString(arr);
System.out.append(codeVerified);
```





## CSFR

CSFR(Cross site request forgery) je mehanizam da neko drugi 3rd party gadja server preko naseg racunara,slicno kao spoofing.

Primer:

Mi smo na poslu i ulogovani smo u aplikaciju preko koje dodajemo brisemo i menjamo fajlove.Stize nam email i mi kliknemo na link i to nas odvede na neku random stranicu,kada se vratimo u app vidimo da su svi fajlovi pobrisani.

Desilo se to da smo mi ulogovani u aplikaciju i neko je izvrsio request umesto nas .To sprecava CSFR protection.

CSFR napad pocinje sa pretpostavkom da je target vec ulogovan u sistem im izvrsimo komande umesto njega



![image-20230213100223601](C:\Program Files\Typora\resources\Docs\img\image-20230213100223601.png)



Kako CSFR funkcionise?

Front mora da posalje get da dobije stranicu,kada se to desi back salje token.Sad svaki request koji dolazi ce sdarzati taj token i tako ce back znati. 

<span style="color:#ff4d4d">CSRF Token</span>

U Spring Security imamo filter <span style="color:#ff4d4d">CsrfFilter</span> koji proverava token,on proverava samo metode HTTP zahteva koje menjaju dokumente





![image-20230213100739744](C:\Program Files\Typora\resources\Docs\img\image-20230213100739744.png)

CsfrFilter koristi komponentu <span style="color:#ff4d4d">CsfrTokenRepository</span> po defaultu ovo ima CRUD opercaije nad tokenom,i svi tokeni se ubacuju u HTTP session,tako da request koji se salje mora da sadrzi sesiju i csfr token

Klasa koje je reprezentacija token je <span style="color:#ff4d4d">CsrfToken</span> i nju uzimamo preko atributa <span style="color:#ff4d4d">_csrf</span>

```java
Object o = request.getAttribute("_csrf"); 
 CsrfToken token = (CsrfToken) o;
```



Mnogo je lakse uraditi CSFR tokene kada imamo 1 frejmvork koji kontrolise sve i front i back,ali ako imamo da se razvijaju odvojeno:

Kada razvijamo app koja ima poseban front moramo da customize malo rad:

1. Configuring paths for which CSRF applies
2. Managing CSRF tokens

1.Po defaultu CSRF je primenjen na svim endpointovima koji rade post,patch,put.

Mi znamo da mozemo da iskljucimo ceo csfr zastitu,ali ovako zelimo da iskljucimo samo na odredjenim endpointima



Jedna ruta ce morati da ima c![image-20230213103819367](C:\Program Files\Typora\resources\Docs\img\image-20230213103819367.png)srf protekciju a druga ne.

```java
http.csfr(
	customizer->customizer.requireCsrfProtectionMatcher(ServerWebExchangeMatcher)
);
```

Ovo <span style="color:#ff4d4d">ServerWebExchangeMatcher</span> je funkcionalni interfjjs pa mozemo x->x.

2.Mi znamo da po defaultu csrf token se cuva u sesiji,ovo je ok za male aplikacije,ali za velike i koje hoce da skaliraju nije.

Moramo podesiti nekako da se ne cuva u sesijji:

1. CsrfToken
2. CsrfTokenRepository



1.

```java
public interface CsrfToken extends Serializable {
 String getHeaderName();
 String getParameterName();
 String getToken();
}
```

Moramo da definisemo nas deafult,vidimo da ima getHeaderName() po defaultu je ovo _csrf,ovo je implemntacija defaulta tj <span style="color:#ff4d4d">DefaultCsrfToken</span>

2.Ovo je kljucno jer ovde je logika kako ce se manage sami csrf tokeni,da li ce se cuvati u sesiji ili ne.Mi za to moramo da implementiramo nasu CsrfTokenRepository



![image-20230213105403821](C:\Program Files\Typora\resources\Docs\img\image-20230213105403821.png)

```java
public class CustomCsrfTokenRepository implements CsrfTokenRepository {
    
     @Autowired
     private JpaTokenRepository jpaTokenRepository;
    
     @Override
     public CsrfToken generateToken(HttpServletRequest httpServletRequest) {
   		String uuid = UUID.randomUUID().toString();
 		return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", uuid)
     }
    

     @Override
     public void saveToken(CsrfToken csrfToken,  HttpServletRequest httpServletRequest,  HttpServletResponse httpServletResponse) {
         String identifier = httpServletRequest.getHeader("X-IDENTIFIER");
     Optional<Token> existingToken = jpaTokenRepository.findTokenByIdentifier(identifier);
     if (existingToken.isPresent()) { 
         //update ga ako postoji
             Token token = existingToken.get();
             token.setToken(csrfToken.getToken());
             } 
         else { 
             //napravi novi
             Token token = new Token();
             token.setToken(csrfToken.getToken());
             token.setIdentifier(identifier);
             jpaTokenRepository.save(token);
    	 }
     }
     @Override
     public CsrfToken loadToken(HttpServletRequest httpServletRequest) {
        String identifier = httpServletRequest.getHeader("X-IDENTIFIER");
         Optional<Token> existingToken = jpaTokenRepository.findTokenByIdentifier(identifier);
         if (existingToken.isPresent()) {
             Token token = existingToken.get();
             return new DefaultCsrfToken(
             "X-CSRF-TOKEN", 
             "_csrf", 
             token.getToken());
          }

         return null;
}
     }
}
```

Kreiranje token je prepusteno defaultu DefaulCsrfToken() i to cemo stavl

saveToken pre po defaultu je uzimao sessionID ali to nemamo vise vec neki jedinstaveni id 



![image-20230213110915171](C:\Program Files\Typora\resources\Docs\img\image-20230213110915171.png)

Spring vec ima definisan filter za proveru da li postoji CSRF to je CsrfFitler,a mi mozemo da napraivmo sami samo moramo da nasledimo OncePerRequestFilter,to ovaj CsrfFIlter i radi.Proveravali bi samo da li posotji i validnost,izvlacenje i cuvanje je u repository delu

## CORS

CORS(Cross origin request  sharing) je mehanizam za deljenje resursa preko public ip-ja

Svaki browser ima svoj CORS policy sto mu omogucava da lokalno moze da fetch resurse,dok resursi koji dolaze van lokala moraju da imaju nesto u sebi kako bi CORS policy to dozvolio

3rd party sharing

To je kada nam npr treba slika sa neko servera daleko pa moramo da posaljemo zahtev,ako server ne podrzava CORS necemo moci da dobijemo sliku.



Kada posaljemo zahtev u requestu se doda origin header koji nam govori  o toj policy,ako se taj  request salje na server kji ima isti origin onda ce da prodje super,ju suprotnom nece dati zato se zove cross origin jer prelazi vise origin-a.

To resavamo dodavanjem header-a koji se po standardu pisu isto,i opstepoznati:

<span style="color:#ff4d4d">Access-Control-Allow-Origin</span> -> koji domen je dozvoljen da salje zahtev

<span style="color:#ff4d4d">Access-Control-Allow-Methods</span> -> koje metode atj domen moze da vrsi(samo get npr...)

<span style="color:#ff4d4d">Access-Control-Allow-Headers</span> -> koje hedere u requestu moze da posalje

 <span style="color:#ff4d4d">Access-Control-Max_Age</span>  -> koliko traje allow origin



![image-20230213113551787](C:\Program Files\Typora\resources\Docs\img\image-20230213113551787.png)

To se resava tako sto server mora da dodja Access-control-Allow-Origin: ime origna,kako bi politika to dozvolila

Ovo je problem na serveru koji mora da doda ovaj header.

Ovo funkcionise tako sto neki requestovi kao PUT POST moraju da budu `preflight`

browser automatski zna kada da preflight request i to salje OPTIONS da vidi da li server dozvoljava prenos tih resursa ,i kao rezultat server salje allow origin koji se salje .

On salje zahtev ali mi ne mozemo da procitamo response?

Takodje server moze da salje i Access-Control-Max_Age koji cache origin



![image-20230213114439384](C:\Program Files\Typora\resources\Docs\img\image-20230213114439384.png)

1)

@CrossOrigin anotacija na controller da ovo ukljucimo u spring-u.

```java
@CrossOrigin(origins = "http://external-system.com", maxAge = 3600,methods=RequestMethod[],allowedHeaders=String[])
```

Mi za origins i za allowedHeaders mozemo da koristimo * sto ce ukljuciti i dozvoliti svima.Ovo moze biti problematicno jer moze izazvati XSS(cross site scripting ) i DDos napade.

Ovo je i method i class level anotation

2) Mozemo da radimo sa CorsConfigurer.

    <span style="color:#ff4d4d">CorsConfigurationSource</span>

    <span style="color:#ff4d4d">CorsConfiguration</span>

   

   ```
   http.cors(c->{
   CorsConfigurationSource source=request->{
   	CorsConfiguration config=new CorsConfiguration():
   	config.setAllowedOrigins(List.of(...));
   	config.setAllowedMethod(List.of(...));
   	return config;
   	};
   	c.configurationSource(source);
   });
   ```

   







- `@RequestParam` ← `application/x-www-form-urlencoded`,
- `@RequestBody` ← `application/json`,
- `@RequestPart` ← `multipart/form-data`,

The main difference is that when the method argument is not a String or raw `MultipartFile` / `Part`, `@RequestParam` relies on type conversion via a registered [`Converter`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/converter/Converter.html) or [`PropertyEditor`](https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/java/beans/PropertyEditor.html) while [`RequestPart`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestPart.html) relies on [`HttpMessageConverters`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html) taking into consideration the 'Content-Type' header of the request part. [`RequestParam`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html) is likely to be used with name-value form fields while [`RequestPart`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestPart.html) is likely to be used with parts containing more complex content e.g. JSON, XML

@RestController vs @Repository ili neki drugo

RestController sadrzi anotacije @Controller i @ResponseBody jer mi kada radimo sa api moramo da vratimo response koji nije stranice ,a ako oznacimo sa @Controller bez @ResponseBody  ce u resource da gelda stranice sa imenom string koje mi vracamo da li postoje ,zato je uz controller bitno da se doda @ResponseBody da bi mu dali do znanja da vraca poruku a ne da trazi stranicu.

RestController->Controller + ResponseBody



@RequestHeader i mozemo da izvucemo header



PATCH -> Partial update

PUT->Update whole resource
