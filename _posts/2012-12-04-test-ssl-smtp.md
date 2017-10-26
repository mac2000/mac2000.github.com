---
layout: post
title: Test SSL SMTP from command line
tags: [openssl, ssl, smtp, telnet]
---

To generate auth code use following command:

```sh
perl -MMIME::Base64 -e 'print encode_base64("\000USERNAME\@gmail.com\000PASSWORD")'
```

Do not forget to change `USERNAME` and `PASSWORD`.

Output will be something like this:

    AZ1hcb1oZW5rb25hbGV4Y45kckBn5WFpbC8jb20ADTM0NDk0MA==

**SMTP.GMAIL.COM**

```sh
openssl s_client -connect smtp.gmail.com:465 -crlf -ign_eof

EHLO localhost
AUTH PLAIN AZ1hcb1oZW5rb25hbGV4Y45kckBn5WFpbC8jb20ADTM0NDk0MA==
MAIL FROM: <marchenko.alexandr@gmail.com>
RCPT TO: <alexandrm@rabota.ua>
DATA
Subject: TEST

TEST
.
QUIT
```

**SMTP.UKR.NET**

```sh
openssl s_client -connect smtp.ukr.net:465 -crlf -ign_eof

EHLO localhost
AUTH PLAIN AZ1hcb1oZW5rb25hbGV4Y45kckBn5WFpbC8jb20ADTM0NDk0MA==
MAIL FROM: <mac2000@ukr.net>
RCPT TO: <alexandrm@rabota.ua>
DATA
Subject: TEST

TEST
.
QUIT
```

As to usual smtp:

```sh
telnet 192.168.5.1 25

EHLO localhost
MAIL FROM: alexandrm@rabota.ua
RCPT TO: marchenko.alexandr@gmail.com
DATA
TEST
.
QUIT
```
