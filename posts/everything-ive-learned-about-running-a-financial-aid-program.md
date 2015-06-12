<!--
.. title: Everything I've learned about running a financial aid program
.. slug: everything-ive-learned-about-running-a-financial-aid-program
.. date: 2015-04-25 21:08:36 UTC-07:00
.. tags: pycon, mathjax
.. link:
.. description:
.. type: text
-->

For the past two years, I've been running PyCon's financial aid
program. Starting this year, the event coordinator has asked all staff
members to document what they do for PyCon. Firstly, this helps to
objectively recognize the hard work done by our (volunteer) staff, and
to help make sure there is continuity when the time comes to pass the
torch. Since organizers of other conferences have expressed interest
in my opinions for creating their own financial aid programs, I am
posting my notes publicly instead.

This is a collection of hard-earned opinions, and is very much work in
progress. It's written as if it were a conversation with a
hypothetical financial aid organizer; so, whenever I say "you", I mean
you, the awesome person running financial aid at a conference
somewhere.

# Basics

Financial aid programs are one of the most effective ways a software
foundation can spend their money. Even if you completely ignore the
effect it has on diversity, the number of speakers, sprinters and
other contributors that attended PyCon thanks to the financial aid
program is staggering.

## Naming your program

I inherited the term "Financial Aid". It's a fine term, but some other
conferences have come up with different terms that you may want to
consider, like "opportunity grants" and "diversity grants".

## Taking care of yourself

Running a financially program is a lot of work. It scales linearly
with the number of applicants; it's your job to create a process that
keeps the work per applicant small, so that you can make the number of
applicants large. (You'll know you've succeeded when the fixed
overhead dominates, and it wouldn't really matter if you added another
dozen people.)

It is also an exercise in deferred gratification. Typically, you will
need to start preparing about a year before the conference. The
gratification part only comes at the conference itself. Since you'll
probably be quite busy as an organizer, it may only come after the
conference is over. You probably want to make sure that you have an
excellent social circle and that you're fairly self-motivated.

As the Financial Aid Chair, I have been on the receiving end of verbal
abuse once. Don't put up with it.

## Beat the drum

Your financial aid program is useless if people don't know about it.

Some very talented speakers refrain from sending talk proposals
because they don't know if they can afford to attend.

## Numbers

PyCon's financial aid program is quite large. There are a number of
reasons for that:

- PyCon's financial aid program has been around for many years, so
  it's quite mature.
- The program can count on support from PyCon leadership and the
  Python Software Foundation.
- Because PyCon US is the largest PyCon in the world, it acts as a
  nexus for people across the globe; therefore, it's important for the
  Python community that as many people as possible have a chance to
  attend.

This is just to give you a ballpark idea of our numbers.

Regardless of what your numbers will be, expect that the primary
bottleneck of your financial aid program will simply be lack of funds.

# Confusing parts and sad truths

## No-shows

A lot of people will not show up, and not notify you (or notify you in
the days around the conference). Sometimes, there's good reasons
(illness, emergencies...). Sometimes, the reasons are less
great. Sometimes, you won't know the reason.

From the perspective of the Financial Aid Chair, no-shows are
terrible. It's a dead grant: money that's been allocated that can't
easily be translated into an extra attendee. Hence, many of the
suggestions I make for running financial aid processes are focused on
minimizing no-shows.

## Visas and travel

Many recipients have to cancel because they are unable to acquire
visas to Canada or the United States. In some cases, the visa process
took several months and simply did not complete in time for the
conference. In others the visas were declined for various reasons.

Occam's razor tells me that many countries, particularly the United
States, to some extent now Canada, but also Schengen zone countries
are simply actively hostile to foreigners visiting their country.

# Planning

## Stand by your Chair

At the end of the day, someone's responsible for making your
conference all it can be. That position is typically called the
Conference Chair. They get help from teams like the Program Committee
and Financial Aid to make that happen. At the end of the day they make
the hard calls, and you have to execute within their guidelines. That
typically includes the budget, but it also includes how you want to
allocate grants. You will almost certainly be resource-constrained, so
there are trade-offs to be made:

- Do you want to help newbies, or advanced programmers?
- Do you want to help marginalized groups? Which ones? How much?
- Do you want repeat attendees, or first timers?
- Do you want a few people from all over the globe, or many locals?
- Do you want to benefit people who directly contribute to the
  conference? Which ones (speakers, staff...)? How much?
- Do we care if people are receiving funds from other places? What if
  their employer is paying? What if their employer is also a sponsor?
  Does it make sense for them to give us *x* sponsorship fees when
  we're giving them a significant portion of that back in financial
  aid?

Your budget is, unfortunately, zero sum. Every group you benefit means
less for everyone else. Helping everyone is the same as helping
no-one. Everyone wants to help everyone, but it's unlikely you'll get
to do that. Make everyone understands exactly what you want to
accomplish; you don't want to have this argument in the middle of
trying to run a financial aid process.

