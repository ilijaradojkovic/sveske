# OAuth2 Google

Da bi uopste radili sa Oauth2 standardima u Spring-u moramo dodati dependency za Oauth2 client.

Dodajemo za client jer se mi ponasamo kao klijenti koji hoce da koriste google authorization server.

## 1)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```





Kada smo dodali ovaj dependency sada moramo namestiti google app da bi dobili client secret i client id.

## 2)

Odemo na https://console.cloud.google.com/apis/credentials to je dashboard koji sadrzi nase app u google,mi tu ako nemamo napravimo nasu i dodamo oauth2.

Bitne stvari koje moramo da podesimo su 

Home uri -> to je base ip od nase aplikacije http://localhost:8080

Authorized redirect uri -> to je ip gde ce da se redirect kada uspesno unesemo kredencijale,i kako bi spring to radio u pozadini sve on ima poseban endpoint za to.

Za google ja ovaj http://localhost:8080/login/oauth2/code/google

ali generalno izgledaju ovako {baseUrl}/login/oauth2/code/{registrationId} pa bi za github bilo

 http://localhost:8080/login/oauth2/code/github gde menjamo ovaj registrationid 

## 3) 

Kada napravimo app dobicemo **Client Id** i **Client Secret**.Seledeci korak je da mi to unesemo u nasu spring konfiguraciju kako bi spring automatski znao sve

Odemo u application.yml fajl i unesemo

```yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: 161670812290-utfovu2c2q5n65skdefjn9845b8ehhk8.apps.googleusercontent.com
            client-secret: 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
```

Ovo bi bila najednostavnija oknfiguracija.Opazimo ovde da imamo registration  pa google e to znaci da je nas registration id iz koraka 2 kada smo registrovali redirect uri bas to ime google,da smo ga drugacije nazvali morali bi druacije da registruje redirect uri

Znaci redirect uri zavisi od registratio nid iz koraka 3.

Kada ovo namestimo mozemo isprobati odmah,ali cemo najpre konfigurisati i security.

## 4) 

Napravimo novu klasu za configu security i u njoj

```java
@Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
    return 
        1)httpSecurity.authorizeHttpRequests().anyRequest().authenticated()
           .and()
        2).formLogin()
            .and()
        3).oauth2Login()
            .and().build();
}
```

1)Podesicemo da sve rute budu auth

2)Pored Oauth2 mogucnosti da se login,ovo nam daje da mi imamo nasu login formu za nasu app

3)Ovime ce se display Oauth2 login izbori



Mi kada smo se login sada imamo pristup to useru preko 

OidcUser->Interfejs 

DefaultOidcUser->Konkrentna implementacija

```java
@GetMapping("/user/openid")
public String openIdUser(@AuthenticationPrincipal OidcUser user){
    return user.toString();
}

@GetMapping("/user/oauth2")
public String oauth2User(@AuthenticationPrincipal OAuth2User user){
    return user.toString();
}
```

```java
@GetMapping("/token/oauth2")
public String oauth2Token(OAuth2AuthenticationToken auth2AuthenticationToken){
    return auth2AuthenticationToken.toString();
}
```

Mi kada se registrujemo preko google po defaultu nam se inject session u nas browser i spring security context pa je to sso.

OidcUser vs OAuth2User vs OAuth2AuthenticationToken

Svi su oni Principal

razlika je u tome sto OAuth2AUthenticationToken je u stvari implementacija Authentication interfejsa iz Spring Security i predtavlja authentication token iz OAtuh2 flow-a.On sadrzi access token i opciono ako je openid i tokenid.

Oidc je interfejs koji sadrzi metoda da se pristupi open id informacijama i predstavlja autentikovanog usera u open id flow.Kako je OAtuh2AuthenticationToken sadrzi i oenid i oatuh2 mi mozemo ovu klasu da kastujemo u zeljnu oidcUser ili OAtuh2User zavisi kakav flow je isao,Tj mi uzmemo principal iz OAuth2AuthenticationToken i njega kastujemo

```java
@GetMapping("/oidcIdToken/v1")
public String getOidcIdToken(OAuth2AuthenticationToken authentication) {
    OidcUser authorizedClient = (OidcUser) authentication.getPrincipal();
    return authorizedClient.getIdToken().getTokenValue();
}
```



Spring nam provide vec autoconfigured konfiguraciju samo nam je potrebno da ubacimo u yml sve ono,ako hocemo vise da bude custom moramo da pisemo bean-ove.

