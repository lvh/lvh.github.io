<!--
.. title: Optimal hotel room pairing
.. date: 2014/03/18 13:05
.. slug: optimal-hotel-room-pairing
.. link:
.. description:
.. tags: pycon, mathjax
-->

The greedy hotel pairing algorithm from my [previous post][previous]
gives good solutions. If you make some fairly reasonable-sounding
assumptions about the problem, *very* good solutions. At any rate
nicer than forcing a human to drudge through it; both in terms of
person-hours spent and quality of the solutions.

However, it turns out I was wrong about them being *optimal*
solutions. There's no guarantee they would be, and in fact a good
chance that they won't.

[previous]: http://blog.lvh.io/blog/2014/03/10/optimization-problems-and-pycon-financial-aid/

## Finding optimal solutions

The good news is that there is a way to find the optimal solution.
This problem can be modeled as a [graph matching problem][matching];
what we're trying to find is a weighted maximal matching.

This is really easy if the graph is bipartite, but in our case we're
actually dealing with two [complete graphs][complete]: we try to pair
people with similar gender. Within that compatible group, anyone can
theoretically pair with anyone.

<img src="https://upload.wikimedia.org/wikipedia/commons/8/86/10-simplex_graph.svg">

People interested in additional reading might be interested in:

- [Notes from a Stanford course by Jan Vondrak][vondrak]
- [Notes from an Inria course by Frederic Havet][havet]

[matching]: https://en.wikipedia.org/wiki/Matching_%28graph_theory%29
[complete]: https://en.wikipedia.org/wiki/Complete_graph
[vondrak]: http://theory.stanford.edu/~jvondrak/CS369P-files/lec6.pdf
[havet]: http://www-sop.inria.fr/members/Frederic.Havet/Cours/matching.pdf

## Picking an algorithm

There are several feasible algorithms for this.

- [Edmonds' blossom algorithm][blossom], the first polynomial-time
  algorithm with running time \\(O(V^4)\\) or \\(O(V^2\cdot E)\\)
- Gabow's 1973 PhD thesis, an \\(O(V^3)\\) algorithm
- [An \\(O(\sqrt{V} \cdot E)\\)) improvement][micvas] by Micali (yes,
  [*that* Micali][micali]) and Vazirani
- Another improvement by Gabow, \\(O(V(E + \log{V}))\\)

The current state of the art in computerized algorithms appears to be
[Blossom V][blossomv] by Vladimir Kolmogorov. This algorithm finds
*perfect* matchings (i.e. all nodes are included), which may be
impossible for us, e.g. because of an odd number of pairs.

[blossom]: https://en.wikipedia.org/wiki/Blossom_algorithm
[micvas]: http://dl.acm.org/citation.cfm?id=1382663
[micali]: https://en.wikipedia.org/wiki/Silvio_Micali
[blossomv]: http://pub.ist.ac.at/~vnk/papers/blossom5.pdf

## Issues

One issue with this approach is that it becomes pretty much impossible
to adapt this to rooms for more than two people. In order to support
that case my previous greedy algorithm remains the best thing I've
found.

I was unable to find an implementation of this in Java or Clojure.
Fortunately, there is a Python library called [NetworkX][networkx]
that does have [an implementation][max_weight_matching].

I have not yet turned this into a fully functional program, because:

- it's too late to apply it for this year
- there's a good chance we won't be doing this again for next year

[networkx]: http://networkx.lanl.gov
[max_weight_matching]: http://networkx.lanl.gov/reference/generated/networkx.algorithms.matching.max_weight_matching.html

## Credits

I'd like to thank Tor Myklebust (`tmyklebu` on Freenode) for pointing
out I was wrong, and shoving me in the direction of the answers.

In this blog post I used several freely available graph illustrations
from Wikipedia. The complete graph illustrations were made by
[koko90][koko90].

[koko90]: https://commons.wikimedia.org/wiki/User:Koko90
