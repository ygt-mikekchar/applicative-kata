# Applicative Kata

This is a kata I have adapted from Mighty Byte's Monad Challenge for Haskell:
http://mightybyte.github.io/monad-challenges/pages/set1.html The intent of the
original challenge is to guide the user into a more natural understanding of
how monads are constructed and why you need monads. I've implemented the Random
Number portion of this kata in many languages and I always enjoy it.  This is a
language agnotistic description of what to do.

While the original kata came from Mighty Byte's challenge, I've modified this
kata and changed it's goal somewhat.  The idea is less to understand monads per
se, but rather to get a good understanding of applicative functors and why you
would want to use them.

FIXME: I'm really not selling this kata very well.  This is by far the best
kata I've done for really cementing certain functional programming ideas in my
head.  I especially like doing this kata in different programming languages.
This helps me imagine how to use those FP techniques in that language, which
may not directly facilitate FP.  I'd like to explain this in the into but I
don't want to spend too much space.

## General Problem Statement

Imagine you have a function that generates a random number.  In pseudo code it
might look like this:

```
function rand_int(int seed) -> (int rand, int new_seed)
```

In other words, we have a function that takes an integer, which is the seed for
a random number generation.  The function returns a tuple (or a structure of
some kind) the contains an integer which is a random number as well as the new
seed.

We can use this function like this:

```
(rand1, seed) = rand_int(42)
(rand2, seed) = rand_int(seed)
(rand3, seed) = rand_int(seed)
etc...
```

Each time we call `rand_int` we get a new seed which we use for the *next* call
to random.  This is how we get our random numbers.

You can imagine writing a function that gets `n` random ints:

```
function rand_ints(int n, int seed) -> (Array<int> rands, int new_seed)
```

It just loops `n` times setting doing the process described above.  It's not
difficult to write.  We're going to play around with this function and
especially consider how we might write such a function immutably.

## Pseudo-Random Number Generation

Computers do not usually generate actual random numbers when you call a
function like `rand_int`.  Discussion of pseudo-random number generators is
outside of the scope of this document, but if you are interested, a good first
step might be Wikipedia:
https://en.wikipedia.org/wiki/Pseudorandom_number_generator

What you need to know for this kata is that a pseudo-random number generator
generates a series of output in a specified range.  The output is
deterministic, but not easily predictable without running the algorithm itself.

The random number function takes a "seed" value and generates an output.  It
always returns the same output for the same seed value.  For example, if your
"seed" is 42, the random number function might always output 2432.  The random
number function *also* outputs the next seed.  You use that seed to get the
next number in the series.  Pseudo-random number generators are built so that
the output is uniformly distributed in the desired range.

### Step 1.  Building a "Pseudo-random" generator function

We could use a good pseudo-random number generator to give realistic output for
our kata, but there is no real need.  Instead our "pseduo-random" generator
will be much simpler:

  - If your language supports types, make a type or type alias
    called `Seed` that is an integer.
  - Also, if possible, create a type called `RandInt` that is a tuple
    or structure composed of an integer value and a `Seed`.  This
    will be the return type of our random number generator.  Note:
    It's tempting to parameterise the integer and make something like
    `Rand<int>`.  Don't do that just yet.
  - Write a function called `rand_int` that takes a `Seed` as input
    and outputs a `RandInt`
  - The "random value" that `rand_int` returns will be equal to the
    seed that you used as input.  The next seed it returns will
	  be equal to the previous seed plus 1.

The effect is that if you call `rand_int(1)` you will receive `(1, 2)`.  If you
call `rand_int(2)` you will receive `(2, 3)`, etc.  Not a great random number
generator, but it will be easy to test.

You should write unit tests for this kata.  There is a lot of refactoring
involved and it will save you a lot of time ensuring that you haven't broken
something along the way.  Since you can't easily test an infinite series (and
such a challenge is not what this kata is for), just test a couple of examples.

### Step 2. An Array of Random Integers

Write a function, called `rand_ints`, that generates an array (or list or
vector, or whatever is appropriate in your language) of `n` "random" integers.
Start with a seed of 1.  If you call `rand_ints(5, 1)` (`n` = 5 and `seed` =
1), the output should be `[1, 2, 3, 4, 5]`.

There are lots of ways to cheat with this, but try not to.  You will need to
call `rand_int(1)` to get the first "random" number.  After that you will need to
use the seed from the previous random number to generate the next random
number.

### Step 3. Different kinds of random values

