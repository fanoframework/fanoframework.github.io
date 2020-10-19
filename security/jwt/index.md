---
title: JSON Web Token (JWT) signing and verification
description: How to sign and verify JSON Web Token (JWT) in Fano Framework
---

<h1 class="major">JSON Web token (JWT)</h1>

According to [jwt.io](https://jwt.io)

> JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties.

JWT consists of three parts, header, claims and signature. They are concatenated with dot character to form JWT. Authenticator party can verify claims using only data contained in JWT. If token is verified, claims is authentic.

You can read [RFC 7519](https://tools.ietf.org/html/rfc7519) for detail information regarding JWT.

## Generate and sign JSON Web token (JWT)

To generate and sign JWT, Fano Framework provides `TJwtTokenGenerator` class. It
implements `ITokenGenerator` interface. Currently supported algorithms are

- HMAC SHA2 256 bit.
- HMAC SHA2 384 bit.
- HMAC SHA2 512 bit.

### Setup TJwtTokenGenerator

You can use `TJwtTokenGeneratorFactory` class and register it to [dependency container](/dependency-container).

```
//setup JWT token generator with HMAC SHA2-256
container.add(
    'jwtGenerator',
    TJwtTokenGeneratorFactory.create()
        .secret('very very secret key')
        .algorithm(THmacSha256JwtAlg.create())
);
```
To generate with HMAC SHA2-384 or HMAC SHA2-512, replace with `THmacSha384JwtAlg` and `THmacSha512JwtAlg`
respectively.

Following code shows how to retrieve `ITokenGenerator`instance, query it from dependency container.

```
var fTokenGenerator : ITokenGenerator;
...
fTokenGenerator := container['jwtGenerator'] as ITokenGenerator;
```

### Generate token

`ITokenGenerator` interface provides `generateToken()` methods. It expects parameter of `TJSONObject` type for input data and returns token as string.

We choose `TJSONObject` class because it allows us to pass arbitrary input data easily.
This does not mean `ITokenGenerator` is always tied to JWT. Implementor decides how token is generated.
For `TJwtTokenGenerator` class, it is JWT, obviously.

```
var payload : TJSONObject;
    token : string;
...
payload := TJSONObject.create([
    'iss', fIssuer,
    'sub', username
]);
token := fTokenGenerator.generateToken(payload);
```

## Verify JSON Web token (JWT)

### Setup TJwtTokenVerifier

You can verify JWT with `TJwtTokenVerifier` class. It implements `ITokenVerifier` interface.
You can use `TJwtTokenVerifierFactory` class and register it to [dependency container](/dependency-container).

```
//setup JWT token verifier
container.add(
    'jwtVerifier',
    TJwtTokenVerifierFactory.create()
        .secret('very very secret key')
        .issuer(('fano')
);
```
Following code shows how to retrieve `ITokenGenerator`instance, query it from dependency container.

```
var fTokenVerifier : ITokenVerifier;
...
fTokenVerifier := container['jwtVerifier'] as ITokenVerifier;
```

### Verify JWT

`ITokenVerifier` interface provides `verify()` methods. It expects one string parameter, token to verify and returns status of verification in `TVerificationResult` type.

```
var verifyRes : TVerificationResult;
...
verifyRes := fTokenVerifier.verify(token);
```

Fano Framework declares `TVerificationResult` as follow

```
TVerificationResult = record
    verified : boolean;
    credential : string;
end;
```
If token matches, `verified` field is set to true. `credential` field content is implementation
specific. For `TJwtTokenVerifier`, it returns subject of JWT claims. If you want different data,
you can inherit `TJwtTokenVerifier` and override its `getCredential()` protected methods.

For example to return whole JWT claims part,
```
uses fpjwt;
type
    TMyJwtVerifier = class(TJwtTokenVerifier)
    protected
        function getCredential(const jose : TJOSE; const claims: TClaims) : string; override;
    end;
...
    TMyJwtVerifier.getCredential(const jose : TJOSE; const claims: TClaims) : string;
    begin
        result := claims.asString;
    end;
```

## Explore more

- [HTTP Authentication](/security/http-authentication)
- [HTTP Bearer Authentication with JWT example project](https://github.com/fanoframework/fano-jwt) demonstrates how to use [JWT](https://tools.ietf.org/html/rfc7519) (RFC 7519) to sign and verify JWT token in Fano Framework.
