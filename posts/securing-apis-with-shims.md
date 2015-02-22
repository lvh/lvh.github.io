<!--
.. title: Securing APIs with shims
.. slug: securing-apis-with-shims
.. date: 2015-02-20 22:24:03 UTC-08:00
.. tags: security crypto
.. link:
.. description:
.. type: text
-->

Imagine that you had a capability URL, except instead of giving you
the ability to perform a specific *action*, it gave you the ability to
perform a (limited) set of operations on a third party API,
e.g. OpenStack. The capability URL wouldn't just be something you
exercise or revoke; it'd be an API endpoint, mostly indistinguishable
from the real API. Incoming requests would be inspected, and based on
a set of rules, either be rejected or forwarded to the API being
shimmed.

# Proof of concept

At my day job, we had a programming task that I thought logic
programming would be well-suited for. Unfortunately, logic programming
is kind of weird and esoteric. Even programmers with otherwise broad
experiences professed to not being quite sure how it worked, or what
to do with it.

Therefore, I used up my hack day (a day where we get to hack on random
projects) to cook up some cool stuff using logic programming. I demoed
the usual suspects ([the monkey with the banana][monkey], and a
[sudoku solver][sudoku]), illustrating the difference between the
relational nature of the logic programs and the imperative nature of
the algorithms you might otherwise write to solve the same problems.
Finally, I demoed the aforementioned proxying API shim. The proof of
concept, codenamed shimmer, is [up on Github][shimmer].

[monkey]: https://github.com/lvh/shimmer/blob/master/src/shimmer/monkey.clj
[sudoku]: https://github.com/lvh/shimmer/blob/master/src/shimmer/sudoku.clj
[shimmer]: https://github.com/lvh/shimmer/

Let's take a look at the handler function, which takes incoming
requests and modifies them slightly so they can be passed on:

```clojure
(defn build-handler
  [target-host target-port]
  (fn [incoming-request]
    (if (match (spy incoming-request))
      (let [modified-request (-> incoming-request
                                 (dissoc :scheme) ;; hack
                                 (assoc :host target-host
                                        :port target-port
                                        :throw-exceptions false))]
        (spy (request (spy modified-request))))
      {:status 403 ;; Forbidden
       :headers {"content-type" "text/plain"}
       :body "Doesn't match!"})))
```

(Those `spy` calls are from the excellent [`timbre`][timbre]
library. They make it easy to log values without cluttering up your
code; a godsend while developing with some libraries you're not
terribly familiar with.)

[timbre]: https://github.com/ptaoussanis/timbre

The matching function looks like this:

```clojure
(defn match
  "Checks if the request is allowed."
  [req]
  (not= (l/run 1 [q]
          (l/conde
           [(l/featurec req {:request-method :get})]
           [(l/featurec req {:request-method :post
                             :headers {"x-some-header"
                                       "the right header value"}})]
           [(l/featurec req {:request-method :post})
            (l/featurec req {:headers {"x-some-header"
                                       "another right header value"}})]))
        '()))
```

# Future work

Make this thing actually vaguely correct. That means e.g. also
inspecting the body for URL references, and changing those to go
through the proxy as well.

Start collecting a library of short hand notations for specific API
functionality, e.g. if you're proxying an OpenStack API, you should be
able to just say you want to allow server creation requests, without
having to figure out exactly what those requests look like.

The spec is hard-coded, it should be specified at runtime. That was
trickier than I had originally anticipated: the vast majority of
`core.logic` behavior uses macros. While some functionality is fairly
easy to port, that's probably a red herring: I don't want to port a
gazillion macros. As an example, here's `conds`, which is just`conde`
as a function (except without support for logical conjunction per
disjunctive set of goals):

```clojure
(defn ^:private conds
  "Like conde, but a function."
  [goals]
  (if (empty? goals)
    l/fail
    (l/conde [(first goals)]
             [(conds (rest goals))])))
```

That's not the worst function, but let's just say I see a lot of
`macroexpand` in my future if I'm going to take this seriously.

URLs and bodies should be parsed, so that you can write assertions
against structured data, or against URL patterns, instead of specific
URLs.

If I ever end up letting any of this be a serious part of my day job,
I'm going to invest a ton of time improving the documentation for both
`core.logic` and `core.typed`. They're *fantastic* projects, but
they're harder to get started with than they could be, and that's a
shame.