We want to make a different random generator.  This one will return "random"
chars.  It will be called `rand_char` and it will take a `Seed` and return a
tuple with a `char` and a `Seed`.   If you are using a language with type
checking, make a type called `RandChar`.  `rand_char(1)` should return
`('a', 2)`, `rand_char(2)` should return `('b', 3)`, etc.

Additionally, write a function, called `rand_chars`, that that takes an
integer, `n` and a `Seed` and returns a *string* of `n` "random" letters.  If
you call `rand_chars(3, 1)` the output should be "abc".  Note that we are
generating a string here, not an array or list.  For reasons.

FIXME: Do we actually have a reason any more?  The original reason was so that
when we implement the applicative we can make sure to pass in some kind of
concatenation function for building the output.  I don't know how easy that is
in some languages, so maybe it's not a great idea.  However, I'll keep this
here for now and we'll see how it goes.

### Step 4. Many kinds of generator functions

We really want to make a variety of different random numbers.
Implement the following:

  - `rand_even_int`: The same as rand, but each random number is
    multiplied by 2.
  - `rand_odd_int`: The same as rand, but rach random number is multplied
    by 2 and has one added to it (`x * 2 + 1`)

### Step 5. Time to refactor

We could do a million of these generator functions.  It would be better to
factor out the common code.  In the next step, we'll talk about how it would
generally be done in a Functional Programming way.  Spend some time thinking
about how you might refactor your code before you start to look at the next
step.

### Step 6. Making a functor

As it happens, what we are trying to do is very common.  There
is a special word for the thing that we are trying to build:
"functor".

A "functor", as a programming tool, is really a special word for a container.
It has to follow some special rules, but any container you are familiar with is
likely to also be a functor.  All functors must have a function associated with
it, usually called `map` or `fmap` (for "functor map").  We will be sticking
with `map` for this kata.

FIXME: Equating a functor to a container is pretty misleading from a category
theory perspective, even though it makes it much easier to understand from
a programming perspective.  It would be better to find a less lossy way of
describing what's going on.

You may be familiar with `map` on arrays from a variety of different languages.
However, `map` can be used on any functor.  To be more precise, if you can
implement `map` for the container, then it is a functor.  `map` takes the
contents out of a container, applies a transformation to the contents and then
puts the results back into the same kind of container.

The `map` that we normally use on an array applies the transformation to
everying in the array.  However, you can imagine writing the same thing for
other kinds of containers.  Remember that our `RandInt` type is just a container
(tuple or structure) that contains an integer and a `Seed`.  We will implement
`map` for `RandInt`, thus making it a functor.  Depending on your language, you
may be able to associate a function called `map` to your `RandInt` type, but
in this documentation we'll refer to the function as `rand_int_map` to make
it a bit more clear what function we're talking about.

FIXME: I'm not sure if object oriented decomposition or Rust style traits
will work well here because the reverse the parameter list.  We want the
`RandInt` to be the *last* parameter so that we can do eta decomposition.
Will have to think about it.

  - Write a function called `rand_int_map` that takes a function which
    transforms an integer and a `RandInt`.  `rand_int_map` should look
    something like:
    `function rand_int_map((function(x int) -> int) f, RandInt r) -> RandInt`
    Or in other words, the function, f, that you pass to `rand_int_map`
    should take an integer and return an integer.
  - `rand_int_map` should take the integer in the `RandInt`, r, and pass it
    to the function `f`.  It should then return a new `RandInt` value
    with the result of `f` and the `Seed` from r.
  - Implement `rand_even_int` and `rand_odd_int` using `rand_int_map`

#### Side note: Why is it called a "Functor"?

It seems like a strange name for a container.  In Category theory, a
"functor" is actually a "mapping" between 2 categories.  You can
consider a category as a set of objects.  For example, the
set of positive integers is a category.  The set of unicode characters
is also a category.

Imagine you have a set of integers from 0-9, called A.  Then imagine
a set of integers from 1-10, called B.  We can easily "map" between
A and B with a function like: `|x| x + 1`, called `add_one`.  Using
`add_one` we can "map" between the categories A and B.

This is what's known as a "functor" and you can probably see why it
has a name that is similar to "function".  We usually say that "A is a
functor", even though we don't specify the function that we are using
to map it with.  What we mean by this is that we can easily write a
function called `map` that will map integers from A to some other
(or the same) category, given a transforming function.

