<!--
.. title: Call for proposal proposals
.. slug: call-for-proposal-proposals
.. date: 2015-06-13 10:07:13 UTC-07:00
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

I'm excited to announce that I was invited to speak at PyCon
PL. Hence, I'm preparing to freshen up my arsenal of talks for this
year.

I'd like to do more security talks. Compared to previous talks, I'd
like to shift focus towards a more technical audience, going more
in-depth and touching on more advanced topics.

# Candidates

## Object-capability systems

Capabilities are a better way of thinking about authorization. A
capability ("cap") gives you the authority to perform some action,
without giving you any other authority. Unlike role-based access
control systems, capability based systems nearly always fail-closed;
if you don't have the capability, you simply don't have enough
information to perform an action. Contrast this with RBAC systems,
where authorization constraints are enforced with pinky swears, and
therefore often subverted.

I think I can make an interesting case for capability systems to any
technical audience with some professional experience. Just talk about
secret management, and how it's nearly always terrifying!

Of course, this gives me an opportunity to talk about `icecap` and
`shimmer`, my favorite pastimes.

## Putting a backdoor in `RDRAND`

I've [blogged about this before][rdrand-blog] before, but I think I
could turn it into a talk. The short version is that Linux's PRNG
mixes in entropy from the `RDRAND` in a way that would allow a
malicious implementation to control the output of the PRNG in ways
that would be indistinguishable by a (motivated) observer.

As a proof of concept, I'd love to demo the attack, either in software
(for example, with QEMU) or even in hardware with an open core. I
could also go into the research that's been done regarding hiding
stuff on-die. Unfortunately, the naysayers so far have relied on
moving the goalposts continuously, so I'm not sure that would convince
them this is a real issue.

## Retroreflection

An opportunity to get in touch with my languishing inner electrical
engineer! It turns out that when you zap radio waves at most hardware,
the reflection gets modulated based on what it's doing right now. The
concept became known as [TEMPEST][tempest], an NSA program. So far,
there's little public research on how feasible it is for your average
motivated hacker. This is essentially [van Eck phreaking][van-eck],
with 2015 tools. I imagine the biggest differences will be that it
should be a lot easier to interpret digital devices.

Demo ideas: read a keyboard. Or "wirelessly JTAG-debugging a device
which may or may not be yours" -- that'd be a catchy subtitle...

# The draft bin

## Underhanded curve selection

Another talk in the underhanded cryptography section I've considered
would be about underhanded elliptic curve selection. Unfortunately,
bringing the audience up to speed with the math to get something out
of it would be impossible in one talk slot. People already familiar
with the math are also almost certainly familiar with the argument for
rigid curves.

## Web app authentication

Some folks asked for a tutorial on how to authenticate to web
apps. I'm not sure I can turn that into a great talk. There's a lot of
general stuff that's reasonably obvious, and then there's highly
framework-specific stuff. I don't really see how I can provide a lot
of value for people's time.

# Feedback

[David Reid][dreid] and [Dwayne Litzenberger][dlitz] made similar,
excellent points. They both recommend talking about object-capability
systems. Unlike the other two, it will (hopefully) actually help
people build secure software. Also, the other two will just make
people feel sad. I feel like those points generalize to all attack
talks; are they just not that useful?

[rdrand-blog]: https://www.lvh.io/posts/2013/10/thoughts-on-rdrand-in-linux.html
[tempest]: https://en.wikipedia.org/wiki/Tempest_%28codename%29
[van-eck]: https://en.wikipedia.org/wiki/Van_Eck_phreaking
[dreid]: https://twitter.com/dreid
[dlitz]: https://twitter.com/DLitz
