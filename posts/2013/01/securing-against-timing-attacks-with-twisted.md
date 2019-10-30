<!--
.. title: Securing against timing attacks with Twisted
.. date: 2013/01/30 18:55
.. slug: securing-against-timing-attacks-with-twisted
.. link:
.. description:
.. tags: security, python
-->


## What are timing attacks?

Timing attacks are side-channel attacks that rely on inferring secret
information from operations by measuring how long they take to execute.

A complete explanation is outside of the scope of this article, but
the [Wikipedia article](https://en.wikipedia.org/wiki/Timing_attack)
might be a good starting point for the interested reader.

## Why should I care about them?

Because they can creep in before you know it, and break your otherwise
fine system.

A common way they're introduced are string comparisons. String
comparisons in many langauge implementations, including all
implementations of Python I know of, short-circuit. They work roughly
like this:

``` python
def strcmp(s1, s2):
    if len(s1) != len(s2):
        return False
    for c1, c2 in zip(s1, s2):
        if c1 != c2:
            return False
    return True
```

This means that as soon as they can prove the two strings are not
equal, they return `False` and ignore the rest of the string. This is
a very simple and effective performance optimization. And that's
precisely why, from a security point of view, it's a liability.

Since it takes longer to compare "The quick brown fox jumps over the
lazy dog" to "The quick brown fox jumps over the lazy god" than it
does to compare it to "Lorem ipsum", or even a Lipsum of the same
length, an attacker can use that timing information to figure out what
the original string is that is being compared to, even when he
shouldn't.

## Why did you care?

Password resets.

The typically recommended way of doing them is to generate a random
number, one that's sufficiently long that an attacker can't guess it.
Then, you relay that number to the user using some alternative channel
(usually a URL in an e-mail).

Bar the random number, those URLs are predictable. That means an
attacker can trivially try any number he wants. If an attacker is
allowed to do that enough, he could measure timing differences between
different numbers (introduced by string comparison functions that
short-circuit or even by the database's index) to efficiently deduce
the value of a "good" number.

Also, if membership is private, an attacker may exploit timing
differences in the password reset *request* function to farm e-mail
addresses.

## How do I protect against timing attacks?

Just to be clear: timing attacks are a complicated problem, and this
article describes just one strategy I've applied to help secure
against them.

The easiest way to prevent a timing attack is to make sure that the
timing your hypothetical attacker can measure is unrelated to the work
you have to do.

Since I was using [Twisted](http://twistedmatrix.com) and
[AMP](http://amp-protocol.net/), this was actually quite easy. I wrote
a decorator for AMP responder functions that does exactly that:

``` python
def _immediateResponder(f):
    """
    A decorator for responder functions that should return immediately and
    execute asynchronously, as a defense against timing attacks.

    The responder decorator should be applied after (above) this decorator::

        @SomeCommand.responder
        @_immediateResponder
        def responder(...):
            ....

    This should be timing attack resistant since it is unconditional: the
    the AMP response is returned immediately, and the real responder is
    scheduled to run at the next chance the reactor has to do so.

    This only works with AMP commands with empty responses. That's probably a
    good idea anyway: almost all information you could add to the response
    is liable to introduce a timing attack vulnerability.

    Since this precludes your ability to communicate success or failure to
    the caller, the decorated function should return quite quickly (or, if it
    can't, that should be clearly documented). Otherwise, you may end up in a
    a race condition, where the caller assumes the operation has completed,
    but it is in progress or hasn't started yet.

    The original responder function is available on the decorated function as
    the ``responderFunction`` attribute.
    """
    @functools.wraps(f)
    def wrapped(self, *args, **kwargs):
        reactor.callLater(0, f, self, *args, **kwargs)
        return {}

    wrapped.responderFunction = f
    return wrapped
```

When I get an incoming RPC call, the function doing the actual work is
scheduled to run at the next reactor iteration. Then, an empty
response is returned. All of this happens in amortized constant time,
and independent of any secrets. As a result, it can't really leak
much about them.

## An abstraction too high

The above was an effective response to a proof-of-concept timing
attack exploit. Unfortunately, that doesn't mean you've fixed every
timing attack.

In particular, this example is a few layers of abstraction removed
from the grit of real-world I/O. Just because I returned `{}` (an
empty response) immediately, doesn't mean the underlying IO happens
immediately.

In particular, the write output latency could be coerced to depend on
the computation time, because the function passed to it could be
executed before the `write`. If that happens, and the time it takes is
dependant on some secret, delaying the write, the latency on the
attacker's side could be used to measure the work done.

I have not yet been able to turn the above into a working exploit.

There are a number of ways this could be mitigated. Since there's no
working exploit, it's unclear if this mitigation would render a timing
attack infeasible.

One way I've considered to mitigate this is to limit the time that the
reactor is blocked. In my concrete example, this was fortunately
already the case, since one of the first things it did was defer to a
thread that released the GIL (to compute the key from the password
using ``scrypt``). Alternatively, if you're doing this for Python
code, you could write your function cooperatively.
