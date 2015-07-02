<!--
.. title: HTTPS requests with client certificates in Clojure
.. slug: https-requests-with-client-certificates-in-clojure
.. date: 2015-07-02 08:53:20 UTC-07:00
.. tags: security
.. category:
.. link:
.. description:
.. type: text
-->

The vast majority of TLS connections only authenticate the
server. When the client opens the connection, the server sends its
certificate. The client checks the certificate against the list of
certificate authorities that it knows about. The client is typically
authenticated, but over the inner HTTP connection, not at a TLS level.

That isn't the only way TLS can work. TLS also supports authenticating
clients with certificates, just like it authenticates servers. This is
called mutually authenticated TLS, because both peers authenticate
each other. At Rackspace Managed Security, we use this for all
communication between internal nodes. We also operate our own
certificate authority to sign all of those certificates.

One major library, [`http-kit`][http-kit], makes use of Java's
`javax.net.ssl`, notably `SSLContext` and `SSLEngine`. These Java APIs
are exhaustive, and very... Java. While it's easy to make fun of these
APIs, most other development environments leave you using OpenSSL,
whose APIs are patently misanthropic. While some of these APIs do
leave something to be desired, [aphyr][aphyr] has done a lot of the
hard work of making them more palatable with
[`less-awful-ssl`][less-awful-ssl]. That gives you an
`SSLContext`. Request methods in `http-kit` have an `opts` map that
you can pass a `:sslengine` object to. Given an `SSLContext`, you just
need to do `(.createSSLEngine ctx)` to get the engine object you want.

Another major library, [`clj-http`][clj-http], uses lower-level
APIs. Specifically, it requires [`KeyStore`][keystore] instances for
its `:key-store` and `:trust-store` options. That requires diving deep
into Java's cryptographic APIs, which, as mentioned before, might be
something you want to avoid. While `clj-http` is probably the most
popular library, if you want to do fancy TLS tricks, you probably want
to use `http-kit` instead for now.

My favorite HTTP library is [`aleph`][aleph] by
[Zach Tellman][ztellman].  It uses Netty instead of the usual Java IO
components. Fortunately, Netty's API is at least marginally friendlier
than the one in `javax.net.ssl`. Unfortunately, there's no
`less-awful-ssl` for Aleph. Plus, since I'm using [`sente`][sente] for
asynchronous client-server communication, which doesn't have support
for `aleph` yet. So, I'm comfortably stuck with `http-kit` for now.

In conclusion, API design *is* UX design. The library that "won" for
us was simply the one that was easiest to use.

For a deeper dive in how TLS and its building blocks work, you should
watch my talk, [Crypto 101][talk], or the matching [book][book]. It's
free! Oh, and if you're looking for information security positions
(that includes entry-level!) in an inclusive and friendly environment
that puts a heavy emphasis on teaching and personal development, you
should get in touch with me at `_@lvh.io`.

[clj-http]: https://github.com/dakrone/clj-http
[http-kit]: https://github.com/http-kit/http-kit
[aphyr]: https://aphyr.com/
[less-awful-ssl]: https://github.com/aphyr/less-awful-ssl
[http-kit-sslengine]: https://github.com/http-kit/http-kit/blob/230d5aabe3f2a410000dca3f4b5de78464963fcc/src/org/httpkit/client.clj#L67-L68
[aleph]: http://aleph.io/
[ztellman]: http://ideolalia.com/
[sente]: https://github.com/ptaoussanis/sente
[talk]: https://www.youtube.com/watch?v=3rmCGsCYJF8
[book]: https://www.crypto101.io
