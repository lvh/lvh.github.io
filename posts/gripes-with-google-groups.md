<!--
.. title: Gripes with Google Groups
.. slug: gripes-with-google-groups
.. date: 2018-05-29 16:14
.. tags: security
.. category:
.. link:
.. description:
.. type: text
-->

If you’re like me, you think of Google Groups as the Usenet client turned
mailing list manager. If you’re a GCP user or maybe one of a handful of SAML
users you probably know Google Groups as an access control mechanism. The bad
news is we’re both right.

This can blow up if permissions on those groups aren't set right. Your groups
were probably originally created by a sleep-deprived founder way before anyone
was worried about access control. It's been lovingly handcrafted and never
audited ever since. Let’s say their configuration is, uh, “inconsistent”. If an
administrator adds people to the right groups as part of their on-boarding, it’s
not obvious when group membership is secretly self-service. Even if someone
can't *join* a group, they might still be able to read it.

You don’t even need something using group membership as access control for this
to go south. The simplest way is a password reset email. (Having a list of all
of your vendors feels like a dorky compliance requirement, but it's underrated.
Being able to audit which ones have multi-factor authentication is awesome.)

Some example scenarios:

*Scenario 1* You get your first few customers and start seeing fraud. You create
a mailing list with the few folks who want to talk about that topic. Nobody
imagined that dinky mailing list would grow out to a full-fledged team, let alone
one with permissions to a third party analytics suite that has access to all
your raw data.

*Scenario 2* Engineering team treats their mailing list as open access for the
entire company. Ops deals with ongoing incidents candidly and has had bad
experiences with nosy managers looking for scapegoats. That’s great until
someone in ops extends an access control check in some custom software
that gates on ops@ to also include engineering@.

*Scenario 3* board@ gets a new investor who insists on using their existing
email address. An administrator confuses the Google Groups setting for allowing out-of-domain
addresses with allowing out-of-domain registration. Everyone on the Internet can
read the cap table for your next funding round.

This is a mess. It bites teams that otherwise have their ducks in a row.
Cleaning it up gets way worse down the line. Get in front of it now and you
probably won’t have to worry about it until someone makes you audit it, which is
probably 2-3 years from now.

Google Groups has some default configurations for new groups these days:

* Public (Anyone in ${DOMAIN} can join, post messages, view the members list,
  and read the archives.)
* Team (Only managers can invite new members, but anyone in ${DOMAIN} can
  post messages, view the members list, and read the archives.)
* Announcement-only (Only managers can post messages and view the members list,
  but anyone in ${DOMAIN} can join and read the archives.)
* Restricted (Only managers can invite new members. Only members can post
  messages, view the members list, and read the archives. Messages to the group
  do not appear in search results.)

This is good but doesn't mean you're out of the woods:

* These are just defaults for access control settings. Once a group is created,
  you get to deal with the combinatorial explosion of options. Most of them
  don't really make sense. You probably don't know when someone messes with the
  group, though.
* People rarely document intent in the group description (or anywhere for that
  matter). When a group deviates, you have no idea if it was supposed to.
* "Team" lets anyone in the domain read. That doesn't cover "nosy manager" or
  "password  reset" scenarios.

Auditing this is kind of a pain. The UI is slow and relevant controls are
spread across multiple pages. Even smallish companies end up with dozens of
groups. The only way we've found to make this not suck is by using the GSuite
Admin SDK and that's a liberal definition of "not suck".

You should have a few archetypes of groups. Put the name in the group itself,
because that way the expected audience and access control is obvious to users
and auditors alike. Here are some archetypes we've found:

* Team mailing lists, should be called xyzzy-team@${DOMAIN}. Only has team
  members, no external members, no self-service membership.
* Internal-facing mailing lists, should be called xyzzy-corp@${DOMAIN}. Public
  self-serve access for employees, no external members, limit posting to domain
  members or mailing list members. These are often associated with a team, but
  unlike -team mailing lists anyone can join them.
* External-facing lists. Example: contracts-inbound@${DOMAIN}. No self-serve
  access, no external members, but anyone can post.
* External member lists (e.g. boards, investors): board-ext@${DOMAIN}. No
  self-serve access, external members allowed, members and either members or
  anyone at the domain can post.

PS: Groups can let some users post as the group. I haven't ran a phishing
exercise that way, but I'm guessing an email appearing to legitimately come from
board@company.com is going to be pretty effective.
