<!--
.. title: Crypto APIs and JVM byte types
.. slug: crypto-apis-and-jvm-byte-types
.. date: 2016-06-18 13:43:30 UTC-07:00
.. tags: cryptography, private
.. category:
.. link:
.. description:
.. type: text
-->

The JVM has several standard byte types. For in-memory cryptography purposes
(specifically, one-shot APIs), the two most relevant ones are byte arrays and
`java.nio.ByteBuffer`. Unfortunately, picking between them isn't easy, because
they have different pros and cons:

`ByteBuffer` can produce slices of itself and slices of byte arrays with
zero-copy semantics. This makes it the API you want to use if you want to
place an encrypted message in a pre-allocated binary format, say, if you're
designing [nonce-misuse resistant APIs][magicnonce]). It is also useful for
generating more than one key out of a single call to a KDF with an unusually
long output.

Byte arrays are easily serializable, but `ByteBuffer` is not. This results in
at least some (possibly unnecessary) copying during serialization, unless your
serialization library .

Byte arrays are constant length, and that length is stored on the array, so
it's cheap and easy to find. Figuring out how much to read from a `ByteBuffer`
requires a (trivial) amount of math, but that might not be constant time.

`ByteBuffer` has a public API for allocating direct buffers; unlike byte
arrays. Direct buffers are interesting because they are not managed by the
JVM, meaning they won't be copied around by the garbage collector, and memory
pinning is free. This means they can be securely zeroed out after
use. Directly-allocated `ByteBuffer` instances might have underlying arrays,
so going back to an array _might_ be zero-copy. In my experiments, these byte
buffers never have underlying arrays, implying copying. The `sun.misc.Unsafe`
class does have options for allocating memory directly; but it's pretty clear
that use of that class is strongly discouraged.

[caesium]: https://github.com/lvh/caesium
[magicnonce]: https://github.com/lvh/caesium/blob/master/src/caesium/magicnonce/secretbox.clj
[libsodium]: https://github.com/jedisct1/libsodium
[byte-streams]: https://github.com/ztellman/byte-streams
