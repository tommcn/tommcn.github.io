---
title: "How does TLS work"
date: 2021-10-24T12:00:00+00:00
featureImage: https://miro.medium.com/max/700/1*ner-yFUzwHIG_nkxW9C29g.png
postImage: https://miro.medium.com/max/700/0*g-u8XnVc7abfIlbH
# tags: c
categories: blog
toc: true
mediumLink: https://medium.com/visionary-hub/how-does-tls-work-87ff948d5410
---
# How Does TLS Work

Introduction

![Photo by [Richy Great](https://unsplash.com/@richygreat?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12000/0*nFVaDLe1uPdfwPFK)*Photo by [Richy Great](https://unsplash.com/@richygreat?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

I’m sure we’ve all been told to make sure that our browser shows a little lock icon when visiting a website. You probably know that the “lock” means that your connection to the web server is secure, no one can read your passwords or credit card information except for you and the server. You also probably know that the “lock” represents HTTPS (with the S standing for “secure”), whereas the lack of a lock represents HTTP. You’ve probably encountered warnings from browsers that tell you that they can’t establish a secure connection to the website you want to reach. All of this is common knowledge or experience. But do you really know what all of this means and how HTTPS works behind the scene? Enters **TLS**.

## What is TLS

Basically, **TLS provides encryption for communication between two peers**. TLS is used when web browsers load a secure webpage and even when you send an email or use VoIP software. Many protocols rely on TLS: HTTPS, SFTP, SMTP… (basically most protocols that have an “S” for secure in their name). TLS provides a secure communication channel that is free of tampering and snooping.

![Photo by [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/11520/0*g-u8XnVc7abfIlbH)*Photo by [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

However TLS is not the end all be all of security, it doesn’t guarantee the safety of your data; it only guarantees that the transportation of information over the internet is safe. The responsibility of safeguarding your information lands on the servers’ end. We also need to keep in mind that TLS relies on encryption; as computing power increases year after year, TLS will need to keep up. Already, 5 of TLS 1.2 predecessors are considered insecure due to various underlying problems.

## Quick history of TLS

![By Indolering — Own work, CC0, [https://commons.wikimedia.org/w/index.php?curid=42758689](https://commons.wikimedia.org/w/index.php?curid=42758689)](https://cdn-images-1.medium.com/max/2234/1*xcPLGhUmiQEn1VZQZd8QsA.png)*By Indolering — Own work, CC0, [https://commons.wikimedia.org/w/index.php?curid=42758689](https://commons.wikimedia.org/w/index.php?curid=42758689)*

In the mid 1990s, web browsers were still emerging. **Netscape**, which had a market share of around 75% at the time, **proposed a protocol for secure communication between servers and clients**. This new protocol was called SSL (Secure Socket Layer). SSL 1.0 never released due to severe security flaws. SSL 2.0 was released in 1995 it also contained serious security flaws and made many assumptions making it unusable at the time. After these 2 failed protocols, a complete redesign of the protocol was released as SSL 3.0 in 1996.
At this time, **Microsoft’s Edge and Netscape’s Navigator were competing for market share**. This led to both browsers implementing many exclusive features in their respective browsers. In 1996, both Microsoft and Netscape met and decided to support the [IETF](https://www.ietf.org/) (Internet Engineering Task Force) in updating and standardizing the SSL protocol, instead of having both of them create their own proprietary encryption software. **A new protocol was released, dubbed TLS 1.0** (Transport Layer Security). This new standard was not widely different from its predecessor. Microsoft proposed the name change so that TLS was not associated with all the problems of SSL. At the time of writing, TLS 1.2 is the most widely used to this day, but the TLS version 1.3 was released in 2018 and contains many cryptographic improvements. Both TLS 1.2 and 1.3 are considered secure. TLS versions 1.1 and 1.0 and all SSL versions are considered insecure and should not be used. In fact, most major browsers will display a warning upon entering a site that uses an outdated protocol.

## Quick primer on cryptography

Before we jump into the nitty-gritty details of TLS’s inner working, here’s a small primer on asymmetric and symmetric cryptography.

### Asymmetric Cryptography — Public-key cryptography

![By Davidgothberg — Own work, Public Domain, [https://commons.wikimedia.org/w/index.php?curid=1028460](https://commons.wikimedia.org/w/index.php?curid=1028460)](https://cdn-images-1.medium.com/max/2000/1*QSe6suUJU4CqEwh-MEh5Lg.png)*By Davidgothberg — Own work, Public Domain, [https://commons.wikimedia.org/w/index.php?curid=1028460](https://commons.wikimedia.org/w/index.php?curid=1028460)*

In asymmetric encryption, the client uses the server’s **public key** to encrypt a message. The message can only be decrypted using the server’s **private key**, which is kept secret. This allows anyone to use the public key to form messages only the server can understand. It is very important that the secret key be kept secret, otherwise, anyone would be able to decrypt messages. Since the private and public key are **mathematically linked**, the keys need to be long enough that brute forcing, while technically possible, would take a very long time (roughly 300 trillions years on a standard computer). The current recommended length for keys is between **1024 and 4096 bytes**. While being cryptographically very strong, the calculations are time-consuming, which makes it impractical for long streams of data (like a viewing a website).

### Symmetric Cryptography

In symmetric cryptography, only one key is used, called a **secret** (often around **150 bits**). This secret is used to encrypt **and** decrypt all messages between both peers. This is significantly faster than asymmetric cryptography and more convenient as it doesn’t require the server to hold onto a private key. However, for this to be effective both parties need to agree on a **secret without someone snooping on the communication gaining access to the secret**.

## TLS

As we will see later on, TLS used a mix of both asymmetric and symmetric cryptography to ensure that all communications are kept secret.

### The process

![TCP 3-way handshale — [https://www.guru99.com/tcp-3-way-handshake.html](https://www.guru99.com/tcp-3-way-handshake.html)](https://cdn-images-1.medium.com/max/2000/1*NUlx_Xi98RBs6LbrkCmtVw.png)*TCP 3-way handshale — [https://www.guru99.com/tcp-3-way-handshake.html](https://www.guru99.com/tcp-3-way-handshake.html)*

TLS is an internet protocol that builds upon the TCP protocol, which powers most high-level protocols. As with any TCP connection between two peers, the connection starts with what’s known as the “three-way handshake”. The client starts by sending a message called **SYN** (for synchronize), where the client sends some information regarding the Initial Sequence Number (this number is used to order all TCP packets). The server then sends a **SYN ACK** message (synchronize — acknowledge). The server sends a synchronize message, followed by an acknowledgment that the server has received the first SYN packet. Finally, the client sends a final **ACK** (acknowledgment) to the server. From there, a TCP connection is established. This is TCP’s answer to the [Byzantine generals problem](https://en.wikipedia.org/wiki/Byzantine_fault).
After a TCP connection is established, the client and servers start the TLS handshake process, which is more complex. Due to the variety of cipher suites (a set of algorithms that help secure a network connection), the exact functioning of the TLS handshakes differs. This is the process used with the **RSA cipher suite**, which is the most commonly used.

1. **Client hello**
Since TLS supports many cipher suites and not all ciphers are supported by all implementations of TLS, the client sends the ciphers it supports in order of preference, followed by the TLS versions it supports and finally it sends 28 bytes generated by a **Cryptographically Secure Random Number Generator**, known as client secret. This message can be sent at any time by the client to the server to start a TLS connection.

1. **Server Hello**
The server sends the server hello message in response to the client hello. It includes the chosen TCP version, the chosen cipher suite (from the list sent by the client), and its SSL certificate (more information on that later). Finally it includes 28 bytes of random data known as server secret

1. **Certificate validation**
At this stage the client verifies the authenticity and credibility of the server’s SSL certificate (more information on those certificates later). The client does this by checking the Certificate Authority which issued the certificate and the signature of the certificate. If the client decides that the certificate is legit, it continues the handshake.

1. **Premaster** **secret**
Now that the client has the server’s public key (available from the server’s SSL certificate), it sends another secret, this time encrypted with the server’s public key. This is very important as it prevents snoopers from gaining access the decrypted premaster secret.

1. **Session key creation**
With the client random, server random and the decrypted premaster secret, the client and the server create a session key. Both session keys should be equal.

1. **Finished**
The client and the server send a “finished” message to indicate they have finished computing the session key.

![The TLS Handshake — Cloudflare](https://cdn-images-1.medium.com/max/5836/1*tsMmW2P9UM6UnD34mmMm0g.png)*The TLS Handshake — Cloudflare*

Now the client and the server have access to a shared secret key and are able to start an encrypted connection using symmetric encryption.

## SSL Certificates

SSL certificates are a way to for server to prove their authenticity. SSL certificates are actually X.509 certificates. An X.509 certificate’s primary purpose is to **communicate the public key of the server and prove its authenticity**. It obviously contains the server’s public key. A typical certificate will also include metadata like the issuing date, expiration date and the name of the user. It also states more information regarding the user, mainly the domain or subdomains the certificate is valid for. Finally, it contains the name of the CA (Certificate Authority, more information below) and the signature of the CA.

![By jaydeep_ — [https://pixabay.com/en/hacking-cybercrime-cybersecurity-3112539/](https://pixabay.com/en/hacking-cybercrime-cybersecurity-3112539/) archive copy, CC0, [https://commons.wikimedia.org/w/index.php?curid=69573226](https://commons.wikimedia.org/w/index.php?curid=69573226)](https://cdn-images-1.medium.com/max/16000/1*ner-yFUzwHIG_nkxW9C29g.png)*By jaydeep_ — [https://pixabay.com/en/hacking-cybercrime-cybersecurity-3112539/](https://pixabay.com/en/hacking-cybercrime-cybersecurity-3112539/) archive copy, CC0, [https://commons.wikimedia.org/w/index.php?curid=69573226](https://commons.wikimedia.org/w/index.php?curid=69573226)*

### Certificate Authorities

SSL certificates can prove their authenticity using Certificate Authorities (CA). **The CA acts as the issuer of certificate**. If a CA is trusted by the client, then all of the certificates issued by it would be trusted by default. CAs are supposed to verify that the person the certificate is issued to is indeed the owner of the domain for which the certificate is issued. **The certificate acts as a proof of ownership of the website** in addition to providing the server’s secret key.
Individuals are also able to create their own CAs, but those will not be trusted by default by most modern browsers. However, they can be useful in instances where connection to internet cannot be guaranteed (for example, communication between pods in Kubernetes)

## Conclusion

In conclusion, TLS provides a much needed layer of security over the TCP protocol. In its most common use case, TLS guarantees the privacy of your personal information. But those benefits only come in play if you ensure that you are using the latest versions of the protocols to not fall victims to common vulnerabilities. As major companies like Google and Mozilla push to make the web safer, TLS will play an essential role in securing the web for all its billions of users.
If you want to learn more about TLS, I would highly suggest reading the [original RFC](https://datatracker.ietf.org/doc/html/rfc5246) (fair warning, it’s 100 pages of highly technical writing), it goes into great details about the standards and includes more information about the cryptography of it all. 
If you want to learn more about your browsers built-in security features, visit [BadSSL](https://badssl.com/).

*If you liked this article, consider following me on Medium, or connect with me on [LinkedIn](https://www.linkedin.com/in/tomas-mc/).*

## References

[https://datatracker.ietf.org/doc/html/rfc5246](https://datatracker.ietf.org/doc/html/rfc5246)
[https://datatracker.ietf.org/doc/html/rfc5280](https://datatracker.ietf.org/doc/html/rfc5280)
[https://datatracker.ietf.org/doc/html/rfc8446](https://datatracker.ietf.org/doc/html/rfc8446)

[https://sectigostore.com/blog/ssl-vs-tls-decoding-the-difference-between-ssl-and-tls/](https://sectigostore.com/blog/ssl-vs-tls-decoding-the-difference-between-ssl-and-tls/)
[http://tim.dierks.org/2014/05/security-standards-and-name-changes-in.html](http://tim.dierks.org/2014/05/security-standards-and-name-changes-in.html)
[https://www.cloudflare.com/en-ca/learning/ssl/what-happens-in-a-tls-handshake/](https://www.cloudflare.com/en-ca/learning/ssl/what-happens-in-a-tls-handshake/)
[https://www.internetsociety.org/deploy360/tls/basics/](https://www.internetsociety.org/deploy360/tls/basics/)
[https://www.cloudflare.com/en-ca/learning/ssl/what-is-an-ssl-certificate/](https://www.cloudflare.com/en-ca/learning/ssl/what-is-an-ssl-certificate/)
[https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/three-way-handshake-via-tcpip](https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/three-way-handshake-via-tcpip)
[https://www.cryptologie.net/article/340/tls-pre-master-secrets-and-master-secrets/](https://www.cryptologie.net/article/340/tls-pre-master-secrets-and-master-secrets/)
[https://sectigo.com/resource-library/what-is-x509-certificate](https://sectigo.com/resource-library/what-is-x509-certificate)
[https://www.guru99.com/tcp-3-way-handshake.html](https://www.guru99.com/tcp-3-way-handshake.html)
[https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)
