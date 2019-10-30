<!--
.. title: Self-compressing pickles
.. slug: self-compressing-pickles
.. date: 2018-07-05 07:59
.. tags: pickle, python, security
.. category:
.. link:
.. description:
.. type: text
-->

I’ve been working on some pickle security stuff. This is a teaser.

Python pickles are extremely flexible: they can run essentially whatever code they want. That means you can create a pickle that contains a compressed pickle. The consumer doesn’t know if an incoming pickle will be compressed or not: the Pickle VM takes care of the details.

To do this, we define a useful little helper class:

```python
class PickleCall(object):
     def __init__(self, f, *args):
        self.f, self.args = f, args
     def __reduce__(self):
         return self.f, self.args
```

PickleCall is nothing but a convenience function for us to encode the function f being called with some args into a pickle. If you’ve used pickle before and you know that it normally encodes classes by name, you might expect that the <s>victi</s>consumer of the pickle also needs to define PickleCall, but that’s not the case. This class accomplishes that by explicitly implementing part of the pickle protocol with the \_\_reduce\_\_ method: it tells pickle how to encode it, and PickleCall isn't involved anymore. Of course, the “obvious” thing to use PickleCall with is “os.system” and something involving /dev/tcp.

Once you have PickleCall, writing the function that dumps an object to a string but with embedded zlib compression is straightforward:

```python
from pickle import loads, dumps
from zlib import compress, decompress

def zdumps(obj):
    zpickle = compress(dumps(obj), level=9)
    unz_call = PickleCall(decompress, zpickle)
    loads_call = PickleCall(loads, unz_call)
    return dumps(loads_call)
```

We can check that it works:

```
zpickle = zdumps(["a"] * 100)
pickle = dumps(["a"] * 100)

loads(pickle) == loads(zpickle)
# => True
len(zpickle), len(pickle)
# => (82, 214)
```

Internally, the structure for this looks as follows:

```
   0: \x80 PROTO      3
   2: c    GLOBAL     '_pickle loads'
   17: c    GLOBAL     'zlib decompress'
   34: C    SHORT_BINBYTES b'x\xdak`\x8e-d\xd0\x88`d``H,d\xcc\x18\x160U\x0f\x00W\xb6+\xd4'     <= this is zlib-compressed pickle
   63: \x85 TUPLE1   <= set up arguments for zlib decompress
   64: R    REDUCE   <= call zlib decompress
   65: \x85 TUPLE1   <= set up arguments for pickle load
   66: R    REDUCE    <=  call pickle load
   67: .    STOP
```

If you're trying to protect pickles the punchline here is that you probably need to whitelist because there are too many ways to hide things inside a pickle. (We're working on it.)

(This post was syndicated on the Latacora blog.)
