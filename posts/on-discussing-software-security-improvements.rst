.. title: On discussing software security improvements
.. slug: on-discussing-software-security-improvements
.. date: 2014-07-28 23:58:27 UTC-07:00
.. tags: crypto, security, python
.. link:
.. description: How to discuss software security improvements.
.. type: text

A common criticism of information security folks is that they tend to
advise people to not do any crypto. Through projects like `Crypto
101`_, I've attempted to make a small contribution towards fixing
that.

In the open source world, various people often try to improve the
security of a project. Because designing secure systems is pretty
hard, they often produce flawed proposals. The aforemetioned tendency
for infosec-conscious people to tell them to stop doing crypto is
experienced as unwelcoming, even dismissive. Typically, the only thing
that's accomplished is that a lot of feelings get hurt; it seems to
only rarely result in improved software.

I think that's quite unfortunate. I think open source is great, and we
should be not just welcoming and inclusive, but aiming to produce
secure software. Furthermore, even if a flawed proposal is
unsalvageable, a clear description of *why* it is flawed will
presumably result in fewer negative interactions. Best case scenario,
the issues with a proposal can be discussed and potentially rectified.

In an effort to improve this situation, I'm documenting what I believe
to be a useful way to discuss security changes and their tradeoffs. As
Zooko has taught me:

  Security isn't about perfect versus imperfect or about better versus
  worse, it's about *this* attack surface versus *that* attack
  surface.

This document aims to be the equivalent of an SSCCE_ for generic bug
reports: a blueprint for making suggestions likely to lead to
productive discourse, as long as we can agree that we're trying to
produce more secure software, as well as provide a welcoming
development environment.

Important points
----------------

A good proposal should contain:

1. A brief description of what you're suggesting.
2. A description of the attack model you're considering, why the
   current system does not address this issue, and why the suggested
   system *does* address this issue.
3. A motivation of the attack model. Why is it important that this
   issue is actually addressed?
4. How does this change affect the attack surface (i.e. all of the
   ways an attacker can attempt to attack a system)?
5. What does the user experience for all users of the system look
   like? Many cryptosystems fall over because they're simply unusable
   for most users of the system.

An example
----------

Wul (the widely underestimated language, pronounced */wool/*) is a
general purpose programming language. It has a package repository,
WuPI (the Wul package index, pronounced */woopie/*), the de facto
standard for distributing and installing Wul software.

WuPI uses TLS as a secure transport. The WuF (Wul foundation,
pronounced */woof/*), maintains a root certificate, distributed with
Wul. Thanks to a well-managed system of intermediary CAs run by a
tireless army of volunteers, this means that both package authors and
consumers know they're talking to the real WuPI.

Alice is the WuPI BDFL. Bob is a Wul programmer, and would like to
improve the security of WuPI.

While consumers and authors know that they're talking to the real
WuPI, there is no protection against a malicious WuPI endpoint. (This
problem was recently made worse because WuPI introduced a CDN, greatly
increasing the number of people who could own a node.). You know that
you're talking to something with a WuF-signed certificate (presumably
WuPI, provided the WuF has done a good job managing that certificate),
but you have no idea if that thing is being honest about the packages
it serves you.

Bob believes WuPI could solve this by using GPG signatures.

He starts with a brief description of the suggestion:

  I would like to suggest that WuPI grows support for GPG signatures
  of packages. These signatures would be created when a package author
  uploads a package. They would optionally be verified when the user
  downloads a package.

He continues with the attack model being considered:

  I believe this would secure WuPI consumers against a malicious WuPI
  endpoints. A malicious WuPI endpoint (assuming it acquires an
  appropriate certificate) is currently free to deliver whatever
  packages it wants.

He explains why the current model doesn't address this:

  The current system assures authenticity and secrecy of the stream
  (through TLS), and it ensures that the server authenticates itself
  with a WuPI/WuF certificate. It does not ensure that the package is
  what the author uploaded.

He explains why he believes his model does address this:

  Because the signatures are produced by the author's GPG key, a
  malicious WuPI endpoint would not be able to forge them. Therefore,
  a consumer is sure that a package with a valid signature is indeed
  from the author.

He explains why this attack model is important:

  With the new CDN support, the number of people with access to such a
  certificate has greatly increased. While I certainly trust all of
  the volunteers involved, it would be nice if we didn't *have* to.
  Furthermore, the software on the servers can always be vulnerable to
  attack; as a high-value target, it certainly isn't inconceivable
  that an attacker would use an unknown vulnerability to take over a
  WuPI endpoint.

He (believes to) address the attack surface:

  Because the signatures are optional, the attack surface remains the
  same.

Finally, he addresses the user experience:

  The weak point of this scheme is most likely the user experience,
  because users historically seem to dislike using GPG.

  I am hopeful that this increased value of participating in the GPG
  web of trust will mean that more people participate.

Alice reviews this, and notes a flaw in the proposal:

  This proposal aims to address a security flaw when the WuPI endpoint
  is malicious by adding signatures. However, a malicious WuPI
  endpoint can lie by omission, and claim a package was never signed
  by the author.

Bob now realizes this issue, and suggests an improvement:

  This could be rectified if the user insists on a signature for
  packages they expect to be signed.

As a side note, Alice notes that the attack surface does increase:

  This places trust in author's ability to manage private keys, which
  has historically been shown to be problematic. That introduces a new
  attack vector: an attacker can attempt to go after the author's
  private key.

Regardless of the outcome of this conversation, there actually was a
conversation. I believe this to be an improvement over the overall
status quo.

.. _`Crypto 101`: https://www.crypto101.io/
.. _SSCCE: http://www.sscce.org/