Bitno beanovi:

## ClientRegistrationRepository

<span style="color:#ff4d4d">ClientRegistrationRepository</span> -> Ovo je interfejs iz oauth2-client biblioteke.Ona definise metode koje cuvaju i uzimaju registrovane klijente iz Oatuh2 autorizacionog secera.Svrha je da mi pomocu apstrakcije mozemo da biramo kako cemo to cuvati.Instanca ovoga uzima registrovane klijente iz yml fajla i provide nama kao end korisnicima da mozemo preko njih da radimo OAuth2 proces.Metode ovoga nam daju <span style="color:#ff4d4d">ClientRegistration</span> za odredjeni registration ID.U springu mi mozemo da impl ovo sami,ili da koristimo postajacsf <span style="color:#ff4d4d">InMemoryClientRegistartionRepository</span>

Ovo sluzi za one koje smo registrovali u yml fajlu,google,github...

ovo googleClientRegistration() vraca RegistrationClient koji se pravi pomocu  CommonOAuth2Provider.GOOGLE

MI kada overide ovako tj kada definisemo bean on ce overide svu tu konfiguraciju,on nece da gleda nas yml fajl pa zato necemo ni imati provajdere kada napravimo samo new InMemory... vec moramo da ubacimo sami onda 



```java
	@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}
```

```java
@Bean
public ClientRegistrationRepository clientRegistrationRepository(){
    return new InMemoryClientRegistrationRepository(
            CommonOAuth2Provider.OKTA.getBuilder("okta")
            .clientId("efwre")
                    .clientSecret("ewqg")
                    .authorizationUri("test")
                    .tokenUri("test")
                    .build());
}
```

Iako imamo u yml falju registrovane i google i github on nece njih da gelda jer smo ovime overide tu default konfiguraciju,i geldace samo ovo sto smo mu mi definisali

VIdimo da nam treba ClientRegistration da bi napravili ovo

ClientRegistrationRepository je na OAuth2 serveru ne na klijentu  i defaultna implementacija ove u springu je InMemoryClientRegistartionRepository

