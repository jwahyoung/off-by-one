---
uid: 788c558c-83c1-4444-81bc-a63c753245fd
layout: post
title: Using asymmetric JWT on the server and the client
date: 2015-07-25
tags: jwt, javascript, node, express
published: true
authorid: jahyoung
author: Jedd Ahyoung
---

The modern web is built upon a stateless protocol: HTTP. However, we turn to older conventions when attempting to store state in web applications - often turning to session storage on the server side. Now, we have a modern alternative: [JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token).

## What is JWT?

JSON Web Token, or JWT for short, is a token used for authentication in web applications as defined by [RFC7519](https://tools.ietf.org/html/rfc7519). It's been around for approximately two years, and it's being pushed by [Auth0](https://auth0.com/). As of the time of this writing, Auth0 has just completed its rebranding of [jwt.io](http://jwt.io/).

Tokens expose claims as part of the payload. A typical token payload, after being converted from Base64, could look something like this:

```json
{
	"iss": "myapp.io",
	"exp": "1536710400",
	"iat": "1437891817",
	"user": {
		"username": "testuser",
		"displayname": "Test User",
		"permissions": ["read", "write"]
	}
}
```

The payload is encapsulated by a header and a footer, containing the token specification (token type, signing algorithm) and signature, respectively.

## So, what are the benefits?

What, didn't you read the RFC? Yeah, neither did I. However, there are clear benefits at first glance:

 - A user can have an application session without the application having to store that session state in some type of persistent storage.
 - Unlike the classic session model, JWTs don't require the use of cookies, which can simplify cross-domain requests.

The implications for the first bullet point are quite large. Not having to store an application session per user means that we can simplify our code, deload our authentication server and persistence server, and save a lot of space. Furthermore, by using the client as a part of the session scheme, we introduce scalability easily without having to throw more hardware at the solution (or a different storage methodology such as Redis).

As for the second point - we live in an age where mobile is a first-class citizen. Native mobile applications don't always do well with cookies, but they have no problem with Authorization headers containing JWTs. Forward-looking APIs recognize that their clients may not always be web browsers.

These two reasons alone make JWT a viable alternative to traditional sessions. In conjunction with a highly concurrent framework like Node.js, I think it's a solution worth investigating.

## The Basic Premise

We're going to use JWT for user identification and authorization - on both the server side and the client side.

For the purposes of my use case, I wanted to provide an authentication endpoint in my server application. Upon a successful login, the user would receive a JSON Web Token, which would be stored in the client application (in LocalStorage, in this case). The token itself would contain a list of permissions applicable to that user, along with any other relevant user information. Logging out would simply clear the token from storage.

The token would be passed in the Authorization header of every web request; once the server verifies the token, the server application logic would use the permissions defined in the token to determine whether or not a user is authorized to perform the specific function associated with that request.

Here's the cool thing - the client-side application, having a copy of the token, could also read the permissions and deny access to certain functionality based on the token itself. Due to the way we'll implement the token signature, the client-side application can check the validity of the token and react to an invalid token, just as the server can.

## Implementing JWT in Express.js

Because we're using the token on both sides of the application - server, and client - we're going to want to sign our token using an [asymmetric key](https://en.wikipedia.org/wiki/Public-key_cryptography). This ensures that while our server and our client (and anyone, really) can verify the tokens using a public key, only the server - using a secured private key - can sign the tokens.

This protects us from someone attempting to edit the token on the client side (since it is visible in LocalStorage) to impersonate a user or spoof permissions on the client or server, as editing the token will invalidate the hash. It also protects from someone attempting to create a valid token with false data; although they may have the public key, without the correct private key they cannot create a signature that the server will verify.

***NOTE:*** A private key is just that; private. Leaking or losing this key compromises the integrity of the entire application. If possible, follow [twelve-factor](http://12factor.net) in your application and [access your keys through a Node environment variable](http://12factor.net/config) - and DO NOT check them into source control.

It's also worth noting that we are not *encrypting* the token - we are only *signing* the token. This is an important distinction.

### Generating our public/private key pair

JWT supports multiple asymmetric key types. A discussion of the different key types and their key sizes is beyond the scope of this post, but you should make sure that you're using a secure key length. For the purposes of this example, we'll be using a 2048-bit RSA key.

I'm not sure how this can be done on a Windows system (sorry!) but if you're on a BSD or Unix-based system, you can use `ssh-keygen` to generate a pair of keys at the terminal prompt, like so:

```
Jedds-MacBook-Pro:~ jahyoung$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/jahyoung/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/jahyoung/.ssh/id_rsa.
Your public key has been saved in /Users/jahyoung/.ssh/id_rsa.pub.
The key fingerprint is:
95:19:dc:88:64:5b:9e:20:7e:02:47:fa:9d:5a:00:bd jahyoung@Jedds-MacBook-Pro.local
The key's randomart image is:
+--[ RSA 2048]----+
|        ==B      |
|       o.B.* .   |
|       .+ +.* .  |
|        .o.+oo   |
|         .oE S   |
|            o    |
|           .     |
|                 |
|                 |
+-----------------+
```

When using ssh-keygen, you'll have to convert the public key to PEM format so that our application can use it:

```
ssh-keygen -f mydirectory/rsa.pub -e -m pem
```

After that, you can either move the keys to your project directory for use in your application (don't check them in!); at startup, the application can read the keys into its process environment.

Note that when generating a key (regardless of which tool you use to do so), for our purposes, do not enter a passphrase. (I ran into problems with token verification due to this. If there is a way to use a passphrase with verification, I'd love to know.)

***NOTE:*** If you're on a Mac, like me, you may have issues transforming the public key from the OpenSSH format to PEM format, which is necessary for use with the JWT libraries we're using. The version of ssh-keygen bundled with Mavericks isn't up to date. You can either install a new version through [Homebrew](http://brew.sh/) (or your favorite package manager) or transform the key on a Linux machine (I used [Cloud9](c9.io)).

### Encoding the token

Now that we've finished setting up our keys, we can encode our token. I ended up using Auth0's [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) library to encode my JWT. Here's some simplified code:

```javascript
var jwt = require('jsonwebtoken');

module.exports.login = function (req, res, next) {
    User.findOne({ where: { username: req.body.user.username } })
        .then(function (user) {
            return User.checkCredentials(user.password, req.body.user.password);
        })  
        .then(function (user) {
            res.send(jwt.sign(user, process.env.MY_PRIVATE_KEY, {
                algorithm: 'RS256',
                issuer: 'myApp',
                expiresInMinutes: 60
            }));
        });
}
```

When a client application hits this endpoint with valid credentials, it gets a JWT back and stores it to use for future requests.

***NOTE:*** Don't put sensitive information in the token! You can put anything you want in there, but know that it can and will be read - it's a JSON object encoded in Base64 format.

### Decoding the token upon request

I wanted to add this to an Express application I was building in Node.js. LUckily, the good folks over at Auth0 have provided a middleware library for Express that handles token authentication, appropriately called [express-jwt](https://github.com/auth0/express-jwt). Decoding the token as part of your server HTTP request lifecycle is simply a manner of plugging the middleware into the Express app or Express routers, like so:

```javascript
var app = require('express');
var jwt = require('express-jwt');

...

app.use(jwt({ secret: process.env.MY_PUBLIC_KEY, algorithms: ['RS256'] }));
```

(Plugging the middleware into the Express router instances allows for more granularity; you wouldn't want this middleware on your login endpoint, for instance.)

The middleware will check the Authorization header of the HTTP request for the JWT and verify it using the key that you provide. In our case, we're verifying our signed token using our public key. If the token has been altered, it will likely be invalid; if it's been replaced or re-signed, the verification won't work using our public key. Either way, the request will fail.

## What's next?

We've established the groundwork for building JWT into the server side. In the next post, we'll talk about implementing verification on the client side.

Stay tuned!