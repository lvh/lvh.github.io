.. title: hypercathexis dev notes part 1
.. slug: hypercathexis-dev-notes-part-1
.. date: 2014-10-18 06:55:15 UTC-07:00
.. tags: game, hex, svg
.. type: text
.. has_math: yes

===============
 hypercathexis
===============

.. epigraph::

   hy·per·ca·thex·is, n, pl hy·per·ca·thex·es \\-kə-ˈthek-səs, -ka-\\:
   excessive concentration of desire upon a particular object

I'm considering renaming the project to its plural, hypercathexes,
because then it can be about a hyper cat in a bunch of hexes.

The only real constraints I started with was that I wanted a
simultaneous turn-based space game on a hex grid.

Amit Patel from `Red Blob Games`_ has basically the awesomest page
about `hex grids`_, and a ton of awesome pages about many other areas
of game development. I think it's a fantastic resource for programmers
like myself who don't do game dev as a day job, but just want to make
a little game on the side.

.. _`Red Blob Games`: http://www.redblobgames.com
.. _`hex grids`: http://www.redblobgames.com/grids/hexagons/

Simultaneous turn-based means that all players plan their moves
simultaneously, and they are then also executed simultaneously. This
has an interesting scaling effect. On the one hand, it clearly scales
better to many players, because players "play" simultaneously. On the
other hand, you start getting interesting problems. For example, clock
synchronization. Does the entire world advance with the same tick-tock
pattern? What happens when a player does *not* make a move within the
allotted time? If you allow different clocks in the world, does time
advance faster if both players submit a move, or do you always wait
until the maximum timeout?

I wanted an excuse to play with Om_ and found Chestnut_.

.. _Om: https://github.com/swannodette/om
.. _Chestnut: https://github.com/plexus/chestnut

The `first version`_ had a working hex grid, but displayed it using
offset coordinates. I wanted to work using axial coordinates as much
as possible, because it makes a lot of math so much easier. Axial
coordinates work together with the "grain" of the hex map:

.. image:: /img/AxialBaseVectors.svg
   :alt: axial base vectors

.. _`first version`: https://github.com/lvh/hypercathexis/tree/d454da2b1d8c1cf491fc3cd7dba83ee1b2bd4c76

Second thing I did was move from individual ``<img>`` tags produced by
Om to a single ``<svg>`` with hexes (``<polygon>``) inside it. This
fixed a number of annoying placement issues with CSS. CSS really wants
to position things based on bounding box, not midpoints. That's great
for web pages, not so much for my hex grid. I ran into a number of
annoying issues where certain hex borders would be wider than others.
Browsers aren't made to draw hex grids based on left/right offsets, I
guess...

I started by expressing distances in the SVG in terms of the hex
width, which would become my unit. The height would then be
\\(\\sqrt{3}/2\\). Then, I realized that I could make my life easier by
expressing all x coordinates in terms of a single hex width, and all y
coordinates in terms of a single hex height; then I could just scale
differently across x and y at the end.

.. image:: /img/BasicSVGHexGrid.png
   :alt: Simple SVG hex grid

Developing in Firefox was mostly painless, but I discovered many
discrepancies once trying it in Chrome. Things that should be the same
aren't, particularly when it comes to transforms. For example, Chrome
would occasionally literally do the inverse of the scaling it was
supposed to:

.. image:: /img/HexGridCrossBrowserScalingIssues.png
   :alt: Differences in scaling behavior across browsers

I guess trying to implement things in browsers was a mistake.
