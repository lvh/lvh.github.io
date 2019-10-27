<!--
.. title: Optimization problems and PyCon financial aid
.. slug: optimization-problems-and-pycon-financial-aid
.. date: 2014/03/10 16:47
.. tags: pycon
.. has_math: yes
-->


This year, I have partly taken on some new responsibilities in
organizing PyCon. Specifically, I've done some work on financial aid.
(Next year, I will be taking on the position of financial aid chair.)
There have been some challenges that I've thrown some code at. Some of
it stuck.

I'm very much interested in your comments, thoughts, alternative
approaches, et cetera. Hit me up on
[Twitter](https://twitter.com/lvh).

## Hotel room allocation

The first issue I solved was one of hotel room allocation. We provide
the financial aid applicants with hotel rooms. To keep costs down, we
pair people up, two by two.

We want to pair people so that we have to book a minimal number of
days. If there's days on the start of the room booking where only one
person is booking the room, we have to pay for the rest.

### A greedy algorithm

I start out with generating every possible pairing. Assuming we have
300 financial aid applicants, that's:

$${300 \choose 2} = \frac{300!}{2! \cdot 298!} = 44850$$

That's more than we can check by hand, but still pretty easy for a
computer. Besides, there are actually fewer pairs than that: some
people request to be paired together explicitly, and we only pair
people of the same gender.

Then, I sort them by number of unmatched days, that is, how many days
would have a single person in the hotel room if we paired them
together. Finally, I just greedily start grabbing pairs, keeping track
of the people that have been allocated rooms as I go along. As soon as
everyone is accounted for, we stop.

Other than perhaps minding the edge conditions where you have an odd
number of people to pair, this isn't too tricky. The implementation
for this is on Github, in
[`lvh/pairing`](https://github.com/lvh/pairing/blob/master/src/pairing/core.clj).

### Solution, meet reality

Before I wrote this, humans did this by hand. That took pretty long,
and the solutions were decent, but suboptimal. All the humans that
have tackled this problem seem to take the same approach: sort by
check-in date, find exact or near matches in check-out dates, rinse,
repeat.

The algorithm above did quite well, coming up with a significantly
better result than the previous human allocation. It also produced
very *interesting* solutions. Turns out that by taking a big hit on
one of the pairs (say, a pair that's 4 days mismatched four days), you
can limit the damage on a large number of other pairs, and still come
out on top. After reviewing with the person who previously had to do
this manually, we quickly agreed that a human would most likely not
have come up with this.

Furthermore, a lot of the pairs have matching check-out dates but with
a mismatched check-in date; whereas humans typically only look for
solutions in the other direction. This is easy to explain in
hindsight; the algorithm has no concept of an ordering of events in
time. It just tries to minimize the number of mismatched days.

I'm quite happy with the result. Any dollar I'm not devoting to a room
that isn't being fully used will instead be going into an increased
grant.

Unfortunately, finding this solution isn't the end of the story. It
rarely survives the clash with reality. Many complex factors affect
it: people will drastically change their room dates, people's visas
get denied, et cetera. All sorts of unforseeable circumstances have
lead to changes to the answer the algorithm originally produced. I
don't expect that to change until PyCon is over; unfortunately, I also
don't think that's a problem I can fix in a few lines of Clojure.

## Grant allocation

The second issue is a much thornier one. Given the financial aid
budget, figure out how to optimally distribute it. The budget is quite
limited, a good deal smaller than the sum of what everyone requests.

There's a few reasons why this problem is complex. First of all,
"optimal" is highly subjective. As I'm sure you'll notice very soon
after thinking about it, it's very easy to verge off from math and
straight into the realm of politics.

### A very simple greedy algorithm

You might say that you want to get as many people as possible to come
to PyCon, and you just greedily allocate the smallest grant requests
first. This has a few disadvantages:

1. It biases against the people that ostensibly need the money the
   most.
1. It is ordering-sensitive: equivalent applications that happen to
   come later in the queue are disadvantaged.
1. It oversimplifies how people react to grants, by making grant
   allocation binary. Someone receiving 90% of their requested grant
   amount will likely still be able to attend. Suppose you have 11
   people, all requesting 100 USD, and you have 1000 USD to spend. The
   greedy algorithm would give the first ten of them 100 USD and find
   its purse empty for the eleventh. It would almost certainly be
   better to give them all 90.91 USD instead.
1. Game-theoretically, it makes a lot of sense for everyone to apply
   for a small grant that they will almost certainly get. (This is not
   a real issue for us, because we still have humans eyeballing all
   the applications.)
1. The only difference between applications is the amount they're
   requesting, but there are people that we would like to bias in
   favor of. For example, we'd like to try very hard to get speakers
   or tutorial presenters to the conference, we might want to reward
   people doing open source work, et cetera.

Selecting these critera is definitely squarely in the realm of
politics, and I'd like to stick to the realm of math; so let's just
assume that we can assign scores to applicants and that those scores
are fair and just. If you feel like we shouldn't bias in favor of
anyone at all, just assume that whenever I say "the score of an
applicant" I mean "the number 1".

### A slightly smarter greedy algorithm

One of the key ideas (which a lot of people seem to have when tackling
this problem) is to translate an application's score into an amount of
the budget. So, for example, let's say that the sum of all scores is
100, and your score is 10, that means you can have up to 10% of the
budget allocated to you.

So, I make a pass over all the remaining applicants. If you're asking
less than your budget slice, you get your requested grant. If you're
asking for more, I save you for a future pass. Even though the budget
goes down between passes, budget slices will go up, because the total
score drops.

You can't do this in one pass, increasing the budget as you go along
if people request less than their slice, because you might introduce
order dependence. For example, consider what happens when Alice and
Charlie both need more than the budget allows for (and they have the
same score and requested amount), and Bob needs less. If you process
them in the order Alice - Bob - Charlie, Charlie may get his requested
grant and Alice may not, just because Bob's in the middle making the
budget bigger for Charlie.

Once everyone left is requesting more than their slice, they just get
their slice instead of their requested amount.

This solution is adequate, but I'm not completely satisfied; everyone
besides the last pass will get full grant amounts. You can find my
implementation of this algorithm
[on Github](https://github.com/lvh/hood/blob/master/src/hood/greedy.clj)
as well.

### A better model

I think we can do better by applying some elementary probability
theory. What we really want is to maximize the total score of the
applicants that we expect to see at PyCon as a consequence of
financial aid. Probability theory has a thing called the expected
value of a random variable, which is pretty much just the integral of
the random variable with respect to the probability. In our case, that
just means that we want to maximize the sum of the probabiliies that a
given applicant will come to PyCon given a particular grant amount,
weighted by their score:

$$\max \sum s_i \cdot p_i$$

That just restates problems we can't solve in terms of problems we
can't solve. What on earth could the probability that someone shows up
be? We can't know the real value, since it's specific to each
individual, and dependent on a lot of random, unknowable events. We
can, however, make an educated guess. If we don't give them any money,
they probably won't come. If we give them the amount they're asking
for, they probably will.

We could conjecture that the probability someone will come to the
conference is approximately the fraction of the amount they requested
that they actually received:

$$p_i = \frac{g_i}{r_i}$$

For example, if someone gets 90% of their
requested grant, we estimate that the odds they will attend are also
90%; if we give them only 10%, we estimate it at just 10%.

Alternatively, we could conjecture that it's closer to the *square* of
that ratio; when giving someone a value that's very close to what they
asked for, their probability of attending is still going to be very
high; but if we only give them half, the odds they will attend will be
closer to 25%, much lower than 50%. So, the expression becomes:

$$p^{\prime}_i = \left(\frac{g_i}{r_i}\right)^{2}$$

*Update:* I now realize that this is probably not the expression that
 I actually want: it falls off very quickly as soon as an applicant
 gets anything less than the full amount. I've elaborated on this in
 [a new blog post][model-update].

[model-update]: http://www.lvh.io/blog/2014/04/06/more-on-financial-aid-grant-optimization/

We will see how the square model emphasizes focusing the available
funds on fewer grants, whereas the linear model will spread smaller
grants more liberally.

### An attempt with constraint solvers

(I would like to thank Mark Engelberg for helping me with the stuff in
this chapter. It may not have panned out, but it was a very
educational experience nonetheless.)

It'd be nice if we could just declaratively describe the problem,
throw it into a computer program, and have it magically tell us the
answer. Of course, there are programs that do exactly that, called
solvers.

When I reached out on the Clojure mailing list, Mark Engelberg
helpfully reached out and immediately handed out some cool runnable
example for me to play with. Sure enough, they worked super, and with
a carefully crafted three-person example we could clearly see the
difference between the linear and quadratic models I was talking about
above: under the square model, the concentrated grants, trying harder
to give high-scoring applications the amount they requested. The
linear model spread money out more evenly. That is, with the following
applications:

```clojure
(def alice {:name "Alice", :score 5, :requested 120})
(def bob {:name "Bob", :score 4, :requested 100})
(def carol {:name "Carol", :score 3, :requested 80})
```

In a linear model, it comes up with:

```clojure
{alice 120
 bob 80
 carol 0}
```

However, in the quadratic model, the optimizer rather gives Carol (who
has a lower score) a complete grant than give Bob a partial one:

```clojure
{alice 120
 bob 0
 carol 80}
```

You can find the code [on
Github](https://github.com/lvh/hood/blob/master/src/hood/constraint.clj).
The tests that confirm the difference in behavior for the linear and
quadratic models are in [the
tests](https://github.com/lvh/hood/blob/master/test/hood/constraint_test.clj).

There's one issue though; it didn't scale. Not bad enough that you'd
notice on the three-person example (I thought my REPL was just being
laggy), but bad enough that any real-world problem would be completely
unsolvable. Why? Clearly constraint solvers are awesome real world
tools with real world usage, it's not like they're supposed to fall
over as soon as you throw a problem of reasonable size at it.

Well, let's do some napkin math. If we have 200 financial aid
recipients that are all getting 1000 possible options (somewhere
between 0 USD and 1000 USD, in whole-dollar increments), there's
200,000 possible places to put any given dollar. If we have 100,000
USD to spend, that's:

$${2 \cdot 10^{5} \choose 10^{5}} = \frac{(2 \cdot 10^{5})!}{10^{5}! \cdot 10^{5}!} \approx 10^{60203}$$

That is a very big number. I needed logarithms to compute it. It is so
huge that it makes the number of atoms in the observable universe
(about \\(10^{80}\\)) look like rounding error.

In an attempt to "fix" that problem, I tried only handing out funds in
chunks of 100 USD each. Then, we have 1000 chunks to hand out and 2000
places to put them:

$${2000 \choose 1000} = \frac{2000!}{1000! \cdot 1000!} \approx 10^{600}$$

I have made the problem 59,603 decimal orders of magnitude easier!
Rejoice!

A constraint solver really doesn't "know" anything about the structure
of the problem you're feeding it. That's a trade-off: in return, it
grants you a lot of freedom in expressing your problem. Constraint
solvers like LoCo are very valuable, they just aren't a good fit for
this problem. They are designed for problems where the constraints
really constraining the solution space. We're clearly not in that
territory. We have many valid solutions, and we're struggling to look
for a *good* one.

There are two ways we can fix this:

1. Compromising on the freedom of expressing the problem. By pouring
   our problem into a specific structured shape, we may be able to use
   a much faster solver, specific to that class of problems. Also, by
   allowing arbitrary real numbers instead of just integers, we can
   use much faster solvers.
1. Compromising on finding the *optimal* solution, instead searching
   for a *decent* solution. You can do that with algorithms like tabu
   search or simulated annealing.

I will be elaborating on these problems in a future blog post.
