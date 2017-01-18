<!--
.. title: Securing Python pickles
.. slug: securing-python-pickles
.. date: 2017-01-16 13:37:11 UTC-08:00
.. tags: private, security
.. category:
.. link:
.. description:
.. type: text
-->

Python's built-in pickle serialization library is a convenient, ubiquitous
(within Python, at least) way of serializing almost arbitrary objects. It has
plenty of issues, but since I'm a security nerd, I'm going to focus on those
aspects in this post. Most of those security issues are well-documented, and
some of them are also reasonably well-known.

# A brief primer

TL;DR: deserializing a pickle is arbitrary code execution. Believe
me? [Skip to the next section.](#hidden-pickles).

Internally, pickle uses a limited stack machine for building objects as it
deserializes. I've found a few sources claiming that machine to be Turing
complete without a citation. As far as I can tell, this is incorrect: the
language itself has no loops or branches, and pickle instructions can't
directly affect the execution of the following instructions. However, that
distinction is academic: pickle can execute arbitrary Python code with some
clever tricks, trivially being able to do whatever it wants after that.

If you're unfamiliar with how a stack machine works, the idea is that you have
a growing list of objects, and then operations that act on them. If you've
ever used a reverse Polish notation calculator, it's the same idea. For
example, to evaluate `1 2 + 3 *`, read it left to right:

1. stack: `[]`, input: `1`, result: `[1]`
2. stack: `[1]`, input: `2`, result: `[1 2]`
3. stack: `[1 2]`, input: `+`, result: `[3]`
4. stack: `[3]`, input: `3`, result: `[3 3]`
5. stack: `[3 3]`, input: `*`, result: `[9]`

The Pickle virtual machine has more kinds of objects than just integers and
plenty more operations, but the general idea is the same. There are many ways that the Pickle VM can turn out to execute some code, but most rely on

```python
>>> class Malicious(object):
...     def __reduce__(self):
...             return os.system, ("/bin/sh",)
...
>>> import pickle
>>> my_pickle = pickle.dumps(Malicious())
>>> pickle.loads(my_pickle)
sh-3.2$ # Uh oh!
sh-3.2$ exit
0
```

<a name="hidden-pickles"></a>

# Hidden pickles

Knowing pickle's caveats is one thing, but identifying where pickle is used is
another. You can search the codebase for `pickle`, but there are plenty of
implicit call sites too. Because it's so convenient, pickle is often the
default storage format for anything that needs to turn an object into bytes
and back again. Examples include:

* multiprocessing libraries, e.g. [`multiprocessing`][multiprocessing]
* session stores, e.g. [Django's][django],
* databases and caches, [redis][redis],
* queues, e.g. [celery][celery],
* scientific computing and machine learning, e.g. [scikit-learn][scikit-learn].

et cetera. This list isn't a repudiation of those libraries! Some of these
don't or at least no longer use pickle by default, and the others come with
appropriate warnings. The only point here is that there's a decent chance your
program uses pickle and you didn't even know.

# Remediation

A quick survey of existing remediation of pickle-induced vulnerabilities shows
that the most common answers include trusting the other party and avoiding
pickles. These are fine suggestions if you can pull them off. Suppose you're
merely using pickle because it's the default, but you could use JSON just
fine; disabling pickle input is an easy security win. But there are plenty of cases where this is harder to do. Maybe it's just inconvenient or poor UX; you can't disable pickle-based session cookies without

Other suggestions involve answers from cryptography: transport level security
and signatures. Both of these have limitations when tested against realistic
threat models.

# Static analysis

I'd like to explore another avenue for remediation: static analysis. Can we
analyze pickles and determine ahead of time if they're definitely bad or
definitely good?

Clearly, we can't solve this problem generally for many of pickle's applications. One of its biggest selling points is


Both the reference implementation and the Pyrolite Java library implement the
Pickle stack machine directly. They do not expose parsers.


# Conclusion

The author would appreciate any consideration for not making any awful puns
about being in a pickle, or analogies about hidden pickles in the back of
fridges.

[multiprocessing]:
[django]:
[redis]:
[celery]: http://celery.readthedocs.io/en/latest/userguide/security.html#serializers
[scikit-learn]: http://scikit-learn.org/stable/modules/model_persistence.html
