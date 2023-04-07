# Auth server

Prvo moramo da ubacimo dependency za ovaj server 

```xml
 <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-authorization-server</artifactId>
            <version>1.0.0</version>
 </dependency>
```



![image-20230130145333620](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230130145333620.png)

Da bi ga napravili moramo ovo da uradimo,sve ove komponente su nam neophodne da bi radio,i savku moramo da setujemo



Mi kada pravimo *Authorization server* na njemu se nalazi i <span style="color:#F24D37">authentication</span> i <span style="color:#F24D37">authorization</span>.

Tako da cemo imati biblioteku *Spring security* i *Authorization Server*.

### Authorization server

Ovde definisemo **JWK Source,RegisteredClientRepository,ProviderSettings,i njegov SecurityFilterChain**

```java
     @Bean
    @Order(1)
    public SecurityFilterChain asSecurityFilterChain(HttpSecurity http) throws Exception {

        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        //If user is not auth redirect it to login page
       http.exceptionHandling(e->e.authenticationEntryPoint(new 			  				LoginUrlAuthenticationEntryPoint("/login")));
        return http.formLogin().and().build();
    }

    //Define endpoints for oid and oauth2 endpoints,default now
    //Provider settings
    @Bean
    public AuthorizationServerSettings authorizationServerSettings() {
        return AuthorizationServerSettings.builder().build();
    }

    @Bean
    //Cela poenta Authorization servera je ako neko 3th party zatrazi neke personal info iz nase app da mu mi kao priredimo postupak kao sto ima google
    // Sa login with google,ovo registedClientRepository je u stvari skup user-a koji su registrovani na app i oni mogu da daju consent .Dosta je slicno
    //Kao user details service ali samo sto ne radi sa user-om nego sa client-om ima razlike ovde
    public RegisteredClientRepository registeredClientRepository(){
   RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
            //username
            .clientId("client")
            //password
            .clientSecret("secret")
        .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
        .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
       	//enable refresh tokens
        .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
        .redirectUri("http://127.0.0.1:3000/authorized")
        .scope(OidcScopes.OPENID)
        .clientSettings(ClientSettings.builder()
            .requireAuthorizationConsent(true)
                //now need PKCE
                //.requireProofKey(true)
                .build())
        .tokenSettings(TokenSettings.builder()
            .refreshTokenTimeToLive(Duration.ofHours(10))
            .build())
        .build();

    return new InMemoryRegisteredClientRepository(registeredClient);
    }

    //Key pair for signing tokens
    @Bean
    public JWKSource<SecurityContext> jwkSource() throws NoSuchAlgorithmException {
     //Key set je jer je ovo set pa moze da ima vise keypaira pa moze da ih rotira
        JWKSet set=new JWKSet(keyManager.rsaKey());
        return new ImmutableJWKSet<>(set);

    }
```

i dodamo za Spring Security autentikaciju:

```java
 @Bean
    @Order(2)
    public SecurityFilterChain appSecurityFilterChain(HttpSecurity http) throws Exception {
        return
                //login page on authorization server
                http.formLogin()
                .and()
                 .authorizeHttpRequests(x->x.requestMatchers("/error").permitAll())
                .authorizeHttpRequests().anyRequest().authenticated()
                .and().build();
    }


    @Bean
    public UserDetailsService userDetailsService(){
        var u= User.withUsername("admin")
                .password("12345")
                .authorities("read")
                .build();

        return new InMemoryUserDetailsManager(u);
    }
    @Bean
    public PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }

```

Kada ovo imamo mozemo da pristupimo OAuth2 rutama

### Default routes for Spring Authorization server:

Authorize -> http://localhost:8080/oauth2/authorize

Token -> http://localhost:8080/oauth2/token

I sada radimo isto kao da imamo OAuth2,u request posaljemo 

http://localhost:8080/oauth2/authorize?response_type=code&client_id=client&scope=openid&redirect_uri=http://test/auth&code_challenge=47DEQpj8HBSa-_TImW-5JCeuQeRkm5NMpJWZG3hSuFU&code_challenge_method=S256

scope,redirect_uri,response_type,client_id,code_challenge,code_challenge_method