@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}

	private ClientRegistration googleClientRegistration() {
		return ClientRegistration.withRegistrationId("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
			.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
			.redirectUri("{baseUrl}/login/oauth2/code/{registrationId}")
			.scope("openid", "profile", "email", "address", "phone")
			.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth")
			.tokenUri("https://www.googleapis.com/oauth2/v4/token")
			.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo")
			.userNameAttributeName(IdTokenClaimNames.SUB)
			.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs")
			.clientName("Google")
			.build();
	}

## ClientRegistration

<span style="color:#ff4d4d">Client Registration</span> -> ovo je klasa koja predstalja registrovanog klijenta na OAuth2 serveru.Ona cuva id,client id,client secret,redirect uri...

```java
private ClientRegistration googleClientRegistration() {
		return CommonOAuth2Provider.GOOGLE.getBuilder("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.build();
	}
```

Ili

```java
ClientRegistration.withRegistrationId()...
```



Ili

```java
ClientRegistrations.fromIssuerLocation().build()
```



CommonOAuth2Provider bitna klasa koja nam daje dafault providere

Mi kada registrujemo klijente on ce prepoznati ovo okta string i pretvorice ga u CommonOAuth2Provider.neki prepoznace da je to enum pa ce znacti odmah vrednosti za ostale elemente kao sto su redirect uri auth_uri...

Ako zelimo sami da definisemo samo kazemo provider i time on nece geldati CommonOAuth2Provider vec ovaj u nasoj config

```java
spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            client-id: okta-client-id
            client-secret: okta-client-secret
        provider:
          okta:	
            authorization-uri: https://your-subdomain.oktapreview.com/oauth2/v1/authorize
            token-uri: https://your-subdomain.oktapreview.com/oauth2/v1/token
            user-info-uri: https://your-subdomain.oktapreview.com/oauth2/v1/userinfo
            user-name-attribute: sub
            jwk-set-uri: https://your-subdomain.oktapreview.com/oauth2/v1/keys
```





## OAuth2AuthorizedClient

je reprezentacija autorizovanog korisnika.Klijent je autorizovan kada end user(Resource owner) dozovli autorizaiju .



Mi mozemo da autowire service ili repository i preko njega uzimamo OAuth2AuthorizedClient

```java
        OAuth2AuthorizedClient google = authorizedClientService.loadAuthorizedClient("google", authentication.getName());

```

"google" -> ovo je registered client u applicatiin.yml fajlu a principal dobijamo iz metode 



Ili da koristimo anotaciju <span style="color:#ff4d4d">@RegisteredOAuth2AuthorizedClient("registeredclient")</span>

```java
@RegisteredOAuth2AuthorizedClient("okta") OAuth2AuthorizedClient authorizedClient
```

## OAuth2AuthorizedClientService 

Ovo nam sluzi da cuvamo informacije o registrovanom koristniku kao sto su access token,refresh token...

Razlika izmedju *OAuth2AuthorizedClientService* i *ClientRegistrationService* je ta sto OAuth2AuthorizedClientService cuva informacije privremeno o tokenima,a ovo drugo cuva info bukvalno o korisnicima ime secret...

Po defaultu je <span style="color:#ff4d4d"> InMemoryOAuth2AuthorizedClientService </span>,takodje mozemo mi nasu da damo  i cuvamo podatke u db <span style="color:#ff4d4d">JdbcOAuth2AuthorizedClientService</span> samo moramo paziti na semu user-a koji postoji.Jer cuvamo OAuth2AuthorizedClient imamo semu 

```sql
CREATE TABLE oauth2_authorized_client (
  client_registration_id varchar(100) NOT NULL,
  principal_name varchar(200) NOT NULL,
  access_token_type varchar(100) NOT NULL,
  access_token_value blob NOT NULL,
  access_token_issued_at timestamp NOT NULL,
  access_token_expires_at timestamp NOT NULL,
  access_token_scopes varchar(1000) DEFAULT NULL,
  refresh_token_value blob DEFAULT NULL,
  refresh_token_issued_at timestamp DEFAULT NULL,
  created_at timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL,
  PRIMARY KEY (client_registration_id, principal_name)
);
```

## OAuth2AuthorizedClientRepository



## AuthorizationRequestRepository

 Authorization request se kreira kada client app zeli da pristupi OAtuh2 zasticenom resorsu.Sadrzi informacije neophodne koje se salju u taj zahtev,scopes...Mi pomocu AuthorizationRequestRepository cuvamo te requestove.Postoje vec default implementacije ovo i to je ***HttpSessionOAuth2AuthorizationRequestRepository*** a mozemo i mi nasu.Ovaj default cuva OAuth2AuthorizationRequest u *Http* *Session*

<span style="color:#ff4d4d">AuthorizationResponseRepository</span> -> Ovo nam sluzi da mi redirect client na autorizacioni server

<span style="color:#ff4d4d">OAuth2AccessTokenResponseClient</span> -> Sluzi da mi uzmemo OAuth2 access token .Ovo mozemo da customize jer ovo vraca tokene

<span style="color:#ff4d4d">OAuth2AccessTokenResponseClient</span> -> Sluzi da radimo sa responsom na token endpointu i defaultna implementacija ovoga je <span style="color:#ff4d4d">DefaultAuthorizationCodeTokenResponseClient</span> koja korsti code kako bi uzela token

Mi moze da customise post proces ovoga i pre proces ovoga,znaci mozemo da customize request koji se zalje za token i response koji dobijamo

##  OAuth2AuthorizationRequestResolver

Sluzi da resolvuje OAuthAuthorizationRequest i da inicijalizuje Authorization Code flow.

Njegova default implementacija je DefaultOAuth2AuthorizationRequestResolver koja match default path /oauth2/authorization/{registrationId}.On pravi OAuthAuthorizationReuqst na osovu registrationId tj ClientRegistration





## OAuth2UserService

**DefaultOAuth2UserService** je default implementacija ovoga za OAuth2

**OidcUserService** je default implementaciza ovoga za Openid

<span style="color:#ff4d4d">OidcUserService</span> -> Sluzi da izvlacimo OAuth2 user profile information,izvlaci se tip OidcUser

<span style="color:#ff4d4d">OAuth2UserService</span> -> Sluzi da izvlacimo OAuth2 user profile information,izvlaci se tip OAuth2User

```java
x.userInfoEndpoint()
```

Ovo namestamo na userInfoEndpoint() jer nam sluzi da posaljemo request i da dodbijemo user-a

Opsti itnterfejs je genericki <T,R>

T->Neki request kao sto su oidcUserRequest ili OAtuh2UserRequest

R->Return type sto moze biti OAtuh2User ili OidcUser

```java
public class CustomOAuth2UserService implements OAuth2UserService<NekiRequest,NekiUser> {
    @Override
    public NekiUser loadUser(NekiRequest userRequest) throws OAuth2AuthenticationException {
        return null;
    }
}
```



OidcUserRequest

OAuth2UserRequest

`OAuth2UserRequest` and `OAuth2User` (when using an OAuth 2.0 UserService) or `OidcUserRequest` and `OidcUser` (when using an OpenID Connect 1.0 UserService).

Korak 3) mozemo da napisemo i u kodu umesto u konfiguraciji preko ClientRegistrationRepository

