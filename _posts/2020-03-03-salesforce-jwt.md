---
layout: post
title: Salesforce OAuth 2.0 JWT Bearer flow
subtitle: To the point
# gh-repo: mannharleen/log
# gh-badge: [star, fork, follow]
tags: [salesforce, oauth, jwt, x509, pem]
comments: true
---

## This is a practical 'to the point' guide of using the Salesforce OAuth 2.0 JWT Bearer flow

The whole setup will be covered in the following steps:
- Step 1: Creating private key and X509 certificate
- Step 2: Creating connected app in Salesforce
- Step 3: One time oAuth 2.0 authorization flow
- Step 4: Let's create our JWT
- Step 5 Using JWT to obtain access_token from Salesforce
- Step 6: Using access_token to hit Salesforce APIs

---
## Step 1: Creating private key and X509 certificate

```bash
$ openssl genrsa -out privatekey.pem 1024
$ openssl req -new -x509 -key privatekey.pem -out publickey.cer -days 3650
..<just press enters>..
......
$ ls -lrt
// privatekey.pem
// publickey.cer
```
---
## Step 2: Creating connected app in Salesforce
Go to Settings > Build > Create > Apps > New (connected app)
- Fill in name, email etc.
- Tick 'Enable OAuth Settings'
- Fill Callback URL = 'https://oauthdebugger.com/debug' // we are using a great online debugger
- Tick 'Use digital signatures'
- Upload <publickey.cer from Step 1>
- Selected OAuth Scopes = api refresh_token offline_access // you can add more if you need to
- Save
- Note the client_id and client_Secret
---

## Step 3: One time oAuth 2.0 authorization flow
For some background on why we need this step, and what it means, refer to the Appendix at the bottom.

Open your browser
- Go to https://oauthdebugger.com/ // you could use any another oauth 2.0 client
- Authorization url = 'https://login.salesforce.com/services/oauth2/authorize'
- Redirect URI = 'https://oauthdebugger.com/debug'
- Scope = 'api refresh_token offline_access' // this is same as what we set on our connected app in salesforce
- Response type = 'code'
- Response mode = 'query'
- Click on "send request"

If all goes well, you will receive an AUTHORIZATION-CODE

- Over to postman (or you could use curl)
```ruby
POST 
https://login.salesforce.com/services/oauth2/token?grant_type=authorization_code&code=<PASTE AUTHORIZATION-CODE HERE>&client_id=<CLIENT_ID FROM STEP 2>&client_secret=<CLIENT_SECRET FROM STEP 2>&redirect_uri=https://oauthdebugger.com/debug
```

You really don't need to do anything with the response

---

## Step 4: Let's create our JWT
You could use a library in your fav landuage to perform this, but I am just using an online tool for this demo.

Head to https://jwt.io/ and se the following
- ALGORITHM = 'RS256'
- Decoded.Payload = 
```json
{
   "iss": "<CLIENT_ID FROM STEP 2>",
  "sub": "<USERNAME OF THE USER WHO YOU WANT TO LOG IN AS>",
  "aud": "https://login.salesforce.com", 
  "exp": "1583238254198"
}
// see below note on exp
// use test.salesforce.com for sandpits
```
> Note: The "exp" value must be withing the next 3 minutes

So, to generate such a time in Unix Epoch format, you could simple use your browser (or nodejs runtime, or python, or anything else)
```js
// generating a valid exp
//
// open console window in your browser (usually F12 -> console)
d = new Date(); d.setSeconds(d.getSeconds() + 180); d.valueOf()
// copy the output! thats your "exp"
```

- Decoded.VERIFY SIGNATURE = 
    - Copy & paste content of <publickey.cer from Step 1> in the first block
    - Copy & paste <privatekey.pem from Step 1> in the second block
- Copy the jwt from the left section under Encoded

---
## Step 5 Using JWT to obtain access_token from Salesforce

- Head to postman (or you could use curl)
```ruby
POST
https://login.salesforce.com/services/oauth2/token?grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer&
assertion=<JWT from Step 4>
```

The response would look like this:
```json
{
    "access_token": "...........",
    "scope": "api",
    "instance_url": "https://ap4.salesforce.com",
    "id": "https://login.salesforce.com/id/xxx/yyy",
    "token_type": "Bearer"
}
```

> This is it! We have the access_token for the "user"! A pat on your back.

---

## Step 6: Using access_token to hit Salesforce APIs
Using the access_token we can now work with Salesforce APIs

eg:
```sh
# run soql select Id, Name, Type From Account
curl -X GET \
  'https://ap4.salesforce.com/services/data/v42.0/query/?q=SELECT%20Id%2CName%2CType%20FROM%20Account' \
  -H 'authorization: Bearer <ACCESS_TOKEN from Step 5>' \
  -H 'cache-control: no-cache'
```

---

## Appendix
Step 3 is a tricky bit that took me a few iterations to figure out. Ref to the below snippet from https://help.salesforce.com/articleView?id=remoteaccess_oauth_jwt_flow.htm&type=5#grants_access
```
The OAuth 2.0 JWT bearer and SAML assertion bearer flow requests look at all previous approvals for the user that include a refresh token. If Salesforce finds matching approvals, it combines the values of the approved scopes. Salesforce then issues an access token. If Salesforce doesn’t find previous approvals that included a refresh token or any available approved scopes, the request fails as unauthorized.
```
In simpler terms it means obtain a refresh token (using authorization flow) at least once before using jwt flow.
> If we dont perform Step 3, we will get the following error in step 5
```json
 {"error":"invalid_grant","error_description":"user hasn't approved this consumer"}
```