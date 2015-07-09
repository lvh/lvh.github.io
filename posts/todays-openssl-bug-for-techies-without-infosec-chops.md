<!--
.. title: Today's OpenSSL bug (for techies without infosec chops)
.. slug: todays-openssl-bug-for-techies-without-infosec-chops
.. date: 2015-07-09 08:26:58 UTC-07:00
.. tags: security
.. category:
.. link:
.. description:
.. type: text
-->

# What happened?

OpenSSL 1.0.1n+ and 1.0.2b+ had a new feature that allows finding an
alternative certificate chain when the first one fails. The logic in
that feature had a bug in it, such that it didn't properly verify if
the certificates in the alternative chain had the appropriate
permissions; specifically, it didn't check if those certificates are
certificate authorities.

Specifically, this means that an attacker who has a valid certificate
for any domain, can use that certificate to produce new
certificates. Those normally wouldn't work, but the algorithm for
finding the alternative trust chain doesn't check if the valid
certificate can act as a certificate authority.

# What's a certificate (chain)?

A certificate is a bit like an ID card: it has some information about
you (like your name), and is authenticated by a certificate authority
(in the case of an ID, usually your government).

# What's a certificate authority?

A certificate authority is an entity that's allowed to authenticate
certificates. Your computer typically ships with the identity of those
certificate authorities, so it knows how to recognize certificates
authorized by them.

In the ID analogy, your computer knows how to recognize photo IDs
issued by e.g. California.

The issue here is that in some cases, OpenSSL was willing to accept
signatures authenticated by certificates that don't have certificate
authority powers. In the analogy, it would mean that it accepted
CostCo cards as valid ID, too.

# Why did they say it wouldn't affect most users?

This basically means "we're assuming most users are using OpenSSL for
vanilla servers", which is probably true. Most servers do use OpenSSL,
and most clients (browsers) don't.

The bug affects anyone trying to authenticate their peer. That
includes regular clients, and servers doing client
authentication. Regular servers aren't affected, because they don't
authenticate their peer.

Servers doing client authentication are fairly rare. The biggest
concern is with clients. While browsers typically don't use OpenSSL, a
lot of API clients do. For those few people affected by the bug and
with clients that use OpenSSL, the bug is catastrophic.

# What's client authentication?

The vast majority of TLS connections only authenticate the
server. When the client opens the connection, the server sends its
certificate. The client checks the certificate chain against the list
of certificate authorities that it knows about. The client is
typically authenticated, but over the protocol spoken inside of TLS
(usually HTTP), not at a TLS level.

That isn't the only way TLS can work. TLS also supports authenticating
clients with certificates, just like it authenticates servers. This is
called mutually authenticated TLS, because both peers authenticate
each other. At Rackspace Managed Security, we use this for all
communication between internal nodes. We also operate our own
certificate authority to sign all of those certificates.

# What's TLS?

TLS is what SSL has been called for way over a decade. The old name
stuck (particularly in the name "OpenSSL"), but you should probably
stop using it when you're talking about the secure protocol, since all
of the versions of the protocol that were called "SSL" have crippling
security bugs.

# Why wasn't this found by automated testing?

I'm not sure. I wish automated testing this stuff was easier. Since
I'm both a user and a big fan of client authentication, which is a
pretty rare feature, I hope to spend more time in the future creating
easy-to-use automated testing tools for this kind of scenario.

# How big is the window?

1.0.1n and 1.0.2b were both released on 11 Jun 2015. The fixes, 1.0.1p
and 1.0.2d, were released today, on 9 Jul 2015.

The "good news" is that the bad releases are recent. Most people who
have an affected version will be updating regularly, so the number of
people affected is small.

The bug affected following platforms (non-exhaustive):

* It did not affect OS X by default, because they still ship
  0.9.8. However, the bug does affect a stable version shipped through
  Homebrew (1.0.2c).
* [Ubuntu is mostly not affected][ubuntu]. The only affected version
  is the unreleased 15.10 (Wily). Ubuntu has already released an
  update for it.
* The bug affects stable releases of Fedora. I previously mistakenly
  reported that the contrary, but that information was based on their
  package version numbers, which did not match upstream. Fedora
  backported the faulty logic to their version of 1.0.1k, which was
  available in Fedora 21 and 22. They have since released patches; see
  [this ticket][fedora] for details. Thanks to Major Hayden for the
  correction!
* The bug does not affect Debian stable, but it does affect
  [testing and unstable][debian].
* The bug affects [ArchLinux testing][arch].

[ubuntu]: http://people.canonical.com/~ubuntu-security/cve/2015/CVE-2015-1793.html
[arch]: https://www.archlinux.org/packages/?sort=-last_update
[debian]: https://security-tracker.debian.org/tracker/CVE-2015-1793s=openssl
[fedora]: https://bugzilla.redhat.com/show_bug.cgi?id=1241544

# In conclusion

The bug is disastrous, but affects few people. If you're running
stable versions of your operating system, you're almost certainly
safe.

The biggest concern is with software developers using OS X. That
audience uses HTTPS APIs frequently, and the clients to connect to
those APIs typically use OpenSSL. OS X comes with 0.9.8zf by default
now, which is a recent revision of an ancient branch. Therefore,
people have a strong motivation to get their OpenSSL from a
third-party source. The most popular source is Homebrew, which up
until earlier this morning shipped 1.0.2c. The bug affects that
version. If you installed OpenSSL through Homebrew, you should go
update right now.
