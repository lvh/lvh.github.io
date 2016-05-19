<!--
.. title: Automatic nonce and nonce-misuse resistant schemes for libsodium
.. slug: automatic-nonce-and-nonce-misuse-resistant-schemes-for-libsodium
.. date: 2016-05-09 14:58:10 UTC-07:00
.. tags: crypto, private
.. category:
.. link:
.. description:
.. type: text
-->

The `libsodium` library is a collection of modern cryptographic primitives and
related tools. It has a handful of authenticated encryption APIs, two for
asymmetric encryption (box, seal) and two for symmetric encryption primitives
(secretbox, aead).

Under the hood, most of these cryptosystems use the same symmetric primitives:
the XSalsa20 stream cipher, and the Poly1305 MAC. The AEAD constructions, use
the ChaCha20+Poly1305 or AES-GCM.



## Automatic nonce schemes


## Security

## Performance

## Adding this to libsodium