## Free or reduced-price tickets

For many conferences, the tickets themselves can be quite
expensive. It makes sense to provide them to financial aid recipients
at no (or reduced) charge as part of their grant.

PyCon previously provided free registration, but now provides
reduced-cost registration. This helps with no-shows, giving
applicants a financial incentive to let you know if they can't attend.

Make sure that you document clearly that people will be receiving
tickets, at what price they'll be receiving them, and that their spots
are reserved. Common concerns from financial aid applicants:

- "Your conference blog says that you're sold out, and I haven't
  received financial aid yet. Will I be able to attend?"
- "I've already registered to reserve my spot; what do I do now that I
  get financial aid?"
- "I'm a student. Will I still be able to register at the student rate
  if I apply for financial aid?"
- "I applied for a larger amount because I didn't think ticket price
  would be included." (Unfortunately, most people tell you this far
  too late.)

As usual, clarity in communication is key here.

Previous years, PyCon optionally provided free registration for people
who asked. This optional part was somewhat confusing. Whatever you do,
make it part of the default grant application. That also means that
you should probably offer the reduced-price ticket to anyone who
applies for financial aid, even if you can't otherwise give them a
grant. Otherwise, someone who applies for financial aid for whom you
simply don't have enough funds will get punished twice: no financial
aid, and no access to early bird ticket price.

## Housing

Housing, in the context of financial aid, means that you pay for a
bunch of hotel rooms for various dates at your conference, and then
put financial aid recipients in them. To save costs, you want to pair
them up, and you want to utilize the rooms maximally.

Some people think that it's a good idea to organize housing as part of
your travel grants. Those people are mistaken. Housing is a terrible
idea all round:

- It doesn't scale, up or down. If you're small, you can't negotiate a
  worthwhile hotel block contract. If you're big, the system crashes
  under its /O(N²)/ weight (see below).
- It's not good for the financial aid recipients. While there are many
  nice things to be said about conference hotels, they are typically
  not economical. When we still organized housing, many financial aid
  recipients opted out: they could get significantly more bang for
  their buck otherwise.
- It's not good for the conference. People leave, people join, people
  change dates, people have preferences (or hard constraints) about
  who they'll stay with... Doing this for any nontrivial number of
  people is a logistic nightmare; doing it for trivial number of
  people isn't worth it.

Having humans solve the allocation problem produces inefficiencies at
larger scales (i.e. humans typically come up with fairly suboptimal
solutions). It's a pretty tricky problem to solve even with computers
(believe me, I've [tried][alloc] [extensively][optim]), but computers
will never solve the logistic issues caused by human factors.

PyCon used to manage housing for financial aid recipients. Getting rid
of this was the single best decision I've ever made for the financial
aid process.

It worked out quite well for the attendees too. Providing simple tools
(i.e. the equivalent of a classifieds section) is more than ample to
help people find great groups to room-share with. This *increased*
opportunities for roomsharing, because it made it much less of a
hassle to mix-and-match between financial aid recipients and other
attendees.

Plenty of FA people stayed in large AirBnBs or the like in groups of 6
or more, and ended up getting fantastic deals that allowed them to
stay an extra few days to attend other events like sprints and
tutorials.

# Before the conference: from applications to allocation

## Applications

Keep it simple. Use a form generator (Google Forms or Wufoo or
something) for data collection. All of the processing was done with
simple Python scripts, most of it in an IPython/Jupyter notebook. This
enables you to create well-documented processes, which helps
everyone. CSV files are your best friend.

Make as many fields on the application form as possible directly
translatable to something in your allocation process. Multiple choice
and boolean values are your friend; the review process will make sure
the applications are accurate. Have free-form fields for documenting
things, but only use them in the review process. For example, you can
have a multiple choice field for Python expertise ranging from
beginner to expert, and then have a free-form field for applicants'
portfolios.

Names are weird. There are lots of falsehoods programmers believe
about names [(link)][names]. You probably want to ask for a legal
name; it's quite likely that you need to keep the legal name around
for your records. Ask a lawyer and/or an accountant for
details. However, you probably want to ask for an (optional) preferred
name as well, which you should always use when communicating with
them. There's a bunch of reasons those might be different, and people
may have excellent reasons for not using their legal names. For
example, a legal name might give someone away as being
transgender. Sometimes, you want to do that just for your /own/
convenience. People will put all sorts of stuff down as their "name",
but full legal names are quite consistent. This can be useful to match
up records from different sources, such as your registration database.

Speaking of gender, asking for people's gender is also tricky. Make
sure you have at least a cursory understanding of how gender works
before you ask. Always have a "decline to answer" button, which is
distinct from "other/nonbinary". If all you really want to know is
whether someone qualifies for earmarked funds, just ask the specific
question you want to know; e.g. "Apply for a PyLadies grant (people
who self-identify as women only, please)".

