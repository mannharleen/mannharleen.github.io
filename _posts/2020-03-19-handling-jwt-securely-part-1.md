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
- [Handling JWT securely on your client - Part-1](https://mannharleen.github.io/2020-03-19-handling-jwt-securely-part-1) [THIS POST]
- [Handling JWT securely on your client - Part-2](https://mannharleen.github.io/2020-04-10-handling-jwt-securely-part-2)
- [Handling JWT securely on your client - Part-3](https://mannharleen.github.io/) - not yet published
- [Handling JWT securely on your client - Part-4](https://mannharleen.github.io/) - not yet published

### Summary
* Part-1 covers the main problem statement around jwt security in web-apps; presents a few options and evaluates them 
* Part-2 dives deep into overcoming limitations around the chosen option in Part-1 e.g. SSO, Silent Authentication/Refresh, etc.
* Part-3 talks about non web-apps i.e. backend rest clients that don't run on web browsers e.g. postman
* Part-4 talks about other values added flows such as jwt expiry, force logout etc.

# Handling JWT securely on your client - Part-1

#### Assumptions
- We will focus on Single Page Application (SPA) web-apps here
- jwt here are treated as access tokens or sessions tokens
- Our Use case:
    - Our website is a SPA and hosted on app1.com
    - The SPA allows login and logout at /login and /logout
    - After logging in the SPA displays a button called "getData"
    - Once the button is pressed, the client-side-js sends a request to /api/data to obtain some data

#### Prerequisites
Basic knowledge around
- what is an SPA 
- what is a JWT
- what is XHR or ajax or fetch
- what is a XSS attack
- what is a CSRF attack

## JWT in web browsers

> The discussion below is equally applicable to server side rendered (SSR) websites, however I decided to cover only SPAs to keep it consistent

* How apps typically use JWT
    * JWT is a token, just like a session token/cookie. If present (and valid) on the client side browser, it signifies that a user may be logged in. I say "may be" because as we will see that the presence of jwt is not enough. 
    * SPAs usually make use of the jwt to render unauthenticated or authenticated HTML elements. e.g. Absence of JWT would trigger the SPA to render a login (and/or a sign-up) box
    * SPAs also use the jwt to send authenticated XHR/ajax/fetch requests to server apis


The pseudo-code flow when the user opens the app in the browser:
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

[user_opens_app1]

"...user_opens_app1"
[clientjs checks if jwt is present "somewhere??", if true]
    // do nothing, [clientjs can use jwt to access resources]
    
[else]        
    [...ask_user_to_login]


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
[server sends 200 response with jwt "somehow??"]
[clientjs stores jwt "somewhere??"]
[clientjs] shows the getData button
```

Now that the JWT is available, the user can click the button "getData" and invoke the following flow:
```rust
//pseudo-code

[button_press_get_data]

"...button_press_get_data"
[clientjs checks if jwt is present "somewhere??", if true]
    [clientjs sends GET /api/data (along with the jwt stored "somewhere??")]
    [server checks jwt, if valid]
        [server sends 200 with some data]
    [else]
        [server sends 400]
        [...ask_user_to_login]
    
[else]        
    [...ask_user_to_login]        

```

This all looks great!
However, if you noticed the pseudo-code, I marked two issues 
- the "somehow??" 
- and the somewhere??"

They both are two sides of the same problem "Storing jwt on browsers". So, lets look at some options to solve our problem

## Problem: Storing jwt on browsers

We will progressively evaluate 4 options here:
1. localStorage
2. sessionStorage
3. cookies
4. in-memory

### Option 1: localStorage
Storing a jwt in localStorage is prone to XSS attack since localStorage is available to javascript running on the same domain

### Option 2: sessionStorage
Storing a jwt in sessionStorage has the same issue, i.e. prone to XSS attack. Besides any data in sessionStorage is erased when a tab or a window is closed, so the user will have to login everytime he closes and reopens (all) tabs or the browser window

### Option 3: cookies
Upon receiving a login request on /login and validating the credentials, the server instead of sending the jwt in the body would send the jwt as a cookie (Set-Cookie)

Issues with this approach and mitigating it: 

| Vulnerability | Brief | Mitigation |
|---------------|------------|--------|
| XSS   | The client side js can read cookies | HttpOnly cookie |
| CSRF  | Cookies are sent to the attacker    | CORS polciy, X-CSRF-TOKEN, SameSite cookie |

> If you are keen to read more, I wrote a brief about how to effectively mitigate CSRF [here](https://mannharleen.github.io)

The cookie approach is thus far the most secure.
However, it has at least two drawbacks
- SameSite cookie is sort of a new concept and isn't supported in older browser version and in some newer ones too. See [here](https://caniuse.com/#feat=same-site-cookie-attribute)
- Your auth server /login and your /api must be hosted on the same domain

The implication is that if the app is served on an unsupported browser, SameSite won't work - CSRF is still possible and now depends on how strongly the other mitigation strategies have been implemented

### Option 4: in-memory
In-memory is definitely the most secure!

However, memory is not shared between processes (browser tabs and windows), so we are left two limitations:
1. Limitation 1: Hampers the ability to implement SSO
2. Limitation 2: Hampers user experience by forcing him/her to login on every tab/window (and everytime a browser is reopened)


#### Overcoming these limitations
As you will see the Part-2 of this series, we can overcome these limitations quite easily. See you in Part-2

Hint: refresh_tokens
