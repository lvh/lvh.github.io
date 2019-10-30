<!--
.. title: Analyzing a simple encryption scheme using GitHub SSH keys
.. slug: analyzing-a-simple-encryption-scheme-using-github-ssh-keys
.. date: 2018-09-30 12:54
.. tags: cryptography
.. category:
.. link:
.. description:
.. type: text
-->

(This is an introductory level analysis of a scheme involving RSA. If you're already comfortable with Bleichenbacher oracles you should skip it.)

Someone pointed me at the following suggestion on the Internet for encrypting secrets to people based on their GitHub SSH keys. I like the idea of making it easier for people to leverage key material and tools they already have.  The encryption instructions are:

```sh
echo "my secret" > message.txt
curl -q github.com/$USER.keys | head -n 1 > recipient.pub
ssh-keygen -e -m pkcs8 -f recipient.pub > recipient.pem
openssl rsautl -encrypt -pubin -inkey recipient.pem -ssl \
    -in message.txt -out encrypted.txt
```

Anything using an openssl command line tool makes me a little uncomfortable. Let's poke at it a little.

We'll assume that that first key is really an RSA key and we don't have to worry about EdDSA or ECDSA (or heaven forbid, DSA). You're encrypting a password for someone. The straightforward threat model is an attacker who has the public key and ciphertext (but no plaintext) and wants to decrypt the ciphertext.

There are a few ways you can try to attack RSA schemes. You could attack the underlying math: maybe the keys were generated with insufficient entropy (e.g. [the Debian weak SSH keys problem][cve-2008-0166]) or bogus prime generation (e.g. [ROCA][cve-2017-15361]). In either case, you can generate the private key from the public key. These keys off of GitHub are likely OpenSSH-generated SSH keys generated on developer laptops and hence unlikely to have that sort of problem. It's also not specific to this scheme. (A real attacker would still check.)

Other attacks depend on the type of RSA padding used. The thing that sticks out about that `openssl rsautl -encrypt` is the `-ssl` flag. The man page claims:


> -pkcs, -oaep, -ssl, -raw
>
>    the padding to use: PKCS#1 v1.5 (the default), PKCS#1 OAEP, special padding used in SSL v2
>    backwards compatible handshakes, or no padding, respectively.  For signatures, only -pkcs
>    and -raw can be used.

iA! iA! I forgot that SSLv2 has its own weird padding variant: I remembered it as PKCSv15 from the last time I looked (DROWN). After some source diving (thanks pbsd!) I figured out that backwards-compatible SSLv2 padding is like PKCS1v15, but the first 8 bytes of the random padding are `0x03` ([PKCS1v15 code][rsa-pad-pkcsv15], [SSLv2 code][rsa-pad-sslv23]). That's weird, but OK: let's just say it's weird PKCSv15 and move on.

PKCS1v15 and its SSLv2 variant are both vulnerable to Bleichenbacher's oracle attack. That attack relies on being able to mess with a ciphertext and learn from how it fails decryption via an error message or a timing side channel. That doesn't work here: this model is "offline": the attacker gets a ciphertext and a public key, but they don't get to talk to anything that knows how to decrypt.  Hence, they don't get to try to get it to decrypt maliciously modified ciphertexts either.

There are lots of ways unpadded ("textbook") RSA is unsafe, but one of them is that it's deterministic. If *c = m<sup>e</sup> mod N* and an attacker is given a *c* and they can guess a bunch of *m*, they know _which_ *m* produced a particular *c*, and so decrypted the ciphertext. That sounds like a weird model at first, since the attacker comes up with *m* and just "confirms" it's the right one. It would work here regardless: passwords are often low-entropy and can be enumerated, that's the premise of modern password cracking.

But that's raw RSA, not PKCS1v15 padding, which is `EB = 00 || BT || PS || 00 || D` (see [RFC][pkcsv15]), where BT is the block type (here 02 for public key encryption). D is the data you're encrypting. PS is the  "padding string", which is a little confusing because the entire operation is padding. It's randomly generated when you're doing an RSA encryption operation. If you call the maximum size we can run through RSA k (the modulus size), the maximum length for PS is k - D - 3 (one for each null byte and one for the BT byte). The spec insists (and OpenSSL correctly enforces) this to be at least 8 bytes, or 64 bits of entropy. You can do about 50k/s public key operations on my dinky virtualized and heavily power throttled laptop. 64 bits is a bunch but not infinity. That's still not a very satisfactory result.

But wait--it's *not* PKCS1V15, it's that weird SSLv2 padding which sets the first 8 bytes of the padding string to 0x03 bytes. If the padding string is just 8 bytes long, that means the padding string is entirely determined. We can verify that by trying to encrypt a message of the appropriate size. For a 2048 bit RSA key, that's 2048 // 8 == 256 bytes worth of modulus, 3 bytes worth of header bytes and 8 bytes worth of padding, so a 256 - 8 - 3 == 245 byte message. You can go check that any 245 message encrypts to the same ciphertext every time. There's no lower bound on the amount of entropy in the ciphertext. A 244 byte message will encrypt to one of 256 ciphertexts: one for each possible pseudorandom padding value.

Practically, is this still fine? Probably, but only within narrow parameters. If the message you're encrypting is very close to 245 bytes and has plenty of structure known to the attacker, it isn't. If I can get you to generate a lot of these (say, a CI system automating the same scheme), it won't be. It's the kind of crypto that makes me vaguely uncomfortable but you'll probably get away with because there's no justice in the world.

There's a straightforward way to improve this. Remember how I said `-ssl` was weird? Not specifying anything would've resulted in the better-still-not-great PKCS1v15 padding. If you are going to specify a padding strategy, specify `-oaep`. OAEP is the good RSA encryption padding. By default, OpenSSL uses SHA1 with it (for both the message digest and in MGF1), which is fine for this purpose. That gives you 160 bits of randomness, which ought to be plenty.

This is why most schemes use RSA to encrypt a symmetric key.

Future blog posts: how to fix this, creative scenarios in which we mess with those parameters so it breaks, and how to do the same with ECDSA/EdDSA keys. For the latter: I asked a smart person and the "obvious" thing to them was not the thing I'd have done. For EdDSA, my first choice would be to convert the public key from Ed25519 to X25519 and then use NaCl box. They'd use the EdDSA key directly. So, if you're interested, we'll could do a post on that. And then we'd probably talk about why you use NIST P-256 for both signatures and ECDH, but different curve formats in djb country: Ed25519 for signatures and X25519 for ECDH. Oh, and we should do some entropy estimation. People often know how to do that for passwords, but for this threat model we also need to do that for English prose.

[pkcsv15]: [tools.ietf.org/html/rfc2...](https://tools.ietf.org/html/rfc2313)
[cve-2008-0166]: [cve.mitre.org/cgi-bin/c...](http://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2008-0166)
[cve-2017-15361]: [cve.mitre.org/cgi-bin/c...](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-15361)
[rsa-pad-sslv23]: [github.com/openssl/o...](https://github.com/openssl/openssl/blob/1212818eb07add297fe562eba80ac46a9893781e/crypto/rsa/rsa_ssl.c#L16-L53)
[rsa-pad-pkcsv15]: [github.com/openssl/o...](https://github.com/openssl/openssl/blob/1212818eb07add297fe562eba80ac46a9893781e/crypto/rsa/rsa_pk1.c#L117-L152)
