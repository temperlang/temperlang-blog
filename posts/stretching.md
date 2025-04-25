---
date: 2025-03-21
authors:
  - mikesamuel
---

# Stretching the strangeness budget to bridge language communities

The tools we have at hand shape how we work with others, especially
the languages we use to express ourselves. When I have a language in
common with you, it’s easier for us to identify common problems, craft
shared solutions, and the effort spent testing and improving benefits
many; without, we each reinvent wonky wheels.

The software community is fragmented. Data scientists use Python, the
web platform uses JavaScript, mobile apps and backends are written in
yet another language, and legacy systems … ugh.

Temper is a language designed to translate well to all. It fills this
gap in our collective toolbox to allows sharing common solutions
across all the language communities in an engineering organisation, or
across the whole open-source community. One developer or a small team
can use Temper to write a library that translates into all the other
languages, supporting many language communities.

A blessing and a curse: different languages are often the right tool
for different jobs, but each has its own quirks. This series of
articles explores these differences. Each starts with a short snippet
of code, and looks at the problems with a naïve translation into
various languages. Translating faithfully and providing idiomatic
interfaces is hard, so understanding these differences is crucial to
bridging language communities.

<!-- more -->

The first article, [*A Tangle of Strings*](./tangle-of-strings.md),
starts small.  It has nothing to do with entertaining cats, but, maybe
`cat`. Can we write simple string functions that translate efficiently
while preserving semantics? Yes, Betteridge’s law notwithstanding, but
to do so in a way that is efficient and sensible to the working
programmer, we need to first disentangle semantic yarn.

*When Words Collide* deals with naming collisions? Languages have subtly
(and grossly) different approaches to naming style, overloading,
overriding, masking, key word reservation.

*Generically Speaking* talks about generic operations. Some languages
don’t distinguish between strings and integers at runtime (Perl, PHP),
others between integers and floats (JavaScript, Lua and many
others). How can we translate a simple generic minimum function that
uses a lexicographic string comparison given strings and numeric
comparison given ints? How can we do this without expanding code size
(as monomorphization tends to) for languages like JavaScript where
code has to be sent over the wire?

*Well, I Never!, But When I Do* discusses bottom types. Some languages
have a type that conveys “expressions of this type never produce a
value” which might be called Never or Nothing. How does Temper’s
bottom-esque type let us translate well to languages that have them
and those that don’t and avoid producing translations that trip up
“Unreachable code” compiler checks.

*Cast Away: TFW Your BFF may be a Volleyball* discusses type casting,
checking whether a value is of a type before proceeding to treat it
that way. As noted above, some languages answer “is this value an int,
a float, a boolean, or a string” with a resounding “yes!” Other
languages represent dates as eight character strings: “20241231” or
“tomorrow”. How can Temper provide reliable casting sufficient for
important use cases?

*Failure Is Not Not an Option* but Some Option is not a Failure?! This
article discusses Nicomachean Ethics: Aristotle’s take on the virtues
of exceptions vs result checking for error handling.
Exception handling is also super important to programmers. An
interface isn’t idiomatic if programmers can’t rely on practices that
have worked before to meet failure-handling responsibilities.

*I Love You, You Love Me, ARGGHH There Goes Timely Deallocation* is
about the big purple dinosaur of memory management: reference
cycles. Can we enable developers used to garbage-collecting runtimes a
way to write memory safe libraries? How does this fit with our “do as
OCaml does and things just work” design principle? Can developers who
aren’t used to thinking about ownership craft APIs, like W3 DOM trees,
that appear cyclic?

*The Problem With Introspection Is That It Has No End*, nor, PKD, does
it have portable semantics. But introspection, and its mirror image
reflection, are necessary in many language ecosystems for critical
operations like serialization. How can Temper support those operations
without sinking into a semantic tarpit?

*When We Go Low, We Go High* talks about an opportunity. Some dynamic
languages pair well with a compied language: CPython with C, JS with
Wasm. When translating, can we get the best of both?: idiomatic
dynamic language interfaces that delegate most of the work to
ahead-of-time optimized code.
