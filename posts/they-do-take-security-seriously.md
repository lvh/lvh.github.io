<!--
.. title: They do take security seriously
.. slug: they-do-take-security-seriously
.. date: 2015-07-05 13:17:18 UTC-07:00
.. tags: security
.. category:
.. link:
.. description:
.. type: text
-->

Earlier today, I read an [article][source] about the plethora of
information security breaches in recent history. Its title reads:

> “We take security seriously”, otherwise known as “We didn’t take it
> seriously enough”

The article then lists a number of companies informing the public that
they've been breached.

I think this article doesn't just blame the victims of those attacks,
but subjects them to public ridicule. Neither helps anyone, least of
all end users.

I'm surprised to hear such comments from Troy Hunt. He's certainly an
accomplished professional with extensive security experience. This is
not the first time people have expressed similar thoughts; the
[HN thread][hn] for that article is rife with them.

The explicit assumption is that these companies wouldn't have gotten
in trouble if only they had taken security more seriously. In a world
where the information services store is increasingly valuable and
software increasingly complex, breaches are going to happen. The idea
that getting breached is their own darn fault is unrealistic.

This idea is also counterproductive. Firstly, there's one thing all of
the victims being ostracized have in common: they disclosed the
details of the breach. That is exactly what they should have done;
punishing them creates a perverse incentive for victims to hide
breaches in the future, a decidedly worse end-user outcome.

Secondly, if any breach is as bad as any other breach, there is no
incentive to proactively mitigate damage from future breaches by
hardening internal systems. Why encrypt records, invest in access
control or keep sensitive information in a separate database with
extensive audit logging? It might materially impact end-user security,
but who cares -- all anyone is going to remember is that you got
popped.

Finally, there's a subtle PR issue: how can the security industry
build deep relationships with clients when we publicly ridicule them
when the inevitable happens?

These commentators have presumably not been the victims of a breach
themselves. I have trouble swallowing that anyone who's been through
the terrifying experience of being breached, seeing a breach up close
or even just witnessing a hairy situation being defused could air
those thoughts.

If you haven't been the victim of an attack, and feel that your
security posture is keeping you from becoming one, consider this:

1. What's your threat model?
2. How confident are you in your estimation of the capabilities of
   attackers?
3. Would you still be okay if your database became three orders of
   magnitude more valuable? Most personal data's value will scale
   linearly with the number of people affected, so if you're a small
   start-up with growth prospects, you'll either fail to execute, or
   be subject to that scenario.
4. Would you still be okay if the attacker has a few 0-days?
5. What if the adversary is a nation-state?
6. How do you *know* you haven't been breached?

That brings me to my final thesis: I contest the claim that all of the
companies in the article didn't take security seriously. It is far
more probable that all of the companies cited in the article have
expended massive efforts to protect themselves, and, in doing so,
foiled many attacks. It's also possible that they haven't; but the
onus there is certainly on the accuser.

Clearly, that's a weak form of disagreement, since "taking something
seriously" is entirely subjective. However, keep in mind that many
targets *actually* haven't taken security seriously, and would not
even have the technical sophistication to detect an attack.

(By the way, if you too would like to help materially improve people's
security, we're hiring. Contact me at `_@lvh.io`.)

[source]: http://www.troyhunt.com/2015/07/we-take-security-seriously-otherwise.html
[hn]: https://news.ycombinator.com/item?id=9834099