code_challenge -> generisemo preko [Online PKCE Generator Tool (tonyxu-io.github.io)](https://tonyxu-io.github.io/pkce-generator/) ovoga.To je jedinstveni id koji garantuje autenticnost(Videti sta je PKCE u fajlu SecurityBasics)





## RegisteredClient

A `RegisteredClient` is a representation of a client that is [registered](https://datatracker.ietf.org/doc/html/rfc6749#section-2) with the authorization server. A client must be registered with the authorization server before it can initiate an authorization grant flow, such as `authorization_code` or `client_credentials`.

During client registration, the client is assigned a unique [client identifier](https://datatracker.ietf.org/doc/html/rfc6749#section-2.2), (optionally) a client secret (depending on [client type](https://datatracker.ietf.org/doc/html/rfc6749#section-2.1)), and metadata associated with its unique client identifier. The client’s metadata can range from human-facing display strings (such as client name) to items specific to a protocol flow (such as the list of valid redirect URIs).

The primary purpose of a client is to request access to protected resources. The client first requests an access token by authenticating with the authorization server and presenting the authorization grant. The authorization server authenticates the client and authorization grant, and, if they are valid, issues an access token. The client can now request the protected resource from the resource server by presenting the access token.

```java
public class RegisteredClient implements Serializable {
	private String id;  
	private String clientId;    
	private Instant clientIdIssuedAt;   
	private String clientSecret;    
	private Instant clientSecretExpiresAt;  
	private String clientName;  
	private Set<ClientAuthenticationMethod> clientAuthenticationMethods;    
	private Set<AuthorizationGrantType> authorizationGrantTypes;    
	private Set<String> redirectUris;   
	private Set<String> scopes; 
	private ClientSettings clientSettings;  
	private TokenSettings tokenSettings;    

	...

}
```

|      | `id`: The ID that uniquely identifies the `RegisteredClient`. |
| ---- | ------------------------------------------------------------ |
|      | `clientId`: The client identifier.                           |
|      | `clientIdIssuedAt`: The time at which the client identifier was issued. |
|      | `clientSecret`: The client’s secret. The value should be encoded using Spring Security’s [PasswordEncoder](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html#authentication-password-storage-dpe). |
|      | `clientSecretExpiresAt`: The time at which the client secret expires. |
|      | `clientName`: A descriptive name used for the client. The name may be used in certain scenarios, such as when displaying the client name in the consent page. |
|      | `clientAuthenticationMethods`: The authentication method(s) that the client may use. The supported values are `client_secret_basic`, `client_secret_post`, [`private_key_jwt`](https://datatracker.ietf.org/doc/html/rfc7523), `client_secret_jwt`, and `none` [(public clients)](https://datatracker.ietf.org/doc/html/rfc7636). |
|      | `authorizationGrantTypes`: The [authorization grant type(s)](https://datatracker.ietf.org/doc/html/rfc6749#section-1.3) that the client can use. The supported values are `authorization_code`, `client_credentials`, and `refresh_token`. |
|      | `redirectUris`: The registered [redirect URI(s)](https://datatracker.ietf.org/doc/html/rfc6749#section-3.1.2) that the client may use in redirect-based flows – for example, `authorization_code` grant. |
|      | `scopes`: The scope(s) that the client is allowed to request. |
|      | `clientSettings`: The custom settings for the client – for example, require [PKCE](https://datatracker.ietf.org/doc/html/rfc7636), require authorization consent, and others. |
|      | `tokenSettings`: The custom settings for the OAuth2 tokens issued to the client – for example, access/refresh token time-to-live, reuse refresh tokens, and others. |

## Registered Client Repository

Mozemo samo da napravimo custom tako sto cemo da impl RegisteredClientRepository,postoje vec neke koje su nam provajdovane:

The `RegisteredClientRepository` is the central component where new clients can be registered and existing clients can be queried. 

The provided implementations of <span style="color:#F24D37">`RegisteredClientRepository` </span> are <span style="color:#F24D37">`InMemoryRegisteredClientRepository`</span> and <span style="color:#F24D37">`JdbcRegisteredClientRepository`</span>

### InMemory

```java
@Bean public RegisteredClientRepository registeredClientRepository() { List<RegisteredClient> registrations = ... 

return new InMemoryRegisteredClientRepository(registrations);
}
```

ili

```java
@Bean
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
	OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
		new OAuth2AuthorizationServerConfigurer();
	http.apply(authorizationServerConfigurer);

	authorizationServerConfigurer
		.registeredClientRepository(registeredClientRepository);

	...

	return http.build();
}
```

The `OAuth2AuthorizationServerConfigurer` is useful when applying multiple configuration options simultaneously.



## OAuth2AuthorizationServerConfigurer 

```java
OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
	new OAuth2AuthorizationServerConfigurer();
http.apply(authorizationServerConfigurer);
```

We need to apply configurer to SecurityFilterChain(AuthServer SecurityFilterChain)



### OAuth2AuthorizationEndpointConfigurer

`OAuth2AuthorizationEndpointConfigurer` provides the ability to customize the [OAuth2 Authorization endpoint](https://datatracker.ietf.org/doc/html/rfc6749#section-3.1). It defines extension points that let you customize the pre-processing, main processing, and post-processing logic for [OAuth2 authorization requests](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1).

```java
@Bean
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
    //Set Configurer
	OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
		new OAuth2AuthorizationServerConfigurer();
	http.apply(authorizationServerConfigurer);

	authorizationServerConfigurer
		.authorizationEndpoint(authorizationEndpoint ->
			authorizationEndpoint
                 //1
				.authorizationRequestConverter(authorizationRequestConverter)   
		.authorizationRequestConverters(authorizationRequestConvertersConsumer) 
                   //2
				.authenticationProvider(authenticationProvider) 
				.authenticationProviders(authenticationProvidersConsumer)   
                   //3
				.authorizationResponseHandler(authorizationResponseHandler) 
                   //4
				.errorResponseHandler(errorResponseHandler) 
                   //5
				.consentPage("/oauth2/v1/authorize")    
		);

	return http.build();
}
```

1. `authorizationRequestConverter()`: Adds an `AuthenticationConverter` (*pre-processor*) used when attempting to extract an [OAuth2 authorization request](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1) (or consent) from `HttpServletRequest` to an instance of `OAuth2AuthorizationCodeRequestAuthenticationToken` or `OAuth2AuthorizationConsentAuthenticationToken`.
2. `authenticationProvider()`: Adds an `AuthenticationProvider` (*main processor*) used for authenticating the `OAuth2AuthorizationCodeRequestAuthenticationToken` or `OAuth2AuthorizationConsentAuthenticationToken`.
3. `authorizationResponseHandler()`: The `AuthenticationSuccessHandler` (*post-processor*) used for handling an “authenticated” `OAuth2AuthorizationCodeRequestAuthenticationToken` and returning the [OAuth2AuthorizationResponse](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2).
4. `errorResponseHandler()`: The `AuthenticationFailureHandler` (*post-processor*) used for handling an `OAuth2AuthorizationCodeRequestAuthenticationException` and returning the [OAuth2Error response](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2.1).
5. `consentPage()`: The `URI` of the custom consent page to redirect resource owners to if consent is required during the authorization request flow