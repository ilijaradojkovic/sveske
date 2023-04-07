

# Auth



Osnovni pojmovi:

1. <span style="color:#ff4d4d">Authentication</span>
2. <span style="color:#ff4d4d">Authorization</span>
3. <span style="color:#ff4d4d">Prinipal</span>
4. <span style="color:#ff4d4d">Granted Authority/Role</span>

## Authentication 

Postavlja pitanje **who are you?**

Utvrdjuje ko smo mi,mi saljemo kredencijale da bi server mogao da potvrdi nas identitet.

## Authorization

Postavlja se pitanje **can this user do this?**

Nemaju svi useri isti pristum resursima ili rutama.Ovo je role base sistem i mi moramo proveriti ko ima koju ulogu.

Pre autorizacije ide autentikacija.

## Tokens

### Opaque 

Ne sadrzi informacije u sebi

### Non-Opaque 

Sadrzi informacije u sebi

JWT

## Encoding

Pretvaranje String-a uz secret nekad ali ne mora da znaci,sluzi da mi predstavljamo neki fromat ovo nije za security nista spec,ali je format koji se postuje tipa Uri encoding gde mi specijalne karakere predstavljamo u jedan format.Znaci format je bitan i lako se decode

## Encrypting

Ovo uz pomoc secret ali ne mora pretvaramo podatke u nesto sto je zasticeno i sto ne mogu sve da dekriptuju.Niko ne moze da cita bez odredjenog dekripcionog kljuca

In summary, encoding is used to represent data in a specific format, while encryption is used to protect data from unauthorized access.

## Hashing

Slizi da napravimo irreversable kod .Kada jednom napravimo hash kod necemo moci da vratimo u prvobitno stanje.Ulazak u hash funkciju zovemo message a sta izlazi iz hash funkcije zovemo diagest.Generise se fixed length

## 





## Principal

Ovo je autentikovani user zovse se principal,mi u spring-u mozemo da uzmemo informacije o njemu,ako je proces uspesno prosao samo mozemo inject varijablu i metodi i spring ce nam provid ovo

# Spring Security

Da bi omogucili spring security  moramo dodati dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Kada dodamo ovu lib spring ce automatski dodati security i napravili default korisnika sa sifrom kako bi se login,sve rute ce biti zasticene.

Cela poenta ovoga je da mi konfiguresemo security kako nama odgovara.

Spring presrece svaki nas request i nesto radi,to presretanje radi<span style="color:#ff4d4d"> Filter</span> koji je deo Spring AOP,on ce da presretne svaki nas reqeust i videti da li smo auth itd.

Po dodavanju biblioteke Spring dodaje neke default Filter-e,mozemo to videti ako ukljucimo

![Desktop - 13](C:\Users\radoj\Downloads\Desktop - 13.png)



Svaki request koji nam od sada dolazi prolazi kroz seriju filtera koji su defaultno namesteni zbog security biblioteke,mi mozemo da definisemo i nase filtere.

![Desktop - 14](C:\Users\radoj\Downloads\Desktop - 14.png)

Svaki filter ce da sadrzi request i njega ce da obradjuje,ovo koristi chain of responsibility pattern,ako jedan punke pucaju svi.

Kako se doda default user mi njega mozemo da promeni u **application.properties**

```properties
spring.security.user.name=foo
spring.secturiy.user.password=foo
```



![image-20230201123458402](C:\Program Files\Typora\resources\Docs\img\image-20230201123458402.png)

## Konfiguracija Security-ja

Da bi manipulisali security-om mi moramo da definisemo bean metode koje ce da se registruju sa nasom config .

```java
    @Bean
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        
    }

    @Bean
    protected void configure(HttpSecurity http) throws Exception {
       
    }

    @Bean
    public void configure(WebSecurity web) throws Exception {
       
    }
```

### Configure Authentication

​	Ovo radimo u metodi **configure(AuthenticationManagerBuilder authd)**

Glavna klasa za ovo je <span style="color:#F24D37">AuthenticationManager</span> koja upravlja proces autentikacije,radi autentikaciju preko metode authenticate().Nas posao je samo da ovo 	konfiguresemo,ne kreiramo mi ovo jer spring vec provajduje intancu AuthenticationManager-a,mi samo konfiuresemo ovo preko <span style="color:#F24D37">AuthenticationManagerBuilder</span> klase.

​	Korisicemo ovu klasi koja primenjuje builder pattern.Bas u ovoj metodi configure  nama spring vec provajduje taj bojekat na nama je samo da ga config.

### How Does It Work?

![Desktop - 15](C:\Users\radoj\Downloads\Desktop - 15.png)

Glavi pojmovi:

- ​	<span style="color:#F24D37">AuthenticationManager</span>
- ​	<span style="color:#F24D37">ProvideManager</span>
- ​	<span style="color:#F24D37">AuthenticationProvider</span>
- ​	<span style="color:#F24D37">UserDetailsService</span>

Filter ne zna kako da radi authentication to radi **AuthenticationManager** koji delegira rad provideru koji smo namestili,jer imamo vise providera za auth(JDBC,InMemory,LDAP...) i **ProviderManager** sadrzi taj konkretni provider koji smo mi registrovali i on sadrzi implementaciju **UserDetailsServic-a** taj service radi samo authentication kako mi impl,fora je da mora da postoji registrovani provider,ako ga ne nadje baca excetion

Ako je authentication uspesan vraca **Principle** ili **Authentication** objekat,ovo su slicni objekti i spring ih provajduje u kontroleru ako kazemo da ga hocemo kao parametar.Principle sadrzi vise detalja.

![Desktop - 16](C:\Users\radoj\Downloads\Desktop - 16.png)

Ako je auth succes vratice se Authentication objekat koji sadrzi prinipal i on cce se pamtiti u **ThreadLocal**



![Desktop - 17](C:\Users\radoj\Downloads\Desktop - 17.png)

Postoje vise vrsta autentikacije.

#### 		1) InMemoryAuth

​	

```java
		auth.inMemoryAuthentication()
					.withUser("user")
					.password("...")
				     .roles("..")
					.authorities("..")
					.and()
					.withUser(...)
```