As usual in programming, state is the bane of your existence. Keep it
in one place whenever possible. A (Google Drive) spreadsheet works
just fine. Your scripts should operate on data extracted from them
(again, CSV works fine), but not store any state. This is trickier
than it sounds, but the alternative is that you'll probably end up
destroying some data.

## Review

Reviews are easy to do in parallel, so get help from volunteers if
needed. Establish clear guidelines for what your discrete values
(e.g. Python experience) mean; not everyone agrees on what "expert"
means.

## Allocation

I've [written about allocation before][alloc].

PyCon allocates approximately 15% over budget; i.e. the budget that we
allocate is 1.15x the actual grant budget. This does not include aid
in the form of e.g. reduced ticket prices; remember to account for
those separately!

PyCon's no-show rate is somewhere between 10% and 20%, but it's very
hard to predict if the factors that contribute to that will affect
your conference equally.

As mentioned in that previous blog post, you don't always want to
allocate the full grant asked for. People will still be able to attend
with partial grants but typically wouldn't with an empty
grant. Therefore it typically makes sense to reduce everyone's grant
slightly if that allows you to provide more grants.

We ended up going with a fairly simple "flood fill" algorithm. An
applicant's score maps to a fraction of the budget:

$$f_i = \frac{s_i}{\sum_j s_j}$$

Where \\(f_i\\) is the fraction of the budget you're willing to assign
to applicant \\(i\\), \\(s_i\\) is an applicant's score.


If \\(f_i \ge q \cdot r_i\\) (where \\(q\\) is the fraction of the
grant you're willing to allocate and \\(r_i\\) is what the applicant
requested, you grant them \\(q \cdot r_i\\); otherwise, grant them 0.

Some applicants will be below this fraction, some will be above. They
could be below this fraction because they have a very high score
(e.g. they are a speaker), or because they're "low hanging fruit" and
not asking for very much money.

That means that if you run the algorithm again, the fractions will be
bigger; the people that received allocations in the previous round
were allocated less than what would've been their "fair share". Rinse,
repeat until you're out of money.

# Communications

Have a central website where people can see their current status and
updates, both specific to the applicant and generic to the entire
process. Training people to expect information there will drastically
reduce the number of repetitive questions you get to answer by e-mail,
which will contribute enormously to your happiness. Therefore, if
people ask questions that are answered there; answer, be kind, but
point out where they could have gotten that information from.

Most of the things you will have to say will be generic points about
the process. Nonetheless, I have spent huge amounts of time answering
individual questions about that generic process, which got get quite
tedious. Hence, even a static page that doesn't show any information
specific to the applicant, but just explaining the process in detail
is extremely valuable.

Being up-front about how your process works is also great for
prospective applicants who wouldn't feel comfortable asking. This also
attracts speakers to submit talk proposals; many speakers would not
propose a talk because they know they can't afford to come without
financial aid. Therefore, it's important to communicate clearly if you
intend to support speakers, both through your financial aid
communication and your call for papers.

Once that fails, send e-mail. Once you're over a dozen or so
recipients, use [Mailgun](http://www.mailgun.com/). Emails
particularly automated from your personal email account is a good way
to get stuck in spam filters.

# Disbursement (giving out money)

First off, talk to your treasure. Possibly talk to a lawyer, too. It's
quite possible, particularly if you're in the United States, that
giving out a bunch of money as a non-profit comes with some fairly
complex strings attached. For example, you probably have certain
standards in terms of what records you have to keep.

As a European, it somewhat comical to still see checks in active use,
but hey; it works. For many parts of the world (apparently not the
United States, though) wire transfers are how you send money to
people. PayPal seems to work better for larger, established
organizations that use it often. There is less of a problem with the
accounts or funds being frozen. It does actively restrict (or, perhaps
more accurately, enforces restrictions on) sending funds to certain
countries, including Brazil and India.

Cash works, but is hard to scale. With PyCon's budget of well over
$100,000.00, managing cash is clearly less than ideal.

At the end of the day, be pragmatic. Do whatever your recipients can
accept. Be sure to ask ahead of time, as your financial aid programs
may prove to be successful in bringing people in from very different
countries, with very different cultures and very different means
available to them for excepting your grant. This is the case for
PyCon, but I consider that a superb problem to have.

# Conclusion

I'd like to thank the following people in no particular order:

- Ewa Jodlowska, for managing PyCon and being the love of my life
- Van Lindberg, for continuously inspiring me to serve
- Diana Clarke, for Charing PyCon US in Montreal

[names]: http://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/
[alloc]: https://www.lvh.io/posts/2014/03/optimization-problems-and-pycon-financial-aid.html
[optim]: http://www.lvh.io/posts/2014/03/optimal-hotel-room-pairing.html
