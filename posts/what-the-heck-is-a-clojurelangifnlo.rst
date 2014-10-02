.. title: What the heck is a clojure.lang.IFn$LO?
.. slug: what-the-heck-is-a-clojurelangifnlo
.. date: 2014-10-02 06:15:17 UTC-07:00
.. tags: clojure
.. link:
.. description:
.. type: text

It's no secret that I love Clojure. Like any tool though, it isn't
perfect. Today, I was trying to write unit tests that use
``clojure.core.async/timeout``, so I wrote a test double analogous to
Twisted's ``Clock``. As I tried to ``with-redefs`` it in, I got the
most inscrutable error message out: ``java.lang.ClassCastException:
icecap.handlers.delay_test$fake_timeout$timeout__22934 cannot be cast
to clojure.lang.IFn$LO``.

Wha? I know ``clojure.lang.IFn``, Clojure's function type, but what
the heck is a ``clojure.lang.IFn$LO``?

Searching for the term didn't give any particularly useful results. It
was clear this happened when I was redeffing the original ``timeout``,
so I looked at its documentation::

  clojure.core.async/timeout
  ([msecs])
  Returns a channel that will close after msecs

Doesn't look too special to me. What's the type of that thing, anyway?
Let's find out::

  > (parents (type timeout))
  #{clojure.lang.IFn$LO clojure.lang.AFunction}

Aha! So that is actually part of ``timeout``, not something else wonky
going on. What does the source say? It's a pretty lame shim::

  (defn timeout
  "Returns a channel that will close after msecs"
  [^long msecs]
  (timers/timeout msecs))

I mean, nothing interesting there, just a type hint.

Oh. Wait. That's not just a type hint. ``long`` is a primitive.
Testing::

  > (parents (type (fn [^long x] x)))
  #{clojure.lang.IFn$LO clojure.lang.AFunction}

Aha! Due to a JVM quirk, functions with a primitive type hint are
special. That works for doubles, too::

  > (parents (type (fn [^double x] x)))
  #{clojure.lang.IFn$DO clojure.lang.AFunction}

And multiple arguments::

  > (parents (type (fn [^double x ^double y] x)))
  #{clojure.lang.IFn$DDO clojure.lang.AFunction}
  > (parents (type (fn [^double x ^long y] x)))
  #{clojure.lang.IFn$DLO clojure.lang.AFunction}

Adding a simple type hint to the function fixed it. Success!
