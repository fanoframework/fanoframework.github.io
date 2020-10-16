---
title: Password hash
description: Password hash in Fano Framework
---

<h1 class="major">Password hash</h1>

## Storing password

[Best practice for storing password](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) is to never store plain password but hash value of password and its salt, so that when password storage is compromised, it is still considerably very difficult task to retrieve plain password.

## Password hash and verification

Fano Framework provides several password hash algorithm based on [HashLib4Pascal](https://github.com/Xor-el/HashLib4Pascal) and [BCrypt](https://github.com/viniciussanchez/bcrypt) libraries.

Fano Framework provides simple wrapper for these libraries through `IPasswordHash` interface.

### Generate password hash

To generate password hash, you call `hash()` method of `IPasswordHash` interface. This method accepts string of plain password and returns hashed password as hexadecimal string.
```
var passwHash : IPasswordHash;
    plainPassw, hashedPassword : string;
...
plainPassw := 'PAssw0rd';
hashedPassword := passwHash.hash(plainPassw);
```

### Verify plain password against password hash

To verify password against password hash, you call `verify()` method of `IPasswordHash` interface. This method accepts string of plain password and hashed password and returns boolean true when they match or false otherwise.

```
var passwHash : IPasswordHash;
    plainPassw, hashedPassword : string;
    verified : boolean;
...
plainPassw := 'PAssw0rd';
hashedPassword := 'edaadeaa....';
verified := passwHash.verify(plainPassw, hashedPassword);
```
### Setting password hash parameters.
Before you generate password hash or verify a password, you need to set some parameters related to passworh hash implementation

#### Salt
To prevent against rainbow table attack, you should salt password. You should use random salt value. This value can be stored along with password hash.

To set salt, you call `salt()` method of `IPasswordHash` interface. This method returns current password hash instance.

```
passwHash.salt('some random value');
```

#### Cost
To make generating password expensive, some password hash algorithm requires you to set number of iterations as cost of algorithm.
Higher value usually means higher computation resources. You should think carefully about this value as this is trade-off between security and performance.

To set cost, call `cost()` method with integer value for cost.
This method returns current password hash instance.

```
passwHash.cost(1024);
```

#### Memory cost
Some algorithm employs memory cost, such as Argon2i. For other, this value is ignored.

To set memory cost, you call `memory()` method of `IPasswordHash` interface. This method returns current password hash instance.

```
passwHash.memory(32);
```
#### Paralleism
Some algorithm employs paralleism cost.

To set paralleism cost, you call `paralleism()` method of `IPasswordHash` interface. This method returns current password hash instance.

```
passwHash.paralleism(1);
```

#### Secret key
In addition to salt, some algorithm allows to use secret key which application must kept secret.

To set secret key, you call `secret()` method of `IPasswordHash` interface. This method returns current password hash instance.

```
passwHash.secret('hush hush value');
```
#### Password hash length
To set how many bytes passwod hash length, you set it using `len()` method and pass integer value of number of bytes of hash.

```
passwHash.len(64);
```

## Available password hash implementations

Currently, Fano Framework provides `IPasswordHash` implementation for [Argon2i](https://en.wikipedia.org/wiki/Argon2), [PBKDF2](https://tools.ietf.org/html/rfc2898), [Scrypt](https://tools.ietf.org/html/rfc7914), [BCrypt](https://en.wikipedia.org/wiki/Bcrypt) and SHA2.

### PBKDF2 password hash

To use PBKDF2 password hash, you need to use `TPbkdf2PasswordHash` class. You can register this class with [dependency container](/dependency-container) using its factory class `TPbkdf2PasswordHashFactory`.

To register password hash,
```
container.add('passwHash', TPbkdfPasswordHashFactory.create());
```
To retrieve password hash instance,

```
var passwHash : IPasswordHash;
passwHash := container['passwHash'] as IPasswordHash;
```

### Argon2i password hash

To use Argon2i password hash, you need to use `TArgon2iPasswordHash` class. You can register this class with dependency container using its factory class `TArgon2iPasswordHashFactory`.

To register password hash,
```
container.add('passwHash', TArgon2iPasswordHashFactory.create());
```
See PBKDF2 code above to retrieve password hash instance.

You can set initial setting values for password hash instance, for example

```
container.add(
    'passwHash',
    TArgon2iPasswordHashFactory.create()
        .cost(4)
        .memory(32)
        .paralleism(4)
        .len(32)
);
```

### Scrypt password hash

To use Scrypt password hash, you need to use `TScryptPasswordHash` class. You can register this class with dependency container using its factory class `TScryptPasswordHashFactory`.

To register password hash,
```
container.add('passwHash', TScryptPasswordHashFactory.create());
```

### BCrypt password hash

To use BCrypt password hash, you need to use `TBcryptPasswordHash` class. You can register this class with dependency container using its factory class `TBcryptPasswordHashFactory`.

To register password hash,
```
container.add('passwHash', TBcryptPasswordHashFactory.create());
```

### SHA2 password hash

While you are advised not to use SHA2 for password hash, Fano Framework provides `IPasswordHash` implementation for SHA1 256 bit and 512 bit using `TSHA2_256PasswordHash` and `TSHA2_512PasswordHash` class.

You can register this class with dependency container using its factory class `TSha2PasswordHashFactory`.

To register SHA2 256 bit password hash,
```
container.add('passwHash', TSha2PasswordHashFactory.create().use256());
```
To register SHA2 512 bit password hash,
```
container.add('passwHash', TSha2PasswordHashFactory.create().use512());
```

## Explore more

- [Dependency Container](/dependency-container)
- [Security](/security)
- [Fano Password example](https://github.com/fanoframework/fano-password).
