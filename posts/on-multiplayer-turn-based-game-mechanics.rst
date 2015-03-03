.. title: On multiplayer turn-based game mechanics
.. slug: on-multiplayer-turn-based-game-mechanics
.. date: 2014-10-26 09:00:02 UTC-07:00
.. tags: hypercathexes, game
.. link:
.. description: Thoughts on turn-based mechanics in multiplayer games
.. type: text

Most classic turn-based games, from chess all the way to Civilization
V, are sequential in nature. A player makes a move, then the next
player makes a move, and so on. The details can vary, for example:

- There could be two players, or multiple. This number is tightly
  bound for scaling reasons, which we'll discuss later.
- The game could have perfect information, like chess, where all
  players see a move as soon as it is played. The game could also have
  imperfect information, like Civilization V, where players see part
  of a move, but the effets may be obscured by fog of war.
- The players may play in a consistent order (chess, Civilization V),
  or in a somewhat random one (D&D's initiative system).

All of those things are more or less orthogonal to the turn system.
Players play turns sequentially, so I'm going to call these
*sequential turn-based games*.

Sequential turns make scaling the number of players up difficult. Even
with only 8 players, any given player will spend most of their time
waiting. While 8 players are a lot for most turn-based games, it's
nothing compared to an MMORPG.

An alternative to *sequential* turn-based play is *simultaneous*
turn-based play. In simultaneous turn-based play all players issue
their moves at the same time, and all moves are played out at the same
time. The simplest example is rock-paper-scissors, but Diplomacy works
the same way. More recently, this system has been explored by the
top-down tactical game `Frozen Synapse`_.

.. _`Frozen Synapse`: http://www.frozensynapse.com/

While simultaneous turn-based play gets us closer to making massively
multiplayer turn-based games feasible by turning a linear scaling
problem into a constant time one, we're not quite out of the woods
yet.

Consider what happens when a player does not make a move. There are a
few reasons that might happen:

- The player is not playing the game right now.
- The player has stopped playing the game altogether.
- The player may be in a hopeless position, where stalling is better
  than losing. (Stalling may tie up lots of enemy resources.)

If you've ever gotten frustrated at a multiplayer game that has a
"ready" system before you begin a game, but had to wait because one of
the players disappeared; this is essentially the problem turn-based
games face *every turn*.

There are a number of ways to mitigate this problem. Games can
duplicate playing fields. That works for both sequential games like
Hero Academy and simultaneous ones like Frozen Synapse. If a player
doesn't make a move, that particular instance of the game world
doesn't go anywhere; but you can play any number of games
simultaneously.

For this strategy to work, the playing fields have to be independent.
You don't lose heroes or soldiers because they're stuck on some stale
game. The worst possible outcome is that your game statistics don't
reflect reality.

That works, but rules out a permanent game world with shared
resources. If there's a larger story being told, you would want these
worlds to be linked somehow: be it through shared resources, or
because they're literally the same game world.

There's a number of creative ways to get out from under that problem,
usually by involving wall-clock time. For example, if a player doesn't
respond within a fixed amount of time, they may forfeit their turn.
Fuel consumption might be based on wall-clock time, not turns. [#f1]_
There's a lot of degrees of freedom here. Do you use a global clock,
or one local to a particular area?

A global clock is probably simpler, but poses some game play
challenges. How long is the tick? Too fast, and a player may see their
empire annihilated while they're sleeping. Too slow, and the most
trivial action takes forever. There isn't necessarily one right
answer, either. In an all-out cataclysmic struggle between two
superpowers, a complete tactical battle plan may take a long time. Any
timescale that isn't frustratingly short for that situation will be
frustratingly long for anyone trying to guide their spaceship (or
kodo, depending which universe you're in) across the Barrens.

Local clocks have their own share of difficulties. You still need to
answer what happens for anything that isn't in a particular battle;
you still need to answer what happens when battles merge or diverge.

I'm currently exploring the shared global clock. In order to mitigate
the issues I described, I'm contemplating two ideas:

- Allow programmable units; a la Screeps, CodeWars...
- Allow players to plan several turns ahead of time.

These are, of course, not mutually exclusive.

.. rubric:: Footnotes

.. [#f1] I don't particularly like this, because it "breaks the fourth
         wall" in a sense. If my engines are still consuming fuel real
         time, why can't the enemy fire missiles? Either time is
         stopped, or it isn't. Sure, games can be abstract, but that
         feels like an undue inconsistency.
