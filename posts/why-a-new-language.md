---
date: 2025-03-21
authors:
  - mikesamuel
draft: true
---

# Why a new language?

![You do not have to reinvent the wheel.  You do not have to reinvent the wheel.  You do not have to ...](./images/you-do-not-have-to-reinvent-the-wheel.svg){ width="1500" height="500" }

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

What we realized was, you just can't translate existing languages,
in a semantics preserving way.  Sure, you can sit down and *port* some
code from Python into Java.  If you know that Python *int*s are more like
Java *BigInteger*s then you can more closely match the original behaviour.
And you can even have AI tools port code for you.  (Now you've got two
codebases to maintain; go you).

If you're porting a few hundred lines from one language to another, you
probably won't run into giant differences in behaviour.  And if you
had a good test suite and you port the tests, you can probably fix them.
If you're porting tens of thousands of lines of code, then you are almost
guaranteed to run into significant differences.  Somewhere in a codebase
that large, there are *embedded assumptions*: the code makes some assumption
about the source language, that just isn't true for the target language.

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
the other three, for the purposes of key hashing to be consistent with
its coercing `==` operator.

```java
|  Welcome to JShell -- Version 21.0.4
|  For an introduction type: /help intro

jshell> import java.util.LinkedHashMap

jshell> var m = new LinkedHashMap<Object, String>()
m ==> {}

jshell> m.put(0, "0"); m.put(0.0D, "0.0D"); m.put(false, "false"); m.put(Double.NaN, "NaN")
$3 ==> null
$4 ==> null
$5 ==> null
$6 ==> null

jshell> m
m ==> {0=0, 0.0=0.0D, false=false, NaN=NaN}

jshell> m.get(0.0)
$8 ==> "0.0D"
```

Java preserves all the values.  Its wrapper objects don't compare equal to each other.

```js
Welcome to Node.js v25.1.0.
Type ".help" for more information.
> let m = {
|   0: '0',
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

JavaScript seems more like Java, but still there at the end, wtw?
JavaScript is actually just coercing all the keys to string values.
You can use JavaScript's *Map* type with heterogeneous keys, but not
the object style, and JavaScript *Map* doesn't extend to user
functions since JavaScript has no standard library support for a
custom equality idioms.

The details of how languages deal with heterogeneous keys is super
language specific.  I've only shown languages that have numbers,
booleans, and support for NaN values, but even so there are too many
differences to reliably translate large codebases.  Temper does not
have heterogeneously keyed maps, but knowing that, we can provide
other strategies to users.

Temper is not just providing a toolbox for writing libraries; you
don't just get a string type and it's up to you to deal with semantic
corner cases.  We want Temper's users to be able to support
communities whose languages they don't know.  That means consistent
semantics, and consistent semantics require carefully choosing
language features that are not only familiar, not only portable, but
which are also translatable.

Temper aims to *translate* reliably to other languages, not just
*port* easily.  In *porting*, you expect to have to go back and fix
bugs due to difference in language features, but then you need to
separately maintain each port.  Temper enables reliable, automatic
translation: the same semantics regardless of where a function is
applied.  You don't maintain each translation, you just change one
Temper source, and re-translate.