<span style="color:#F24D37">inMemoryAuthentication()</span> -> daje springu do znanja da radi sa in memory korisnicima

<span style="color:#F24D37">withUser("userneki")</span>

<span style="color:#F24D37">password("...")</span>

<span style="color:#F24D37">roles("...")</span>

<span style="color:#F24D37">authorities("...")</span>

### 	2)JDBC Authentication

Ovo nije JPA,vec jdbc konektor ,jpa koristi ovo u pozadini ali dodaje nove stvari,ovo je raw sql koji koristi jdbc.

```java
auth.jdbcAuthentication()
    .datasouce(datasource)
    .withDefaultSchema()
    .withUser("...")
    .password("..")
    .roles("...")
```

Kako radimo sa jdbc konektorom mi moramo reci gde je nasa baza da bi se auth vrsio nad njom.To je<span style="color:#F24D37"> DataSource</span>

Mi ako konfiguresemo bazu u application.properties moci cemo da uzmemo DataSouce  preko DI jer spring provajduje.Mi cemo time reci Spring-u da gelda tu bazu kada radi auth,nesto u njoj neku tableu.

Ako kazemo da hocemo **withDefaultSchema()** on ce sam da napravi User i Authorities tabelu,i mi necemo moci da kontrolisemo sta ide u nju jer ce po defaultu on da stavi neophodne stvari

Ako ne zelimo default napravimo mi nasu

<span style="color:#F24D37">* </span>  To mozemo uraditi tako sto definisemo 2 fajla u **resources**

- <span style="color:#F24D37">schema.sql</span>
- <span style="color:#F24D37">data.sql</span>

i ovime ce on da overide ,ovo su fajlovi koji se pokrecu odmah sa app,ne smeju biti prazni.Uz ovo namestimo u propoerties

```properties
 spring.jpa.hibernate.ddl-auto=create
 spring.sql.init.mode=always
 spring.jpa.properties.hibernate.dialect=MYSql5Dialect
```

<span style="color:#F24D37">* </span> Ne zelimo da kucamo sql u springu,vec da definisemo tabele u nasoj bazi kako mi hocemo.

```java
auth.
.
.
.
.usersByUsername("Query to get username from our table in db")
.authoritiesByUsernameQuery("Quewry to get username,authorities from our table in db")
```

Mi mu ovde dajemo do znanja kako da izvrsi query nad tabelama i da uradi authentication





### 	3)JPA 

Glavni koncepti:

- <span style="color:#F24D37">UserDetailsService</span>
- <span style="color:#F24D37">LoadByUsername(String username)</span>
- <span style="color:#F24D37">UserDetails</span>
- <span style="color:#F24D37">GrantedAutherity</span>

Ovo je moderniji pristup gde koristimo prednosti koje JPA nudi,on u pozadini koristi JDBC ,ali ima mnogo naprednije funkcije.

```java
auth.userDetailsService(ourDetailsService)
```

Glavna stavar je ovde da definisemo <span style="color:#F24D37">UserDetailsService</span>,to je interfejs koji nas servis mora da implementira gde ce definisati metodu 

<span style="color:#F24D37">LoadByUsername(String username)</span>

i mi cemo tu uraditi query nad bazom kao znamo i vratiti objekat <span style="color:#F24D37">UserDetails</span>.Mi cemo vec u ovoj metodi dobiti username,samo treba da uradimo query nad bazom i vratimo objekat UserDetails.

Imamo 2 opcije:

1. Vratimo predefinisanu Spring klasu UserDetails (new UserDetails(username,password,Collection<GrantedAutherity>))
2. Nasa klasa User nasledi ovu klasu UserDetails pa cemo vratiti nasu definisanu klasu User (new MyUser(...))

Pravljenje klase UserDetails uz pomoc default builder-a koji spring vec provide:

```java
User.withUsername("bill")
 .password("12345")
 .authorities("read", "write")
 .accountExpired(false)
 .disabled(true)
 .build()
```

Neka praksa je da mi napravimo naseg user-a entity u bazi koji ima samo polja za bazu,i onda da njega wrap-ujemo sa SecurityUser koji ce da impl UserDetails da imamo jasnu razliku(separaciju koncepta),mogli smo i u User klasi da imlpementujemo UserDetails,ali ta separacija je bitna.

```java
public class SecurityUser implements UserDetails {
 
 private final User user;
 
 public SecurityUser(User user) {
 this.user = user;
 }
 @Override
 public String getUsername() {
 return user.getUsername();
 }
 @Override
 public String getPassword() {
 return user.getPassword();
 }
 @Override
 public Collection<? extends GrantedAuthority> getAuthorities() {
 return List.of(() -> user.getAuthority());
 }
 // Omitted code
}
```

User-> nas entity

<span style="color:#F24D37">GrantedAuthority</span> ovo je kao role koji spring secutiry ocekuje da provajduje zajedno sa username i password,ovo je abstraktna klasa,moramo vratiti konkretnu implementaciju u zavisnosti koji authentication provider imamo

<span style="color:#F24D37">SimpleGrantedAuthority</span> -> ovo je implementacija za JPA

<span style="color:#F24D37">LdapAuthority</span> -> ovo je implementacija za Ldap

### 	4)Ldap

```
auth.ldapAuthentication()
	.userDnPattern("cn={0}.ou=muUsers") -> po ovome poredi id name email
	.groupSearchBase("ou=groups") 
	.contextSource()
	.url(Url ldap servera)
	.passwordComaprer()
	.passwordEncoder(Encoer)
	.passwordAttribute("userPassowrd")-> ime polja gede se cuva password
```



### 5)Custom

Pored toga sto imamo da definisemo custom auth imamo i da definisemo custom AuthenticationProvider

```java
auth.authenticationProvider(NAS)
```

Postoje predefinisani provajderi 

Mi nas provajder definisemo tako sto nasledimo interfejs <span style="color:#F24D37">AuthenticationProvider</span>

```java
public class Myprovider implements AuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        //if authenticaiton fails return AuthenticationException
        //if authentication provider not supported return null
        //if all good return Authentication boject
        return null;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        //if auhtentiation provider is supported
        return false;
    }
}
```

vidimo da moramo da vratimo objekat tipa Authentication:

Dva najvaznije Authentication implementacije koje cemo vratiti u ovoj metode authenticate su :\

<span style="color:#F24D37">UsernamePasswordAuthenticationToken</span>

<span style="color:#F24D37">AnnonymousAuthenticationToken</span> -> vrsta authentication objekta koji ne odaje informacije o useru



## Spring Security Context

Ovde se storuju in memory authentication objekti koji us uspesno prosli kod svog provajdera

SecurityContext dobijamo iz klase SecurityContextHolder preko:

```java
SecurityContext s=SecurityContextHolder.getContext();
```

Postoje nekoliko strategija kojima se sluzi SecurityContext

<span style="color:#F24D37">MODE_THREADLOCAL </span>-> Svaka thread cuva detalje o security contextu.Thread per request i svaka ce  imati svoj thread i pristup securityContextu

<span style="color:#F24D37">MODE_INHERITABLETHREADLOCAL</span>->Ovo je slicno kao MODE_THREADLOCAL ali  kopira kontext trenutne thread u novu asinhronu.Ovo je kada stavljamo @Async pa se izdvoji novi thread.

```java
@GetMapping("/bye")
@Async 
public void goodbye() {
 SecurityContext context = SecurityContextHolder.getContext();
 String username = context.getAuthentication().getName();
 //
}
```

Kada imamo Async on ce izdvoiji poseban thread i u taj thred nece biti dostipan po deafult strategiji security context.Zato stavljamo novi mode na interitable thread local da bi nasledio context od prethodne thread koja je pozvala ovu async

spring.security.strategy ovako mozemo da stavimo u yml fajlu

<span style="color:#F24D37">MODE_GLOBAL</span>->

5.1)DaoAuthenticationProvider

Ovo je provider kojie radi na principu UserDetailsService i to trazi za rad.

```java
@Bean 
DaoAuthenticationProvider getProvider(){
	DaoAuthenticationProvider provider=new DaoAuthenticationProvider();
	d.setPasswordEncoder(...);
	d.setUserDetailsService(...);
}
```



### Configure Route Secutiry

Ovo radi metoda

```java
 @Bean
    protected void configure(HttpSecurity http) throws Exception {}
       
```

Ovo radi klasa HttpSecutiry,mi ovde dobijemo gotov objekat koji config.

SessionManagement nam sluzi da kazemo springu da ne kreira sesije jer je to default ponasanje

``` java
http.authorizeRequests()        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
	.authorizeHttpRequets() ->radi isto sto i gore
	.antMatchers("pattern") ->match rutu koju hocemo da zastitimo
	.hasRole("role")
	.hasAnyRole("...","...")
	.and()
    .access(expression)
	.formLogin() -> da imamo login formu,ovo se koristi kod Spring MVC
	.permitAll() -> svi mogu da koriste tu rutu
	.fullyAuthenticated() -> mora da se auth da bi pristupili ruti
	.anyRequest()
	.logout
	.successHandler(AuthentiationSuccessHandler)
	failureHandler(AuthenticationFailureHandler)
	.sessionManagment().sessionCreationPolicy(SessionCreationPolicy.STATELESS)-> kada config jwt
    .addFilter(Filter)
    .addFilterBefore(Filter,BeforeWhatFIlter.class)
    .addFilterAfter(Filter,AfterWhatFilter.class)
    
```

npr:

```java
http.authorizeRequests().antMatchers("/properties/add").permitAll() -> bilo ko ce moci da pristupi ovoj ruti
http.authorizeRequests().antMatchers("/properties/add").fullyAuthenticated() -> samo auth users ce moci
http.authorizeRequests().antMatchers("/properties/add").hasRole("Admin") -> samo ce admin moci da priostupi ovoj ruti
```

### Configure http basic

```
http.httpBasic()
```

## AuthenticationEntryPoint 

Ovo je klasa koja je odgovorna za handlovanje procesa autentikacije .Ona je odgovorna ako dolazi greska 401 ili redirektovanje usera na login page.

Ovo se poziva samo kada user pokusa da pristupii zasticenom resursu gde nema pristup pa se baca 401 pa sta da radi u tom slucaju app zato deifinsemo custom

```java
public interface AuthenticationEntryPoint {
    void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException;
}
```

Vidimo da ima metodu gdem ozemo da menjamo request i response objekata,mi ovde mozemo i redirect da radimo

Postoje vec neke implementacije ovoga:

LoginUrlAuthenticationEntryPoint



## Kako pristupiti autentikovanom user-u

Postoje 2 objekata koje mozemo da **inject** u endpoint na kontroleru

- <span style="color:#F24D37">Principle</span>
- <span style="color:#F24D37">Authentication</span>
- <span style="color:#F24D37">@AuthenticationPrincipal</span>

```java
@GetMapping
public Product getProdut(Principal principal){
    return new Product(1,"test");
}
@GetMapping
public Product getProdut(Authentication authentication){
    return new Product(1,"test");
}
```

Authentication je opsirniji od principal klase

Authentication  je jedan od esencijalnih interfejsa koji ucestuje u procesu spring securitija.Authentication obj je  obimniji od Principal objekta i pruza nam vise informacija.Authentication i Principal u univerzalni objekti u SpringSecuritiju.

Authentication obj sadrzi Principal.

Ovaj objekat vadimo iz Security Context-a.

Mozemo taokdje da prepustimo springu da ga automatski izvadi ako smo authentikovani



Principle jeste Details nas pa ga je potrebno cast



Takodje mozemo anotiramo objekat @AuthenticationPrincipal,ova anotacija prima Authentication obj i automatski uradi aujthentication.getPrincipal i ga kastuje u objekat koji zelimo,ovo je kao brzi proces da ne bi imali dodatan korak oko kastovanja Principal nego ovo odmah to odradi,ali svede se na isto

```java
@GetMapping
@CustomAnotation()
public Product getProdut(@AuthenticationPrincipal UserDetails principal){
    return new Product(1,"test");
}
```

## Password Encoders

Ovo je jako bitno da imamo jer nam security nece raditi bez toga.Mi ovime kazemo spring-u kako da encode ili decode password

Postoje neke default encoderi