To really be a functor, the `map` function must adhere to 2
conditions:

  - There must be a kind of "identity" mapping: a mapping that maps
    from every item in the category to itself.  For numbers, that function
    would be `|x| x + 0`.
  - The functions must be "composable".  So if we have the functions
    `let f = |x| x + 1` and `let g = |x| x + 2`, we could "compose"
	the 2 functions into `let fg = |x| (x + 2) + 1`.  To be a functor,
	`f(g(x))` must be equal to `fg(x)` for all functions `f` and `g`.

Things like numbers tend to be functors because you can easily write
`map` so that those conditions apply.

You might be wondering, "Why is a functor a container?"  Imagine a
single number as being a "container" of one number.  We can also
have a set of tuples: for example the set of `(0, 0), (0, 1), (1, 0),
(1, 1)`.  You can probably imagine that this is a functor as well
because we can easily write `map` to work on each value in the tuple.
The same goes for arrays, strings, and many other containers.

That's why we say that a container that implements `map` is a functor.

### Step 7. What about `rand_char`?

  - Create a `rand_char_map` implementation for RandChar.

At this point it's tempting to implement the `rand_char` function
with `rand_int_map`.  But there is a problem.  `rand_int_map` maps
an `int` to an `int`.  Similarly `rand_char_map` maps a `char` to
a `char`.  We have nothing that maps an `int` to a `char`.

If you are using a language without type checking, you will not
really have a problem.  You should notice that the implementation
of `rand_int_map` and `rand_char_map` are identical. In fact, you can
pass *any* function into it.  So at this point, you can just have
a single function called `rand_map` and use it for both.  You can
implement `rand_char` using the `rand_map` function easily.

With a language that does type checking we have a problem.

### Step 7a. Without type checking

If your language does not use type checking, it's easy:

  - rename `rand_int_map` to be `rand_map`
  - implement `rand_char` using `rand_map`

### Step 7b. Generics if you've got 'em

For languages with generics:

  - Create a single `Rand<T>` type to replace `RandInt` and `RandChar`.

Now we have to implement `rand_map`.  This is a bit awkward as the
type needs to look a bit like this:

`function<T,U> rand_map(function f(T x) -> U, Rand<T>) -> Rand<U>`

  - Implement this function if you can with the language you are using
  - Implement `rand_char` using your new `rand_map`

### Step 7c. Type checking languages without generics.

FIXME: I don't know what to do.  Possibly just live with the fact
that you can't mix your functor types...  It's not the end of the
world.  `rand_letter` just has to have an ugly implementation.

### Step 8. Generating Random Pairs

Write a function, called `rand_pair` that takes a `Seed` and outputs
a tuple like the following: `((char, int), Seed)`. For example,
`rand_pair(1)` should return `(('a', 2), 3)`.  Basically you have
to use the seed in the return value from `rand_letter` as the seed
for `rand`.

  - Create a type `Pair` that is a tuple of a `char` and an `int`
    if your language has type checking.
  - Create a type `RandPair` if your language has type checking but
    no generics.
  - implement `rand_pair`

Note: It will be very tempting to implement this function using
`rand_map`, but for now resist that temptation.

### Step 9. Generator Types

It would be nice to be able to output pairs of any kind of type.

First, if you have generics:
  - Implement `Pair` as `Pair<T, U>`

If you have a type checking language and don't have generics, it's
not really possible to do this.  We'll have to make a different
type for each kind of pair, which kind of sucks.

We're going to write a helper function called `general_pair`.
We should be able to implement `rand_pair` something like:

`function rand_pair(Seed seed) -> RandPair { general_pair(rand_char, rand_int, seed) }`

However, the function definition for `general_pair` is a bit
difficult to write.

If you don't have generics, you can write something like:
  - `function general_pair(function(Seed) -> RandInt, function(Seed) -> RandChar, Seed) -> RandPair`

However, you will have to implement that for each kind of RandPair.
This is the price you pay for not having generics: you become king of cut and
paste.

If you have generics, you need something like:
  - `function<T,U> general_pair(function(Seed) -> Rand<T>, function(Seed) -> Rand<U>, Seed) -> Rand<Pair<T, U>>`

Either way, it's kind of awful.  If you don't have type checking, the
following might seem not very useful, but it solves some problems later,
so please just go with the flow.

Really, our problem is that we need a type for the generator functions.
For languages with generics:

  - Create a type `Gen<T>` that is `function(Seed) -> Rand<T>`

For languages without generics:

  - Create `GenInt`, `GenChar` and `GenPair` types.

For languages without type checking:

  - Keep in mind that we're going to be passing these generator
    functions around and doing some clever things with them.  When
    you are naming a variable that contains a generator, consider
    calling is `gen_t`, for example.

