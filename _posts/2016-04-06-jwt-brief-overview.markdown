---
layout: post
comments: true
title:  "Authentication with JWT brief overview"
date:   2016-04-06 16:09:02 +0000
categories: jwt auth
---

Last month I was searching about the best way to implement authentication/authorization in a new service. Then I found JWT. At a first glance, I feel confused about the way it works, but after reading a few articles and examples, JWT showed up beautiful and simple so that I decided to write about it as the first post of my blog.

# JWT Definition

Let's start with JWT definition, from Wikipedia:

> JSON Web Token (JWT) is a JSON-based open standard (RFC 7519) for passing claims between parties
> in web application environment. The tokens are designed to be compact, URL-safe and usable especially 
> in web browser single sign-on (SSO) context.

From that, we can assume JWT can be used as authentication mechanism and for secure message interchange. 

Another good definition comes from http://jwt.io:

> JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims 
> securely between two parties.

## JWT Structure

To understand better about that, we should take a look on JWT structure:

JSON Web Tokens consist of a string with three parts separated by dots (.), which are:

* Header
* Payload
* Signature

The **header** consists of two parts: the type of token (witch is JWT) and the hashing algorithm, witch can be a symetric or asymetric hashing algorithm.

For example

	{
		"alg": "HS256",
		"typ": "JWT"
	}

The second part of the token is the **payload**, which contains the claims. Claims are statements about an entity (typically, the user) and additional metadata. There are a set of predefined claims, like 'iat' (issued at), which are not mandatory but recommended,  and custom claims, witch can whatever you want, like 'username', 'usermail', 'permissions' and others.

The third part is the **signature**. To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.

For example:

	HMACSHA256(
	base64UrlEncode(header) + "." +
	base64UrlEncode(payload),
	secret)
	
The resulting token would be something like that below:

	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ


# Traditional session based authentication vs Token based authentication

In authentication mechanisms context, JWT is a kind of a protocol to implement a stateless token based authentication, witch has many advantages over traditional session based authentication. Let's briefly describe these two types of authentication:

* **Traditional session based authentication:** a client authenticates with its credentials and receives a session_id and attaches this to every subsequent outgoing request. The session_id is just an identifier and the server does everything else.

* **Token based authentication:** a client authenticaties with is credentials and receives signed token which is then attached to every subsequent request. The server validates the token in every request. The token can carry any type of information and can be read on client (security aspects discussed later on) or server side. 

![Requests sequence diagram](/images/jwt1.png "Requests/Response flow for each type of authentication")


Now let's see which are the token based authentication main advantages:

1. **Token based authentication is stateless** A stateless web architecture is dependent only on the input parameters that are supplied. To achieve a stateless app, the underlying authentication mechanism has to be stateless. This property makes the scalability more cheap and simple. Here is a quick picture of how to scale a stateless service.

   ![Scaling stateless](/images/jwt2.png "Scaling stateless architecture")

   On the other hand, the traditional session based authentication is stateful, because it depends on server side data to be aware about user session state. The session data can be stored in memory or in database. To scale a stateful service, there are two options: using sticky sessions or non-sticky sessions. The sticky sessions option add rules on the load balancer to guarantee that every requests of the same user sessions goes to the same webserver that authenticated that user. The non-sticky sessions option is more efficient and complex. It adds another level of load balancing flow, like the image below:

   ![Scaling stateful](/images/jwt3.png "Scaling stateful sticky session architecture")


2. **Token based authentication is easily extensible** Suppose that an user is authenticated in a service A so that it receives a token signed with a private key holded by service A. This token can be used to share session state of that user on service A on the whole internet. If a service B, that holds the public key of service A, wants to use the session state of users on service A, it can rely directly on that token.  

   On the other hand, sharing session_id is not that simple. Session data is usually stored in databases not exposed directly on the internet. When the session data is stored in memory, that session_id would be shareable but not before a criptographic sign.

There other advantages also, like CORS issues and security issues, but I will leave those to another post.

# JWT vs other token standards

Now let's compare JWT against other token standards: **Simple Web Tokens (SWT)** and **Security Assertion Markup Language (SAML)**

As JSON is less verbose than XML, when it is encoded its size is also smaller, making JWT more compact than SAML. This makes JWT a good choice to be passed in HTML and HTTP environments. In fact, a JWT could be passed as an URL parameter.

Security-wise, SWT can only be symmetricly signed by a shared secret using the HMAC algorithm. However, JWT and SAML tokens can use a public/private key pair in the form of a X.509 certificate for signing. Signing XML with XML Digital Signature without introducing obscure security holes is very difficult when compared to the simplicity of signing JSON.

JSON parsers are common in most programming languages because they map directly to objects. Conversely, XML doesn't have a natural document-to-object mapping. This makes it easier to work with JWT than SAML assertions.

# Conclusion

Those advantages are enougth in many cases to decide for using a compact, url-safe and standized way (JWT) to obtain a token based authentication mechanism. But how to use it? What are the best practicties? Let's see later on next posts!
