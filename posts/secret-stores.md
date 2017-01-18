<!--
.. title: Secret stores
.. slug: secret-stores
.. date: 2017-01-18 13:15:16 UTC-08:00
.. tags: private, security, crypto
.. category:
.. link:
.. description:
.. type: text
-->

Applications need secrets. TLS certificates, credentials for your database or
cloud provider: there are plenty of things your application may need access to
that you don't want anyone else to have access to.

This post is not about password managers. Password managers have many similar
design constraints, but are still fundamentally different beasts with, most
importantly, different users.

## Why secret stores?

At first sight, secret stores might sound like a fool's errand. Clearly, access
to secrets is a sensitive operation that you want to properly authenticate.
Machines authenticate themselves to other machines with high entropy secrets.
So, you still need to give the machine a secret anyway; and it seems like all
you've done is kick the can down the road a little.

There are plenty of useful features a secret store might get you:

* It knows how to encrypt and store secrets securely. Having one specialized
  application have an opinion on how to do that is much better than having a
  hundred ones that do it incidentally. The central one will be audited and
  monitored. The hundreds will invariably mess it up. As the old adage goes:
  defenders have to be right all the time, attackers only have to be right once.

* It tracks who accessed a secret and when. This is critical information for
  remediation and ongoing scope reduction. Knowing who accessed what and when
  probably gives you the context for why; all three tell you how to further
  reduce the authority that application has, or reconstruct a detailed audit
  trail.

* It can generate "minimal" credentials on-demand. Minimal means you get a key
  that only lets you access what you need and for a limited amount of time. For
  example, if you need to access specific AWS resources, you'd only be able to
  access those. On-demand means that they key can be created when you ask for
  it, and isn't shared with anyone else. (This is something dear to my heart;
  while [icecap][icecap] is anything but production software, it's essentially
  this idea turned up to eleven.)

* It can encrypt things on behalf of the requester, such that the requester
  never sees the key. That is good, because it can be one-way. It is also good
  because if a service is compromised, the compromise may be detected and
  remediated (access revoked) before all data is dumped and compromised. Having
  the secret store lets you do e.g. rate limiting and centralized monitoring,
  for example.

* Secret stores can know how secrets are linked; making it easier to do
  revocation, and easier to determine the impact of a breach or misuse incident.

* Once secrets are managed in a centralized place, it becomes feasible to have a
  coherent rotation strategy.

## Criteria

If I can help it, open source is better than closed source. I assume that major
cloud platform providers, like Google's GCP and Amazon's AWS, are already
fundamentally trusted; although less required trust is intrinsically better.
Having had professional audits is clearly important.

# Implementations

Oh boy. There are a few. This is by no means intended to be a complete overview,
although it's pretty comprehensive.

## As part of your orchestration

### Ansible Vault

### Chef encrypted bags and Chef vault

### Citadel

https://github.com/poise/citadel

## Other

### [PAL][pal]

### [SOPS][sops]

### [Torus][torus]

### [Ithos][ithos]

While it is by its own admission not yet ready for prime-time, it gets an
honorable mention for being built on an append-only cryptographically verifiable
log, a la Certificate Transparency. This has interesting security properties.

# Conclusions and recommendations

I'd like to thank the following people:

...


[icecap]: https://github.com/lvh/icecap
[pal]: https://www.youtube.com/watch?v=G_JXv059UY0
[sops]: https://github.com/mozilla/sops
[torus]: https://www.torus.sh/
[ithos]: https://github.com/cryptosphere/ithos
