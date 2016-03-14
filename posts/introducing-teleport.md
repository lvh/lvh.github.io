<!--
.. title: Introducing Teleport
.. slug: introducing-teleport
.. date: 2016-03-12 09:35:56 UTC-08:00
.. tags: private,security
.. category:
.. link:
.. description: Teleport is modern SSH infrastructure management.
.. type: text
-->

I'm happy to introduce [Teleport][teleport], a new open source platform for
managing SSH infrastructure. Teleport is built by [Gravitational][grav], a Y
Combinator company that ships SaaS on any platform. While I'm not a part of
Gravitational, I have been advising them on the Teleport project.

The authentication story at most teams just isn't very good. Some teams rely
on passing passwords around haphazardly, while others rely on copying
everyone's `~/.ssh/id_rsa.pub` to every new box. More complex homegrown
systems exist, but they quickly become unwieldy. These methods are problematic
both operationally and from a security perspective: when security and
usability are at odds, at some point security will lose out. For a lot of
teams, a single compromised key off of a developer machine spells disaster,
on-boarding new team members is painful, and key rotation doesn't happen.

In the last few years, strong multi-factor authentication has become the
norm. Tokens are only valid for a brief period of time, use challenge-response
protocols, or both. Teleport helps bring the same level of sophistication to
infrastructure. It helps system administrators leverage the security benefits
of short-lived certificates, while keeping the operational benefits of
decoupling server authentication from user authentication. It lets you run
isolated clusters, so that a compromise of staging credentials doesn't lead to
a compromise in production. It automatically maintains clear audit logs: who
logged in, when and where they logged in, and what they did once they got
there.

Teleport comes with a beautiful, usable UI, making it easy to visualize
different clusters and the available machines within them. The UI is optional:
many system administrators will prefer to use their existing SSH client, and
Teleport supports that natively.  Because it implements the `SSH_AUTH_SOCK`
protocol, integrating your current CLI workflow is a simple matter of setting
a single environment variable.

As someone with an open-source background, I'm glad to see this software
released and developed out in the open. A decent SSH key management story
should be available to everyone, and that's what Teleport does. A handful of
commercial systems with the same core functionality exist, but they are only
used in a fraction of environments, and don't always have the same feature
set. Making this technology more accessible is an important step in the right
direction for everyone. That includes commercial vendors; democratizing a good
infrastructure management story helps shift their product from an esoteric
security gadget to the battle-hardened and commercially supported version of
best practice. As a principal engineer at [Rackspace Managed Security][rms],
I'm excited to start working towards better authentication stories, both
internally and for our customers, with Teleport as the new baseline.

Releasing early and often is also an important part of open source
culture. That can be at odds with doing due diligence when releasing
security-critical systems like Teleport, especially when those systems have
non-trivial cryptographic components. We feel Teleport is ready to show to the
public now. To make sure we act as responsibly as possible, I've helped the
Teleport team to join forces with a competent independent third-party
auditor. We're not recommending that you bet the farm on Teleport by running
it in production as your only authentication method just yet, but we do think
it's ready for motivated individuals to start experimenting with it.

Some people might feel that a better SSH story means you're solving the wrong
problem. It seems at odds with the ideas behind immutable infrastructure and
treating server as [cattle, not pets][cattle]. I don't think that's
true. Firstly, even with immutable infrastructure, being able to SSH into a
box to debug and monitor is still incredibly important. Being able to rapidly
deploy a bunch of fixed images quickly may be good, but you still have to know
what to fix first. Secondly, existing systems don't always work that way. It
may not be possible, let alone economically rational, to "port" them
effectively. It's easy to think of existing systems as legacy eyesores that
only exist until you can eradicate them, but they do exist, they're typically
here to stay, and they need a real security story, too.

Teleport is still in its early stages. It's usable today, and I'm convinced it
has a bright future ahead of it. It's written in a beautiful, hackable Go
codebase, and [available on Github][teleport] starting today.

[teleport]: https://github.com/gravitational/teleport
[grav]: http://www.gravitational.com/
[rms]: https://www.rackspace.com/security/
[cattle]: https://blog.engineyard.com/2014/pets-vs-cattle
