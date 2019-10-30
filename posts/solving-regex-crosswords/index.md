<!--
.. title: Solving regex crosswords
.. slug: solving-regex-crosswords
.. date: 2019-10-25 19:36:29 UTC-07:00
.. tags: clojure
.. status: private
.. category:
.. link:
.. description:
.. type: text
-->

**DRAFT!**

[Regex Crossword](https://regexcrossword.com/) is a puzzle game to help you
practice regular expressions. I wrote a program to solve them. You can find it
on GitHub as [`lvh/regex-crossword`](https://github.com/lvh/regex-crossword).
This blog post walks you through how I wrote it using logic programming. 

To me this game feels more like Sudoku than a crossword. When you solve a
crossword, you start by filling out the words you're certain of and use those
answers as hints for the rest. Backtracking is relatively rare. In a sudoku, you
might start by filling out a handful of certain boxes, but in most puzzles you
quickly need to backtrack. That distinction shapes how I think of solving it:
searching and backtracking is a natural fit for logic programming.

Logic programming is a niche technology, but in its sweet spot it's miraculous.
I'll introduce the concept to help you recognize what sorts of problems it's
good at. Worst case, you'll enjoy a cool hack. Best case you'll get the
satisfaction of writing a beautiful solution to a problem some time in the
future.

My programming language of choice is Clojure, and Clojure has
[`clojure/core.logic`][ccl], a logic programming library. Because logic
programming is such a specialized tool, it's more useful to have libraries that
let you "drop in to" logic programming than to do everything in a full-blown
logic programming language.

# Approach

It would not be a bad idea to read [the instructions for Regex
Crossword][how-to-play]. I'll also assume you have seen some basic regular
expressions. Hopefully the Clojure will be piecemeal enough you can
just follow along, but reading a tutorial wouldn't hurt.

Let's take a look at the first beginner puzzle:

<img src="/img/regex-crossword/beginner1.png">

Each regex part applies to some number of unknowns (empty boxes). The first row
regex `HE|LL|O+` applies to the unknowns of the first row. The first column
regex `[^SPEAK]+` applies to the unknowns of the first column. Both constrain
the top left unknown: we're counting on our logic programming engine at being
convenient for expressing that "cascading" of constraints.

An unknown is just a logic variable in constraint logic programming. For brevity
we'll call them "lvars" (*[ell vars]*).

Rows, columns (and in later puzzles hexagon lines) are all just layout.
Fundamentally it's all just a regex applying to some lvars.

# Breaking down regular expressions

Most regular expressions have structure, like `(AB|CD)XY`. We'll solve this
problem recursively: if `(AB|CD)XY` matches lvars `p, q, r, s`, then presumably
`pq` must match `AB|CD` and `rs` must match `XY`.

(There are some counterexamples to the idea that we can solve the entire problem
recursively! For example, backrefs within a regex would require a second pass,
and backrefs across regular expressions would require another top-level pass.
We'll deal with those later. There are ties to language theory here, but that's
a story for another day.)

We'll parse the regular expression into a data structure that's easier to
handle. Fortunately, there's a piece of code out there that knows how to
generate strings matching a given regex, which has a parser we can reuse. That
parser is designed for Java's regular expression syntax. It's not quite
identical to that of JavaScript, but we're hoping that the puzzles avoid those
tricky edge cases for now.

To parse, we use `[com.gfredericks.test.chuck.regexes :as cre]`.

```clojure
(cre/parse "HE|LL|O+")
;; => (read: produces)
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :character, :character \H} {:type :character, :character \E})}
  {:type :concatenation,
   :elements
   ({:type :character, :character \L} {:type :character, :character \L})}
  {:type :concatenation,
   :elements
   ({:type :repetition,
     :elements [{:type :character, :character \O}],
     :bounds [1 nil]})})}
```

Progress! It is indeed an alternation (a or b) of a concatenation of H and E, or
L and L, or the letter O one or more times. 

# Logic machinery

The logic programming engine is going to search a tree of possibilities for
solutions that fit your constraints. Logic programming is inherently
declarative: instead of telling the computer how to find the answer, we describe
what the answer looks like.

An lvar that doesn't have a value assigned to it yet (it could still be
anything) is called "fresh". An lvar with a definite value is called "bound".
There is nothing preventing your program from returning fresh variables: that
just means the answer that doesn't rely on what that lvar taking any particular
value (like the *x* in *0x = 0*).

In core.logic, a program consists of a series of expressions called *goals*.
`(l/== a b)` is a goal to make the values of `a` and `b` equal. It does not, by
itself, compare `a` to `b` or assign anything to anything. It just expresses the
idea of comparing the two. Logic programming calls this "unification".

As the engine searches the space of possible answers, sometimes `a` will be
equal to `b` already, or `a` will be bound and `b` will be fresh in which case
`b` will become bound to whatever `a` was bound to. Either way, the two
variables can be *unified* and the goal is said to *succeed*. But in many parts
of the tree this doesn't work out: `a` and `b` will already be bound to
incompatible values. In that case, the goal is said to fail. Unification is just
one of many goals, but it's the most important one in most programs including
ours.

(You'll probably be happy to learn `l/==`, full name `clojure.core.logic/==`, is
just a function. It requires learning a ton of deep logic machinery to grok, but
at least it's not magic.)

Finally, we need a way to actually run the logic programming engine. that's
`clojure.core.logic/run`'s job. Something like:

```clojure
(l/run 1 [q]            ;; run to find up to one answer with logic vars [q]
  (l/== q 'fred)        ;; where q is unified with 'fred
```

... will return `('fred)` (the 1-list with the symbol `'fred` in it) because
there's only one answer for `q` that makes all the goals (here just one goal)
succeed. `run` has a sibling `run*` that gets you all the answers. It returns
the same result, because there's only one value for `q` that makes all goals
succeed.

It takes the _maximum number of answers_ as a parameter. Since you're describing
what the answer looks like, there might be zero, one, or any other number of
answers. Sometimes the engine will be able to prove there are no other options
(because the search was exhaustive) and it'll return fewer. Some pathological
programs run forever, some so long it might as well be forever.

# Direct match

The simplest possible regular expression is just a character, which matches
itself: `A`. In our parse tree, this is an entry of `:type` `:character`. We'll
write a multimethod dispatching on `:type` to make this work so we can implement
other types later.

```clojure
(defmulti re->goal :type)
```

A character looks like this: `{:type :character, :character \L}`. We'll use
destructuring to extract the `:character` key. In general, this multimethod will
take multiple lvars, though incidentally in the case of `:character` it'll
be just one, so we can destructure it as well.

```clojure
(defmethod re->goal :character
  [{:keys [character]} [lvar]]
  (l/== character lvar))
```

We'll write a test to verify this works.

```clojure
(t/deftest re->goal-character-test
  (t/is (= '(\A)
           (l/run* [q]
             (rcl/re->goal {:type :character :character \A} [q])))))
```

# Concatenation

We want to be able to solve squares, not just individual letters. The simplest
example of that is concatenation. The simplest case is a concatenation of two
letters:

```clojure
(def a {:type :character :character \A})
(def aa {:type :concatenation :elements [a a]})
```

A simple test:

```clojure
(t/deftest re->goal-concatenation-test
  (t/is (= '((\A \A))
           (l/run* [p q]
             (rcl/re->goal aa [p q])))))
```

A simple implementation passes this test:

```clojure
(defmethod re->goal :concatenation
  [{:keys [elements]} lvars]
  (l/and* (for [e elements
                v lvars]
            (re->goal e [v]))))
```

(`l/and*`is a function that returns a goal that succeeds when all its goals
succeed. We'd use the macro version `l/all` if we were explicitly writing out
our goals. Because we're generating them, it's easier to use a function.)

This passes the test, but we're clearly abusing the fact that we happen to know
that the parts are characters and therefore length 1. Those constituent parts
could be anything, and so they could be longer than 1. We can't write a test for
that yet, because we don't have any regex parses that express different options
yet. Concatenations of characters only match exactly one thing, so they have
fixed length. We already saw the remaining two things to implement in our first
parse tree: `:alternation` (*or*, `A|B`) and `:repetition` (`A{x,y}` plus
shorthand spellings `A*` and `A+`). Alternation came first, so let's try that
next.

(We could technically produce a concatenation of concatenations, or character
literals that contain multiple characters, but our parser produces neither and
I'd rather work from real examples. Plus, we have to implement alternation
anyway.)

# Alternation

## Parse trees

Let's look at a few parse trees for some simple alternations:

```clojure
(cre/parse "A|B")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation, :elements ({:type :character, :character \A})}
  {:type :concatenation, :elements ({:type :character, :character \B})})}
```

Per our note above, we should also think of an example where the options are
different length.

```clojure
(cre/parse "AAA|B")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :character, :character \A}
    {:type :character, :character \A}
    {:type :character, :character \A})}
  {:type :concatenation
   :elements ({:type :character, :character \B})})}
```

It's also probably a good idea to consider something with more than two alternatives:

```clojure
(cre/parse "A|B|C")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation, :elements ({:type :character, :character \A})}
  {:type :concatenation, :elements ({:type :character, :character \B})}
  {:type :concatenation, :elements ({:type :character, :character \C})})}
```

## Alternation in logic

Again, we write a simple test:

```clojure
(t/deftest re->goal-alternation-test
  (t/is (= '(\A \B)
           (l/run* [q]
             (rcl/re->goal (cre/parse "A|B") [q])))))
```
We need a way to express disjunction. Like `l/and*`, there's a `l/or*`.

This primer is a little unorthodox because we're walking through a problem that
requires rule generation. Most introductory texts make you build up static
rules. Just like `l/and*` had a macro variant `l/all`, disjunction is usually
expressed with the macro `l/conde`. One difference is that `conde` can express a
disjunction of conjunctions in one go:

```clojure
(conde [a b] [c])
;; is equivalent to
(or* [(and* a b) c])
```

# Fixing :concatenation

As we discovered earlier: concatenation is broken. It assumes each part is a
character, and specifically that each part is length 1.

# Fixing :character

What the concatenation problem tells us is that character is broken, too. It
returned a successful goal when given more than one lvar.

We communicate a problem with a failing goal. If some goal fails, the engine
will stop exploring that branch. The always-failing goal `l/fail` is designed
for this sort of thing. (You can probably guess what its counterpart `l/succeed`
does.)

We wrote our `:character` rule to assume that there would only be one lvar. That
made sense because we knew that was always the case. But now `:concatenation`
may attempt to call it with more, so we use `l/fail` to communicate the problem.

```clojure
(defmethod re->goal :character
  [{:keys [character]} [lvar :as lvars]]
  (if (-> lvars count (= 1))
    (l/== character lvar)
    l/fail))
```

You could also do this in logic programming so that it's checked at runtime, but
since we're generating our goals ahead of time here and only giving them to the
engine at the end, we can short-circuit here.

# Repetition

## Sample repetition parses

Let's see what the parser does for a few simple repetitions:

```clojure
(cre/parse "A{1}")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :repetition,
     :elements [{:type :character, :character \A}],
     :bounds [1N 1N]})})}

(cre/parse "A{1,}")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :repetition,
     :elements [{:type :character, :character \A}],
     :bounds [1N nil]})})}

(cre/parse "A{1,2}")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :repetition,
     :elements [{:type :character, :character \A}],
     :bounds [1N 2N]})})}

(cre/parse "A*")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :repetition,
     :elements [{:type :character, :character \A}],
     :bounds [0 nil]})})}

(cre/parse "A+")
;; =>
{:type :alternation,
 :elements
 ({:type :concatenation,
   :elements
   ({:type :repetition,
     :elements [{:type :character, :character \A}],
     :bounds [1 nil]})})}
```

This matches our expectation, since:

* `A{n}` means exactly *n* A's
* `A{m,n}` means between *m* and *n* A's, inclusive
* `A{n,}` means `n` or more A's
* `A*` means 0 or more A's
* `A+` means 1 or more A's

"No bound" is represented by `nil`. Some numbers are represented as just `1`,
others as `1N`. The latter are `clojure.lang.BigInts`, not longs. The parser
does this so it can support regexes with pathologically large sizes (longer than
a `long`, a signed 64-bit integer). We can ignore that distinction: both behave
identically for our purposes.

# Conclusion

This introduction was a little unorthodox. Most texts focus on introducing
different primitives, have you write small static rules and build up from there.
You'll see `conde` long before anyone talks to you about `or*`. That's just a
consequence of how I wrote this post, not a principled stance. Reading [The
Little Schemer][tls] is still a good idea if you want to learn to write your own
programs.

# Next steps

These are things I thought were neat but didn't make sense in the original text.

## Running programs backwards

A truly mind-bending feature of relational programs is that they can be ran
"backwards". Usually, you write a program "forwards": it has some inputs and you
expect some outputs. If you write your logic program a particular way, it
doesn't actually know what's an input and what's an output, and so you can give
it an "output" and it will come up with "inputs" that would have led to that
output. This post did not demonstrate that, but it's one of the better hooks
I've found to get people excited about logic programming. I could not give this
talk better than [Will Byrd presenting miniKanren][byrd]. I won't spoil the
ending, but go watch that talk.

Our program does not work that way. All our "inputs" are rules, all our lvars
are the same kind (unknown boxes), so running it "backwards" doesn't really mean
anything. It could, if our parser and rule generation were also fully
relational. In that case, we could fill out some boxes and our program would
tell us progressively more creative regular expressions that would match them.
Instead of a puzzle solver, we would have written a puzzle creator.

## Palindromes

Some puzzles have hints that help you solve them. Usually they're thematic cues,
(e.g. "Hamlet") which are hard to tell a logic engine about. Structural cues are
easy. For example, if you think some lines should be palindromes, you can just
add the following constraint:

```clojure
(l/all* (map (fn [vars] (l/== vars (reverse vars))) lines))
```

If evaluated at the right time, this should constrain the search tree. It's
tricky to make sure that happens without measurement and fiddling.

## Thematic cues

If you really wanted to add thematic cues, e.g. "Hamlet", you could grab the
Wikipedia page, use something like TF/IDF to find important words, filter by the
appropriate size, and introduce them as constraints. I don't expect this will
speed anything up much even if you discount the initial step.

[how-to-play]: https://regexcrossword.com/howtoplay
[ccl]: https://github.com/clojure/core.logic
[tls]: https://www.amazon.com/Little-Schemer-Daniel-P-Friedman/dp/0262560992
[byrd]: https://www.youtube.com/watch?v=RVDCRlW1f1Y
