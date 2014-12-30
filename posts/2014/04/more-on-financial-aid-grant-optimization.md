<!--
.. title: More on financial aid grant optimization
.. date: 2014/04/06 20:34
.. slug: more-on-financial-aid-grant-optimization
.. link:
.. description:
.. tags: pycon, mathjax
-->


In [a previous post][prev], I talked about ways to optimize PyCon
financial aid grants. This is a follow-up on those efforts. Quick
recap:

- There is a fixed budget \\(b\\) available for grants, between 100k
  and 200k USD.
- There are a number of people (approximately 300) requesting various
  amounts \\(r_i\\) (approximately between 100 USD and 2000 USD) of
  financial aid, and receive a grant \\(g_i\\) so that \\(0 \le g_i
  \le r_i\\).
- Financial aid applicants can be assigned scores \\(s_i\\), a
  relative value describing how much we'd like to have them at PyCon.
- PyCon wants to optimize the total expected value of scores. That
  means getting as many people as possible to come, weighted by score.
- We've conjectured that we can estimate the probability \\(p_i\\)
  that someone attends as either \\(g_i/r_i\\), or \\((g_i/r_i)^2\\).
  The former prefers to spread the budget across a larger number of
  smaller grants, whereas the latter prefers to focus the budget into
  a smaller number of larger grants.

If any of that doesn't make sense, you should read
[the previous blog post][prev] for more details.

[prev]: http://www.lvh.io/blog/2014/03/10/optimization-problems-and-pycon-financial-aid/

In short, we're trying to solve the optimization problem:

$$\max \sum E[S_i] = \sum s_i \cdot p_i$$

Since most optimization texts appear to prefer minimization,
alternatively:

$$\min - \sum s_i \cdot p_i$$

Subject to a budget constraint and an individual grant constraint:

$$\sum g_i \le b$$

$$0 \le g_i \le r_i$$

The greater-than-zero constraint for individual grants is fairly
important, otherwise the algorithm might casually give you answers
like:

```
k = 1: [ 10.  20. -20.  30.  50.  10.  50.], sum: 150.0
```

That list in the middle are the per-person grants. Notice how the
algorithm feels that the third person really ought to pony up some
cash so that some of the other people can go to PyCon ;-)

## Squared problems

In the [previous blog post][prev] I ran into issues using a very
generic constraint solver. I ended that post saying that I would try
to remeedy that by applying a more specific solver that takes
advantage of a particular structure of the problem.

When you set \\(p_i = g_i/r_i\\), this turns into a linear programming
problem, since \\(r_i\\) is a constant. When you set \\(p_i =
\left(g_i/r_i\right)^2\\), it turns into a quadratic programming
problem. Turns out there's two things I missed about the quadratic
problem:

- The resulting problem is not convex. That means it's (probably)
  difficult to solve.
- The estimated probability that someone will attend falls off sharply
  as soon as they don't receive the full amount they requested. At 50%
  of the requested grant, the estimated probability of attending is
  only 25%; at 90%, it's 81%. This is the opposite of what we want.

## Fixing the model

That doesn't mean we should put the \\((g_i/r_i)^k\\) out to pasture:
it just means that I didn't pick the \\(k\\) I really wanted.
Specifically, if I were to pick \\(k=1/2\\), I'd get:

$$p_i = \left(\frac{g_i}{r_i}\right)^{1/2} = \sqrt{\frac{g_i}{r_i}}$$

In general, if  \\(k = 1/n\\):

$$p_i = \left(\frac{g_i}{r_i}\right)^{1/n} = \sqrt[n]{\frac{g_i}{r_i}}$$

That problem *is* convex, but it isn't linear, quadratic, or some
other easy specific problem. It's just constrained multivariate convex
optimization. That's okay, there are still a couple of applicable
optimization algorithms.

Some of these algorithms require the derivative of the goal function
with respect to a particular grant size \\(g_j\\) at a particular
point. In case we don't have the real derivative, we can still provide
a numerical approximation. In our case, we don't really need to
approximate, since the derivatives are fairly easy to compute
analytically:

$$\frac{\partial}{\partial g_j} \sum_i s_i \cdot
\sqrt[n]{\frac{g_i}{r_i}} = \frac{s_j \cdot {g_j}^{\frac{1}{n} -
1}}{\sqrt[n]{r_j} \cdot n}$$

## Finding a solution with Python

There are a few Python packages that contain optimization algorithms:

- SciPy
- cvxopt
- pyopt

Someone originally pointed me towards cvxopt. While I'm sure it's
excellent software, I already knew SciPy, so I went with that.

SciPy's optimization module provides the following algorithms:

- `fmin_l_bfgs_b` - Zhu, Byrd, and Nocedal's constrained optimizer
- `fmin_tnc` - Truncated Newton code
- `fmin_cobyla` - Constrained optimization by linear approximation
- `fmin_slsqp` - Minimization using sequential least-squares programming
- `nnls` - Linear least-squares problem with non-negativity constraint

The first two are not applicable because they only appear to support
bounds on individual variables. I also need a constraint over the
*sum* of variables for the budget:  \\(\sum g_i \le b\\). The last one
isn't applicable because this isn't a linear least-squares problem.
That leaves `cobyla` and `slsqp`.

## Experimenting with linear approximation (COBYLA)

Making COBYLA work ended up being pretty easy. The only non-trivial
part is expressing all your constraints as expressions greater than or
equal to zero.

I've uploaded my IPython notebook
([viewer](http://nbviewer.ipython.org/gist/lvh/10107818),
[gist](https://gist.github.com/lvh/10107818)).

This produced the following results:

```
scores: [1 1 1 2 3 5 5]
requested: [10 20 30 30 50 10 50] total: 200, budget: 150
k = 1/0.5: [ 10.  20.  30.  30.  50.  10.   0.], sum: 150.0
k = 1/1: [ 10.  -0.   0.  30.  50.  10.  50.], sum: 150.0
k = 1/2: [ 10.  10.   7.  27.  36.  10.  50.], sum: 150.0
k = 1/3: [ 10.  11.   9.  25.  35.  10.  50.], sum: 150.0
k = 1/5: [ 10.  11.  10.  24.  35.  10.  50.], sum: 150.0
k = 1/10: [ 10.  11.  11.  23.  35.  10.  50.], sum: 150.0
k = 1/100: [ 10.  11.  11.  23.  34.  10.  50.], sum: 150.0
```

Some takeaways:

- As predicted, as \\(1/k\\) increases, the optimization gradually
  starts spreading the budget out more evenly; preferring to give many
  partial grants rather than a few large ones.
- This effect is mostly only pronounced for \\(1/k = 2, 3\\); after
  that, increasing \\(1/k\\) doesn't make much of a difference
  anymore. This is what I'd expect when I visualize the \\(p_i\\)
  functions in my head, but I haven't ruled out numerical instability.
- Even at high \\(1/k\\), the two high scorers get their full grant
  amount. That's quite understandable for the one that's only asking
  for 10, but even the one asking for a large grant gets it
  unconditionally. This probably means that tweaking the scoring
  functions is going to be very important.
- The linear version is apparently already quite brutal: the person
  with score 3 requesting 50 simply gets it entirely, leaving no
  budget left over for the people with score 1.

Since I'm so pleased with these results, I'm skipping `slsqp` until I
feel like it. Next up, I'll try to compare the COBYLA results above
with the results of the greedy algorithm under various fitness
metrics.