<span style="color:#F24D37">NoOPasswordEncoder</span> -> bez encoder-a,bitno je samo da se ovo koristi u test svrhe

```java
        NoOpPasswordEncoder.getInstance()

```

<span style="color:#F24D37">BCryptPasswordEncoder</span>

```java
        BCryptPasswordEncoder p=new BCryptPasswordEncoder();

```

<span style="color:#F24D37">Custom PasswordEncoder</span> tako sto implementiramo interfejs PasswordEncoder



<span style="color:#F24D37">SCryptPasswordEncoder</span>

## Key Generators

Spring takodje provajduje klasu <span style="color:#F24D37">KeyGenerators</span> preko koje mozemo dobiti instance dve bitne klase koje generisu kljuceve:

<span style="color:#F24D37">StringKeyGenerator</span>

```java
KeyGenerators.string()
```

<span style="color:#F24D37">BytesKeyGenerator</span>

```java
KeyGenerators.secureRandom()
```

Pozivamo im jednu metodu a to je *generateKey()*

## Encriptors

Ovo su objekti koje opet spring security provajduje koji rade proces enkripcije to radi sve <span style="color:#F24D37">Encryptors</span> i preko toga mi imamo staticke instance posebnih enkriptora.

<span style="color:#F24D37">TextEncryptor</span>

<span style="color:#F24D37">BytesEncryptor</span>



## Filters

Spring infrastruktura pravi nekoliko filtera po defaultu kada dodamo spring security biblioteku.

Svaki filter je **spring bean**.Kada se filter definise kao bean njega pokupi <span style="color:#F24D37">DelegationFilterProxy</span> koji upravlja filterima,ako ga on ne pokupi tj ako se ne registruje (automatski se to radi) kod njega onda se nece primeniti.

Filter pravimo pomocu interfejsa <span style="color:#F24D37">Filter</span>.Bitni koncepti:

- <span style="color:#F24D37">Filter</span>
- <span style="color:#F24D37">ServletRequest</span>
- <span style="color:#F24D37">ServeltResponse</span>
- <span style="color:#F24D37">FilterChain</span>

```
@Component
@Order(broj)
public class TestFilter implements Filter{
	init(FilterConfig)
	doFilter(ServletRequest,ServletResponse,FilterChain)
}
```

Kada ga registrujemo kao bean on ce imati najmanji prioritet,automatski ce ga pokupiti DelegationFilterProxy.

**ServletRequest** -> je klasa iz koje mi izvlacimo podatke o requestu koji je stigao

**ServletResponse** -> je klasa iz koje mi izvlacimo podatke o responsu koji mi vracamo

**FilterChain** -> je klasa uz pomock oje kazemo da li se nastavlja dalje u prolasku kroz filtere kroz lanac,ili puca sve

filterchain.doFilter(request,response) -> pomocu ove metode nastavljamo dalje da obradjujemo rquest u filterchain-u.

Mi kada filter definisemo kao bean on ce biti dostupan uvek i uvek ce se primenjivati,DelegationFilgerProxy ce ga uvek ukljuciti,ovo ponasanje mozemo da promenimo



```java
@Bean
public FilterRegistrationBean<NasFilter> setFilterBean(){
    FilterRegistrationBean<NasFilter> filerConfig=new FilterRegistrationBean();
    filterConfig.setOrder(int)
                .setFilter(new NasFilter())
                .addUrlPattern("...")
                .setUrlPattern("...")
                .setName("..")
                .addinitParameter(String ,String)
    return filterConfig;
}
```

### ServletRequest

Ovo je interfejst koji sadrzi informacije o request-u.

![Desktop - 18](C:\Users\radoj\Downloads\Desktop - 18.png)

Mi mozemo da konvertujemo ServeltRequest u HttpServletRequest  koji sadrzi vise info.

```java
HttpServletRequest http=(HttpServletRequest) serveltRequest;
```



### ServeltResponse

![Desktop - 19](C:\Users\radoj\Downloads\Desktop - 19.png)

Ista prica

#### UsernamePasswordAuthenticationFilter

Autentifikaciju radi <span style="color:#F24D37">UsernamePasswordAuthenticationFilter</span> i tu imamo metode attemptAuthentication i successAuthentication,mi mozemo da implementiramo ovaj filter i onda cemo mi upravljati auth

#### OncePerRequestFilter

Ovaj filter raidmo pre svakog requesta koji nam stigne



## WebFilter

Ovo je isto sto i filter samo radi async

## JWT(Json Web Token)

 ovo je nacin da se vrsi authentikacija,moderna pristup.Sastoji se od 3 kljucna dela

1. <span style="color:#F24D37">Header</span> -> npr tip algoritma za kodovanje otkena
2. <span style="color:#F24D37">Payload</span> -> osnovne informacije kao sto je mail roles...
3. <span style="color:#F24D37">Signature</span> -> algoritam potpisa

On sadrzi polja u payload delu koja su encode u base64 i potpisana od stran nekog algoritama.Ovo je siguran nacin za authentication koji se koristi,gde 1 token ako je tacan ima s ve privilegije korisnika.

JWT je za autorizaciju a ne autektikaciju

Mi kada se uspesno login nama server vraca ovaj token gde ga mi pamtimo na frontu i za svaki request posaljemo ovaj token u headr-u

<span style="color:#F24D37">Autorization : "Bearer NasToken"</span> -> jako je bitno da imamo Bearer jer je ovaj token jos nazvan i bearer token.

Server je dovoljnmo pametan da extract ovo sve lepo i proveri validnost tokena.

Svaki JWT token ima svoje vreme dokle vazi,posle koga ce isteci i necemo moci opet sa njim da pristupimo nekim rutama.To bi znacilo da opet moramo da se login ,ali to je malo naporno,zato postoji i <span style="color:#F24D37">refresh token</span> koji ima duze vreme trajanja i uz pomoc njega mi dobavljamo novi jwt.

Postoje vise algoriama koji se koriste za Signature deo tokena,jako je bitno koristiti dobar signature.

**SHA256** je ok pristup ako se token deli izmedju mikroservisa,ali sa klijentom nije,jer on sadrzi samo 1 key koji sign i decode token.Znaci ako jwt poasljemo klijentu imace key pomocu koga moze da kreira nove jwt tokene to ne bi bilo dobro da dospe u ruke hakera koji bi mogao da pravi nove tokene