Ako definisemo bean za nju na client strani ona ce biti u stvari kredencijali za google





Takodje http.oauth2Login sadrzi brojne konfiguracije

```java
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
			.oauth2Login(oauth2 -> oauth2
			    .authorizationEndpoint(authorization -> authorization
			            ...
			    )
			    .redirectionEndpoint(redirection -> redirection
			            ...
			    )
			    .tokenEndpoint(token -> token
			            ...
			    )
			    .userInfoEndpoint(userInfo -> userInfo
			            ...
			    )
			);
		return http.build();
	}
```

```java
@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
			.oauth2Login(oauth2 -> oauth2
			    .clientRegistrationRepository(this.clientRegistrationRepository())
			    .authorizedClientRepository(this.authorizedClientRepository())
			    .authorizedClientService(this.authorizedClientService())
			    .loginPage("/login")
			    .authorizationEndpoint(authorization -> authorization
			        .baseUri(this.authorizationRequestBaseUri())
			        .authorizationRequestRepository(this.authorizationRequestRepository())
			        .authorizationRequestResolver(this.authorizationRequestResolver())
			    )
			    .redirectionEndpoint(redirection -> redirection
			        .baseUri(this.authorizationResponseBaseUri())
			    )
			    .tokenEndpoint(token -> token
			        .accessTokenResponseClient(this.accessTokenResponseClient())
			    )
			    .userInfoEndpoint(userInfo -> userInfo
			        .userAuthoritiesMapper(this.userAuthoritiesMapper())
			        .userService(this.oauth2UserService())
			        .oidcUserService(this.oidcUserService())
			    )
			);
		return http.build();
	}
```



## Customize Login Endpoint

```java
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
			.oauth2Login(oauth2 -> oauth2
			     .oauth2Login(x->
                    x
                            	.authorizationEndpoint().baseUri("/login/oauth2/authorization")
                            .and()
                            .redirectionEndpoint().baseUri("/login/oauth2/callback/*")
            )
			);
		return http.build();
	}
```



ako promenimo samo loginPage("...") moramo imati stranu koja ima prikaz html elemenata.

authorizationEndpoint menjamo za auth server

