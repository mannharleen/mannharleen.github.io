---
layout: post
title: Cryptography
subtitle: To the point
# gh-repo: mannharleen/log
# gh-badge: [star, fork, follow]
tags: [cryptography, encryption, signing, hashing]
comments: true
---
Of course! Cryptography is a very technical subject; **it's difficult to understand the "how"**

But there ubiquitous presence warrants to **understand the "why & when"**

Surprisingly, these questions aren't difficult to understand, so let's run through them quickly.

## Some terms
- Encryption and signing make use of keys
- Private keys: are keys that are treated as secret and are usually used by a single entity
- Public keys: are keys that are shared and available for anyone to use
- Symmetric keys: where encryption and decryption is done using the same key
- Asymmetric keys: where encryption and decryption is done using the separate keys belonging to a key-pair
- Hashing makes use of a hashing function
- Encoding (which is not cryptography) is merely changing how the data is represented e.g. ASCII, UTF-8, base64, base62, HEX etc.

## Encryption, Signature & Hash

### Encryption
```
Encrypt some data called X:
[X] --(public key)--> [Y]

Decrypt Y to obtain the data:
[Y] --(private key)--> [X]
```
Implying that anyone can encrypt a message/data, but only those systems/apps that have access to the private key can decrypt it. e.g. if an app wants to receive data securely, it can publish a public key. This public key can be used by another app to encrypt the data and send to the former app. The former app can then use the private key to read the data.

Under symmetric encryption, i.e. when there is only 1 key (no public private key pair), typically the same system/app is performing the encryption and decryption of the message/data. e.g. encrypting data at rest. It is also used in TLS process.

### Signature (aka Digital Signature)
```
Encrypt some data called X:
[X] --(private key)--> [Y]

[X,Y] - both are sent from sender to receiver

Decrypt Y to obtain the data:
[Y] --(public key)--> [X]
```
Digital signatures are used to verify that a message was sent by a particular source, and none else. The destination can verify this by comparing X that was sent and X that was obtained by decrypting Y. If the two are not equal, then either the message was tampered or not sent by the sender we expected it from.

### Message authentication
```
Encrypt some data called X:
[X] --(key)--> [Y]

[X,Y] - both are sent from sender to receiver

Decrypt Y to obtain the data:
[Y] --(same key)--> [X]
```
Message authentication works in the same manner as Signing does, except that it uses symmetric keys. 

### Hash
```
[X] --(hashing <DOESN'T USE A KEY>)--> [Y]

[Y] --[NOT POSSIBLE]--> [X]
```
Implying, that once Y is created it is not possible to obtain X from Y.
Typically used to store passwords and perform data integrity checks

## Popular algorithms

|Cryto type|Popular algorithms| Typical applications|
-----------|----------|-------|---------------------|
Symmetric encryption    | AES family | Encrypting hard drives
Asymmetric encryption   | RSA family | Sending encrypted data over the network
Digital Signing         | RSA family | SSL certificates
Message Authentication  | HMAC       | Authentication (see notes)
Hashing                 | SHA family | Storing data

# Notes:
HMAC is used by AWS to provide programmatic access via AWS APIs. An access_key and secret_key is provided such that the access_key identifies the user and the secret_key is the private_key. When data is sent to AWS API, the AWS SDK (or CLI) hashes then signs the data using secret_key to produce HMAC. Then sends access_key, HMAC and data. AWS then cross-references the access_key internally to obtain public_key. AWS then decrypts the HMAC using this public_key to get decrypted_hash. It then hashes the data to produce a hash. Finally, AWS verifies the integrity of the message by comparing the decrypted_hash and hash, and authenticates since public_key was corresponds to the private_key that is with the user.