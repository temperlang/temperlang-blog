---
date: 2025-03-25
authors:
  - mikesamuel
draft: true
---

# Generically Speaking

<small>(This is part of a [series of articles](./stretching.md) each of which highlights a difficulty in translating a familiar programming concept across programming languages and highlights design choices to bridge that gap)</small>

Many programmers cut their teeth on sorting algorithms. Indeed, the French word for *computer*, *ordinateur*, comes from a word that means sorter.

Everyone makes mistakes, but if there's one thing programmers get right, it's sorting.

<div class="grid" markdown>

```py
🐚$ python3
Python 3.13.2 (main, Feb  4 2025, …
Type "help", "copyright", "credits"…
>>> array_of_nums = [9, 7, 10, 11, 8]
>>> array_of_nums.sort()
>>> array_of_nums
[7, 8, 9, 10, 11]
```

```js
🐚$ node
Welcome to Node.js v23.11.0.
Type ".help" for more information.
> let arrayOfNums = [9, 7, 10, 11, 8]
undefined
> arrayOfNums.sort()
[ 10, 11, 7, 8, 9 ]
```

</div>

That array! *Quel désarroi?!*

Temper aims to translate well to all the other programming languages. To do that, we need to impose some order on generic operations like&mdash;well&mdash;ordering. The more generic the code, the more important context is. Join us as we look at languages' differing approaches to genericity and find some novel tweaks that let us translate generic functions well to many many languages.

<!-- more -->

Which language is the odd duck here? Let's try a few more languages.

<div class="grid" markdown>

```perl
🐚$ perl -de1

Loading DB routines from perl5db.pl version 1.80
Editor support available.

Enter h or 'h h' for help, or 'man perldebug' for more help.

main::(-e:1):	1
  DB<1> @nums = (9, 7, 10, 11, 8);

  DB<2> print(join(", ", (sort @nums)))
10, 11, 7, 8, 9
```

```php
🐚$ php -a
Interactive shell

php > $nums = array(9, 7, 10, 11, 8);
php > sort($nums);
php > print(implode(", ", $nums));
7, 8, 9, 10, 11
```

```ruby
🐚$ irb

WARNING: This version of ruby is …

irb(main):001:0> nums = [9, 7, 10, 11, 8]
=> [9, 7, 10, 11, 8]
irb(main):002:0> nums.sort
=> [7, 8, 9, 10, 11]
irb(main):003:0> [9, 7, "10", 11, 8].sort
Traceback (most recent call last):
        …
        1: from (irb):3:in `sort'
