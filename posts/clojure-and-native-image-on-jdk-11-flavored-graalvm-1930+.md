<!--
.. title: Clojure and native-image on JDK 11-flavored GraalVM (19.3.0+)
.. slug: clojure-and-native-image-on-jdk-11-flavored-graalvm-1930+
.. date: 2019-11-24 08:50:15 UTC-08:00
.. tags: clojure
.. category:
.. link:
.. description:
.. type: text
-->

My takeaways from [the release notes][https://www.graalvm.org/docs/release-notes/19_3/]:

1. JDK code inlining support
1. Optional JDK11 support
1. Better Windows support

These may not seem like much, but JDK code inlining fixes my one major niggle
with native-image: it was too hard to get top-notch single-binary TLS, and
now it just works.

(There are lots of other great things that happened in this release! They're
just not in parts of Graal I use.)

# Updating to JDK 11

Updating to JDK 11 is optional, but you might as well get it over with now.

## A brief summary of Jigsaw (JDK9+) breakages

There are two things that bit Clojure-using early adopters of JDK 9, both
consequences of [Project Jigsaw][jigsaw]:

* The module system hiding previously-available classes
* Changes to the way classloaders work

These two changes broke Clojure and a whole host of common libraries (mostly
because of the now-unavailable classes) and the two common build tools
(leiningen and boot, mostly because of the bootclassloader). These were quickly
resolved and only affect you now if you care about supporting a long range of
Clojure versions or a long range of JDKs. Since this blog post is about
native-image, your output is a standalone binary and you get to pick the JDK
version. However, you still need to know a bit about this background in order to
understand some of the workarounds necessary for supporting GraalVM native-image
targeting JDK11 and above. This is happening now because Graal was previously
targeting JDK8, avoiding all of these issues.

The two classes that tend to come up that were often used but hidden in modules
are `java.sql.Timestamp` and `javax.xml.bind.DatatypeConverter`. Despite their
package names, they don't have anything to do with SQL or XML. Clojure used them
because `Timestamp` was the good instant type (`java.util.Date` being famously
bad), and `DatatypeConverter` was the good Base64 implementation available
everywhere.

[jigsaw]: https://openjdk.java.net/projects/jigsaw/

## Example: DatatypeConverter in clj-http-lite

Outside of Clojure, clj-http and clj-http-lite used `DatatypeConverter` as well
(also for base64). clj-http-lite is very popular in native-image Clojure
projects. Like other libraries, they were quickly patched to support JDK9. The
patch still attempted to import `DatatypeConverter` (see [the actual patch in
clj-http-lite][clj-http-lite-base64]), because the Base64 implementation
replacing it isn't available on every JDK those libraries wanted to support.
Normally, this is fine: the import fails and the alternative library gets used.
However, the static analysis step in GraalVM sees the trial import and
complains:

```
Error: com.oracle.graal.pointsto.constraints.UnresolvedElementException: Discovered unresolved type during parsing: javax.xml.bind.DatatypeConverter. To diagnose the issue you can use the --allow-incomplete-classpath option. The missing type is then reported at run time when it is accessed the first time.
```

The classic workaround was to add the module back with `--add-modules
java.xml.bind`. Since it's just a trial import (see patch), you can instead use
the workaround suggested in the error message (`--allow-incomplete-classpath`)
and it'll work fine. The downside is this moves _all_ errors to runtime. There's
a [Graal ticket][specific-incomplete-classpath] for a more precise command line
argument limiting the suppressed error to that class. I'm confident there's
already a way to express this in Graal command line arguments, but I haven't
tried to figure out the right incantation yet.

## Single binary TLS!

Once you fix the above issue with clj-http-lite, as long as you enable the TLS
subsystem (`--enable-https`), you'll just get single-binary HTTPS with
libsunec.so under the hood, meaning I can finally close
[#1336][single-binary-tls].

## `locking` errors are fixed

Clojure 1.10+ introduces a dependency on clojure.spec. That library is unusual
because it uses the `locking` macro. Ordinarily, the `locking` macro is pretty
rare because Clojure has different, higher-level concurrency primitives.

Bytecode verifiers that are more aggressive than those in the JDK would balk at
the resulting bytecode. Historically this was the Android toolchain, but the
issue resurfaced with Graal. This got documented in [CLJ-1472][locking-macro].
This issue had a whole myriad of workarounds that mostly involved replacing the
`locking` implementation and then hooking clj loads with dynapath. The most
common workaround was to just downgrade Clojure to 1.9.0.

With Graal 19.3.0, the error appears to have simply disappeared. I don't know
which Graal change precipitated this, but I'll happily take being able to use
the current release of Clojure.

[locking-macro]: https://clojure.atlassian.net/browse/CLJ-1472

# Example project

I updated https://github.com/lvh/cljurl-graalvm-demo if you want to try any of
this at home. If you're on Linux and want to debug the TLS issues, I wrote
https://github.com/lvh/nscap specifically for this purpose. It leverages Linux
namespaces to elegantly capture network traffic for a single process. You can
then throw the resulting PCAP into e.g. wireguard.

# What I'd still love to see in native-image

The compiler is slow. It's in the range of rustc speed: typically faster than
C++, certainly slower than Go. It eats a lot of RAM. It's fine because I don't
iterate on the binary version. I develop Clojure apps targeting native-image as
if they're normal Clojure apps and then eventually run some end-to-end tests on
the binary. But you knocked out my #1 feature so now I have a new one 😊

[specific-incomplete-classpath]: https://github.com/oracle/graal/issues/1664
[clj-http-lite-base64]: https://github.com/martinklepsch/clj-http-lite/commit/3f41fc53a1b692549c88a8602e753cfb887330ae
[single-binary-tls]: https://github.com/oracle/graal/issues/1336
[speculation]: https://www.youtube.com/watch?v=oyLBGkS5ICk
[maybenot]: https://www.youtube.com/watch?v=YR5WdGrpoug
