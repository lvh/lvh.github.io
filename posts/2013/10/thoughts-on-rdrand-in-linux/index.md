<!--
.. title: Thoughts on RDRAND in Linux
.. date: 2013/10/19 21:47
.. slug: thoughts-on-rdrand-in-linux
.. link:
.. description:
.. tags: cryptography
-->


This is a response to [Linus' response to the petition to remove `RDRAND` from
/dev/random][linus]. `RDRAND` is a CPU instruction introduced by Intel on recent
CPUs. It (supposedly) uses a hardware entropy source, and runs it through AES in
CBC-MAC mode, to produce random numbers. Out of fear that `RDRAND` may somehow
be backdoored, someone petitioned to remove `RDRAND` support to "improve the
overall security of the kernel". If `RDRAND` contains a back door, and an
unknown attacker can control the output, that could break pretty much all
userland crypto.

Linus fulminated, as he does. He suggested we go read `drivers/char/random.c`. I
quote (expletives and insults omitted):

 > we use rdrand as _one_ of many inputs into the random pool, and we
 > use it as a way to _improve_ that random pool. So even if rdrand
 > were to be back-doored by the NSA, our use of rdrand actually
 > improves the quality of the random numbers you get from
 > /dev/random.

I went ahead and read `random.c`. You can read it for yourself [in Linus'
tree][randomc]. The function I'm interested in is `extract_buf`:

```c

    /*
     * If we have a architectural hardware random number
     * generator, mix that in, too.
     */
    for (i = 0; i < LONGS(EXTRACT_SIZE); i++) {
        unsigned long v;
        if (!arch_get_random_long(&v))
            break;
        hash.l[i] ^= v;
    }
```

This is in the extraction phase. This is after the hash is being mixed back in
to the pool (and that's for backtracking attacks: not intended as an input to
the pool). The output of `arch_get_random_long` is being XORed in with the
extracted output, not with the pool.

If I were to put on my tin-foil hat, I would suggest that the difficulty has now
been moved from being able to subvert the pool as one of its entropy sources
(which we believe is impossible), versus being able to see what you're about to
be XORed with. The latter seems a lot closer to the realm of stuff a microcode
instruction can do.

To put it into Python:

```python
from inspect import currentframe
from random import getrandbits

def extract_buf():
    """Gets 16 bytes from the pool, and mixes them with RDRAND output.

    """
    pool_bits = extract_from_pool()
    rdrand_bits = rdrand()
    return  pool_bits ^ rdrand_bits

def extract_from_pool():
    """Pretend to get some good, unpredictable bytes from the pool.

    Actually gets a long with some non-cryptographically secure random
    bits from random.getrandbits, which is usually a Mersenne Twister.

    """
    return getrandbits(32)

def rdrand():
    """
    A malicious hardware instruction.
    """
    pool_bits = currentframe().f_back.f_locals["pool_bits"]
    return pool_bits ^ 0xabad1dea

if __name__ == "__main__":
    assert extract_buf() == 0xabad1dea
```

Why can't RDRAND work like this?

Some comments based on feedback I've gotten so far:

1. This attack does not need to know where the PRNG state lives in memory. First
of all, this isn't an attack on the PRNG state, it's on the PRNG output.
Secondly, the instruction only needs to peek ahead at what is about to happen
(specifically, what's about to be XORed with) the RDRAND output. That doesn't
require knowing where the PRNG state (or its output) is being stored in memory;
we're already talking register level at that point.

2. While it's certainly true that if you can't trust the CPU, you can't trust
anything, that doesn't really make this problem go away. `RDRAND` being broken
wouldn't make software crash, which is a lot harder for almost all other
instructions. `RDRAND` being broken wouldn't result in measurable side-effects,
unlike what would happen if `PCLMULDQ` contained a back door. Furthermore, it's
a lot easier to backdoor one single microcode instruction and a lot more
plausible and feasible for a CSPRNG to be backdoored than it is to think of a
CPU as some kind of intelligent being that's actively malicious or being
remotely controlled.

For what it's worth, it seems [Zooko agrees with me][zooko].

[linus]: https://www.change.org/en-GB/petitions/linus-torvalds-remove-rdrand-from-dev-random-4/responses/9066
[randomc]: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/drivers/char/random.c
[zooko]: https://twitter.com/zooko/status/392334674690723840
