.. title: Reverse ungineering
.. slug: reverse-ungineering
.. date: 2014-10-07 02:44:39 UTC-07:00
.. tags:
.. link:
.. description:
.. type: text

Title with apologies to Glyph.

Recently, some friends of mine suggested that "software engineer" is
not a good job title. While they are of course free to call their
profession whatever they like, I respectfully disagree: I think
"engineer" is a perfectly cromulent description of what we do.

This is an opinion piece. Despite arriving at opposite conclusions,
the disagreement is feathery at best.

What if buildings failed as often as software projects?
=======================================================

When it comes to getting things done, we're just not very good, as
Glyph points out:

  Most software projects fail; as of 2009, 44% are late, over budget,
  or out of specification, and an additional 24% are canceled
  entirely. Only a third of projects succeed according to those
  criteria of being under budget, within specification, and complete.

He then rightly points out that those shenanigans would never be
accepted in civil engineering, a Serious Engineering Discipline:

  Would you want to live in a city where almost a quarter of all the
  buildings were simply abandoned half-constructed, or fell down
  during construction? Where almost half of the buildings were missing
  floors, had rents in the millions of dollars, or both?

I certainly wouldn't. Computers are terrible, but not quite *that*
bad, as Glyph points out. "Failure" simply means something different
for software projects than it does for construction projects. Many of
those "failed" projects were quite successful by other measures; the
problem isn't with software projects, it's with applying civil
engineering standards to a project that isn't.

I emphatically agree that software projects aren't civil engineering
projects, and that attempts to treat the former like the latter have
done much more harm than good. That said, I think the point could be
nuanced in several important ways.

Firstly, I believe civil engineering is really the outlier here: other
engineering disciplines don't do as well by the civil engineering
success yardstick either. The few engineering endeavors in other
fields that do do well, are typically civil engineering in disguise,
such as nuclear and chemical plants.  Rank-and-file projects in most
fields of engineering operate a lot more like a software project than
building a skyscraper. Projects are late and over budget, often highly
experimental in nature, and in many cases also subject to changing
requirements. It's true just can't plan ahead in software, but we're
not alone.

On a related note, we may be confounding cause and effect here, even
if we overlook that not all engineering is civil engineering. Are
software projects unable to stick to these standards because it's not
engineering, or is civil engineering the only thing that sticks to
them because *they have no other choice*? Conversely, do we fail early
and often because we're not engineering, or because, unlike civil
engineering projects, we *can*? [#anthropic]_

Finally, software has existed for decades, buildings for
millennia. Bridges used to collapse all the time; Tacoma Narrows
wasn't so long ago. If the tour guide on my trip to Paris is to be
believed, one of those bridges has collapsed four times already.

But this isn't science!
=======================

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

That was just one example from a single discipline, but it holds more
generally, too. I analyze circuits by recognizing subsections. If you
show me a piece that looks like a low-pass filter, I am not distracted
by Maxwell's equations to figure out what that little capacitor is
doing.  I could certainly derive its behavior that way: in fact,
someone made me do that once. Quite instructive. But I'm not bothered
with the electrodynamics of a capacitor right now; I'm just trying to
understand this circuit.

This isn't just how engineers happen to do their jobs in practice,
either. Engineering breakthroughs live on both sides of science's
cutting edge. When Shockley et al. first managed to get a transistor
to work, we didn't really understand what was going on. [#tor]_ Carnot
was building engines long before anyone realized he had stumbled upon
one of the most fundamental properties of the universe. Nobody was
doing metaphysics. Sadi just wanted a better steam engine.

To me, saying that I-beam was dimensioned with the help of beam theory
is about as far from the truth as saying that a software project was
built with the help of category theory. I'm sure that there's some way
that that thing I just wrote is a covariant functor and you can
co-Yoneda your way to proving natural isomorphism, but you don't
really have to let that affect you if you don't want it to. It's easy
to reduce an applied field to *just* the application of that field,
but that doesn't make it so, especially if we haven't even really
figured out the field yet.

So, even if the math and science behind computer engineering is
somehow less real than that other math and science, I think that
difference is immaterial, and certainly not enough to make us an
entirely different profession.

But *that* isn't art!
=====================

Many people smarter than I have made the argument that programming is
an art, not dissimilar from painting, music or even cooking. I'm
inclined to agree: many talented programmers are also very talented
artists in other fields. However, I do disagree that those things are
art-like *unlike* engineering, which is supposedly just cold, hard
science.

There's a not-so-old adage that science is everything we understand
well enough to explain to a computer, and art is everything else. If
that's true, there's definitely plenty of art to be found in
engineering.

I don't think anyone wants to get dragged into a semantic argument
about what art *is*. I think even with a much narrower view of art,
engineers do plenty of it, as I've tried to argue before. Not all
engineering calls are a direct consequence of quantum mechanics;
sometimes, it is really just down to what the engineer finds most
platable. Even civil engineers, the gray predictable stalwarts of our
story, care about making beautiful things.

Conclusion
==========

I think the similarities run deep. I don't want to throw that away
essentially just because our field is a little younger. We're all
hackers here; and we're all engineers, too.

.. rubric:: Footnotes

.. [#anthropic] I suppose this is really analogous to the anthropic
                principle, except applied to engineering disciplines
                instead of humans.

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
