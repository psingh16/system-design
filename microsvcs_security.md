# MicroSvcs\_Security

* [Security](microsvcs_security.md#security)
  * [Authentication](microsvcs_security.md#authentication)
    * [Http basic](microsvcs_security.md#http-basic)
    * [Token based - Session cookie](microsvcs_security.md#token-based---session-cookie)
      * [Cookie security attributes](microsvcs_security.md#cookie-security-attributes)
      * [Session fixation attacks](microsvcs_security.md#session-fixation-attacks)
      * [CSRF attacks](microsvcs_security.md#csrf-attacks)
        * [Solution 1: Samesite cookies](microsvcs_security.md#solution-1-samesite-cookies)
        * [Solution 2: Double-Submit cookie](microsvcs_security.md#solution-2-double-submit-cookie)
      * [Allow cross origin requests](microsvcs_security.md#allow-cross-origin-requests)
      * [Pros](microsvcs_security.md#pros)
      * [Cons](microsvcs_security.md#cons)
    * [Database token based](microsvcs_security.md#database-token-based)
      * [Replace cookie](microsvcs_security.md#replace-cookie)
      * [Def](microsvcs_security.md#def)
      * [Web storage XSS attack](microsvcs_security.md#web-storage-xss-attack)
      * [Harden the database token store](microsvcs_security.md#harden-the-database-token-store)
    * [Self contained tokens based](microsvcs_security.md#self-contained-tokens-based)
      * [ID token claims](microsvcs_security.md#id-token-claims)
      * [Access token](microsvcs_security.md#access-token)
      * [JWT Format](microsvcs_security.md#jwt-format)
        * [JOSE header](microsvcs_security.md#jose-header)
      * [JWS signing algorithms](microsvcs_security.md#jws-signing-algorithms)
        * [JWK header](microsvcs_security.md#jwk-header)
        * [Claims](microsvcs_security.md#claims)
      * [Types of JWT](microsvcs_security.md#types-of-jwt)
  * [Authorization with OAuth2](microsvcs_security.md#authorization-with-oauth2)
    * [Simple guide](microsvcs_security.md#simple-guide)
    * [Roles](microsvcs_security.md#roles)
    * [OAuth 2 modes](microsvcs_security.md#oauth-2-modes)
  * [Auth architecture revolution](microsvcs_security.md#auth-architecture-revolution)
    * [Single server cookie based auth](microsvcs_security.md#single-server-cookie-based-auth)
    * [Multi server sticky session based auth](microsvcs_security.md#multi-server-sticky-session-based-auth)
    * [Auth Service and token](microsvcs_security.md#auth-service-and-token)
    * [Gateway and token](microsvcs_security.md#gateway-and-token)
    * [Gateway and JWT](microsvcs_security.md#gateway-and-jwt)
  * [Microservices security architecture](microsvcs_security.md#microservices-security-architecture)
    * [External access token internal JWT token](microsvcs_security.md#external-access-token-internal-jwt-token)
    * [Encrypted JWT token](microsvcs_security.md#encrypted-jwt-token)
    * [External access token internal JWT token with token cache](microsvcs_security.md#external-access-token-internal-jwt-token-with-token-cache)
  * [Real world examples](microsvcs_security.md#real-world-examples)
    * [Auth at Netflix](microsvcs_security.md#auth-at-netflix)

## Security

### Authentication

#### Http basic

* Overflow chart

![](.gitbook/assets/microSvcs_security_basicAuth.png)

* Cons:
  * User password being passed along in every request. 
  * Verifying a password is an expensive operation. Modern password hashing algorithm is designed to take around 100ms per operation. 

#### Token based - Session cookie

* Session cookie is the simplest and most widespread token issuing mechanism. 

![](.gitbook/assets/microSvcs_security_tokenauth.png)

![](.gitbook/assets/microSvcs_security_sessionCookie.png)

**Cookie security attributes**

![](.gitbook/assets/microSvcs_security_cookieattributes_one.png)

![](.gitbook/assets/microSvcs_security_cookieattributes_two.png)

**Session fixation attacks**

![](.gitbook/assets/microSvcs_security_sessionfixationAttack.png)

**CSRF attacks**

* Cross-site request forgery \(CSRF, pronounced “sea-surf”\) occurs when an attacker makes a cross-origin request to your API and the browser sends cookies along with the request. The request is processed as if it was genuine unless extra checks are made to prevent these requests. 
* Impacts: The malicious site could create fake requests to your API that appear to come from a genuine client. 

![](.gitbook/assets/microsvcs_security_csrf.png)

**Solution 1: Samesite cookies**

![](.gitbook/assets/microSvcs_security_samesite.png)

**Solution 2: Double-Submit cookie**

![](.gitbook/assets/microSvcs_security_doublehash.png)

![](.gitbook/assets/microSvcs_security_doublehash_two.png)

**Allow cross origin requests**

![](.gitbook/assets/microsvcs_security_cors.png)

![](.gitbook/assets/microsvcs_security_cors_headers.png)

**Pros**

* Using cookies in authentication makes your application stateful. This will be efficient in tracking and personalizing the state of a user.
* Cookies are small in size thus making them efficient to store on the client-side.
* Cookies can be “HTTP-only” making them impossible to read on the client-side. This improves protection against any Cross-site scripting \(XSS\) attacks.
* Cookies will be added to the request automatically, so the developer will not have to implement them manually and therefore requires less code.

**Cons**

* It is vulnerable to Cross-site request forgery attack. It often needs other security measures such as CSRF tokens for protection.
* You need to store the session data in a database or keep it in memory on the server. This makes it less scalable and increases overhead when the site is having many users.
* Cookies normally work on a single domain. For example, it is impossible to read or send a cookie from a domain like jabs.com to a boo.com domain. This is an issue when the API service is from different domains for mobile and web platforms.
* The client needs to send a cookie on every request, even with the URLs that do not need authentication for access.

#### Database token based

**Replace cookie**

* [https://www.section.io/engineering-education/cookie-vs-token-authentication/](https://www.section.io/engineering-education/cookie-vs-token-authentication/)

![](.gitbook/assets/microsvcs_security_replace_cookie.png)

**Def**

* Bearer authentication scheme: [https://datatracker.ietf.org/doc/html/rfc6750\#page-7](https://datatracker.ietf.org/doc/html/rfc6750#page-7)

![](.gitbook/assets/microsvcs_security_bearer.png)

**Web storage XSS attack**

![](.gitbook/assets/microsvcs_security_xss.png)

![](.gitbook/assets/microsvcs_security_xss_two.png)

**Harden the database token store**

![](.gitbook/assets/apidesign_database_hardening.png)

![](.gitbook/assets/microsvcs_security_hmac_tag.png)

![](.gitbook/assets/microsvcs_security_authtag.png)

#### Self contained tokens based

* JWT is a type of by value token which does not need to be verified at the authorization server. 
  * Def for by reference token: Randomly generated string value. Upon receiving the token, resource server needs to verify it against OAuth authorization server to obtain information such as claims/scopes. 
  * Def for by value token: A token which contains key value pair of \(issuer, audience, scope, claims\). It could be verified locally and does not need to be verified against authorization server. 

**ID token claims**

![](.gitbook/assets/microsvcs_security_idtoken_claims.png)

**Access token**

* Used to access target resource.

**JWT Format**

![](.gitbook/assets/microsvcs_security_statelessJwt.png)

![](.gitbook/assets/microsvcs_security_jwt_format.png)

**JOSE header**

![](.gitbook/assets/microsvcs_security_jose_header.png)

**JWS signing algorithms**

![](.gitbook/assets/apidesign_jws_algorithm.png)

**JWK header**

![](.gitbook/assets/microsvcs_security_jwt_header.png)

![](.gitbook/assets/apidesign_example_jwk.png)

![](.gitbook/assets/apidesign_example_jwk_two.png)

**Claims**

![](.gitbook/assets/microsvcs_security_standardclaims.png)

**Types of JWT**

* HMAC JWT

![Gateway and token](.gitbook/assets/security_gateway_jwt_hmac.png)

* RSA JWT

![Gateway and token](.gitbook/assets/security_gateway_jwt_rsa.png)

### Authorization with OAuth2

![](.gitbook/assets/apidesign_permissionVsScope.png)

![](.gitbook/assets/apidesign_accesstoken.png)

#### Simple guide

* Simlplest guide: [https://medium.com/@darutk/the-simplest-guide-to-oauth-2-0-8c71bd9a15bb](https://medium.com/@darutk/the-simplest-guide-to-oauth-2-0-8c71bd9a15bb)

#### Roles

* Third party client app: it needs to access users' protected resource
* Resource server: a web server which expose users' protected resource to outside users
* Authorization server: issue access token to client app after resource owner grant the permission
* Resource owner: the owner of a resource who wants to share it with third party apps.

#### OAuth 2 modes

* Good reference: Ruanyifeng overview - [http://www.ruanyifeng.com/blog/2014/05/oauth\_2\_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
* Choose between OAuth modes: 

![Choose between](.gitbook/assets/security_oauth_chooseMode.svg)

![](.gitbook/assets/apidesign_jwt_accesstokens.png)

![](.gitbook/assets/microsvcs_security_public_key_crypto.png)

### Auth architecture revolution

#### Single server cookie based auth

![Auth version 1](.gitbook/assets/security_singlemachine-cookiebased.png)

#### Multi server sticky session based auth

* Cons:
  * Sticky session binds a session to a server. If the server goes down or needs to be maintained. 
  * Sticky session needs to store session data in load balancer. 
* Possible solutions:
  1. Session synchronization by replicating across web servers
  2. Store session data completely inside users' browser
     * Cons: Limited size of cookie
  3. Store session data in a shared storage

![Sticky session](.gitbook/assets/security_multimachine-stickysession.png)

#### Auth Service and token

* Pros:
  * Encapsulate everything related with token issuing
  * Introduce the concept of token, which could be passed around between services
* Cons:
  * Services need to implement the logic to validate the token. 
  * All services need to talk to authSvc, which might become a performance bottleneck.     
  * All requests need to be verified via auth service. 

![Auth service and token](.gitbook/assets/security_authservice_token.png)

#### Gateway and token

* Pros:
  * Gateway centralizes the logic of parsing userInfo. Only gateway need to validate the token with auth service. 
* Cons:
  * All requests need to be verified via auth service. Auth service needs to be maintained and scaled in a manageable way. 

![Gateway and token](.gitbook/assets/security_gateway_token.png)

#### Gateway and JWT

* Pros:
  * Compact and lightweight
  * Low pressure on Auth server
  * Simplify the implementation of auth server
* Cons:
  * Could not invalidate a JWT token if it has been leaked
  * JWT might become big

![Gateway and token](.gitbook/assets/security_gateway_jwt.png)

### Microservices security architecture

#### External access token internal JWT token

* Cons: Still rely on gateway to switch access token with JWT token. 

![first arch](.gitbook/assets/security_futureMicroservice_firstArch.png)

#### Encrypted JWT token

* Pros: Stateless token

![second arch](.gitbook/assets/security_futureMicroservice_secArch.png)

#### External access token internal JWT token with token cache

* Most widely used in practice

![third arch](.gitbook/assets/security_futureMicroservice_thirdArch.png)

### Real world examples

#### Auth at Netflix

* [https://netflixtechblog.com/edge-authentication-and-token-agnostic-identity-propagation-514e47e0b602](https://netflixtechblog.com/edge-authentication-and-token-agnostic-identity-propagation-514e47e0b602)
* A talk on InfoQ: [https://www.infoq.com/presentations/netflix-user-identity/](https://www.infoq.com/presentations/netflix-user-identity/)
* Access control at Netflix: [https://netflixtechblog.com/consoleme-a-central-control-plane-for-aws-permissions-and-access-fd09afdd60a8](https://netflixtechblog.com/consoleme-a-central-control-plane-for-aws-permissions-and-access-fd09afdd60a8)
* Netflix container security: [https://netflixtechblog.com/evolving-container-security-with-linux-user-namespaces-afbe3308c082](https://netflixtechblog.com/evolving-container-security-with-linux-user-namespaces-afbe3308c082)
* Netflix detect credential leak: [https://netflixtechblog.com/netflix-cloud-security-detecting-credential-compromise-in-aws-9493d6fd373a](https://netflixtechblog.com/netflix-cloud-security-detecting-credential-compromise-in-aws-9493d6fd373a)
* Netflix viewing privacy: [https://netflixtechblog.com/protecting-netflix-viewing-privacy-at-scale-39c675d88f45](https://netflixtechblog.com/protecting-netflix-viewing-privacy-at-scale-39c675d88f45)
