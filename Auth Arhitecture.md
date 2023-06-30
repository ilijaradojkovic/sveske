# Auth Arhitecture

![image-20230201141207587](C:\Program Files\Typora\resources\Docs\img\image-20230201141207587.png)

## Monolith approach

Vidimo da monolith app radi jednostavan *Basic Authentication*  gde ce dobiti cookie\(Session) preko koga ce moci da pristupi

One key part of the security architecture is the session, which stores the principal’s ID and roles. The FTGO application is a traditional Java EE application, so the session is an HttpSession in-memory session. A session is identified by a session token, which the client includes in each request. It’s usually an opaque token such as a cryptographically strong random number. The FTGO application’s session token is an HTTP cookie called JSESSIONID

The other key part of the security implementation is the security context, which stores information about the user making the current request. The Spring Security framework uses the standard Java EE approach of storing the security context in a static, thread-local variable, which is readily accessible to any code that’s invoked to handle the request. A request handler can call SecurityContextHolder.getContext() .getAuthentication() to obtain information about the current user, such as their identity and roles

## Microservice

 A microservice architecture is a distributed architecture. Each external request is handled by the API gateway and at least one service. Consider, for example, the getOrderDetails() query.The API gateway handles this query by invoking several services, including Order Service, Kitchen Service, and Accounting Service. Each service must implement some aspects of security. For instance, Order Service must only allow a consumer to see their orders, which requires a combination of authentication and authorization. In order to implement security in a microservice architecture we need to determine who is responsible for authenticating the user and who is responsible for authorization

 One challenge with implementing security in a microservices application is that we can’t just copy the design from a monolithic application.

1. In-memory security context—Using an in-memory security context, such as a threadlocal, to pass around user identity. Services can’t share memory, so they can’t use an in-memory security context, such as a thread-local, to pass around the user identity. In a microservice architecture, we need a different mechanism for passing user identity from one service to another
2. Centralized session —Because an in-memory security context doesn’t make sense, neither does an in-memory session. In theory, multiple services could access a database-based session, except that it would violate the principle of loose coupling. We need a different session mechanism in a microservice architecture



 There are a couple of different ways to handle authentication. One option is for the **individual services to authenticate the user**. The problem with this approach is that it permits unauthenticated requests to enter the internal network. It relies on every development team correctly implementing security in all of their services. As a result, there’s a significant risk of an application containing security vulnerabilities.

Another problem with implementing authentication in the services is that different clients authenticate in different ways. Pure API clients supply credentials with each request using, for example, basic authentication. Other clients might first log in and then supply a session token with each request. We want to avoid requiring services to handle a diverse set of authentication mechanisms.

 A better approach is for the API gateway to authenticate a request before forwarding it to the services. Centralizing API authentication in the API gateway has the advantage that there’s only one place to get right. As a result, there’s a much smaller chance of a security vulnerability. Another benefit is that only the API gateway has to deal with the various different authentication mechanisms. It hides this complexity from the services

 Figure 11.3 shows how this approach works. Clients authenticate with the API gateway. API clients include credentials in each request. Login-based clients POST the user’s credentials to the API gateway’s authentication and receive a session token. Once the API gateway has authenticated a request, it invokes one or more services.

**Pattern: Access token** The API gateway passes a token containing information about the user, such as their identity and their roles, to the services that it invokes. 

A service invoked by the API gateway needs to know the principal making the request. It must also verify that the request has been authenticated. The solution is for the API gateway to include a token in each service request. The service uses the token to validate the request and obtain information about the principal. The API gateway might also give the same token to session-oriented clients to use as the session token.

![image-20230201151218756](C:\Program Files\Typora\resources\Docs\img\image-20230201151218756.png)

The sequence of events for login-based clients is as follows:

 1 A client makes a login request containing credentials.

 2 The API gateway returns a security token. 

3 The client includes the security token in requests that invoke operations. 

4 The API gateway validates the security token and forwards it to the service or services

One place to implement authorization is the API gateway. It can, for example, restrict access to GET /orders/{orderId} to only users who are consumers and customer service agents. If a user isn’t allowed to access a particular path, the API gateway can reject the request before forwarding it on to the service. As with authentication, centralizing authorization within the API gateway reduces the risk of security vulnerabilities. You can implement authorization in the API gateway using a security framework, such as Spring Security

One drawback of implementing authorization in the API gateway is that it risks coupling the API gateway to the services, requiring them to be updated in lockstep.To znaci da kada bi dodali user-a morali bi da saljemo sa gateway-ja update objekat ka ostalim mikroservisima kojima treba update info da se neko register

 *The other place to implement authorization is in the services. A service can implement role-based authorization for URLs.

When implementing security in a microservice architecture, you need to decide which type of token an API gateway should use to pass user information to the services. There are two types of tokens to choose from. One option is to use opaque tokens, which are typically UUIDs. The downside of opaque tokens is that they reduce performance and availability and increase latency. That’s because the recipient of such a token must make a synchronous RPC call to a security service to validate the token and retrieve the user information

