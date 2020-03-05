---
layout: post
title: Go lambda bcrypt
subtitle: A ready to use aws lambda bcrypt microservice
gh-repo: mannharleen/go-lambda-bcrypt
gh-badge: [star, fork, follow]
tags: [go, golang, aws, lambda, bcrypt]
comments: true
---

# go lambda bcrypt

## Lambda, Dynamodb and Bcrypt
Bcrypt is the go to encryption algorithm for hashing passwords. 
If you have worked long enough with AWS Lambda and API Gateway, you will be well aware of the following pattern:
```
[api-gw] -> [lambda authorizer] -> [my lambda]
```

Lambda authorizer is used to verify the credentials against a database, usually dynamodb. Dyanmodb table contains username and password pairs where password are stored as bcrypt hashes

## Microservice for bcrypt
Given the importance of bcrypt with lambdas, I decided to write a microservice that performs the following:
1. convert password to hash
2. compare a password and hash and return whether valid or not

> To start using the microservice simply upload the zip file on the repo to your lambda

# Introducing go lambda bcryptit
https://github.com/mannharleen/go-lambda-bcrypt

An AWS lambda function that sits behind the API Gateway and performs the following:
- Given a password, return its bcrypt hash
- Given a password and a hash, verify that they are valid

# Usage

| Path | Expected input | Expected output | Description
|---|---|---|---|
| /bcrypt/hash | ``` { "password" : "secret" } ```| ``` {"result" : "@#$@#$" } ```| Returns a bcrypt hash of the provided password |
| /bcrypt/verify | ``` { "password" : "secret", "hash" : "$#$%#$%" } ```| ``` {"message" : "valid" } ``` or ``` {"message" : "invalid" } ```| Returns a bcrypt hash of the provided password |


The API GW should be configured as follows:
```
/
  /bcrypt
  OPTIONS
    /{proxy+}
    ANY
    OPTIONS
```
where ANY method is integrated with our Lambda via lambda proxy integration


# Deployment to AWS

## Method 1: Simply upload
- Create a new lambda function with go1.x as runtime
- Upload main.zip
- Rename "handler" to "main"
- And your lambda is ready to be used

If you fancy, use the deploy script :)

## Method 2: Build from source
- Run the build.cmd script to create an executable
- The same script should work on a Linux system as well
- Then deploy
