<!--
.. title: Smaller Clojure Docker builds with multi-stage builds
.. slug: smaller-clojure-docker-builds-with-multi-stage-builds
.. date: 2017-06-16 10:12:46 UTC-07:00
.. tags: docker, clojure
.. category:
.. link:
.. description: Creating smaller Docker images by separating build and runtime environments
.. type: text
-->

A common pattern in Docker is to use a separate build environment from the
runtime environment. Many platforms have different requirements when you're
generating a runnable artifact than when you're running it.

In languages like Go, Rust or C, where the most common implementations produce
native binaries, the resulting artifact may require nothing from the environment
at all, or perhaps as little as a C standard library. Even in languages like
Python that don't typically have a build step, you might indirectly use code
that still requires compilation. Common examples include OpenSSL with
pyca/cryptography or NETLIB and other numerical libraries with numpy/scipy.

In Clojure, you can easily build "uberjars" with both lein and boot. These are
jars (the standard JVM deployable artifact) that come with all dependencies
prepackaged, requiring nothing beyond what's in the Java standard library
(rt.jar). While this still requires a JRE to run, that is still much smaller
than the full development environment.

There are a few advantages to separating environments. It all boils down to them
not having anything in them they don't need. That has clear performance
advantages, although Docker has historically mitigated this problem with layered
pulls. It can have security benefits as well: you can't have bugs in software
you don't ship. Even software that isn't directly used in the build process can
be affected: some build environments will contain plenty of software that is
never used that would normally carry over into your production environments.

Historically, most users of Docker haven't bothered. Even if there are
advantages, they aren't worth the hassle of having separate Docker environments
and ferrying data between them. While different ways of effectively sharing data
between containers have been available for years, people who wanted a shared
build step have mostly had to write their own tooling. For example,
my [icecap][icecap] project has a batch file with an embedded Dockerfile that builds
libsodium debs.

The upcoming release of Docker will add support for a new feature called
multi-stage builds, where this pattern is much simpler. Dockerfiles themselves
know about your precursor environments now, and future containers have full
access to previous containers for copying build artifacts around. This
requires Docker 17.05 or newer.

Here's an example Dockerfile that builds an uberjar from a standard lein-based
app, and puts it in a new JRE image:

```
FROM clojure AS build-env
WORKDIR /usr/src/myapp
COPY project.clj /usr/src/myapp/
RUN lein deps
COPY . /usr/src/myapp
RUN lein uberjar :uberjar-name myapp-standalone.jar


FROM openjdk:8-jre-alpine
WORKDIR /myapp
COPY --from=build-env /usr/src/myapp/myapp-standalone.jar /myapp/myapp.jar
ENTRYPOINT ["java", "-jar", "/myapp/myapp.jar"]
```

The full clojure base image is a whopping 629MB (according to `docker images`),
whereas `openjdk:8-jre-alpine` clocks in at 81.4MB. That's a little bit of an
unfair comparison: `clojure` also has an alpine-based image. However, this still
illustrates the savings compared to the most commonly used Docker image.

There are still good reasons for not using multi-stage builds. In the icecap
example above, the entire point is to use Docker as a build system to produce
a deb artifact *outside of Docker*. However, that's a pretty exotic use case:
for most people this will hopefully make smaller Docker images an easy
reality.

*Edited:* The original blog post said that the Docker version to support this
feature was in beta at time of writing. That was/is correct, but it's since
been released, so I updated the post.

*Edited:** Łukasz Korecki pointed out that `lein uberjar ` has an `:uberjar-name`
parameter. The previous line in the Dockerfile was much harder to read:

```
RUN mv "$(lein uberjar | sed -n 's/^Created \(.*standalone\.jar\)/\1/p')" myapp-standalone.jar
```

Thanks Łukasz!


[icecap]: https://github.com/lvh/icecap/blob/master/utils/build-libsodium-package.sh
