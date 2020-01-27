[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)
![Misk Logo](https://i.ibb.co/KmXhJbm/Webp-net-resizeimage-1.png)

# Authentication with JWTs

## Learning Objectives

- Describe JSON web tokens (JWTs)
- Identify parts of JWTs
- Add JWT authentication with Passport to a MERN app

## Intro - What is Authentication and Encryption?

### Authentication

Today, we are going to learn about making our API more secure. Authentication is about making sure the server knows the identity of the person accessing the API and the data that is stored.

Authentication should be used whenever you want to know exactly who is using your API. To know which user is currently logged-in, an API needs to store sensitive data - this data will, therefore, be *encrypted*.

### What is a JSON Web Token?

[Official definition](https://tools.ietf.org/html/rfc7519): compact, URL-safe means of representing claims to be transferred between two parties. 

In other words: A JSON web token is JSON-formatted data sent securely between the server and the browser via HTTP requests. 

### Authentication with JWTs

The problem that JWTs seek to overcome: HTTP is stateless, but we need a way to tell the server that a user is logged in.

When making requests or performing actions that are only for authorized users, there needs to be a way to keep track of whether a user is logged in, since that information isn't stored in HTTP by nature. 

For instance, when we implemented *devise* for user authentication, we used sessions to remind the server of "logged-in status" with every request made to the server. A session is a place to store data on the browser during one request which can be read during later requests. The session is a Ruby/JS object that allows us to keep track of this information. When a new user signs into an application, we create a new session in the server, and a cookie for this session is sent in a response back to the browser. In future HTTP requests from the browser, the client sends a session cookie to the server to retrieve the user from the database to then authenticate the authorized interaction with the database (e.g. saving a post, editing data).

Another approach to keeping track of a user being logged in is to use JWTs with Passport. With JWTs, the user info is embedded in a token. Upon initial log in, the server creates a JSON "token" to store the user info. These tokens are "signed" by the server, and only the server holds the private key to read the token.

#### How It Works

![JWT vs. Sessions Diagram](https://cdn-images-1.medium.com/max/1600/1*d6YcPvq7TeU0DTamj629xw.png)

1. Client browser makes a request sending user login credentials and password (only has to do this once)
2. Server validates the credentials and sends a JSON response to the client that encodes user login data
3. Client stores this JSON web token
4. When the client sends a request to a route that requires authentication, it will send this token to the API to present its authorization for access

#### Advantages of using JWTs:

- JWTs are self-contained
    - You have all the information about the user within the token. After inital request from browser, the server doesn't need to interact with the database to know who the user is. Using JWTs limit database lookups.
- JWTs are compact, and transmission through HTTP actions is fast.
- JWTs work the same for browser clients and native mobile apps.

### What does a JWT look like?

A string with three parts, each separated by dots (`.`):

    - header
    - payload
    - signature

#### Header

**Header** is a JSON object consisting of two parts: the type of token (typ) and the hashing algorithm being used on the token (alg).

> Example...
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

#### Payload

**Payload** is a JSON object containing claims. Claims refer to statements about an entity (e.g. user data). You can put as many claims into the payload as you want, though you want to be cognizant of keeping the JWT compact so as not to impact performance of HTTP actions.

> Example...
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "userId": "5z-9328477bz",
  "admin": true
}
```

There are three different types of claims: public claims, private claims, and registered claims.

Registered claims refer to claims that have predetermined key names - e.g. common fields like issuer ("iss"), subject ("sub"), and expiration time ("exp").

Public claims are claims that we create - e.g. "name", "userId", and "admin" above.

Private claims are used when JWTs are sent between two parties. Only these two parties know what the claims respresent.

#### Signature
**Signature** is encoded header and payload signed with a secret key.

The header is encoded, and the payload is encoded. They are joined together with a `.` in between. This string is then hashed with the server's secret key, using header's hashing algorithm. This produces a new string, which is added onto the `<header>.<payload>` string with another `.` between.

The signature allows the receiver to ensure that the JWT was sent from an authentic source (the holder of the secret key). This encoding does not serve to encrypt the data, but to transform the data.

> Note: [Refresher on difference between encoding, encrypting, and hashing.](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/)

#### Final Product

Encoded string vs. decoded JSON:

![JWT: encoded string vs. decoded JSON](https://cdn-images-1.medium.com/max/2000/1*LAo6s2tlszZdk2x-uE1lqA.png)

### Encryption

When we talk about passwords, the commonly used word is "encryption", although the way passwords are used, most of the time, it is a technique called "hashing". Hashing and Encryption are pretty similar in terms of the processes executed, but the main difference is that hashing is a one-way encryption, meaning that it's very difficult for someone with access to the raw data to reverse it.  


|     | Hashing |   Symmetric Encryption -|  
|-----|---------|-----------------------|
|     |One-way function | Reversible Operation |
|Invertible Operation? |    No, For modern hashing algorithms it is not easy to reverse the hash value to obtain the original input value | Yes, Symmetric encryption is designed to allow anyone with access to the encryption key to decrypt and obtain the original input value |


#### Bcrypt

Hashing is when a function is called on a variable - in this case a password - in order to produce a constant-sized output; it being a one-way function, there isn't a function to reverse or undo a hash and calling the function again - or reapplying the hash - isn't going to produce the same output again.

From another [stack post](http://stackoverflow.com/questions/1602776/what-is-password-hashing):

_"Hashing a password will take a clear text string and perform an algorithm on it (depending on the hash type) to get a completely different value. This value will be the same every time, so you can store the hashed password in a database and check the user's entered password against the hash."_

This prevents you from storing the clear-text passwords in the database (bad idea, _very bad_).

Bcrypt is recognized as one of the most secure ways of encrypting passwords because of the per-password salt. Even with it being slower than any other algorithms, a lot of companies still prefer to use bcrypt for security reasons.

#### But wait, what's a salt?

A salt is random data that can be added as additional input to a one-way function, in our case a one-way function that  hashes a password or passphrase. We use salts to defend against dictionary attacks, a technique for "cracking" an authentication mechanism by trying to determine the decryption key.

### Using JWTs with Passport in a MERN app

Passport allows you to store the user object in requests instead of in session cookies. Upon the log-in request, the server will create a token and pass it to the browser in the HTTP response. The token is saved to local storage in the browser.

When the user wants to access a route that requires authorization, the client will send a JWT with the request to the server. Since the server has the secret key to decode the JWT, it can (a) verify that the JWT has the right signature to ensure that the JWT originally came from that server, and (b) verify the user and then perform the action that needed authorization.

## Steps for authentication
So, let’s start by mapping out what we want in an authentication system. First, we will want to be able to have people register for an account. This will require some sort of personal identifier. Either a username or an email would do us well, and actually it would be beneficial to have both! We want an email address for the purpose of contacting the user later in other parts of our app, and a username in addition to an email address allows us to use the username as a public ID for our users in the app without compromising their email address which would inundate the user with spam. Second, to log-in we would need to have a password or some sort of challenge for users signing in to pass in order to be Authorized for the application. Let’s take a few notes:

Requirements:
1. User Identifier - User Name.
2. Email Address - for contact purposes only.
3. Password - challenge part of authentication

I believe that in the common terminology, Authentication is the act of challenging a user’s sign-in attempt. Authorization is the reward for passing the challenge. That’s the most basic concept. Also, we need to salt and hash the password when we save it to the database so that it’s not stored in a recoverable way. When we compare the user’s entered password to the stored one, we will use methods that actually salt and hash the entered password to compare to the stored password.

Process 1: Authentication
Required parameters: Username, Password.
1. Grab User from Database.
2. Salt/Hash Entered password from User
3. Compare Entered vs. Stored passwords.
3.1 if password hashes match, generate a token and send to the user
3.2 if password hashes don't match, return an error.

We now have a process for the Authentication challenge. Let’s take a second to also write out our procedure for registering a user. We’ll need the user’s email address, user name, their desired password, and a confirmation of that password. First, we’ll want to check to see if the user already exists in the database then compare the two password fields to make sure they match each other. Then we’ll salt and hash the password field to store in the database. Let’s document this process quickly.

Process 1: Registration
Required Parameters: Username, Email, Password, Password2
1. Check the database for an existing user by Email and Username.
1.1 If there is not a user already, move on.
1.2 If there is a user already, return an error.
2. Compare the password fields.
2.1 If they match, move on.
2.2 If they don't, return an error 
3. Salt and Hash the password
4. Save User to Database.
5. Return success to the client.

Now that we have our procedures listed out with some basic steps, let’s take a look at tokens, what they are, and why they’re desirable in Authorization. A token, in the simplest sense is a claim that the bearer, or owner of the token, has the right to access a resource.

### Token

The token is Base64 encoded JSON object that securley transfers information between services. The token can be signed by synchronous methods such as a secret string that is used as the encryption key with the HMAC algorithm, or asynchronously with private and public key pairs with RSA or ECDSA. 
-Synchronous simply means that anyone that needs to verify the token is valid, must do so with the same key. 
-In Asynchronous signing, the Authorizing Entity, our Authentication server, signs the token with its private key, and the Client Entity verifies the signature of the token with the public key.
The base token itself has three components: the header, the payload, and the signature. The header usually only has two parts, a type that describes the type of token which will always be JWT, and an algorithm that defines the type of encryption used to sign the token. This is where we will define whether we want to use HMAC’s SHA265 algorithm for synchronous signing, or RSA signing in asynchronous.

### Payload

The payload of the token is all the information that needs to be sent to the client application. This can be the information relating to our user. This information is setup in the form of “Claims” these claims can be either Registered, Public, or Private. Registered Claims are those that are predefined by the JWT standard. Some simple ones are iss or issuer, exp or expiration, sub or subject

The next type of claims are public claims. These are defined at will by JWT users, but should be defined here. Alternatively, you can define your public claims in your own namespace.
Finally, we get to have private claims, which is where you would want to place your information for your client application. A private claim is simply one that two parties agree to, and requires no setup, and can be just used. This is how we’ll get our specific application information to the client. Let’s just quickly review the Claim Types:
Registered Claims: Registered Claims that are provided to basic, consistent information within the token.
Public Claims: Claims that are publicly agreed upon, but have nothing to do with the specification. These should be used if considering more of an OAuth situation where users are authenticating for an ecosystem of applications.
Private Claims: Claims that only the developer and/or the specific app uses. These are “Wild West” style, and can be pretty much anything.

```
Do not put any data that can be considered sensitive in the JWT. Decoding a JWT from Base64 is trivial. Treat the token as clear text. The purpose of signing the token is not to mask its contents, but to verify that the signing party believes the claims of the token to be true.
```

## We Do: Implementing Authentication with JWT
Let's fork and clone the blogy app attached with this lesson. As we're going to be adding authentication on top of it.

### Additional Resources on using JWTs in MERN apps
- https://medium.com/vandium-software/5-easy-steps-to-understanding-json-web-tokens-jwt-1164c0adfcec
- https://jwt.io/introduction/
- https://medium.com/@rajaraodv/securing-react-redux-apps-with-jwt-tokens-fcfe81356ea0
- https://hptechblogs.com/using-json-web-token-react/
- https://blog.jscrambler.com/implementing-jwt-using-passport/
- [FAQs: Authentication with tokens (vs cookies)](https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/#token-oauth)
- [Using bcrypt](https://jonathas.com/token-based-authentication-in-nodejs-with-passport-jwt-and-bcrypt/)