With this, we can define `general_pair` as with generics:
  - `function<T, U> general_pair(Gen<T> gen_t, Gen<U> gen_u) -> RandPair<T, U>`

or without generics:
  - `function general_pair(GenChar gen_t, GenInt gen_u) -> RandPair`

or without type checking:
  - `function general_pair(gen_t, gen_u)`

  - implement `general_pair`
  - Refactor `rand_pair` to use `general_pair`.

### Step 10. Returning a Function

`general_pair` is potentially a nice improvement (as long as you have
generics or don't do type checking).  However, there is no particular
reason that `general_pair` needs to be called immediately.  Imagine that
instead of returning a `RandPair`, it returned a function that takes
a seed and returned a `RandPair`: which is, of course, a `GenPair`.

Generics:
  - `function<T, U> general_pair(Gen<T> gen_t, Gen<U> gen_u) -> Gen<Pair<T, U>>`

No Generics:
  - `function general_pair(GenChar gen_t, GenInt gen_u) -> GenPair`

For a variety of reasons, this will be helpful later.

  - Refactor `general_pair` to return a `GenPair`
  - Call the `GenPair` returned from `general_pair` in `rand_pair`.

### Step 11. An Inconvenient Truth

Looking at the previous step, you might realise that `general_pair`
is doing something rather interesting.  It takes 2 `Gen<T>`s
and combines them into a final `Gen<T>`.  Looking at the signature
of the version with generics makes that a bit more obvious.

What's *really* interesting, though, is that we're using the seed
generated from `gen_t` in the application of `gen_u`.  This should
give you the idea that we can do some general transformations
of these `Gen` types to produce different kinds of random values.
The other thing that's really interesting is that `general_pair`
doesn't take a `Seed` parameter.  We do all of our transformations
without taking the `Seed` -- yet.  Instead we return a function
that allows us to specify the seed and then generate our random
values.

Remember that `Gen<T>` is just a function, so we are combining these
functions and returning another function.  Often in functional
programming this is called "function composition".  We can do all
sorts of composition on the functions and when we are done, we run
the result passing in the parameters.

In our quest to create an "applicative functor" we need to do these
kinds of transformations.  However, the `Seed` is definitely going
to get in our way.  In fact, while `Rand<T>` is a functor (we
implemented `map` on it), it is *not* an applicative functor.

We aren't going to go down the road of failing to implement an
applicative functor on `Rand<T>` right now.  I highly recommend
going back later and trying to implement it as it is very
instructive.  But for now we'll stick to the path that will give
us the results we want.

While `Rand<T>` is not an applicative functor, `Gen<T>` *is*.
That might seem very strange.  `Gen<T>` is a function!  Didn't
we say that functors are "containers"?  How is a function a container?
Well, you can imagine that a function "contains" the value that
it evaluates to.

For example: `function foo { 42 }` returns `42` when called.  You
can imagine that it contains the value `42`.  The function
`function bar(int x) { x }` contains a value as well.  However
the value is not defined until we actually call the function,
specifying the parameters.  We can pass `bar` around, but it's value
is essentially *deferred* until we call it.  If you have ever
worked with promises in some languages, it is a very similar concept.

Since `Gen<T>` is a functor, let's implement `map` for it (although
we'll call it `gen_map`.  It should look like:

Generics:
  - `function<T,U> gen_map(function(T x) -> U, Gen<T>) -> Gen<U>`

No Generics:
  - `function gen_int_map(function(int x) -> int, GenInt) -> GenInt`

  - Implement `gen_map` for `Gen<T>` (or without generics implement
    `gen_int_map`, etc).  `gen_map` should look like
  - Rewrite `rand_letter`, `rand_even` and `rand_odd` to use `gen_map` on
    `Gen<T>`.

FIXME: I have not updated the document past this point

### Step 12. Working with Pairs

It would be nice to build pairs of random values, but not just in a
tuple.  For example, making a pair of random values in a `String`.

  - Write a function `gen_lift2` that allows you to do this.

Hint: It's very similar to `gen_map`, but with 2 `Gen<T>` values
instead of one.

#### Step 13. Applicative Functors

`gen_lift2` is very similar to `gen_map`, but combines the output of 2
`Gen<T>`s into one.  The function you pass to `gen_lift2` should have a type
signature of `fn(A, B) -> C`.  Similar to `general_pair`,
it's very tempting to use `gen_map` in the implementation.  `gen_map`
uses a function with a signature of `fn(A) -> B`, though.  If we could
somehow convert `fn(A, B) -> C` into `fn(A) -> B`, then we could use it.

In a more usual functional language, you would simply "curry" the
value.  For example, if I am supposed to call a function like
`my_func(a, b)`, I can "partially apply" the function by passing only
the first parameter.  The result is a function that takes the second
parameter. In other words, I can do `my_func(a)(b)` and it will have
the same output.

Let's say that I am writing `gen_lift2` and I just naively pass the
incoming function directly to `gen_map`. Assume `f: Fn(A, B) -> C`,
`ga: Gen<A>` and `gb: Gen<B>`.  So ideally, what would `gen_map(f, ga)`
return?

In an ideal world, it would return `Gen<Fn(B) -> C>` because it would
just partially apply the first parameter to the function and return a
new function wrapped in a `Gen`.

What we need now is a function that can apply the *second* (and
potentially subsequent) parameters to the function.  That function is
called `apply`.  If you can write `apply` for your functor, then you
have an "applicative functor".  As we saw, not every functor is
applicative.

  - Write a function `gen_apply` that takes a `Gen<Fn(A) -> B>` and
    a `Gen<A>` and returns `Gen<B>`.
  - Refactor `gen_lift2` to use `gen_map` and `gen_apply`.
  - Make a functor called `Applicative` that gives you `apply`, `pure` and
    `lift2`.  Make `Gen<T>` `Applicative`.

Note: This is quite tricky and requires that you understand the lifetimes
of your variables/functions.  I *think* you also need to make the function
that you pass to `apply` `FnOnce` because you need to partially apply
the function when you run `map`.  I'm still a Rust novice, so it's possible
I'm missing something.

### Step 17. Vectors of Generators

Being able to generate a pair of random values is slightly useful.
However, we really want to be able to generate an arbitrarily sized
vector of random values.  What would be nice is:

  - Write a function, `gen_sequence`, that takes a `Vec` of `Gen<T>` and
    generates a `Gen<Vec<T>>`.  In orther words it takes a vector of
    generators and returns a Generator that gives you a random vector
    of values.

As a usage example:
```
  let gs = vec![rand, rand, rand, rand, rand];
  assert_eq!(vec![1, 2, 3, 4, 5], gen_sequence(gs)(1).0);
```

  Note: you can implement this with and without `apply`.  This is
  because Rust iterators give you enough capability to avoid using
  `apply` in this instance.  Implement both so you can see the
  difference.

### Step 18. Cleaning Up

You may think the battle has been won at this point, and you are
correct.

  - Refactor `five_rands` and `three_rand_letters` using
    `gen_sequence`.  Do not use `mut`.

We may need to clean up a bit.  When you implemented `gen_sequence`
you will have had to deal with the case where the vector passed to the
function is empty (you *did* handle that didn't you?).  In that case
you will have had to make a `Gen` that held an empty array.

  - Use `pure` for that case if you haven't already.

This is why you need `pure`.

#### Step 19. Traversable

`Applicative` is a natural consequence of wanting to use functors with
functions that have more than one parameter.  Usually `apply` is used
as a kind of stepping stone to get what you actually want.  We saw
that when we implemented `gen_sequence`.

`sequence` converts a vector containing applicative functors to an
applicative functor that contains a vector.  It is normally paired
with another function called `traverse`.

  - Implement `gen_traverse`:  It takes a `fb(A) -> Gen(A)`, a `Vec<A>`
    and returns `Gen<Vec<A>>`.
  - Create a `Traversable` trait with `sequence` and `traverse`.

Note: `traverse` can always be implemented with `sequence` and
vice versa.  Which one you implement first when you are making
a `Traversable` is up to you.

Special Note: This version of `Traversable` is actually incorrect.
`Traversable` works on *any* traversable functor, not just vectors.
However, to keep this already very long kata within bounds, we will
just implement it for vectors.

### Step 20. Chaining functions

Note: This section needs some work to make it more obvious why you
want to do this.

Recall the generators `rand_even` and `rand_odd`.  They are almost the
same.  One of them applies the function `x * 2` and the other applies
the function `x * 2 + 1`.  It would be interesting to define one in
terms of the other.

  - Write a function called `gen_bind` that takes a `Gen<A>`
    and a `Fn(A -> Gen<B>)` and returns a `Gen<B>`.
  - Make a trait called `Monad` that requires `bind` and `pure` and
    give `Gen<T>` that trait.
  - Implement `rand_odd` usinging `rand_even` and `bind`
