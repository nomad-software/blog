![](/articles/images/more-hidden-treasure-in-the-d-standard-library-banner.jpg)

# More hidden treasure in the D standard library

<time>Posted on 31st August 2015 by [Gary Willoughby](/pages/about.html)</time>

After the success of the last article detailing [hidden treasures in the D standard library](/articles/hidden-treasure-in-the-d-standard-library.html), I thought I would write another highlighting why the [D programming language](https://dlang.org/) coupled with its great standard library is surprisingly useful. The library itself is a vast beast and has been written by some exceptional programmers, so occasionally you stumble across some truly useful and well designed nuggets of code. This article shows a few more of these hidden treasures and provides examples of where they could be useful when used in your projects.

The following code examples make good use of D’s uniform function call syntax and template metaprogramming capabilities. Don’t be put off by this as very simple explanations for these can be found [here](/articles/templates-in-d-explained.html) and [here](/articles/alternative-function-syntax-in-d-banner.html).

## std.algorithm

This package is huge and implements generic algorithms oriented towards the processing of sequences. Sequences processed by these functions (for the most part) define range based interfaces.

### predSwitch

This function returns one of a defined collection of expressions based on the value of a switch expression. While this sounds quite complicated, here’s a simple example implementing the standard [fizzbuzz](https://en.wikipedia.org/wiki/Fizz_buzz) program.

<script src="https://gist.github.com/nomad-software/5cf58ca213f0a112228c441163ab8ff7.js"></script>

The `predSwitch` function takes one template parameter, which defines a [predicate](https://en.wikipedia.org/wiki/Predicate_(mathematical_logic)). This predicate is used to choose which expression to return from the function. The first function parameter is a switch expression, which will replace the `a` in the predicate. (This is shown in the above example as the `input` variable using [UFCS](https://en.wikipedia.org/wiki/Predicate_(mathematical_logic))). The next parameters are [variadic](https://en.wikipedia.org/wiki/Variadic_function) and define pairs of test expressions and return expressions. In the above code `15` will replace the `b` in the predicate, while its pair `"fizzbuzz"` defines the return expression if the predicate evaluates to true depending on the value of `input`. Many pairs can be defined and the last parameter is the default return expression if there are no matches.

At first glance it looks like it’s just a reimplementation of an [if](https://en.wikipedia.org/wiki/Conditional_(computer_programming)#If.E2.80.93then.28.E2.80.93else.29) or [switch](https://en.wikipedia.org/wiki/Switch_statement) statement until you realise the function parameters can actually be expressions and all variadic arguments are evaluated [lazily](https://en.wikipedia.org/wiki/Lazy_evaluation). This helps write cleaner code where before it would be littered with conditional statements. [View Documentation](https://dlang.org/phobos/std_algorithm_comparison.html#.predSwitch).

## std.bitmanip

This module was created solely for the purpose of manipulating [bits](https://en.wikipedia.org/wiki/Bit). It contains tools for [low-level programming](https://en.wikipedia.org/wiki/Low-level_programming_language) and can be used in many surprisingly useful ways.

### bitfields

This template is used for the creation of [struct](https://en.wikipedia.org/wiki/Struct_(C_programming_language)) or [class](https://en.wikipedia.org/wiki/Class_(computer_programming)) fields while controlling how many bits each field uses. Using this template, you specify the types, names and bit sizes of each field as parameters. The total bit size must be equal to the size of a built-in integer type, which is used as the underlying data store. During compile time, code is generated by the template which can in-turn be mixed-in to be compiled into the executable. Take a look at the following code.

<script src="https://gist.github.com/nomad-software/eb9242e2b642d82ac4e386e0d91b7f4f.js"></script>

Here `foo` is only two bytes in size but contains many fields, all utilising the same [short](https://en.wikipedia.org/wiki/Integer_(computer_science)) for storage and providing access to a subset of bits. Defining structs like this can be very useful to define programmer friendly ‘views’ into binary data. [View Documentation](https://dlang.org/phobos/std_bitmanip.html#.bitfields).

## std.conv

This module implements functions for conversion between values and types. At first glance, there doesn’t seem to be a lot in here but what is here is very comprehensive.

### castFrom

This template is a simple wrapper over the standard cast operator, but its perceived redundancy is more than made up for by its usefulness. For example, when casting a value to another type, for reasons of safety and legality you need to know and verify what type that value is before the cast. Once the cast is made, the code from then on usually assumes the source type will never change. This template makes this assumption an explicit rule to avoid potential bugs from occurring. Take a look at the following snippet.

<script src="https://gist.github.com/nomad-software/3bc57639275b6fa34dbf76bdb6ccb968.js"></script>

This code makes an assumption that the `foo` variable will always be an `int`. If we change `foo` to be a pointer, everything will still compile fine but the program is obviously bugged. Here is another snippet showing how you can avoid situations like this by safely enforcing the source type.

<script src="https://gist.github.com/nomad-software/da49462fe389aabc84e8629d289167e0.js"></script>

Now if the type of `foo` changes for whatever reason you get a helpful [compile time](https://en.wikipedia.org/wiki/Compile_time) error stating what the cast was originally expecting. [View Documentation](https://dlang.org/phobos/std_conv.html#.castFrom).

## std.functional

This module implements functions that manipulate other functions. I guess this is where D implements library functions to accommodate some aspects of functional programming. It’s a bit thin on the ground in there but what _is_ there seems to be very useful.

### pipe

This template [composes functions](https://en.wikipedia.org/wiki/Function_composition_(computer_science)) together to create a chain. In this chain the result of each composed function is passed as an argument to the next and the result of the last one is the result of the whole. Here is an example.

<script src="https://gist.github.com/nomad-software/e70640daaba3362644bf0e699bb0164a.js"></script>

Here we are composing various built-in library functions to create a function called `sumString`. This function when called will chain the functions as follows.

<script src="https://gist.github.com/nomad-software/0ebaca293ac44ea4b42743eb1db0678f.js"></script>

This is great for creating complicated functions by composing existing ones in clever ways. [View Documentation](https://dlang.org/phobos/std_functional.html#.pipe).

### compose

The `compose` template functions in exactly the same way as the `pipe` template with the exception that the arguments passed are defined in the opposite order. This is purely for readability cases. Here is the above example now using `compose`.

<script src="https://gist.github.com/nomad-software/437021e1c37c1ee0ffb6087044f474af.js"></script>

Notice the order of the arguments when using `compose`. [View Documentation](https://dlang.org/phobos/std_functional.html#.compose).

## std.range

This module defines the notion of a [range](http://ddili.org/ders/d.en/ranges.html). Ranges generalise the concept of arrays, lists, or anything that involves sequential access. This abstraction enables the same set of algorithms to be used with a vast variety of different concrete types.

### generate

This is a function that when passed a callable argument will create an infinite [InputRange](https://dlang.org/phobos/std_range_primitives.html#isInputRange) whose `front` method returns the value from successive calls to the argument. This is especially useful when using a function with global side effects i.e. random functions, or to create ranges expressed as a single delegate, rather than defining the entire `front`, `popFront`, `empty` interface manually. Here’s an example.

<script src="https://gist.github.com/nomad-software/bb96126cf5b8b95de2e539800ac1806c.js"></script>

The above code generates a range from the `randNum` function and prints three elements from it. The returned range can be considered [lazy](https://en.wikipedia.org/wiki/Lazy_evaluation) because the values are only generated as they are needed. In the above case they are only generated when we call `take(3)`. During generation, `front` is called on the range which in turn then calls `randNum`. The returned range can also be considered [infinite](https://dlang.org/phobos/std_range_primitives.html#isInfinite), because it could continue to keep generating values forever.

There is however a problem with this range. Ranges should always return the same value for successive calls to `front`, this one doesn’t because it calls `randNum` each time. To achieve the desired behaviour we can leverage another function from a different package `std.algorithm.iteration.cache`. This function can be chained onto the returned range to cache the value of `front` without needing to call it again. `cache` [eagerly evaluates](https://en.wikipedia.org/wiki/Eager_evaluation) `front` on each range construction or call to `popFront`, to store the result in a cache. The result is then directly returned when `front` is called, rather than re-evaluated. Here is an amended example.

<script src="https://gist.github.com/nomad-software/f7bea2754f2b9ad4556454dfc33d2de5.js"></script>

Now successive calls to the range’s `front` method always return the same cached value, which also means it’s now semantically correct and more performant. [View Documentation](https://dlang.org/phobos/std_range.html#.generate).

## std.typecons

This module implements a variety of type constructors, i.e. templates that allow construction of new, useful general purpose types.

### Unique

This struct enables you to encapsulate unique ownership of a particular resource. Once used, the resource is deleted at the end of the current scope unless it’s transferred. A transfer can be explicit, by calling the release method, or implicit, when returning the struct from a function. The following code shows an example of how to use it.

<script src="https://gist.github.com/nomad-software/77600d79289120bbb5036c44a2cbcebd.js"></script>

The above example shows how a unique resource is created, borrowed by reference and then ownership transferred to a function by calling `release`. `isEmpty` is used to determine whether a resource is encapsulated in the struct. [View Documentation](https://dlang.org/phobos/std_typecons.html#.Unique).

## Conclusion

D provides an absolute wealth of fantastic features and advanced ideas in the standard library. Featured above are only a few of the treasures to be found. Click the documentation links above and read more about each one and see for yourself the appeal and productivity of a modern system programming language such as D.