<!--
.. title: Conflicting threat models
.. slug: conflicting-threat-models
.. date: 2015-03-07 08:56:05 UTC-08:00
.. tags: security
.. link:
.. description:
.. type: text
-->

As I mentioned [in my previous post][prev], we have a long way to go when it comes to information security. I'll be presenting [a talk on building secure systems][sec-talk] at PyCon 2015 next month, and I hope to blog more about interesting bits of comprehensible security.

I believe strongly in the importance of a threat models. A threat model is your idea of what you're protecting against. It may seem obvious that you can't effectively protect anything without knowing what you're protecting it from. Sadly, simply contemplating your threat model puts you ahead of the curve in today's software industry.

Threat models often simply deal with how much effort you're willing to spend to prevent something from happening. In a world with finite resources, we have to make choices. Some models are unrealistic or prohibitively expensive to defend against. These questions aren't all strictly technical: perhaps some risk is adequately covered by insurance. Perhaps you have a legal or a compliance requirement to do something, even if the result is technically inferior. These questions are also not just about *how much* you're going to do: different threat models can lead to mutually exclusive resolutions, each a clear security win.

Consider your smartphone. Our phones have a lot of important, private information; it makes sense to protect them. The iPhone 6 provides two options for the lock screen: a passcode and a fingerprint sensor. Passcodes have been around for about as long as smartphones have, while fingerprint sensors are new and exciting. It's clear that either of them is more secure than not protecting your phone at all. But which one is more secure?

Most people instinctively feel the fingerprint sensor is the way to go. Biometric devices feel advanced; up until recently, they only existed in Hollywood. Fingerprints have their share of issues. It's impossible to pick a new key or have separate keys for separate capabilities; you're stuck with the keys you have. A fingerprint is like a password that you involuntarily leave on everything you touch. That said, turning a fingerprint into something that will unlock your iPhone is out of reach for most attackers.

Passcodes aren't perfect either. People generally pick poor codes: important dates and years are common, but typically not kept secret in other contexts. If you know someone's birthday, there's a decent chance you can unlock their phone. At least with a passcode, you have the option of picking a good one. Even if you do, a passcode provides little protection against shoulder surfing. Most people unlock their phone dozens of times per day, and spend most of that day in the presence of other people. A lot of those people could see your passcode inconspicuously.

Two options. Neither is perfect. How do you pick one? To make an informed choice, you need to formalize your threat models.

In the United States, under the Fifth Amendment, you don't have to divulge information that might incriminate you. I am not a lawyer, and courts have provided conflicting rulings, [but currently it appears that this includes computer passwords.][passwords] [However, a court has ruled that a fingerprint doesn't count as secret information.][ruling] If you can unlock your phone with your fingerprint, they can force you to unlock it.

If your threat models include people snooping, the fingerprint sensor is superior. If your threat model includes law enforcement, the passcode is superior. So, which do you pick? It depends on your threat model.

Disclaimer: this is an illustration of how threat models can conflict. It is *not* operational security advice; in which case I would point out other options. It is not legal advice, which I am not at all qualified to dispense.

[prev]: http://www.lvh.io/posts/were-just-getting-started.html
[sec-talk]: https://us.pycon.org/2015/schedule/presentation/342/
[distsys-talk]: https://us.pycon.org/2015/schedule/presentation/386/
[passwords]: https://en.wikipedia.org/wiki/Fifth_Amendment_to_the_United_States_Constitution#Computer_passwords
[ruling]: http://hamptonroads.com/2014/10/police-can-require-cellphone-fingerprint-not-pass-code?wpisrc=nl-swbd&wpmm=1#
