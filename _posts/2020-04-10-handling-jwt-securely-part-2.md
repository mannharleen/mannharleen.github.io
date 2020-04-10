---
layout: post
title: Handling JWT securely on your client
subtitle: To the point
# gh-repo: mannharleen/log
# gh-badge: [star, fork, follow]
tags: [jwt, security, cookie, xss, csrf, token]
comments: true
---
# Handling JWT securely on your client
This is a series of posts divided into the following parts:
- [Handling JWT securely on your client - Part-1](https://mannharleen.github.io/2020-03-19-handling-jwt-securely-part-1)
- [Handling JWT securely on your client - Part-2](https://mannharleen.github.io/2020-04-10-handling-jwt-securely-part-2) [THIS POST]
- [Handling JWT securely on your client - Part-3](https://mannharleen.github.io/) - not yet published
- [Handling JWT securely on your client - Part-4](https://mannharleen.github.io/) - not yet published

### Summary
* Part-1 covers the main problem statement around jwt security in web-apps; presents a few options and evaluates them 
* Part-2 dives deep into overcoming limitations around the chosen option in Part-1 e.g. SSO, Silent Authentication/Refresh, etc.
* Part-3 talks about non web-apps i.e. backend rest clients that don't run on web browsers e.g. postman
* Part-4 talks about other values added flows such as jwt expiry, force logout etc.

# Handling JWT securely on your client - Part-2

## Quick recap
In the part 1 of this blog series, we established that in-memory storage is most secure way of storing jwt.

We also established that this approach brings about two limitations:
1. Limitation 1: SSO: Hampers the ability to implement SSO
2. Limitation 2: Session: Hampers user experience by forcing him/her to login on every tab/window
A subset of Limitation 2 is that the user is forced to login when they close their browser and reopen it

## More on the limitations 

### Limitation 1: SSO
Limitation 1 can only be solved with some sort of state management on the Authorization server. The server needs to retain information on all users who have logged in. 

**But this goes directly against the principles of using jwt, which are meant to be stateless.**

### Limitation 2: Session
Limitation 2 can only be solved with some sort of state management on the client browser side. The client needs to persist some information that is available across browser tabs and also across browser restarts. 

**But if you are thinking of persisting jwt (through localStorage or cookies), go to PART-1 of this series! I repeat - we will keep the jwt in-memory only**

## Overcoming these limitations
To solve both the limitations, we need to persist "something??" "somewhere??" on both the server and client-browser side.

Let's tackle the "somewhere??" first:
- On the server side, it may be anywhere secure e.g. a distributed cache or a database
- On the client side, as we saw in PART-1, the second most secure storage after memory is a Cookie. So let's store this information in a cookie

> Q. Doesn't using a cookie expose a risk of CSRF exploit?

> A. Read on....

What about the "something??":
- Both on the server and the client-browser side, we store an opaque token. This token is called **refresh_token**
- Additionally, on the server side we keep a map of refresh_token and the corresponding jwt

## refresh_token
The **refresh_token** is opaque, meaning that it does not give away any information to an attacker who gets hold of it.
The client-browser in possession of a refresh_token can send it to the server to obtain jwt (and a new refresh_token)

### refresh_token stored as a cookie is secure (jwt as a cookie is not)
As we saw in PART-1, any cookie is vulnerable to CSRF exploit. However, a refresh_token in itself cannot be used to POST data to the server. It can only be used to obtain jwt. Hence, refresh_token as a cookie is not vulnerable to CSRF

In other words, if one doesn't want to or cannot implement SameSite cookie for refresh_token, it is still safe to use; The attacker may be able to send the refresh_token cookie to the server but the endpoint /refresh_token only allows a GET request. The attacker cannot get his hands on the response that contains the jwt - the response flows directly to the client browser.

## Implementing refresh_token 

### Server side API changes
- We introduce a new server endpoint "/refresh_token" that accepts a GET to exchange an existing refresh_token (cookie) for a jwt and a new refresh_token (Set-Cookie)
- Our existing endpoint at "/login" that allows POST now also sets the refresh_token cookie in the response

### Client side js changes
- The client-js must now implements the "Silent Login" feature, such that in the presence of a refresh_token it should attempt to obtain a jwt if not present in-memory, and that the attempt should be in the background without the user having to do anything.

## Illustration
Lets assume there are two apps named "app1.com" and "app2.com". Both apps are configured to use app1.com as the authorization server; in a real life example it could be a separate auth server e.g. Auth0
- In simpler terms, they both use app1.com/login and app1.com/refresh_token
- In terms of CORS both app1.com and app2.com are allowed origins

With the refresh_token implemented, the pseudo-code flow becomes:

```rust
/*  terms used
    user = a person interacting with the browser
    app = a SPA application that contains the client side js and the html
    clientjs = client side js
    browser = client side browser engine
    server = stateless service side api (think serverless compute like lambda)
*/
/* legend
    "...abcd"   = a fragment that describes the flow, like a function definition
    [....abcd]  = calls the fragment, like a function call
*/
/*  
    pseudo-code
*/

[...user_opens_app1_or_app2]

"...user_opens_app1_or_app2"
[clientjs checks if jwt is present in memory, if true]
    // do nothing, [clientjs can use jwt to access resources]
    
[else]        
    [clientjs sends a GET request to /refresh_token]
    [browser attaches refresh_token cookie if available] 
        [server checks if refresh_token cookie is absent, if true]
            [send a 400 response] 
            [...ask_user_to_login]

        [server check if refresh_token cookie has expired, if true]
            [send a 400 response] 
            [...ask_user_to_login]

        [server check if refresh_token cookie has been marked invalid, if true] (more on this later when we discuss /logout)
            [send a 400 response] 
            [...ask_user_to_login]

        [else] 
            [...login_success]

"...ask_user_to_login"
[clientjs then asks user to login]
    [user enters credentials]
    [clientjs sends a POST /login]
        [server checks credentials, if valid]
            [...login_success]          
        [else]
            [server sends 400]
            [...ask_user_to_login]

"...login_success"
[server sends 200 response with jwt (in body) and a new refresh_token (as a cookie)]
[server inserts refresh_token into it's stateStore]
[clientjs stores jwt in memory]
```
