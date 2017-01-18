<!--
.. title: In defense of GPG
.. slug: in-defense-of-gpg
.. date: 2017-01-09 18:16:07 UTC-08:00
.. tags: private
.. category:
.. link:
.. description:
.. type: text
-->

Several smart people have written articles about [giving][guop] [up][moxie] on
GPG. This post isn't a repudiation; the points they raise are entirely valid.
Instead, I'll highlight some of the nuances.

GPG has serious issues, especially around UX. It isn't how we get the world
communicating securely. You want [Signal][signal] or maybe [WhatsApp][whatsapp]
if you're talking to non-nerds. If you want to send a file to each other and you
already have some reasonably secure medium to communicate a short string over,
check out [Magic Wormhole][mw]. Heck, even for computers talking to each other
you want [Fernet][fernet] or [libsodium][libsodium], not GPG.

And yet, there are things GPG is good at. In this post, I'm going to make one
specific case and generalize it. At [Latacora][latacora], we set up GPG with
clients so we can securely communicate with each other. We also use GPG to give
security researchers an option for the `security@` responsible disclosure
program.

The URL slugs for Filippo's article already show a little nuance.

There are a number of parts of the OpenPGP standard that I heartily suggest you
ignore. Even among security nerds, most people don't really understand how
subkeys work. Just generate new keys once in a while. It's fine. Nobody cares
about that ancient trust root. I know very few people that have the patience
for. Web-of-trust is overrated for almost everyone, too.

I know a few people with keys generated in the mid-to-late nineties. This feels
just like a low-digit ICQ number used to feel. It's for bragging rights, and
that's it.

Long-held keys are of course still useful. When you browse this website, you or
your computer don't first have to make a decision about what to trust on


[moxie]: https://moxie.org/blog/gpg-and-me/
[guop]: https://blog.filippo.io/giving-up-on-long-term-pgp/
[mw]: https://github.com/warner/magic-wormhole
[latacora]: https://latacora.com/
[libsodium]: https://download.libsodium.org/doc/
[fernet]: https://cryptography.io/en/latest/fernet/