redirectionEndpoint menjamo za uath server default ide uvel **/login/oauth2/code/***



## Mapping Authorities

Ovo radimo preko interfejsa <span style="color:#ff4d4d">GrantedAuthoritiesMapper</span>.Fora je da mi imamo 2 tipa authorties za oauth to su <span style="color:#ff4d4d">OAuth2UserAuthority</span> i <span style="color:#ff4d4d">OidcUserAuthority</span>



```java
 public Collection<? extends GrantedAuthority> mapAuthorities(Collection<? extends GrantedAuthority> authorities) {
        HashSet<GrantedAuthority> myauthorities=new HashSet<>();
       authorities.forEach(authority->
       {
           if (OidcUserAuthority.class.isInstance(authority)) {
               OidcUserAuthority oidcUserAuthority = (OidcUserAuthority)authority;

               OidcIdToken idToken = oidcUserAuthority.getIdToken();
               OidcUserInfo userInfo = oidcUserAuthority.getUserInfo();

               // Map the claims found in idToken and/or userInfo
               // to one or more GrantedAuthority's and add it to mappedAuthorities

           } else if (OAuth2UserAuthority.class.isInstance(authority)) {
               OAuth2UserAuthority oauth2UserAuthority = (OAuth2UserAuthority)authority;

               Map<String, Object> userAttributes = oauth2UserAuthority.getAttributes();

               // Map the attributes found in userAttributes
               // to one or more GrantedAuthority's and add it to mappedAuthorities

           }

       });
       return myauthorities;
    }
```



## OAuth2 Client

```java
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
			.oauth2Client(oauth2 -> oauth2
				.clientRegistrationRepository(this.clientRegistrationRepository())
				.authorizedClientRepository(this.authorizedClientRepository())
				.authorizedClientService(this.authorizedClientService())
				.authorizationCodeGrant(codeGrant -> codeGrant
					.authorizationRequestRepository(this.authorizationRequestRepository())
					.authorizationRequestResolver(this.authorizationRequestResolver())
					.accessTokenResponseClient(this.accessTokenResponseClient())
				)
			);
		return http.build();
	}
```

## OAuth2AuthorizedClientManager

je odgovoran za mapiranje autorizacije ili autentikacije OAuth klijenta,on saradjuje sa jednim ili vise OAuth2AuthorizedClientProvider (SLicna je situacija kao kod obicnog security gde imamo AuthenticationManager i AuthenticationProvider-a)

Defaultna implementacija je <span style="color:#ff4d4d">DefaultOAuth2AuthorizedClientManager</span>

On upravlja registrovanim provajderima 

Kada authorization pokusaj uspe *DefaultOAuth2AuthorizedClientManager* delegira na  *OAuth2AuthorizationSuccessHandler* koji po defaultu sacuva *OAuth2AuthorizedClient* preko OAuth2AuthorizedClientRepository

When an authorization attempt succeeds, the `DefaultOAuth2AuthorizedClientManager` delegates to the `OAuth2AuthorizationSuccessHandler`, which (by default) saves the `OAuth2AuthorizedClient` through the `OAuth2AuthorizedClientRepository`. In the case of a re-authorization failure (for example, a refresh token is no longer valid), the previously saved `OAuth2AuthorizedClient` is removed from the `OAuth2AuthorizedClientRepository` through the `RemoveAuthorizedClientOAuth2AuthorizationFailureHandler`. You can customize the default behavior through `setAuthorizationSuccessHandler(OAuth2AuthorizationSuccessHandler)` and `setAuthorizationFailureHandler(OAuth2AuthorizationFailureHandler)`.

```java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
		ClientRegistrationRepository clientRegistrationRepository,
		OAuth2AuthorizedClientRepository authorizedClientRepository) {

	OAuth2AuthorizedClientProvider authorizedClientProvider =
			OAuth2AuthorizedClientProviderBuilder.builder()
					.authorizationCode()
					.refreshToken()
					.clientCredentials()
					.password()
					.build();

	DefaultOAuth2AuthorizedClientManager authorizedClientManager =
			new DefaultOAuth2AuthorizedClientManager(
					clientRegistrationRepository, authorizedClientRepository);
	authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

	return authorizedClientManager;
}
```

Ovime omogucavamo da on podrzava da se autorizacije ili autentikacija rade pomocu authorizationCode ili refreshTOken ili clientCredentails ili password jer su to 4 nacina sa kojima OAuth2 radi

## 

## OAuth2 Success Handler

Samo implementiramo interfejs <span style="color:#ff4d4d">OAuth2AuthorizationSuccessHandler</span>

```java
  @Override
    public void onAuthorizationSuccess(OAuth2AuthorizedClient authorizedClient, Authentication principal, Map<String, Object> attributes) {
            attributes.put("cao","cao");
    }
}
```



## OAuth2 Failure Handler

Samo implementiramo interfejs <span style="color:#ff4d4d">OAuth2AuthorizationFailureHandler</span>

## Authorization Code Flow



<span style="color:#ff4d4d">OAuth2AuthorizationRequestRedirectFilter</span> koristi  <span style="color:#ff4d4d">OAuth2AuthorizationRequestResolver</span> da resolvuje OAuth2AuthorizationRequest i init Authorization Code flow tako sto redirektuje end user-a na authorization server authorization endpoint

Primarna uloga *Auth2AuthorizationRequestResolverje* da resolvuje *OAuth2AuthorizationRequest* iz dolazeceg requesta.Default implementacija je <span style="color:#ff4d4d">DefaultOAuth2AuthorizationRequestResolver</span> koja radi tako sto macuje default put /oauth2/authorization/{registrationId},to je ono sto mi definisemo u yml file

<span style="color:#ff4d4d">AuthorizationCodeOAuth2AuthorizedClientProvider</span> je implementacija  *OAuth2AuthorizedClientProvider* koji pokrece Authorization Code grant

## Grant Types

### Authorization code

The Authorization Code grant type is used by confidential and public clients to exchange an authorization code for an access token.

After the user returns to the client via the redirect URL, the application will get the authorization code from the URL and use it to request an access token.



### Client  Credential

The Client Credentials grant type is used by clients to obtain an access token outside of the context of a user.

This is typically used by clients to access resources about themselves rather than to access a user's resources.