ArgumentError (comparison of Integer with String failed)
```

```lua
🐚$ lua
Lua 5.4.7  Copyright (C) 1994-2024 Lua.org, PUC-Rio
> nums = {9, 7, 10, 11, 8}
> table.sort(nums)
> print(table.concat(nums, ", "))
7, 8, 9, 10, 11
```

</div>

Dynamic languages seem to slightly prefer `[7, 8, 9, 10, 11]` over `[10, 11, 7, 8, 9]`. But is that what's actually going on?

When everything else fails, read the manual. [JavaScript's Array sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) documentation notes:

> If `compareFn` is not supplied, all non-undefined array elements are sorted by converting them to strings and comparing strings in UTF-16 code units order.

Ok. Perhaps we missed an input. Let's try again in JavaScript.

```js
🐚$ node
Welcome to Node.js v23.11.0.
Type ".help" for more information.
> let arrayOfNums = [9, 7, 10, 11, 8];
undefined
> arrayOfNums.sort((a, b) => a - b)
[ 7, 8, 9, 10, 11 ]
> let arrayOfMostlyStrings = ["9", "7", 10, "11", "8"];
undefined
> arrayOfMostlyStrings.sort((a, b) => a - b)
[ '7', '8', '9', 10, '11' ]
```

But how did PHP, which was heavily influenced by Perl know to compare numerically above?[^php.operators.comparison]

[^php.operators.comparison]: [php.net Manual &sect; Comparison Operators](https://www.php.net/manual/en/language.operators.comparison.php)

> If both operands are numeric strings, or one operand is a number and the other one is a numeric string, then the comparison is done numerically. These rules also apply to the switch statement. The type conversion does not take place when the comparison is `===` or `!==` as this involves comparing the type as well as the value.

PHP's default sort flag, *SORT\_REGULAR*, tries to side-step confusion between numeric and string order by defining a different order for *numeric strings* than for *regular strings*. The authors can see the appeal for this design choice in a language that often deals with lists of strings from numeric database columns, but potentially picking a different sort order for different pairs of elements trades one set of bugs for another.

So any sorting code just need to always specify an order. Problem solved.

## Generic Functions

But this article isn't about ordering. It's about **generic functions**. A generic function is one that can work with many types. For example, sorting is generic when the sorting algorithm can be *generalized* to sort lists of numbers, strings, dates, etc. But as shown, context matters; you often need to specialize a generic function with a specific type's conception of, in the case of *sort*, ordering.

As seen below, a language for generic programming has to allow mixing a general framework of *type-agnostic* instructions with a few specialized instructions whose behaviour depends on the *actual* type *T*.

```ts
/** A string form of the given list like "[..., ..., ...]". */
let stringifyList<T>(list: List<T>): String {
  let sb = new StringBuilder();
  sb.append("[");
  var needsComma = false;
  for (let element of list) {
    if (needsComma) { sb.append(", "); }
    sb.append(
      ⚠️ element ⚠️
    );
    needsComma = true;
  }
  sb.append("]");
  return sb.toString();
}
```

All of that code except for the line between the ⚠️s is generic.
Its behaviour does not depend on the type of list element.

But that one line really matters to the semantics of the function.
Here's boolean list formatting code in three widely used languages when
we *don't* take care to specify exactly how the stringification happens.
We get three different strings: `[True, False]`, `[1, ]`, and `[true, false]`.

<div class="grid" markdown>

```py
🐚$ python3
Python 3.13.2 (main, Feb  4 2025, …
Type "help", "copyright", "credits"…
>>> some_bools = [1 + 1 == 2, 1 + 1 != 2]
>>> print("[%s]" % ", ".join([str(x) for x in some_bools]))
[True, False]
```

```sh
🐚$ perl -e '@some_bools = (1 + 1 == 2, 1 + 1 != 2)' \
>        -e 'print("[" . join(", ", @some_bools) . "]\n")'
[1, ]
```

```js
🐚$ node
Welcome to Node.js v23.11.0.
Type ".help" for more information.
> let someBools = [1 + 1 === 2, 1 + 1 !== 2];
undefined
> console.log(`[${someBools.join(", ")}]`)
[true, false]
```

</div>

## Why Generics Matter?

Generics functions are important tools in library authors' tool kits, but there's a more basic reason for paying attention to generics in Temper.

Generic operations, like stringifying a list of booleans above, are bedrocks for well tested code.  They're essential to programmers' testing and debugging flow.

Unit tests often bundle and compare results using **generic** collections.
Unit tests can be **readable and expressive** by using strings that consistently and unambiguously describe the expected output.

```ts
assert(
  functionThatReturnsListOfBooleans(...) == "[true, false]"
);
```

Debugging and logging code **converts arbitrary types** to a standard form.
When that conversion is consistent, the developer has to do less cognitive work to model the system and is led down fewer false trails.

```ts
console.log(
  "Got %s",
  myListOfBooleans
);
```

Temper aims to help developers write high-quality, well tested libraries.
Programmers debugging a library translated into many languages should be able to write unit tests and insert debugging statements as they would in every other language they work in.

Without semantically consistent generic operations, developers will suffer a steeper learning curve when testing and debugging Temper code, and the same debugging effort will yield less improvement.

## A Sample Generic Function: Least Value Of

For the rest of this article, let's consider a function that's a bit more complex than list stringifying but is simpler than a full sorting algorithm.

Below is some pseudo code for a function, *leastValueOf*, that takes a list of some type, *T*, and returns the least element.
It also takes a *fallback* value which it returns when the list is empty.
Unless specified otherwise it compares values using *T*'s natural order.

```js
leastValueOf①(elements, fallback, ②):
  if elements is empty:
    return fallback

  // Handled the no-element-0 case above.
  let leastSoFar = elements[0]

  // Loop over the elements afer 0 comparing each once.
  for i in [1, len(elements)):
    let elementᵢ = elements[i]
    if leastSoFar > elementᵢ③:
      leastSoFar = elementᵢ

  // leastSoFar has been compared to every element in elements, so
  // assuming the ordering is transitive, leastSoFar is the least
  // among elements.
  return leastSoFar
```

The circled numbers shows that this pseudocode glosses over some important details:

1. Typed language will often require some kind of type parameter declaration, like `<T>` possibly with type bounds that show that `T`s can be compared to each other.
2. We talked about taking an order as a parameter when natural order is not desired.  What if you want the least string in a case-insensitive comparison?  Different languages support that in different ways.
3. The `>` performs some *type-specific* comparison.

The next few sections will look at common features of programming languages and how they might be used to implement this function and the pros and cons of trying to translate generic functions using those features.

Afterwards, this article details Temper's goals for type genericity, outlines the approach Temper uses and gives examples of how this will translate in languages that are representative of the groups discussed.


## Runtime type information (RTTI)

In typed languages the compiler has access to type information, but even dynamic languages often have type information.

Runtime type information (RTTI) is available when a language provides operators that distinguish between types of values as the program is running.

<div class="grid" markdown>

```js
🐚$ node
Welcome to Node.js v23.11.0.
Type ".help" for more information.
> typeof "string"
'string'
> typeof 123
'number'
> typeof false
'boolean'
```

```py
🐚$ python3
Python 3.13.2 (main, Feb  4 2025, …
Type "help", "copyright", "credits"…
>>> type('string')
<class 'str'>
>>> type(123)
<class 'int'>
>>> type(False)
<class 'bool'>
>>> isinstance(False, bool)
True
>>> isinstance(False, int)
True
```

</div>

We could implement our generic function in JavaScript/TypeScript as below, using RTTI checks to pick a default comparison function when none is specified.

```ts
export function leastValueOf<T>(
  elements: Array<T>, fallback: T,
  compare: (a: T, b: T) => (-1 | 0 | 1) = pickComparisonFnFor(fallback)
):
  if (!elements.length) {
    return fallback;
  }

  let [leastSoFar, ...rest] = elements;

  for (let element of rest) {
    if (compare(leastSoFar, element) > 0) {
      leastSoFar = element;
    }
  }

  return leastSoFar;
}

function pickComparisonFnFor<T>(x: T): (a: T, b: T) => (-1 | 0 | 1) {
  if (typeof x === 'object' || typeof x === 'function') {
    // A referency type.
    // Try using .compareTo method.
    return (a: T, b: T) => {
      if (a === n) { return 0; }
      if (!a) { return -1; } // Sort null early
      if (!b) { return 1; }
      let delta = +(a?.compareTo(b) ?? 0);
      return !delta ? 0 : Math.sign(delta);
    };
  }
  return ((a, b) =>
    (a === b) ?  0 :
    (a <   b) ? -1 : 1);
}
```

This code is dodgy, but it demonstrates that if you have an instance, in languages with RTTI you can sometimes pick a strategy as the program is running.

How widespread is RTTI? Perl5 has types[^perldata]:

> Perl has three built-in data types: scalars, arrays of scalars, and associative arrays of scalars, known as "hashes". A scalar is a single string (of any size, limited only by the available memory), number, or a reference to something

Oops. Perl has one *scalar* type which is used to represent strings, integers, floating point numbers, and booleans.

[^perldata]: From [Perldoc &sect; perldata](https://perldoc.perl.org/perldata)

Perl and PHP both have added [experimental functions](https://perldoc.perl.org/builtin#created_as_number) like *is_bool* that report approximate runtime type information. When used carefully, they allow libraries to do generic conversion of records to JSON, for example emitting JSON `false` instead of `0` depending on whether the scalar came from a logical operator like `!` or `||`. The Temper designers want to avoid reliance on brittle RTTI because it imposes a recurring tax on library users to write code in an unnatural way.

JavaScript distinguishes between strings and numbers but not between small integers and floating point numbers.

RTTI is not sufficient, even among widely used dynamic languages, to let Temper meet its goal of specializing behaviour based on actual type parameter bindings.

- RTTI is not available when there are no inputs of that type.  How for example, could a function using RTTI determine a type's *distinguished zero value* given an empty list.
- RTTI often conflates related types: Besides Perl 5's scalar, JavaScript's `number` type does not distinguish between integers and floating point numbers.

## Monomorphization

Monomorphization means "one-shaping."  In monomorphizing languages, you can define a generic function, but then the language makes sure that each function call is to a function that only deals with one shape of input.  Typically this is by creating, under the hood, a copy[^rust-mono] of the generic function where type-specific operations are specialized for the actual type.  Rust traits and C++ templates are monomorphized.

```rust
fn leastValueOf<T>(elements: Vec<T>, fallback: T) -> T
where T: Ord + Copy,
{
    if elements.is_empty() {
        return fallback;
    }

    let mut leastSoFar = elements[0];
    for &element in &elements[1..] {
        if element < leastSoFar {
            leastSoFar = element;
        }
    }

    leastSoFar
}
```

The `<` operation in the Rust code is rewritten, by the monomorphizer to be expressed in terms of `<T>`'s *impl*ementation of *Ord.cmp*.  So if you used *leastValueOf* with a concrete struct *MyStruct* it might be calling into a specialized copy as if you had written this:

```rust
fn leastValueOf<MyStruct>(elements: Vec<MyStruct>, fallback: MyStruct) -> MyStruct
{
    if elements.is_empty() {
        return fallback;
    }

    let mut leastSoFar = elements[0];
    for &element in &elements[1..] {
        if MyStruct::cmp(element, leastSoFar) == Ordering::Less {
            leastSoFar = element;
        }
    }

    leastSoFar
}
```

When monomorphization happens is highly implementation dependent, but it requires one of two things: **whole program analysis** or **tight integration of the monomorphizer with the runtime**.

Whole program analysis lets the language define all needed parameterizations ahead of time.  For example, if the only calls to *leastValueOf* are *leastValueOf\<String\>(&hellip;)* and *leastValueOf\<Int\>(&hellip;)* then the compiler can generate those two variants and link the calls to them.

Unfortunately, Temper is a language for libraries, not whole programs. Temper never has access to the whole program.  The Temper compiler can't generate monomorphized variants of generic functions for code written in other languages which use libraries translated from Temper as it never sees them. Also, this variety of monomorphization tends to increase code size which is terrible for languages like JavaScript which ship source code to the browser.

Tight runtime integration involves running the monomorphizer when a previously uncompiled parameterization is found. The Julia language takes a just-in-time compiler[^julia-ORCv2] approach to monomorphization[^julia-code-caching].

Unfortunately, Temper is a language that needs to produce translations that load into any runtime.  Temper has no control over the runtime, because it has no runtime.  We can't expect other languages' runtimes to support dynamic code loading much less embed the Temper compiler. Even if we could, doing so would conflict with our goal to support runtimes that use [non-executable stacks](https://en.wikipedia.org/wiki/Executable-space_protection) as a protective measure against buffer overflows.

[^rust-mono]: [Rust Compiler Development Guide &sect; Monomorphization](https://rustc-dev-guide.rust-lang.org/backend/monomorph.html) explains monomorphization thus: compiler stamps out a different copy of the code of a generic function for each concrete type needed.

[^julia-ORCv2]: The [Julia JIT](https://docs.julialang.org/en/v1/devdocs/jit/) allows pausing function calls to specialize a function, invoke the compiler, and load code before resuming.

[^julia-code-caching]: "[Julia's latency: Past, present and future](https://viralinstruction.com/posts/latency/index.html#a_brief_primer_of_julia_code_caching)" talks about monomorphization as a way to balance performance with Julia's design goals: interactive use by a programmer prevents whole program analysis but for performance they need devirtualization.

There are definite use cases for monomorphization, and Temper may one day support explicit, coarse-grained, ahead-of-time monomorphization based on a mechanism like OCaml's parameterized modules. But it is the wrong tool for translating generic function definitions to other programming languages.

## Virtual Method Calls

In Java, our *leastValueOf* function could delegate the generic operation to an abstract method.

TODO



## Java brittleness

Even typed languages have some brittleness around. Java's numeric primitives don't always behave the same as their boxed equivalents[^java-Double-compareTo-caveat].

[^java-Double-compareTo-caveat]: Java's comparison operators perform NaN poisoning comparison, but the [*Double.compareTo*](https://docs.oracle.com/javase/8/docs/api/java/lang/Double.html#compareTo-java.lang.Double-) method implements a valid ordering predicate.  See "There are two ways in which comparisons performed by this method differ from those performed by the Java language numerical comparison operators &hellip;"

```java
🐚$ jshell
|  Welcome to JShell -- Version 21.0.4
|  For an introduction type: /help intro

jshell> double a = Double.NaN, b = 1.0D;
a ==> NaN
b ==> 1.0

jshell> Double A = a, B = b;
A ==> NaN
B ==> 1.0

jshell> a > b
$5 ==> false

jshell> A.compareTo(B) > 0
$6 ==> true
```



## Implementing leastValueOf

```temper
/**
 * The least value in *list* or *fallback* if *list* is empty.
 * All comparisons are according to *cmp*.
 */
let leastElementOf<T supports Comparison<T>>(
  list: List<T>,
  fallback: T,
  cmp reifies T,
): T {
  let n = list.length;
  if (n == 0) { return fallback }

  var min = list[0];

  for (var i = 1; i < n; ++i) {
    let candidate = list[i];
    if (cmp.compare(min, candidate) < 0) {
      min = candidate;
    }
  }

  min
}

// We should be able to apply leastElementOf to lists of
// strings or numbers and in both cases, get the least according to
// the "natural" order.
test("leastElementOf") {
  assert(leastElementOf<String>(["c", "a", "b"], "") == "a");
  assert(leastElementOf<Int   >([  3,   1,   2],  0) ==   1);
}

```

This is code in Temper that doesn't currently work. We're implementing parts of this scheme as I write this.  Note though that:

- `leastElementOf` is a generic function; it declares a type parameter `<T>` using a syntax familiar to C++, C#, Rust, and TypeScript developers
- That type parameter has an upper type bound, `Comparison<T>`, declared with an unfamiliar keyword: `supports`
- The function takes three inputs and the last one, `cmp` doesn't have a type but it says it *reifies* the type parameter `T`.
- That input, `cmp`, is used to compare values of type `T` obtained from the list.
- In the unit test at the bottom, no value is explicitly passed for `cmp`.



## Goals

Temper needs **libraries of common algorithms**. Specialist library authors should be able to write generic functions that many generalists can use to support their own efforts. Not every Temper user will need to write generic functions, but most will need to use them.

Parts of generic functions need to **specialize behavior** based on type bindings.  *leastElementOf\<String\>* should internally be able to compare based on string order, differently from *leastElementOf\<Int\>*. Ideally the generalist would not have to specify *String* vs *Int* comparison; type inference should handle that in the common case.

Temper needs the **flexibility to connect** multiple Temper types to the same target language type. When translating to a language that does not often distinguish between strings and numbers, Temper code should not conflate string operations and numeric operations.

Temper needs to **provide generic libraries to other languages**. A Java developer, for example, using the Java translation of a Temper library should be able to call a generic Temper function with a Java type that Temper has never heard of.


In Java our function might look like the below:

```java
public static <T>
T leastValueOf(
  Iterable<T> elements,
  T fallback,
  Comparator<T> comparator
) {
  Iterator<T> it = elements.iterator();
  // Use fallback if there's nothing
  if (!it.hasNext()) {
    return fallback;
  }
  T leastSoFar = it.next();
  // Loop over the elements after the first swapping
  // them if leastSoFar is strictly greater according to
  // comparator.
  while (it.hasNext()) {
    T candidate = it.next();
    if (comparator.compare(leastSoFar, candidate) > 0) {
      leastSoFar = candidate;
    }
  }
  // We've considered all elements so leastSoFar is the least
  // assuming comparator is transitive.
  return leastSoFar;
}

public static <T extends Comparable<? super T>>
T leastValueOf(
  Iterable<T> elements,
  T fallback
) {
  return leastValueOf(elements, fallback, Comparator.<T>naturalOrder());
}
```
