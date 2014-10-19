.. title: Reverse ungineering
.. slug: reverse-ungineering
.. date: 2014-10-07 02:44:39 UTC-07:00
.. tags: private
.. link:
.. description:
.. type: text

Title with apologies to Glyph.

Recently some friends of mine suggested that "software engineer" is
not a good job title. While they are of course free to call their
profession whatever they like, I respectfully disagree. I think
"engineer" is a perfectly cromulent description of what we do.

This is an opinion piece. Despite arriving at opposite conclusions,
the disagreement is feathery at best.

What if buildings failed as often as software projects?
=======================================================

When it comes to getting things done, we're not very good, as Glyph
points out:

  Most software projects fail; as of 2009, 44% are late, over budget,
  or out of specification, and an additional 24% are cancelled
  entirely. Only a third of projects succeed according to those
  criteria of being under budget, within specification, and complete.

But it's not science!
=====================

Supposedly, software engineering isn't "real" engineering because,
unlike "real" engineering, it is not backed by "real" science or math.
This statement is usually paired with a dictionary definition of the
word "engineering".

I feel this characterization is incongruent with what engineers do
daily.

Consider the civil engineer, presumably the engineeringest engineer
there is. [#civil]_ If you ask me to dimension an I-beam for
you, I would:

* spitball the load,
* draw a free-body diagram,
* probably draw a shear and moment diagram,
* and pick the smallest standard beam that'll do what you want.

If you want to know how far that beam is going to go, I'll draw you
some conjugate beams. I would also definitely *not* use the
moment-area theorem, even though it wouldn't be too difficult for the
reasonable uses of an I-beam.

Once upon a time, someone inflicted a variety of theories on me.
Euler-Bernouilli beam theory, for example. Very heavy textbooks with
very heavy math. Neither my physical therapist nor my regular one
expect me to ever *truly* recover. Nonetheless, area moments and
section moduli are the only way to understand where the ``I`` in
I-beams comes from.

Nasty math didn't prevent me from dimensioning that I-beam. And I do
really mean *math*, not physics: Euler-Bernouilli is a math hack. You
get it by taking Hooke's law and throwing some calculus at it. Hooke's
law itself is more math than physics, too: it's a first-order
approximation based only on the observation that stuff stretches when
you pull it. It's wrong all the time, even for fairly germane
materials and setups. [#hooke]_ Both theories were put together long
before we had serious [#serious]_ materials science. We use them
because they (mostly) work, not because they are a consequence of a
physical model.

That was just one example from a single discipline, but it holds
generally. I analyze circuits by recognizing subsections. If you show
me a piece that looks like a low-pass filter, I am not distracted by
Maxwell's equations to figure out what that little capacitor is doing.
I could certainly derive its behavior that way: in fact, someone made
me do that once. Quite instructive. But I'm not bothered with the
electrodynamics of a capacitor right now; I'm just trying to
understand this circuit.

This isn't just how engineers happen to do their jobs in practice,
either. Engineering breakthroughs live on both sides of science's
cutting edge. When Shockley et al. first managed to get a transistor
to work, we didn't really understand what was going on. [#tor]_ Carnot
was building engines long before anyone realized he had stumbled upon
one of the most fundamental properties of the universe. Nobody was
doing metaphysics. Dude wanted a better steam engine.

To me, saying that I-beam was dimensioned with the help of beam theory
is about as far from the truth as saying that a software project was
built with the help of category theory. I'm sure that there's some way
that that thing I just wrote is a covariant functor and you can
co-Yoneda your way to proving natural isomorphism, but unless we're
writing Haskell, that doesn't really affect me. It's easy to reduce an
applied field to *just* the application of that field, but that
doesn't make it so, especially if we haven't even figured out the
field yet.

So, even if the math and science behind computer engineering is
somehow less real than that other math and science, I think that
difference is immaterial, and certainly not enough to make us an
entirely different profession.

Conclusion
==========

I think the similarities run deep. I don't want to throw that away
essentially just because our field is a little younger.

We're all hackers here. Engineers, too.

.. rubric:: Footnotes

.. [#civil] I'm using civil engineer here in the strict American sense
            of person who builds targets, as opposed to the military
            engineer, who builds weapons. Jokes aside, perhaps this is
            related to the disagreement. Where I come from, "civil
            engineer" means "advanced engineering degree", and
            encompasses many disciplines, including architectural (for
            lack of better word; I mean the American "civil engineer"
            here), chemical, electrical, and yes, computer.

.. [#hooke] Got a rubber band?

.. [#serious] I don't mean to characterize previous efforts as not
              serious. They simply didn't have the tools to do what we
              can do today.

.. [#tor] While it is very easy to make up a sensible-sounding
          narrative time line after the fact for the breakthroughs in
          physics and engineering that eventually made the transistor
          possible, this ignores the strong disagreements between
          theoretical predictions and practical measurements of the
          time. Regardless of their cause, it would be foolish to
          assume that Shockley just sat down and applied some theory.
          The theory just wasn't there yet.

..  LocalWords:  engineeringest
