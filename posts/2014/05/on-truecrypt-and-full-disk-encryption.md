<!--
.. title: On TrueCrypt and full-disk encryption
.. date: 2014/05/29 10:53
.. slug: on-truecrypt-and-full-disk-encryption
.. link:
.. description:
.. tags: crypto
-->


Since the early hours of May 29th (CEST), [the TrueCrypt website
(`https://www.truecrypt.org`)](https://www.truecrypt.org) has pointed
to `http://truecrypt.sourceforge.net/`, with an ominous-looking error
message:

> WARNING: Using TrueCrypt is not secure as it may contain unfixed
> security issues
>
> This page exists only to help migrate existing data encrypted by
> TrueCrypt.
>
> The development of TrueCrypt was ended in 5/2014 after Microsoft
> terminated support of Windows XP. Windows 8/7/Vista and later offer
> integrated support for encrypted disks and virtual disk images. Such
> integrated support is also available on other platforms (click here
> for more information). You should migrate any data encrypted by
> TrueCrypt to encrypted disks or virtual disk images supported on
> your platform.

The website then explains how you can install BitLocker, a proprietary
disk encryption system available in many versions of Windows, as well
as how you could "rescue" existing TrueCrypt volumes.

People were pretty unhappy, for a variety of reasons:

- Lack of trust in proprietary full-disk encryption software.
- Many versions of Windows didn't even ship with BitLocker, only a few
  "premium" versions did.

As a proprietary system, BitLocker is not susceptible to the same
amount of public scrutiny as a publicly available system. This is in
stark contrast with TrueCrypt, where Matt Green recently raised around
70k USD to perform an [audit](http://istruecryptauditedyet.com/).

There are variety of scenarios that could've caused the TrueCypt
website to suddenly sport that message. The live Internet audience has
speculated wildly. I'll share my thoughts on that near the end of this
post, but there are two points I'd like to make that I think are far
more important:

1. Consider if full-disk encryption is really what you want.
2. If it is, consider if it should be TrueCrypt.

### Full-disk encryption

Something I learned from the inimitable [Zooko](https://zooko.com):

> Security isn't about perfect versus imperfect or about better versus
> worse, it's about *this* attack surface versus *that* attack
> surface.

You can't just add more crypto junk to something and expect to somehow
get better security. You have to consider what it is buying you, and
what it's costing you.

Full-disk encryption buys you one simple thing: if someone steals your
device while the encrypted volume is locked, they probably can't read
it.

If the encrypted volume is unlocked, it's over; and that's the state
it's probably usually in. If the key lives in RAM (it usually does),
there's a variety of ways that can be extracted. There's devices that
have complete direct memory access, including everything that speaks
FireWire or has FireWire-compatibility built-in. Even if you shut off
your machine, cold boot attacks mean that the key can be extracted for
a limited about of time. Various jurisdictions can try to force you to
hand over the keys.

If an attacker can write to the (encrypted!) volume, it's probably
over. Virtually all sector-level full-disk encryption formats are
unauthenticated (with good reason), they're all malleable and
vulnerable to (adaptive) chosen-ciphertext attacks.

If you're encrypting files or blobs of data within other files, an
authenticated encryption scheme like GPG is far more useful.

Don't get me wrong. Full-disk encryption is a good idea, and you
should do it. I'm just saying that what it actually protects against
is pretty limited. If there's files you want to keep secret, full-disk
encryption is probably not enough.

For more details, try Thomas & Erin Ptacek's
[blog post](http://sockpuppet.org/blog/2014/04/30/you-dont-want-xts/)
on why XTS isn't what you want. XTS is a way to build tweakable
ciphers that can be used to create full-disk encryption; but the post
applies to full-disk encryption generically as well.

### TrueCrypt as a full-disk encryption mechanism

So, you probably want full-disk encryption, and you probably want to
make sure that sensitive data is encrypted on top of that. Great. What
full-disk encryption scheme do you use?

I don't want to bash TrueCrypt. It is (was, perhaps?) a great go-to
project for full-disk encryption. It got a lot of things right. It
also got a bunch of things wrong.

TrueCrypt was made by a bunch of people we don't know occasionally
throwing a bunch of binaries over the wall. The source is available
(under a non-OSI license), so you could audit that and compile your
own binaries, but the truth is that the vast majority of TrueCrypt
users never actually did that.

The TrueCrypt disk format and encryption standards are a bit iffy. It
has terrible file system support. It has major performance issues,
partially due to questionable cryptographic practices such as cipher
cascades. It was great for when it was originally conceived and
full-disk encryption was still new and exciting, it's still pretty
decent now, but we can certainly do better.

### What actually happened to the website?

We don't really know. There's a couple of guesses.

First of all, it looks like it are the original authors that folded:
the key is the same one that was being used to sign releases months
ago. Of course, that could mean that it was compromised or that they
were forced to hand it over.

Since the DNS records changed, the e-mail server behavior changed, the
same key was used, the Sourceforge client was involved, and fairly
major changes to the source code were involved, it's unlikely that
it's a simple defacement.

It's unlikely that they folded because they felt discovery of some
backdoor was imminent. Folding wouldn't actually stop that discovery,
because the source was already open.

It's strange that they would point to alternatives like Bitlocker for
Windows and even more dubious alternatives for other operating
systems. The authors certainly knew that this would not be an
acceptable alternative for the vast majority of their users.

Additionally, the support window for XP ending isn't a particularly
convincing impetus for migrating away from TrueCrypt *right now*. This
has lead some to believe that it's an automated release. Worse, it
could be a gagged response: a big and powerful three-letter agency
might be twisting their arm and forcing them to fold, in return for
something else (like, say, not being thrown in a Gitmo cell and
forgotten about). Again: pure speculation.

Another option is that some of the developers just really, genuinely
folded. They felt that for atechnical users, BitLocker was good
enough, while the particularly discerning TrueCrypt user would be able
to find some other alternative.

Bottom line is that none of this really matters. What matters is what
you should do next if you want to have full-disk encryption.

### So what do I do now?

I think LUKS is probably the best system that we have right now, and
one of the few that actually improves on TrueCrypt.

If you're on Linux, dm-crypt + cryptsetup + LUKS is probably what you
want. If you're on OS X, FileVault 2 is probably what you want. If
you're on Windows, your options are looking a bit thin right now:

- Keep using old versions of TrueCrypt.
- Give up and use BitLocker.
- Use a Linux virtual machine to use dm-crypt/LUKS, eventually
  migrating to native support.

Personally, I think the first option is the most reasonable right now,
and then wait until someone actually writes the native LUKS support.

Oh, and you should probably install GPG to encrypt some of those files
stored on that encrypted volume.