**RSA256** je dobar za deljenje sa klijentima,jer je to princip public private key,jer se pomocu private keya potpisuje token i dekodira,a mi saljemo public key sa tokenom klijentu

Mi da bi radili sa jwt tokenom moramo ukljuciti biblioteku neku

**JJWT** je biblioteka 

#### JWT Encoding

```java
Jwts.builder().setClaims(Map<String,Obj> claims) -> ovo ide u body
			 .setSubject(String) -> username
			 .setIssuedAt(Date) 
			 .setExpiration(Date)
			 .signWith(Signatureneki)
			 .compact()->napravi jwt ,vratice string reprezentaciju tokena mi to vracamo
			 .setId("")
			 .setHeaderParam(key,val)
```

#### JWT Decoding

```java
Jwts.parser().isSigned(String jwt)
			.parse(jwt)
			.setSigningKey("secret") ->pre svih operacija moramo da stavimo ovo
			.parseClaims(token)
			.require(key,val)
			
```

```java
Path path = Path.of(ClassLoader.getSystemResource("private.der").toURI()); // todo: extract to properties file
byte[] privateKeyByteArray = Files.readAllBytes(path);

PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(privateKeyByteArray);

KeyFactory keyFactory = KeyFactory.getInstance("RSA");


return keyFactory.generatePrivate(keySpec);
```

ovo ce da  vrati PrivateKey klasu koju smestamo u

```java
  .signWith(SignatureAlgorithm.RS256, generatePrivateKey())
```

jwt nacin auth mozemo duraditi u filteru UsernamePasswordAuthenticationFilter

## GlobalMethodSecurity

Ovo je nacin da radimo lakse sa rolama.Prvo moramo da omogucimo ovo preko 

Dodaje se aspekt pre poziva same service metode,i on proverava:

Ovo je pre uvodjenja Gloval method security-ja

![image-20230214094929219](C:\Program Files\Typora\resources\Docs\img\image-20230214094929219.png)

Ovo je nakon uvodjenja

![image-20230214095028948](C:\Program Files\Typora\resources\Docs\img\image-20230214095028948.png)

<span style="color:#F24D37">@EnableGlobalMethodSecurity(</span>

<span style="color:#F24D37">prePostEnabled ->@PreAuthorize,@PostAuthorize,@PreFilter,@PostFilter</span>

<span style="color:#F24D37">Jsr250Enabled-> @RolesAllowed</span>

<span style="color:#F24D37">securedEnabled-> @Secured</span>

<span style="color:#F24D37">)</span>

zavisi sta omogucimo,te anotacije cemo moci da pisemo.

Za role moramo ROLE_IME ,mora imati ROLE prefiks,a u bazi ne 

```
@Secured([...]) -> stavimo String rola 
@RolesAllowrd([...]) -> barem 1 role mora da ima
```



```
@PreAuthorized(String) -> ovo radi spel
@PostAuthoreized(String)->ovo radi spel
```

ovo prethodno ne koristi spel vec direktno encode role

```java
@PreAuthorized("hasRole("ROLE_VIEVER)")
```



```
@PreFIlter
@PostFIlter
```



## OAuth 2

OAuth (Open Authorization) is an open standard for token-based authentication and authorization. It allows users to share their private resources (e.g. photos, videos, contact lists) stored on one site with another site without having to hand out their credentials, typically a username and password. Instead, the user is redirected to the site that holds their information, where they can grant permission for the other site to access it. This is done through the use of access tokens, which are issued by the authorization server and can be used by the client application to access the user's resources.

OAuth2 nam sluzi za <span style="color:#F24D37">autoriaciju</span> ne za autentikaciju.

OAuth2 je prvo  bio napravljen za service-service(izmedju servisa),ne za user-e da se oni autentikuju.

Services access other services on behaf of user.

OAuth je rodjen po primeru onog coveka sto mu damo kljuceve od kola da ih parkira,fora je da postoji poseban kljuc koji samo se daje njima i on ima organicen pristup,ne dajemo mu secret kljuc vec samo kljuc sa pristupom sta je zahtevao.

Access on behaf of user

<span style="color:#F24D37">Limited Key</span> dajemo a ne <span style="color:#F24D37">Master Key</span>.

![Frame 3](C:\Users\Ilija\Desktop\Frame 3.png)

<span style="color:#F24D37">Resource</span> ->Protected resources,ovo je sta se daje na access,tj sta treba da damo na access.Nesto nase na drugom servisu.

<span style="color:#F24D37">Resource owner</span> -> person who grants access

<span style="color:#F24D37">Resource server</span> -> ono gde se nalazi resource server

<span style="color:#F24D37">Client </span>-> Onaj koji trazi dozvolu

<span style="color:#F24D37">Authorization Server </span>-> Server koji radi autorizaciju

<span style="color:#F24D37">Redirect URI</span>

<span style="color:#F24D37">Callback URI</span>



<span style="color:#F24D37">Access Token</span>

<span style="color:#F24D37">Authorization grant </span>-> kada user klikne yes,dozvoljavam

<span style="color:#F24D37">Scope</span> -> Sta sve zeli da odobri.

<span style="color:#F24D37">Consent</span> -> moje odobrenje

<span style="color:#F24D37">Authorization code</span> -> Kod koji se vraca pri prvom zahtevu za auth,sa njim idemo na token endpoint i dobijemo access token

<span style="color:#F24D37">Backchannel </span>-> sa bekenda se salje zahtev na eksterni servis

<span style="color:#F24D37">Frontchannel </span>-> sa browser-a tipa se salje



<span style="color:#F24D37">State</span> -> Ovo je opcioni parametar koji se koristi da se odrzi stanje izmedju client oatuh2 app i authorization server app.State parametar se prosledjuje iz klijenta do autorizacionog servara u delu requesta i onda se vraca nazad iz autorizacionog servera u klijenta kada se radi redirect.On nam pomaze da sprecimo CSRF napade.Generise random string vrednost i ubacije je u authorization request,i onda kada nam stigne odogovor od severa on ukljuci tu vrednost.Kad astigne odgovr mi proveravamo state parametar iz responsa sa onim koji je poslat u requetu.Ako se match onda je sve ok