An alternative approach, which eliminates the call to the security service, is to use a transparent token containing information about the user. One such popular standard for transparent tokens is the JSON Web Token (JWT). JWT is standard way to securely represent claims, such as user identity and roles, between two parties. A JWT has a payload, which is a JSON object that contains information about the user, such as their identity and roles, and other metadata, such as an expiration date. It’s signed with a secret that’s only known to the creator of the JWT, such as the API gateway and the recipient of the JWT, such as a service. The secret ensures that a malicious third party can’t forge or tamper with a JWT

 One issue with JWT is that because a token is self-contained, it’s irrevocable. By design, a service will perform the request operation after verifying the JWT’s signature and expiration date. As a result, there’s no practical way to revoke an individual JWT that has fallen into the hands of a malicious third party. The solution is to issue JWTs with short expiration times, because that limits what a malicious party could do. One drawback of short-lived JWTs, though, is that the application must somehow continually reissue JWTs to keep the session active. Fortunately, this is one of the many protocols that are solved by a security standard calling OAuth 2.0. Let’s look at how that works. 

OAuth 2.0

Although the original focus of OAuth 2.0 was authorizing access to public cloud services, you can also use it for authentication and authorization in your application.



![image-20230201152501137](C:\Program Files\Typora\resources\Docs\img\image-20230201152501137.png)

An OAuth 2.0-based API gateway can authenticate session-oriented clients by using an OAuth 2.0 access token as a session token. What’s more, when the access token expires, it can obtain a new access token using the refresh token. Figure 11.5 shows how an API gateway can use OAuth 2.0 to handle session-oriented clients. An API client initiates a session by POSTing its credentials to the API gateway’s /login endpoint. The API gateway returns an access token and a refresh token to the client. The API client then supplies both tokens when it makes requests to the API gateway

There are several different protocols on which SSO can be based, such as OAuth, OpenID Connect, SAML, and Json Web Token.

There are two possible ways for a microservice to recognize that the Access Token received is actually from the Auth Microservice and not a malicious impostor (and/or that the payload has not been modified). The two ways are by a **shared secret key** or by a **public-private key pair**.

A shared secret key is used when the Auth Microservice uses **symmetric** cryptography to sign the payload. Another microservice would need to know the same secret key (hence a shared secret) in order to verify if the payload is true.

A public-private key pair is used when the Auth Microservice uses asymmetric cryptography to sign the payload, that is, it still requires a private key to sign it, but the verification can be done with just the public key, which as its name implies, it can be publicly distributed to the world without compromising the private key.

It is safer to keep the signing key (private key) in the Auth Microservice and only share the public key, no matter how much you trust the other microservices. That's why this project uses **asymmetric** cryptography.

1)Za svaki request se salje preko webclienta zahtev da validira auth server token.Ovo je naporno

![image-20230213155649713](C:\Program Files\Typora\resources\Docs\img\image-20230213155649713.png)

2)Zajednicka baza (blackboarding)



![image-20230213155714485](C:\Program Files\Typora\resources\Docs\img\image-20230213155714485.png)







![image-20230213155739135](C:\Program Files\Typora\resources\Docs\img\image-20230213155739135.png)

Ovo postizemo tako sto na gateway-u stavimo oauth2 client i oauth2 resource server kao i da disable session.

## Spring Gateway as OAuth 2.0 Resource Server

Ako ga stavimo da bude Resource server on ce samo da proverava validnost tokena i da ga forwarduje dalje na mikroservise

Kada imamo ovo mi moramo nekako da uzmemo token  sa keycloak-a.Najlakse je da se uradi `password grant flow`na keycloak gde cemo mi da posaljemo username i password i dobiti tokene

##  Spring Gateway as OAuth 2.0 Client

To znaci da ce on izvrsavati zahtev za OAuth2 token,a ne mi(front) to znaci da mozemo da koristimo drugi grant type koji je bezbedniji

Ovde cemo imati samo sesiju koja se uspostavlja izmedju gateway-a i user-a.Ovo mora ovako ,gateway ce da storuje nekako token sa id sesije .Nikada ne saljemo id token nazad jer je losa praksa vec imamo tu sesiju izmejdu user-a i gateway-a.Pa ako zapocne 3 requesta i pomocu prvgo se login gateay dobija od auth servera tokene i cuva ih interno,i uspostavlja sesiju sa user-om,tako da kada on open nesto hoce on ce da proveri validnost tokena.

1. User Login: The user logs in using the authentication server, providing their credentials.
2. Token Generation: Upon successful authentication, the authentication server generates an access token and possibly an ID token.
3. Gateway Session Initialization: The gateway establishes a session with the user, associating the session with the authenticated user's identity. This session can be stored in memory, a distributed cache, or a database.
4. Token Storage: The gateway retains the received token (e.g., access token or ID token) within the session or in a separate storage mechanism. This allows the gateway to validate the token and extract user information for subsequent requests.
5. Subsequent Requests: When the user makes subsequent requests to the gateway, the gateway checks for the presence of the token associated with the user's session.
6. Token Validation: The gateway validates the token to ensure its authenticity and integrity. It verifies the token's signature, expiration, and other relevant claims to ensure its validity.
7. User Identification and Authorization: After successful token validation, the gateway uses the token to identify the user associated with the session. It can extract user information from the token and perform any necessary authorization checks based on the user's identity and associated permissions.
8. Request Forwarding: If the user is authorized, the gateway forwards the request to the appropriate downstream service on behalf of the user.

Znaci sada necemo moci da bas lepo testiramo u postamnu jer ce biti sesija,to resavamo tako sto na postmanu radimo OAuth Authentication Flow direktno na auth server,ono ubacimo client id ,client secret...