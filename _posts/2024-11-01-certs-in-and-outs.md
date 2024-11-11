---
layout: post
title:  "Certificates in and outs"
date:   2024-11-01 10:32:20 +0100
categories: security webdev
---

This is my humble summary regarding every important aspect of the certificates that a developer likely will encounter. I will update this article as I learn more about this topic.

## Some basics

### Hashing "#"

Hashing is the process of transforming a character string to an another format using a mathematical function which makes it:

- fast -> using trapdoor functions which are easy to calculate but hard to reverse e.g.: product of two prime number but finding out which two numbers were they is much much harder
- theoriticaly unreversable
- provides the same output for the same input
- prevents collisions -> cannot produce the same outfor for the same input
- reduces the input size -> it devides the input to equal blocks then compress them before running the hash function

Ok, but what is it good for❓

Usually hashes are sent along side with a payload/data to determine if it was tempered after the sender initially sent it to the client. But the are also used for authentication when the user's password wont be stored in the database just the **salt**ed hash itself.

**Salting** is the process of concatenating an already known propery of the user to the password and hash it together. This is important to prevent [rainbow table attacks](https://www.beyondidentity.com/glossary/rainbow-table-attack) for leaked databases which contains the hashes of the user passwords.

For example each certificate contains the hash fingerprint of itself and this way when client recieves it, then it can be hashed again and see if the fingerprints are matching.

**Peppering** is one more additional layer of security when storing hash. It is basically doing one more round of hashing but now using some shared secret which wil be the same for all of the entries in the database.

#### Hashing algorithms

There are multiple algorithms which is used for hashing, but they have differnet characteristics. Some of them lost their reliability by proving it can create collisions when we are doing lot of hashes, or simply the computers evolved so much that the hash can be reversed because it is not using big enough prime numbers or complexity.

##### Lets see which algo should be used in 2024

| **Algorithm**   | **Suggested in 2024** | **Digest Size** | **Speed**        | **Security Level**              | **Typical Applications**           |
|-----------------|-----------------------|-----------------|------------------|---------------------------------|------------------------------------|
| **MD5**         | ❌ No                 | 128 bits        | Very Fast       | Weak (prone to collisions)      | Checksums, basic data integrity    |
| **SHA-1**       | ❌ No                 | 160 bits        | Fast            | Weak (collisions found)         | Legacy systems, data integrity     |
| **SHA-2 (SHA-256)** | ✅ Yes          | 256 bits        | Moderate        | Strong                           | Cryptographic protocols, blockchain, SSL |
| **SHA-2 (SHA-512)** | ✅ Yes          | 512 bits        | Slower           | Very Strong                      | High-security applications, SSL    |
| **SHA-3 (SHA3-256)** | ✅ Yes         | 256 bits        | Moderate        | Very Strong                      | Cryptographic protocols, secure storage |
| **SHA-3 (SHA3-512)** | ✅ Yes         | 512 bits        | Slower          | Very Strong                      | High-security applications         |
| **BLAKE2**      | ✅ Yes               | Variable (up to 512 bits) | Very Fast | Strong                        | General purpose, cryptography, file hashing |
| **BLAKE3**      | ✅ Yes               | 256 bits        | Very Fast       | Strong                          | File hashing, cryptography, real-time systems |
| **RIPEMD-160**  | ⚠️ Limited use       | 160 bits        | Moderate        | Moderate                         | Digital signatures, blockchain    |
| **Whirlpool**   | ✅ Yes               | 512 bits        | Slow            | Very Strong                      | High-security storage, cryptographic protocols |
| **Tiger**       | ⚠️ Limited use       | 192 bits        | Fast            | Moderate                         | Data integrity verification      |
| **Argon2**      | ✅ Yes               | Variable        | Slow (memory-intensive) | Very Strong           | Password hashing                   |
| **PBKDF2**      | ✅ Yes               | Variable        | Slow (iteration-based) | Very Strong           | Password hashing, key derivation   |
| **Scrypt**      | ✅ Yes               | Variable        | Slow (memory-intensive) | Very Strong           | Password hashing, key derivation   |

#### Hashing best practices 💡

Never store password in the database plaintext, use a hashed and salt&peppered string preferebly using at least sha2 with multiple iterarations/work factors. This way the
cracking process itself wont worth for the attacker.

---

### What is a certificate?

> A **certificate** or **digital certificate** is a unique, [digitally signed](https://www.computerhope.com/jargon/d/digisig.htm) document which authoritatively identifies the identity of an individual or organization. Using [public key cryptography](https://www.computerhope.com/jargon/p/pkc.htm), its authenticity can be verified to ensure that the software or website you are using is legitimate. On the Internet, a certificate is signed by a trusted [CA](https://www.computerhope.com/jargon/c/certificate-authority.htm) (certificate authority), and verified with the authority's public key. The decrypted certificate contains a verified public key of the certificate holder (website operator), with which encrypted [HTTPS](https://www.computerhope.com/jargon/h/http.htm#https) communications can be established.
[source](https://www.computerhope.com/jargon/c/certific.htm)
{: .prompt-info }

So basically the certificate is a unique generated chain of characters in a specific format which can be validated by a third party (CA) therefore can be used for authentication and authorization.

## Use cases

Certificates are used for:

- securing communication between two parties -> like ssh or http
- prooving access rights for a server -> mutual tls (mTLS)
- proving integrity of a document -> digitally signed pdf

## How to use certificate to secure communication: Public-key cryptography or Asymmetric cryptography

### Public Key Pair Generation

- RSA -> using big prime numbers and based on factoring trap door function, using big keys to provide security
- ECC (Elliptic curve) -> based on mathematical properties of elliptic curves from which next point can be calculated easily (public key), but reversing it almost impossible -> using shorter keys due to it
- DSA -> using discrete logaritmic properties and private key generated from the content that we want to sign -> used for ensuring file integrity and verification

#### Communication Flow

{% plantuml %}
@startuml
actor Alice
actor Bob

@enduml

{% endplantuml %}

![asymetrics flow](/assets/certs-in-and-outs/asymetrics-flow.jpg){:style="display:block; margin-left:auto; margin-right:auto"}

In very oversimplified manner this happens

```ts
 const privateKey = randomString();
 const publicKey = generatePublicKey(privateKey);
 const plainMessage = 'Hello Mr Bug!🐜';
 
 const encryptedMessage = encrypt(publicKey, plainMessage);
 const decryptedMessage = decrypt(encryptedMessage, privateKey);
 console.log(decryptedMessage); //'Hello Mr Bug!🐜'
```

---
### The roles of  🟢 Public and 🔵Private keys

| **What one key does**   | **The other will validate** |
|-----------------|-----------------------|
| 🟢 Public encrypts         | 🔵Private decrypts             |
| 🔵Private digitally signs | 🟢Public verifies signature| 
| 🔵Private authenticate    | 🟢Public verifies authentication| 
---

## Public-Key Infrastructure (PKI)

> How the heck validate the cert? 😵
{: .prompt-tip }

### Certificate Authorities (Verification Authority)

- A company that is considered trustworthy and produces digital certificates for other individuals and
companies (i.e. subjects) bearing that subject’s public key

#### Tasks

- issues digital certificates;
- helps establish trust between communicating entities over the internet;
- verifies domain names and organizations to validate their identities; and
- maintains [certificate revocation lists](https://www.techtarget.com/searchsecurity/definition/Certificate-Revocation-List).⛔

![certs-in-and-outs](/assets/certs-in-and-outs/Pasted image 20230712170056.png)

### Chain of thrust

> Hierarchy of certificates is used to verify the validity of a certificate’s issuer. This hierarchy is known as a _chain of trust_. In a chain of trust, certificates are issued and signed by certificates that live higher up in the hierarchy.
{: .prompt-info }

#### Contains three main parts

#### Trust Anchor

- A public cert key named as CA cert which is trusted by both of the communicating parties

#### Intermediete Cert

- you cannot issue new cert from thrusted public CA certs since you need a new layer
- extra security to provide validation for the CA cert
- cheaper then issueing new CA cert for every need

#### End Entity Cert / Leaf Cert

- no other cert can be above it
- represents the end entity of the chain

What is CSR (Certificate Signing Request)?
> It is a request which sent to the CA to issue a new certificate with the given properties
{: .prompt-info }

##### What happens?

- public key pair is generated
- requested cert field are asked from the user
- CSR generated and contains:
  - public key
  - data fields are included with the input from the user

#### Example

Decoder <https://ssltools.godaddy.com/views/csrDecoder>

![certs-in-and-outs](/assets/certs-in-and-outs/Pasted image 20230712181327.png)

---

## X.509

> **Ok but how does everyone understands the certs?**
>
##### X.509 for the win! 🏆
>
>This standard make possible to  that information within a digital certificate are all placed in the same location and in the same order, which makes it possible for all kinds of certificates to be shared across organizations
>
{: .prompt-tip }

### Parts 🎁

#### How to dissect certificate

#### Data Section

![certs-in-and-outs](/assets/certs-in-and-outs/Pasted image 20230712164200.png)

#### Signature Section

- The cipher algorithm – the algorithm used by the issuer to create a digital signature
- The digital signature for the CA – a hash of all the information in the certificate encrypted with the CA private key

![certs-in-and-outs](/assets/certs-in-and-outs/Pasted image 20230712170609.png)

## Certificate key sizes

- sha

## Certificate formats ☘️

### Binary

#### .DER (Distinguished Encoding Rules)

- A single binary certificate, platform-independent format, the default format for most browsers
- Use: Used for Certificate Requests, which are always DER-encoded and then base64-encoded

#### PKCS#12 (Public-Key Cryptographic Standards)

- One or more certificates packed together, password-encrypted
- Use: When the CA wants to ship a package confidentially that contains the private key

### Base64 (ASCII)

#### .PEM

- Default format for OpenSSL. Suitable for sending files as text between systems
- Prefixed with a “--BEGIN…” line
- Use: When making a Certificate Request in an email

#### PKCS#7

- One or more certificate packaged together but not signed or encrypted
- Use: When the CA wants to deliver multiple certificates to a destination

![certs-in-and-outs](/assets/certs-in-and-outs/Pasted image 20230712170647.png)

---

## Cert Validity Scopes

- Single Domain Certs -> for single domain
- Wildcard SSL Certs -> valid for any subdomain
- Multi-Domain SSL Cert -> valid for multiple domain
- Self signed certificates

---

### Cert Validation Levels

- Domain Validation -> dns record or https challange (lets encrypt)
- Organization Validation -> CA will contact the organization and will check every detail of the company that will be included in the cert
- Extended Validation -> CA will make full background check of the organization

### How to retrieve certificates

#### Self signed

##### How to trust them

#### Lets encrypt and cert bot

---

## SSL / TLS

TLS, short for Transport Layer Security, and [SSL](https://kinsta.com/knowledge/base/how-ssl-works/), short for Secure Socket Layers, are both cryptographic protocols that encrypt data and authenticate a connection when moving data on the Internet.

> Well, **TLS is actually just a more recent version of SSL**

- SSL 1.0 – never publicly released due to security issues.
- SSL 2.0 – released in 1995. Deprecated in 2011. Has known security issues.
- SSL 3.0 – released in 1996. Deprecated in 2015. Has known security issues.
- TLS 1.0 – released in 1999 as an upgrade to SSL 3.0. Planned deprecation in 2020.
- TLS 1.1 – released in 2006. Planned deprecation in 2020.
- TLS 1.2 – released in 2008.
- TLS 1.3 – released in 2018.

> **Why do we calling the certs SSL cert?**
>The reason why most people still refer to them as SSL certificates is basically a branding issue. Most major certificate providers still refer to certificates as SSL certificates, which is why the naming convention persists.
{: .prompt-info }

In reality, all the “SSL Certificates” that you see advertised are really **SSL/TLS Certificates**

> What is HTTP**S**?!
>This is also where HTTPS comes in (HTTPS stands for “HTTP over SSL/TLS”).
> What will be encrypted during a https request?
> only the request part of the http
{: .prompt-tip }

## SSL termination

## Mutual TLS