<span style="color:#F24D37">code_challenge</span>

<span style="color:#F24D37">code_challenge_method</span>

<span style="color:#F24D37">code_verifier</span>

![Frame 4](C:\Users\Ilija\Desktop\Frame 4.png)

Mi cemo da se login na PhotoService i zahtevati print,ali to ce nam trebati slike za google naloga,tj sa njegovog servera.Nas resurs na drugom nepoznatom serveru.Pa ce nas servis pitati goolge,google nam ne veruje i mora licno da nas pita,mi potvrdjujemo i google salje authorization code sa kojim moze nas backedn da izvuce slike iz google

Postoje vise oauth2 nacina autorizacije tj vise flow typa,mi obradjujemo ovde kao main type **authorization code flow**

The authorization code flow offers a few benefits over the other grant types. When the user authorizes the application, they are redirected back to the application with a temporary code in the URL. The application exchanges that code for the access token. When the application makes the request for the access token, that request can be authenticated with the client secret, which reduces the risk of an attacker intercepting the authorization code and using it themselves. This also means the access token is never visible to the user or their browser, so it is the most secure way to pass the token back to the application, reducing the risk of the token leaking to someone else.![Frame 5](C:\Users\Ilija\Desktop\Frame 5.png)

Glavna fora je da mi ovo mozemo da koristimo izmedju mikroservisa za zahtevanje podataka,mikrosevice 1 nece imati pristup podacima koji ce imati microservice 2 pa ce od auth servera da trazi dozovlu.





![Frame 6](C:\Users\Ilija\Desktop\Frame 6.png)

![1_7xjn6iM_MOV5Syleep61Yg](C:\Program Files\Typora\resources\Docs\img\1_7xjn6iM_MOV5Syleep61Yg.png)

### OAuth2 request flor

1) Saljemo GET request na auth server

   ​	auth_server_uri_authorize_endpoint?client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=SCOPE&response_type=RESPONSE_TYPE

   ​	redirect_uri je uri koji smo mi registrovali na google app.Tu ce da nas redirect kada se uspesno prijavimo na google auth server,i kada redirect u parametre ce da ubaci code,i radice post odmah,ali kako mi nemamo tu rutu registrovanu na springu nece se nista desiti.Mi kada radimo default iz springa on automatski stavlja redirect_uri na onaj default jer taj je vec unutar springa registrovan(base/login/oauth2/registeredprovider) . Gde ces me redirect kada sam uspesno login na auth server 3rd partyija
   
   **Redirect uri** i **callback uri** su isti koncepti,predtavljaju uri na kojima cemo se redirect ako sve bude uspesno.
   
   Redirect uri redirektuje nakon uspesnog logina gde prosledjuje code kao parametar ,dok callbackrui redirektuje tek kada uzmemo token.Moze se redi ca prvo ide redirce uri jer njega saljemo u get zahtevu za code,a callback uri(iako se zove redirect  u query isto ali ih ovako razlikujemo u teoriji) nas redirect kada dobijemo token tj u post zahtevu.Za security razloge najbolje je da **budu isti**

