---
date: 2026-01-09
authors:
  - mikesamuel
draft: false
---

# Why a new language?

<picture>
  <source srcset="/blog/images/you-do-not-have-to-reinvent-the-wheel-dm.jpg" media="(prefers-color-scheme: dark)">
  <img src="/blog/images/you-do-not-have-to-reinvent-the-wheel-lm.jpg" alt="[Abstract picture of a man with a large wagon wheel tiled horizontally] You do not have to reinvent the wheel.  You do not have to reinvent the wheel.  You do not have to ..." width="1536" height="512">
</picture>

Temper works by translating to other languages.  Why design a whole
new language?  Why not just translate an existing langauge?  We
wrestled with this question an awful lot.  The tldr is that, language
translation is hard. Really, really hard.

Existing languages co-evolved with their runtimes; Python makes it
easy to express functions that the Python runtime can run. That's
different from what the JVM, Java's runtime, can do.

We did a lot of *compartive linguistics*, looking at how programming
languages do the same things in subtly (frustratingly (mind bogglingly
(just plain &hellip; deep breathes))) different ways.  (More dives into
that in future blog posts)

What we realized was, you just can't translate code in existing languages,
in a semantics preserving way.  Sure, you can sit down and *port* some
code from Python into Java.  If you know that Python *int*s are more like
Java *BigInteger*s then you can more closely match the original behaviour.
And you can even have AI tools port code for you.

If you're porting a few hundred lines from one language to another, you
probably won't run into giant differences in behaviour.  And if you
had a good test suite and you port the tests, you can probably fix them.

If you're porting tens of thousands of lines of code, then you are almost
guaranteed to run into significant differences.  Somewhere in a codebase
that large, there are *embedded assumptions*: the code makes some assumption
about the source language, that just isn't true for the target language.

But note here, *porting* involves writing equivalent code, then
testing and fixing.  After you've ported, you've got a new codebase to
maintain.  It's not like automatic translation, where you maintain one
codebase, one single source of truth, and just retranslate when you need
to publish a new version.

----

Let's look at some concrete examples.  Many languages allow for
*heterogeneous collections*.  Storing values of different *types*
alongside each other in a list or key/value map.

```py
Python 3.14.0 (main, Oct  7 2025, 09:34:52) [Clang 17.0.0 (clang-1700.0.13.3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from math import nan
>>> m = {
...   0: '0',
...   False: 'False',
...   -0.0: '-0.0',
...   0.0: '0.0',
...   nan: 'nan',
... }
>>> m
{0: '0.0', nan: 'nan'}
>>> 0 == False == 0.0
True
>>>
```

Here, we created a map (dictionary in Python) with some values:
integer zero, boolean false, floating point zero, and a special float
value nan.

But the resulting dictionary only has two entries.  Python conflated
the first four entries; it's hashing is consistent with its
coercing `==` operator.

Let's try Java.

```java
|  Welcome to JShell -- Version 21.0.4
|  For an introduction type: /help intro

jshell> import java.util.LinkedHashMap

jshell> var m = new LinkedHashMap<Object, String>()
m ==> {}

jshell> m.put(0, "0"); m.put(0.0D, "0.0D"); m.put(-0.0D, "-0.0D"); m.put(false, "false"); m.put(Double.NaN, "NaN")
$3 ==> null
$4 ==> null
$5 ==> null
$6 ==> null
$7 ==> null

jshell> m
m ==> {0=0, 0.0=0.0D, -0.0=-0.0D, false=false, NaN=NaN}

jshell> m.get(0.0)
$9 ==> "0.0D"

jshell> m.get(-0.0)
$10 ==> "-0.0D"
```

Java preserves all the values.  Its wrapper objects don't compare equal to each other.
Let's give JavaScript a spin.

```js
Welcome to Node.js v25.1.0.
Type ".help" for more information.
> let m = {
|   0: '0',
|   [-0.0]: '-0.0',
|   0.0: '0.0',
|   false: 'false',
|   NaN: 'NaN',
| };
undefined
> m
{ '0': '0.0', false: 'false', NaN: 'NaN' }
> m[0.0]
'0.0'
> m[NaN]
'NaN'
> m['NaN']
'NaN'
```

The JavaScript `{...}` looks like Python but the results are closer
but not the same as Java.  But at the end there: wtw?!?  JavaScript is
actually just coercing all the keys to string values.  JavaScript
object syntax isn't analogous to either Python *dict*s or Java *HashMap*s.

How languages deal with heterogeneous keys is super language specific.
Language features that look equivalent actually turn out to be a
semantic tarpit.  Temper does not have heterogeneously keyed maps, but
since we're designing a language from scratch, we can plan for that.

Temper is not just providing a toolbox for writing libraries; we don't
just provide a dict type and a string type and leave it up to users to
deal with semantic corner cases.  We want Temper's users to be able to
support communities whose languages they don't know.  That means
consistent semantics, and carefully choosing language features that
are not only familiar, not only portable, but which are also
automatically translatable.

So translation is hard, really really hard.  But designing a language
for this one purpose, gives us the flexibility we need to let Temper
developers support many language communities, including users of
languages they have no idea about.
