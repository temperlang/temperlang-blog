---
date: 2025-03-21
authors:
  - mikesamuel
draft: true
---

# Stretching the strangeness budget to bridge language communities

This series of articles explores differences between programming
languages&mdash;differences that might make each language the right
tool for a different job, but that complicate translating concepts
across languages.

What good is a programming language that translates to all the others?
If its semantics don't translate faithfully, not a lot. If they do,
developers can easily identify common problems and craft shared
solutions even across language communities&mdash;the software
community becomes less divided despite working on different
stacks. Data scientists use Python, the web platform uses JavaScript,
mobile app and backend people each use their own preferred languages
(and legacy systems&mdash;ugh). But our different stacks shouldn't divide us.

*Temper* is a programming language designed for one thing: translating
well to the others.  It fills this gap in our collective toolbox; it
allows sharing common solutions across all the language communities in
an engineering organisation, or across the whole open-source
community. One developer or a small team can use Temper to write a
library that translates into all the other languages, supporting many
language communities.

Each article in this series starts with a code fragment and explains
problems with straightforward translations of it into various
languages. Translating faithfully and providing idiomatic interfaces
is hard, so understanding these differences is crucial to bridging
language communities and preventing the languages we work in from
dictating whom we can collaborate with.

<!-- more -->

The first article, [*A Tangle of Strings*](./tangle-of-strings.md),
starts small.  Strings are great for entertaining `cat`s, but did you
know they're good for other things too! Can we write simple string
functions that translate efficiently while preserving semantics? To do
so in a way that is both efficient and sensible to the working
programmer, we need to first disentangle some semantic yarn.

*Generically Speaking* talks about generic operations. Some languages
don’t distinguish between strings and integers at runtime (Perl, PHP),
others between integers and floats (JavaScript, Lua and many
others). How can we translate a simple, generic *minimum* function so
that it performs string comparison given strings and numeric comparison
given ints? How can we do this while keeping code size small for
languages like JavaScript where code is sent over the network?

*When Words Collide* deals with naming collisions. Languages have
subtly (and grossly) different approaches to naming style,
overloading, overriding, masking, keyword reservation. What can one
do when the perfect name for a function in one language is forbidden
in another?

*Well, I Never!, But When I Do* discusses bottom types. Some languages
have a type that conveys “expressions of this type never produce a
value,” which you might've seen called *Never* or *Nothing*. How does
Temper’s bottom-esque type let us translate well to both languages that
have them and those that don’t and avoid producing translations that
trip up unreachable-code compiler checks?

*Cast Away: TFW Your BFF is a Volleyball* discusses type casting,
checking whether a value is of a type before proceeding to treat it
that way. As noted above, some languages answer “is this value an int,
a float, a boolean, or a string” with a resounding “yes!” Other
languages represent dates as eight character strings: “20241231” or
“tomorrow.” How can Temper provide reliable casting sufficient for
important use cases?

*Failure Is Not Not an Option*?! This article discusses
Nicomachean Ethics: Aristotle’s take on the virtues of exceptions vs
result checking for error handling. Failure handling is also super
important to programmers. An interface isn’t idiomatic if programmers'
hard-learned habits don't help them meet their failure-handling
responsibilities.

*I Love You, You Love Me, We're Surrounded by Uncollected Garbage* is
about the big purple dinosaur of memory management: reference
cycles. Can we give developers used to garbage-collecting runtimes a
way to write memory-safe libraries? How does this fit with our “when
you do as OCaml does, things just work” design principle? Can
developers who aren’t used to thinking about ownership craft
interfaces that appear cyclic, like W3 DOM trees?

*The Problem With Introspection Is That It Has No End*, nor, PKD, does
it have portable semantics. But introspection, and its mirror image
reflection, are necessary in many language ecosystems for critical
operations like serialization. Can Temper support those operations
without sinking into a semantic tarpit?

*When We Go Low, We Go High* talks about an opportunity. Some dynamic
languages pair well with a compiled language: CPython with C, JS with
Wasm. When translating, can we get the best of both?: idiomatic
dynamic language interfaces that delegate most of the work to
ahead-of-time optimized code. Can Temper make it easier for an
organization gradually adding types to pre-exisitng code in a language
that acquired a type system late in its development?