| `client_id`     | `string` | **Required**. The client ID you received from GitHub when you [registered](https://github.com/settings/applications/new). |
| --------------- | -------- | ------------------------------------------------------------ |
| `redirect_uri`  | `string` | **Required**.The URL in your application where users will be sent after authorization. See details below about [redirect urls](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps#redirect-urls). |
| `response_type` | `string` | **Required**.Koji authroization code flow koristimo(code je najbolji,to je sve ovo objesnjen do sada) |
| `scope`         | `string` | **Required**.A space-delimited list of [scopes](https://docs.github.com/en/apps/building-oauth-apps/understanding-scopes-for-oauth-apps). If not provided, `scope` defaults to an empty list for users that have not authorized any scopes for the application. For users who have authorized scopes for the application, the user won't be shown the OAuth authorization page with the list of scopes. Instead, this step of the flow will automatically complete with the set of scopes the user has authorized for the application. For example, if a user has already performed the web flow twice and has authorized one token with `user` scope and another token with `repo` scope, a third web flow that does not provide a `scope` will receive a token with `user` and `repo` scope. |
| `state`         | `string` | .An unguessable random string. It is used to protect against cross-site request forgery attacks.(Optional) |
| code_challenge  | String   | Ovo je Optional ,za PCKE flow                                |
| code_method     | String   | s256 uglavnom,metoda koja se koristila da se hash code_challenge |

2) User se redirekt nazad od strane Github-a koji je resource server u ovom slucaju.Redirekt sadrzi parametar **code** koji nam sluzi da uzmemo token

3) Saljemo POST zahtev na token endpoint i ukljucujemo code kako bi dobili id token ili auth token

   ​	 auth_server_uri_token_endpoint?code=CODE_STRING&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&redirect_uri=CALLBAC_URI&grant_type=GRANT_TYPE

4. If you look in the browser tools (F12 on Chrome or Firefox) and follow the network traffic for all the hops, you will see the redirects back and forth with GitHub, and finally you’ll land back on the home page with a new Set-Cookie header. This cookie (JSESSIONID by default) is a token for your authentication details for Spring (or any servlet-based) applications.

So we have a secure application, in the sense that to see any content a user has to authenticate with an external provider (GitHub).

| Name            | Type     | Description                                                  |
| :-------------- | :------- | :----------------------------------------------------------- |
| `client_id`     | `string` | **Required.** The client ID you received from GitHub for your OAuth App. |
| `client_secret` | `string` | **Required.** The client secret you received from GitHub for your OAuth App. |
| `code`          | `string` | **Required.** The code you received as a response to Step 2. |
| `redirect_uri`  | `string` | **Required**.Gde ce da nas redirect kada se sve uspesno zavrsi |
| `code_verifier` | `string` | **Optional.**Ovo saljemo da bi proverili code_challenge kako bi sprecili CSRF napad(PKCE) |
| `grant_type`    | `string` | **Required**.Ovo je koji flow smo koristili (authorization_code postoje jos neki ali ovo je main i do sda objasnjen) |

An example of how OAuth 2.0 is used is when a user wants to log in to a third-party website or application using their Facebook account.

1. The user clicks on the "Sign in with Facebook" button on the third-party website.
2. The website redirects the user to the Facebook authorization server.
3. The user is prompted to log in to their Facebook account if they are not already logged in.
4. The user is then presented with a permissions page asking them to confirm that they would like to share their basic profile information and email address with the third-party website.
5. If the user agrees, the authorization server sends an authorization code to the third-party website.
6. The third-party website will then exchange the authorization code with an access token by making a request to Facebook token endpoint, along with client_id,client_secret and redirect_uri.
7. The third-party website can then use the access token to request and access the user's basic profile information and email address from the Facebook API.

This example is a simplified version of the OAuth 2.0 flow, but it demonstrates how OAuth allows users to share their information with third-party sites without having to give out their password.

If Google trusts us then i trust him too.

Access token je uvek na backendu nikad na frontendu.

Znamo da je Oauth samo za autorizaciju ali se moze modifikovati i koristini i za autentikaciju,ali to nije najbolje.Za to je napravljen novi protokol <span style="color:#F24D37">OPENID</span>



Ok, this is a two parts authentication.

- First you will do a `HTTP Get` in order to get the code. So the client will be redirected to the authentication server.

  Once he enters login/ passwords and successfully authenticate to the Oauth2 app he will be redirected back to his client app with the code added as a parameter in the URL.

- The client gets the code from the URL and calls back the authentication server with a `HTTP POST` with the code as request parameter and he will get the access token in the response the access token is then used as a header to access the





klijent dobije code ali taj kod je validan samo ako ga posalje nas backend pa zato mora post da uradi.Kako je validan samo na backendu?Pa imamo client secret sacuvan na backendu i po tome google zna



Figure 1 shows the attack graphically.  In step (1), the native
   application running on the end device, such as a smartphone, issues
   an OAuth 2.0 Authorization Request via the browser/operating system.
   The Redirection Endpoint URI in this case typically uses a custom URI
   scheme.  Step (1) happens through a secure API that cannot be
   intercepted, though it may potentially be observed in advanced attack
   scenarios.  The request then gets forwarded to the OAuth 2.0
   authorization server in step (2).  **B**ecause OAuth requires the use of**
   **TLS, this communication is protected by TLS and cannot be**
   intercepted**.  The authorization server returns the authorization code
   in step (3).  In step (4), the Authorization Code is returned to the
   requester via the Redirection Endpoint URI that was provided in step
   (1).
    Note that it is possible for a malicious app to register itself as a
   handler for the custom scheme in addition to the legitimate OAuth 2.0
   app.  Once it does so, the malicious app is now able to intercept the
   authorization code in step (4).  This allows the attacker to request
   and obtain an access token in steps (5) and (6), respectively.



![image-20230131113944713](C:\Program Files\Typora\resources\Docs\img\image-20230131113944713.png)



   To mitigate this attack, this extension utilizes a dynamically
   created cryptographically random key called "code verifier".  A
   unique code verifier is created for every authorization request, and
   its transformed value, called "code challenge", is sent to the
   authorization server to obtain the authorization code.  The
   authorization code obtained is then sent to the token endpoint with
   the "code verifier", and the server compares it with the previously
   received request code so that it can perform the proof of possession
   of the "code verifier" by the client.  This works as the mitigation
   since the attacker would not know this one-time key, since it is sent
   over **TLS and cannot be intercepted.**



INtelij app je client pa ce ona da generise code challenge i code verifier



![image-20230131114339373](C:\Program Files\Typora\resources\Docs\img\image-20230131114339373.png)

An example of how OAuth 2.0 can be implemented in a Spring application is by using the Spring Security OAuth project.

1. First, you need to add the Spring Security OAuth dependency to your project.
2. Next, you need to configure an OAuth2 client in your application.yml file. This will typically include information such as the client_id and client_secret, as well as the authorization and token endpoints for the OAuth2 provider you are using (e.g. Google or Facebook).
3. In your Spring Security configuration, you need to enable OAuth2 client support by adding the `@EnableOAuth2Client` annotation.
4. You can then create a `OAuth2RestTemplate` bean, which will be used to make requests to the OAuth2 provider's API on behalf of the user.
5. In your controller, you can use the `OAuth2RestTemplate` to exchange the authorization code for an access token, and then use the access token to make requests to the OAuth2 provider's API to fetch user information.
6. You can then use this information to create a user in your local database and log the user in to your application.

Nas backend redirect user-a i u url mu ubaci code_challenge,znaci da mi kao backend tj gde stigne zahtev generisemo PKCE





#### Response

By default, the response takes the following form:

```
access_token=gho_16C7e42F292c6912E7710c838347Ae178B4a&scope=repo%2Cgist&token_type=bearer
```

pecifically, refresh tokens must be valid for only one use, and the authorization server must issue a new refresh token each time a new access token is issued in response to a refresh token grant. This provides the authorization server a way to detect if a refresh token has been copied and used by an attacker, since in normal operation of an app a refresh token would be used only once.

Refresh tokens must also either have a set maximum lifetime, or expire if they are not used within some amount of time. This is again another way to help mitigate the risks of a stolen refresh token.

The browser-based app will need to temporarily store some pieces of information during the authorization flow, and then permanently store the resulting access token and refresh token. This provides some challenges in the browser environment since there are currently no general-purpose secure storage mechanism in browsers.

Generally, the browser’s `LocalStorage` API is the best place to store this data as it provides the easiest API to store and retrieve data and is about as secure as you can get in a browser. The downside is that any scripts on the page, even from different domains such as your analytics or ad network, will be able to access the `LocalStorage` of your application. This means anything you store in `LocalStorage` is potentially visible to third-party scripts on your page.

Due to the inherent risks of performing an OAuth flow in a pure JavaScript environment, as well as the risks of storing tokens in a JavaScript app, it is also advisable to consider an alternative architecture where the OAuth flow is handled outside of the JavaScript code by a dynamic backend component. This is a relatively common architectural pattern where an application is served from a dynamic backend such as a .NET or Java app, but it uses a single-page app framework like React or Angular for its UI. If your app falls under this architectural pattern, then the best option is to move all of the OAuth flow to the server component, and keep the access tokens and refresh tokens out of the browser entirely. Note that in this case since your app has a dynamic backend, it is also considered a confidential client and can use a client secret to further protect the OAuth exchanges.



It is generally a best practice to request scopes incrementally, at the time access is required, rather than up front. For example, an app that wants to support saving an event to a calendar should not request Google Calendar access until the user presses the "Add to Calendar" button; see



## OpenId

Ovo je nadogradnja na OAuth2.

OAuth2 sluzi za autorizaciju,ali se moze koristiti i za autentikaciju mada to niej dobro,nije namanjen za to iako ce raditi sve ok.Za to je uvedem OpenId koji je specificno za autentikaciju.

Razlika je sto kada se posalje AutorizeToken dobije se <span style="color:#F24D37">Access Toekn</span> i <span style="color:#F24D37">Id Token</span>,gde je id token podaci o user-u.I mi uzimmao IdToken jer je to za openid



- An ID token is a JSON Web Token (JWT) that contains information about the authenticated user, such as their user ID, name, and other claims. It is used to authenticate the user and to pass information about the user to the client.
- An access token, on the other hand, is a token that grants access to a protected resource. It is used to authenticate API requests made by the client. Access tokens have a limited lifetime and are typically short-lived.

In a typical OAuth 2.0 flow, the authorization server issues both an ID token and an access token to the client after the user has granted access. The client can then use the access token to access protected resources on behalf of the user, and use the ID token to authenticate the user and get information about the user.

Ako je openid protokol vratice i token id uz access token,ali ako nije vratice se samo access token sto jedefault za oauth2![`](C:\Users\Ilija\Desktop\Frame 7.png)

bitn oje da kada trazimo scope mi ubacimo openid i tako ce auth server automatski da zna o kom protokolu je rec



kada radimo openid samo u scope ubacimo openid u .yml file

Nije htelo da mi radi za github,dok je za google radilo lagano

DEBBUGER

Idemo na [OAuth 2.0 debugger (oauthdebugger.com)](https://oauthdebugger.com/)

i moramo da unesemo

<span style="color:#F24D37">Authorize URI</span>-> ovo je URL od authorization server-a (npr hocemo google i mi moramo da znamo authorization server url) svaki authorization server velikih kompanija je javan i mozemo da ga nadjemo za google je npr:https://accounts.google.com/o/oauth2/v2/auth

<span style="color:#F24D37">Redirect URI</span> -> ovo je URL koji smo prethodno registrovali na nasu app(da bi imali client secert i client id mi moramo da napravimo app npr u google api,i moramo da registrujemo redirect uri,redirect uri su svi uri koji smo deklarisali da su bezbedni i da google veruje),jer testiramo default je njihov za debug i mi taj za debug moramo da stavimo u google app kako bi bio bezbedan za podnosenje zahteva.

<span style="color:#F24D37"> Client ID</span> ->ovo je id koji dobijemo kada registrujemo app 

<span style="color:#F24D37">Scope</span> profile,openid...

https://accounts.google.com/.well-known/openid-configuration see all google endpoint info





5. 

## Resource Server

A resource server is a server that hosts protected resources and provides access to those resources through a secure API. In the context of OAuth 2.0 and OpenID Connect, the resource server is responsible for serving resources that are protected by a OAuth 2.0 access token, and for validating the access token presented by the client to access those resources. The resource server needs to ensure that the access token presented by the client is valid and authorized to access the requested resources.

Yes, you can have multiple resource servers in a microservices architecture. Each resource server can be responsible for managing a specific set of resources and can have its own security policies, access control mechanisms, and APIs. This allows for better modularity, scalability, and maintenance of the system.



as data or APIs, and ensuring that access to those resources is controlled based on the authorization policies defined for the system. It can interact with the authorization server to validate incoming requests and determine if the requester has the necessary permissions to access the protected resources.

Having a resource server as a separate microservice can help improve the modularity, scalability, and maintainability of the system by allowing you to isolate the resource management logic from other parts of the system.

We need to add 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

and then configure it 

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
  http.cors(c -> {
    CorsConfigurationSource source = s -> {
      CorsConfiguration cc = new CorsConfiguration();
      cc.setAllowCredentials(true);
      cc.setAllowedOrigins(List.of("http://127.0.0.1:3000"));
      cc.setAllowedHeaders(List.of("*"));
      cc.setAllowedMethods(List.of("*"));
      return cc;
    };

    c.configurationSource(source);
  });

  http.authorizeHttpRequests().anyRequest()
      .authenticated().and()
      .oauth2ResourceServer()
      .jwt();
  return http.build();
}
```

MOgucnost da neka 3rd party app koristi nase resurse

## CSRF

Cross site request forgery





Ovo je najlaksi nacin da uradimo

```java
httpSecurity.csrf().disable()
```



Ovo je naprednije i odnosi se na to da se token kreira za svaki request

Postoje 2 tipa ove respository koje su implementacije ovog interfejsa CsrfTokenRepository 

**HttpSessionCsrfTokenRepository **-> store token u HTTP sessino
**CookieCsrfTokenRepository** -> store token u Cookie (Stateless ovo )



```java
.csrf(x->x.csrfTokenRepository(CsrfTokenRepository))
```

```java
.csrf(x->x.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()))
.csrf(x->x.csrfTokenRepository(new HttpSessionCsrfTokenRepository()))
```

```java

@GetMapping("/csrf")
public CsrfToken csrf(CsrfToken token){
    return token;
}
```

Ovako mozemo da uzmemo CsrfToken.

Kako bi povecali security jos vise ovaj token se mora poslati i u Cookie i u Header-u **X-XSRF-TOKEN**



## OAuth2 Client Credentials Grant Type

Ovo je kada nemamo clienta definisanog



## OAuth2 Token Grant type

Ako je acces token invalid,pa da ne prolazi kroz ceo proces on smao uz refresh token uzme novi,taj proces je ovaj grant type

![image-20230206164846047](C:\Program Files\Typora\resources\Docs\img\image-20230206164846047.png)
