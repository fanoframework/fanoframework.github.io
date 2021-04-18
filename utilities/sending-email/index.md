---
title: Sending Email
description: Tutorial on how to send email with some utilities provided by Fano Framework
---

<h1 class="major">Sending Email</h1>

## Send email message

Free Pascal users have some options when sending email. Some library such as Indy has TIdSMTP which connect directly to SMTP server.
As alternative, Fano Framework offers a way to send email by using sendmail binary.

## IMailer interface

To abstract away mail sender implementation, `IMailer` interface is provided. It is a contract for any class having capability to send email.

## Sending email with sendmail

Current implementation of `IMailer` interface only supports sending email using sendmail binary, from [ssmtp library](https://linux.die.net/man/8/ssmtp) thorough class `TSendmailMailer`. To use it, register its factory class with [container](/dependency-container).

```
container.add('mailer', TSendmailMailerFactory.create());
```

```
var amailer : IMailer;
...
amailer := container['mailer'] as IMailer;
amailer.recipient := 'recipient@example.com';
amailer.sender := 'no-reply@example.com';
amailer.subject := 'Hello';
amailer.body := 'Hello world';
amailer.send();
```

## Setting up ssmtp library

To install ssmtp in Debian-based distribution,

```
$ sudo apt install ssmtp
```
or Fedora-based,

```
$ sudo yum install ssmtp
```

Edit `/etc/ssmtp/ssmtp.conf` and add following lines,

```
mailhub=smtp.yourownserver.com:25
AuthUser=YourUsername
AuthPass=YourOwnSecretPassword
FromLineOverride=YES
```

Replace `mailhub`, `AuthUser`, `AuthPass` with your own value. If you do not setup dedicated SMTP server, you can use external SMTP server such as gmail or mailtrap, or MailHog. Both mailtrap and MailHog is fake SMTP server which helps sending test email without fear of becoming email spammer during development.

For example, this is configuration for sending email with mailtrap.io. You should replace `YourUsername` and `YourOwnSecretPassword` with username and password of your mailtrap.io account.

```
mailhub=smtp.mailtrap.io:2525
AuthUser=YourUsername
AuthPass=YourOwnSecretPassword
FromLineOverride=YES
```

To test that you configuration works, try to send an email from command line

```
$ echo -e 'Subject: test\n\nTesting ssmtp' | sendmail -v tousername@example.com
```

## Explore more

- [Utilities](/utilities)
- [Fano Email](https://github.com/fanoframework/fano-email) example project demonstrates how to send email.
