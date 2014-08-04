.. title: Secure key derivation performance
.. slug: secure-key-derivation-performance
.. date: 2014-08-04 04:29:15 UTC-07:00
.. tags: mathjax, private, crypto, security
.. link:
.. description:
.. type: text

I'm primarily interested in Clojure and Python.

Please note that I'm examining *high-entropy* key derivation
functions: I'm attempting to produce key material from an existing
high-entropy key. I am *not* trying to produce key material from a
low-entropy password; therefore PBKDF2, bcrypt, scrypt... are all poor
choices.

Requirements
------------

- The function takes two bytestrings with at least 128 bits of entropy
  each, and produces a bytestring of length 64 (bytes).
- The key derivation function must be secure.
- The key derivation must support `HKDF-style info parameter`_.
- The key derivation must be fast.
- Wide support is preferred, but not a blocker.

Contenders
----------

The obvious thing to use here is HKDF_. HKDF takes a hash function;
the most obvious one to use is SHA-256.

HKDF consists of two phases: an extract phase, which condenses the
entropy in some source material, and an expand phase, which turns that
entropy into a bunch of keying material. Since our keys are
high-quality keying material, we could omit the extraction phase.
Unfortunately, that also means we can't use a salt, so this is not
tested.

As an alternative, BLAKE2_ is examined. BLAKE2 is an improvement upon
BLAKE, a SHA-3 finalist. Like other SHA-3 finalists, no weaknesses
have been discovered in BLAKE, nor BLAKE2. However, BLAKE2 is
particularly interesting because it outperforms MD5, SHA-1, SHA-2, and
SHA-3 on modern Intel CPUs, despite being significantly more secure
than both SHA-1 and MD5.

Due to some of the differences between SHA-2-era hashes like SHA-256
and (post-)SHA-3-era hashes like BLAKE2, BLAKE2 does not necessarily
require the padding provided by HMAC (and by extension HKDF) in order
to be a secure PRF/KDF. BLAKE2 comes with a highly efficient PRF/MAC
mode built-in.

Furthermore, BLAKE2 directly provides a "personalization" parameter
similar to the "info" parameter of HKDF. While this is more limited in
BLAKE2 (a 16 byte bytestring) than it is in HKDF (an arbitrary
bytestring), this is arguably a non-limitation since it's quite easy
to hash an arbitrary length info parameter to a fixed-length 16 byte
string, particularly if you already have a BLAKE2 implementation lying
around.

There are two BLAKE2 implementations for Python, python-blake2_ and
pyblake2_. I used the latter, because the former failed to document
how to use the salt and personalization parameters. pyblake2, on the
other hand, had awesome docs with lots of examples.

Implementation
--------------

All implementations are on Github at `lvh/kdfperf`_. It'd be nice if
these were easier to run. In particular, if anyone feels inclined to
add a few Dockerfiles, that would be much appreciated.

Both the HKDF implementation and the BLAKE2 implementation come in
"one pass" and "multipass" varieties:

- in "one pass" mode, the total amount of key material is generated
  once and then chopped up into separate Python strings.
- in "multipass" mode, the key material is generated separately.

Results
-------

Here be dragons! Horribly unscientific, only run once, on my laptop,
with a bunch of other things open, code not peer reviewed yet! No error
bars! This might as well be total nonsense!

Results: https://gist.github.com/lvh/e2e4dfd0ce0d90776616

TL;DR:

- BLAKE2 outperforms the HMAC-SHA256 setup by ~80x (multipass) to
  ~100x (one-pass).
- Multipass is only about 7% slower than one-pass for BLAKE2, but the
  difference is almost 20% for HMAC-SHA256.

.. _HKDF: http://tools.ietf.org/html/rfc5869
.. _`HKDF-style info parameter`: http://tools.ietf.org/html/rfc5869#section-3.2
.. _`BLAKE2`: https://blake2.net/
.. _`BLAKE2 paper`: https://blake2.net/blake2_20130129.pdf
.. _python-blake2: https://github.com/darjeeling/python-blake2
.. _pyblake2: https://github.com/dchest/pyblake2
.. _`lvh/kdfperf`: https://github.com/lvh/kdfperf
