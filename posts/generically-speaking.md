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

To translate well across programming languages, we need to impose some order on generic operations like&mdash;well&mdash;ordering. The more generic the code, the more important context is. Join us as we pick over a soup of generic jargon: runtime type information, monomorphization, inheritance and erasure. Finally we'll discuss how some type declaration tweaks lets Temper translate well to many many languages.

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

> If both operands are numeric strings, or one operand is a number and the other one is a numeric string, then the comparison is done numerically. These rules also apply to the switch statement. The type conversion does not take place when the comparison is === or !== as this involves comparing the type as well as the value.

PHP's default sort flag, *SORT\_REGULAR*, tries to side-step confusion between numeric and string order by defining a different order for *numeric strings* than for *regular strings*. The authors can see the appeal for this design choice in a language that often deals with lists of strings from numeric database columns, but potentially picking a different sort order for different pairs of elements trades one set of bugs for another.

So any sorting code just need to always specify an order. Problem solved.

## Generic Functions

But this article isn't about ordering. It's about **generic functions**. A generic function is one that can work with many types. For example, sorting is generic when the sorting algorithm can be *generalized* to sort lists of numbers, strings, dates, etc. But as shown, context matters; you often need to specialize a generic function with a specific type's conception of, in the case of *sort*, ordering.

Rather than a full sorting algorithm, let's consider a simpler generic function.

- *leastValueOf* takes a list of some type, *T*, and returns the least element.
- It also takes a *fallback* value which it returns when the list is empty.
- Unless specified otherwise it compares values using *T*'s natural order.

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

We'll discuss Temper's goals for genericity as a language meant to
translate well to all these others.

Then, we'll discuss common features of languages that someone implementing
an equivalent function in a specific language might lean on to implement
*leastValueOf*.

Finally, we'll show *leastValueOf* implement in Temper and explain how
we achieve faithful translation by lowering some trait-like definitional
elements into more widely available language affordances.

## Goals

Temper needs **libraries of common algorithms**. Specialist library authors should be able to write generic functions that many generalists can use to support their own efforts. Not every Temper user will need to write generic functions, but most will need to use them.

Parts of generic functions need to **specialize behavior** based on type bindings.  *leastElementOf\<String\>* should internally be able to compare based on string order, differently from *leastElementOf\<Int\>*. Ideally the generalist would not have to specify *String* vs *Int* comparison; type inference should handle that in the common case.

Temper needs the **flexibility to connect** multiple Temper types to the same target language type. When translating to a language that does not often distinguish between strings and numbers, Temper code should not conflate string operations and numeric operations.

Temper needs to **provide generic libraries to other languages**. A Java developer, for example, using the Java translation of a Temper library should be able to call a generic Temper function with a Java type that Temper has never heard of.

## Runtime type information

In typed languages the compiler has type information, but even dynamic languages often have type infromation.

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

Maybe generic functions could get important context by looking at the "type" of an input when the function is called.

Even Perl has types[^perldata]:

> Perl has three built-in data types: scalars, arrays of scalars, and associative arrays of scalars, known as "hashes". A scalar is a single string (of any size, limited only by the available memory), number, or a reference to something

Oops. Perl has one *scalar* type which is used to represent strings, integers, floating point numbers, and booleans.

[^perldata]: From [Perldoc &sect; perldata](https://perldoc.perl.org/perldata)

Perl and PHP both have added [experimental functions](https://perldoc.perl.org/builtin#created_as_number) like *is_bool* that report approximate runtime type information. When used carefully, they allow libraries to do generic conversion of records to JSON, for example emitting JSON `false` instead of `0` depending on whether the scalar came from a logical operator like `!` or `||`. The Temper designers want to avoid reliance on brittle RTTI because it imposes a recurring tax on library users to write code in an unnatural way.

RTTI is not sufficient, even among dynamic languages, to let Temper meet its goal of specializing behaviour based on actual type parameter bindings.

- RTTI may be slow to access.
- RTTI is not available when there are no inputs of that type.  How for example, could a function using RTTI determine the zero value for a type given an empty list.
- RTTI is often partial. RTTI might identify an array as an array, but in not a list of anything in particular.
- RTTI often conflates related types: Besides Perl 5's scalar, JavaScript's `number` type does not distinguish between integers and floating point numbers.

## Monomorphization

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
