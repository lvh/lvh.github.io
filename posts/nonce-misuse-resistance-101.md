<!--
.. title: Nonce misuse resistance 101
.. slug: nonce-misuse-resistance-101
.. date: 2016-05-19 12:25:44 UTC-07:00
.. tags: crypto, private
.. category:
.. link:
.. description: An introduction to nonce-misuse resistant cryptosystems
.. type: text
-->

*This post is an introduction to nonce-misused resistant cryptosystems and why
I think they matter. The first part of this post is about nonce-based
authenticated encryption schemes: how they work, and how they fail. If you're
already familiar with them, you can skip to the section on
[protocol design](#proto). If you're completely new to cryptography, you might
like my free introductory course to cryptography, [Crypto 101][c101]. In a
future blog post, I'll talk about some nonce-misuse resistant schemes I've
implemented using libsodium.*

Many stream ciphers and stream cipher-like constructions such as CTR,
GCM, (X)Salsa20... take a nonce. You can think of it as a pointer that lets
you jump to a particular point in the keystream. This makes these ciphers
"seekable", meaning that you can decrypt a small part of a big ciphertext,
instead of having to decrypt everything up to that point first. (That ends up
being trickier than it seems, because you still want to authenticate that
small chunk of ciphertext, but that's a topic for another time.)

The critical security property of a nonce is that it's never repeated under
the same key. You can remember this by the mnemonic that a *nonce* is a
"number used once". If you were to repeat the nonce, the keystream would also
repeat. That means that an attacker can take the two ciphertexts and XOR them
to compute the XOR of the plaintexts. If `C_n` are ciphertexts, `P_n`
plaintexts, `K_n` keystreams, and `^` is bitwise exclusive or:

```
C_1 = K_1 ^ P_1
C_2 = K_2 ^ P_2
```

The attacker just XORs `C_1` and `C_2` together:

```
C_1 ^ C_2 = K_1 ^ P_1 ^ K_2 ^ P_2
```

Since XOR is commutative (you can rearrange the order), `K_1 = K_2`, and
XOR'ing two equal values cancels them out:

```
C_1 ^ C_2 = P_1 ^ P_2
```

That tells an attacker a lot about the plaintext, especially if some of one of
the plaintexts is predictable. If the attacker has access to an encryption
oracle, meaning that they can get encryptions for plaintexts of their
choosing, they can even get perfect decryptions. That is not an unrealistic
scenario. For example, if you're encrypting session cookies that contain the
user name and e-mail, I can register using a name and e-mail address that has
a lot of `Z` characters, and then I know that just XORing with `Z` will reveal
most of the plaintext.

<a id="proto">

## Protocol design

For many on-like protocols like TLS, the explicit nonce provides a convenient
way to securely send many messages under a per-session key. Because the
critical security property for a nonce is that it is never repeated with the
same key, it's perfectly safe to use a counter. In protocols where both peers
send messages to each other, you can just have one peer use odd nonces and
have the other use even ones.

For off-line (or at-rest) protocols, it's a little trickier. You don't have a
live communication channel to negotiate a new ephemeral key over, so you're
stuck with longer-term keys or keys derived from them. If multiple systems are
participating, you need to decide ahead of time which systems own which
nonces. Even then, systems need to keep track of which nonces they've
used. That doesn't work well, especially not in a distributed system where
nodes and connections can fail at any time.

One solution is to use randomized nonces. Since nonces can't repeat, random
nonces should be large: if they're too small, you might randomly select the
same nonce twice, per the birthday bound. That is the only difference between
Salsa20 and XSalsa20: Salsa20 has a 64 bit nonce, whereas XSalsa20 has a 192
bit nonce. That change exists explicitly to make random nonces secure.

Picking a random nonce and just prepending it to the secretbox ciphertext is
secure, but there are a few problems with this approach. It's not clear to
practitioners that that's a secure construct. Doing this may seem obvious to a
cryptographer, but not to someone who just wants to encrypt a
message. Prepending a nonce doesn't feel much different from e.g. appending a
MAC. A somewhat knowledgeable practitioner knows that there's plenty of ways
to use MACs that are insecure, and they don't immediately see that the
prefix-nonce construction is secure. Not wanting to design your own
cryptosystems is a good reflex which we should be encouraging.

Random nonces also mean that any system sending messages needs access to
high-quality random number generators while they're sending a message. That's
often, but not always true. Bugs around random number generation, especially
userspace CSPRNGs, keep popping up. This is often a consequence of poor
programming practice, but it can also be a consequence of poorly-configured
VMs or limitations of embedded hardware.

## Nonce-misuse resistant systems

To recap, not all protocols have the luxury of an obvious nonce choice, and
through circumstances or poor practices, nonces might repeat
anyway. Regardless of how cryptographers feel about how important nonce misuse
is, we can anecdotally and empirically verify that such issues are real and
common. This is true even for systems like TLS where there is an "obvious"
nonce available ([Böck et al, 2016][bock16]). It's easy to point fingers, but
it's better to produce cryptosystems that fail gracefully.

[Rogaway and Shrimpton (2006)][rog06] defined a new model called nonce-misuse
resistance. Informally, nonce-misuse resistance schemes ensure that a repeated
random nonce doesn't result in plaintext compromise. In the case of a broken
system where the attacker can cause repeated nonces, an attacker will only be
able to discern if a particular message repeated, but they will not be able
to decrypt the message.

Rogaway and Shrimpton also later developed a mode of operation called SIV
(synthetic IV), which Gueron and Lindell are refined to GCM-SIV, a SIV-like
that takes advantage of fast GCM hardware implementations. Those two authors
are currently working with Adam Langley to standardize the AES-GCM-SIV
construction through CFRG. AEZ and HS1-SIV, two entries in the CAESAR
competition, also feature nonce-misuse resistance. CAESAR is an ongoing
competition, and GCM-SIV is not officially finished yet, so this is clearly
a field that is still evolving.

There are parallels between nonce-misuse resistance and length extension
attacks. Both address issues that arguably only affected systems that were
doing it wrong to begin with. (Note, however, in the embedded case above, it
might not be a software design flaw but a hardware limitation.) Fortunately,
the SHA-3 competition showed that you *can* have increased performance and
still be immune to a class of problems. I'm hopeful that CAESAR will consider
nonce-misuse resistance an important property of an authenticated encryption
standard.

## Repeated messages

Repeated messages are suboptimal, and in some protocols they might be
unacceptable. However, they're a fail-safe failure mode for nonce
misuse. You're not choosing to have a repeated ciphertext, you're just getting
a repeated ciphertext instead of a plaintext disclosure (where the attacker
would also know that you repeated a message). In the case of a secure random
nonce, a nonce-misuse resistant scheme is just as secure, at the cost of a
performance hit.

In a context where attackers can see individual messages to detect repeated
ciphertexts, it makes sense to also consider a model where attackers can
replay messages. If replaying messages (which presumably have side effects) is
a problem, a common approach is to add a validity timestamp. This is a feature
of Fernet, for example. A device that doesn't have access to sufficient
entropy will still typically have access to a reasonably high-resolution
clock, which is still more than good enough to make sure the synthetic IVs
don't repeat either.

## OK, but how does it work?

Being able to trade plaintext disclosure for attackers being able to detect
repeated messages sounds like magic, but it makes sense once you realize how
they work. As demonstrated in the start of this post, nonce re-use normally
allows an attacker to have two keystreams cancel out. That only makes sense if
two *distinct* messages are encrypted using the same (key, nonce) pair. NMR
solves this by making the nonce also depend on the message itself. Informally,
it means that a nonce should never repeat for two distinct
messages. Therefore, an attacker can't cancel out the keystreams without
cancelling out the messages themselves as well.

This model does imply off-line operation, in that the entire message has to be
scanned before the nonce can be computed. For some protocols, that may not be
acceptable, although plenty of protocols work around this assumption by simply
making individual messages sufficiently small.

*Thanks to Aaron Zauner and Kurt Griffiths for proofreading this post.*

[c101]: https://www.crypto101.io
[rog06]: http://web.cs.ucdavis.edu/~rogaway/papers/keywrap.pdf
[siv]: http://web.cs.ucdavis.edu/~rogaway/papers/siv.pdf
[bock16]: https://eprint.iacr.org/2016/475.pdf
