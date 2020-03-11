---
layout: post
title: Types of tokens in oidc and oauth 
subtitle: Understanding id_token, access_token and refresh_token
# gh-repo: mannharleen/log
# gh-badge: [star, fork, follow]
tags: [oauth, oidc, tokens, authentication, authorization]
comments: true
---
 ## OIDC, oAuth2.0, JWT, RS256, HS256, Tokens - don't let the jargons be a barrier!

# Key points
## id_token
- An id_token is a JWT - make note of that! 
- It contains claims about the identity of the user/resource owner
- Having a valid id_token means that the user is authenticated

## access_token
- An access_token is a bearer token
- A bearer token means that the bearer can access the resource without further identification
- An access_token can be a JWT (see Appendix point 1.) or opaque

## refresh_token
- Are long lived opaque tokens used to obtain new access_token

# Details 
id_token is typically used for:
- Stateless sessions: The presence of a valid id_token (jwt) provides information about the session. e.g. For browsers, cookies can store id_token 
- User information: The client app receives claims about the user as part of the id_token
- Token exchange: The ID token may be exchanged for an access token at the token endpoint of an OAuth 2.0 authorization server. (see Appendix 3.)

access_token:
- Has one and only one use, i.e. to establish which resource the bearer has access to
- The token is meant to be opaque to the client application
- But for the resource server, the access_token maps to the scope or claims that the bearer has
- If this is a jwt, it may contain scope as part of the token; which the resource server can verify on the fly (without introspection)
> If you are looking for a practical example on how jwt are used as access_tokens, I wrote an authentication-authorization piece for graphql here: https://github.com/mannharleen/graphql-auth-implementation

## The story of scopes & claims
Remember: Claims are grouped under Scope. e.g. claims 'email', 'name', 'age', etc. are part of the scope called 'profile'

### scope
A scope is a space separated list of identifiers that specify what access privileges are being requested by the client application

### claims
A claim is a key-value pair that contain some information, e.g. user information

## Appendix
1. JWT as access_token: this is in practice by providers like Auth0 & Okta, but the specification is still in draft stage, see: https://tools.ietf.org/html/draft-ietf-oauth-access-token-jwt-00
2. If you are eager to read more, checkout the following literature:
- OpenID Connect Core 1.0: https://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation
- The OAuth 2.0 Authorization Framework: https://tools.ietf.org/html/rfc6749
3. OAuth 2.0 Token Exchange: https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-12
