---
# How The Rust Compiler Will Assist You In Catching Bugs

Welcome everybody!

Let's start with a few questions:

- Who knows what Rust is?
- Who knows what is, but only knows that?
- Who knows what is and has read some posts and docs, but never done anything with it?
- Who has tinker it with some exercises or pet projects?
- Who has used for some professional work?
- Who usually uses it for professional work?

This talk is mainly oriented for the first two groups of people, the third and the forth, may get
something beneficial, and the last two groups won't get anything new, unfortunately.

But hey, stay with me here if it's in your interests and help me with clarifications and to answer possible questions that
any of the attendees may have.

---
# A little bit about me

I'm Ivan Fraixedes.

I'm a software engineer interested and tinkering with Rust when I get some free time outside of
work, personal, and family duties.

Hey, realize that I'm not a Rust expert and I only dedicate time to it a little amount of hours a month.

[click]
I work for Storj. An enterprise grade decentralized cloud storage, compatible with Amazon S3, but secure and distributed
by default, and cheaper and more sustainable than Amazon S3.

Are you curious about it? Don't hesitate to visit storj.io.

---
# License

All the content of this presentation is licensed under Creative Commons By Share Alike 4.0, except the media which retains their original license.

Feel free to redistribute the material under this license terms.

---
# What's Rust?

Let's start from the beginning.

[click]
You can build anything with Rust, and in general, not focusing niches, Rust is a great language to
build operative system drivers, embedded systems, command-line applications, network services, and
Web servers, to list a few.

[click][click]
It's a compiled language, not interpreted. You compile your code and get a binary. It compiles to
multiple architectures and operative systems, between them the most spread (Linux, Mac, Windows).

[click]
It doesn't manage the memory through a garbage collection, but it doesn't require that you manually
handle the memory.

[click]
All the constants and variables require to have a type that cannot change once they are declared.

[click]
High level syntax, without compromising performance. You won't feel that you are writing code with a
low level language, but at the same tie, you will control how the data is represented in memory.

[click]
The compiler is your friend, although the beginning you will think the opposite. It will prevent you
to write code with invalid data access.

---
# Strong/Statically typing

Let's start with this straightforward feature, which isn't unique to Rust because they are many other
languages that also have it. If you are familiar with any of then, you're not going to see anything
different between Rust and them.

As mentioned in the previous slide, rust is a strongly/statically typed language.
This means that constants and variables have an initial type and never change.

This guarantees that we can never assign to a variable or a constant a value of a different type.

Here you can see an example of declaring a constant and a variable.

[click]
If we would try to declare that constant and variable with values of a different types, our code
won't compile.

That's likely a mistake that we won't do, but what about this

[click]
That won't compile either, because `n` is an `i64` and `f` an `f64`.

With these examples you are likely getting the essence of what the strongly/statically typing
provides.

Obviously, Rust types go far more than primitive types, there are advanced types that allow to
define your own types to model your business logic and use them to enforce constraints at compile
time, for example

[click]
We define a type to express an alive human temperature, only allowing to change to the bounds that
we consider that an alive human may be.

[click]
Another example are enums. A variable of the `HumanStatus` type has as a value one variant at a time.
In this example, it's `Healthy`, `Sick`, or unfortunately `Dead` ;D

[click]
Last, but not least, Rust has type inference. We don't need to specify the type when the compiler
can inference it.

In this example, `i` and `f` takes the default numeric fallback, which are `i32` and `f64`. That could
be different if those variables are used in places where `i64` or `f32` are specified, in those cases
they would be of those types.

In the case of the line 6, the vector (a vector is a dynamic array of a concrete type) doesn't specify
what's the type of its elements at the time that's declared, but the Rust compiler inferences it from
the following line when it adds the `i2` variable which is a `i32`.

The last case, the `after` variable, it's declared in the first line without specifying what types has.
The compiler inferences the type in the line 9, where the variable is initialized when it's assigned
to `f`, which is of the type `f64`, hence, `after` is a `f64`.

---
# Variables & Mutability

In Rust, the variables are immutable by default.
You can mutate them, but you have to explicitly indicate it.

[click]
You use the `mut` keyword to specify that a variable is mutable.

Explicitly having to indicate that a variable is mutable, it isn't only beneficial to force you to
think when a variable's value needs to change, it's a way to clarify in your code what variables
change their value. While you may think that you don't need this kind of hint, other people reading
your code or you reading code written by other people will appreciate it and makes easier to
understand what the code does.

[click]
This is a more clear example. These 2 functions take 2 strings and return another string.

While we can write the behavior on the functions' documentation and that's a good practice, not
always everybody signals that one of the function arguments gets modified and write a short sentence
like the ones that I did in this example. However, in Rust, we know that append should modify the
`base` string, and we have to pass a mutable string.

[click]
You can see here how `s` is a mutable string. Moreover, we have to specify the `mut` keyword on `sr`
in line 9. That's because `append` receives a mutable reference of `String`, so we declare `sr` as so.
The first `mut` is needed because we cannot get a mutable reference to something that isn't mutable.

We wouldn't likely write that code as I wrote it here because the `sr` is only used to passed to `append`
and we would just replace `sr` by the expression

[click]
The call to `append` with the `mut` keyword allow us to see that append is mutating the variable,
independently, while we may pass the same mutable variable to a function that doesn't mutate it,
for example

[click]
calling `concat`.

---
# Uninitialized variables

Rust doesn't allow you to use variable before they are initialized.

[click]
Here is fine, `after` is initialized before it's first usage.

NOTE I don't need to declare `after` as mutable despite it's defined in the line 1 and we assign
the value in the line 3 because the line 1 declare the variable and the line 3 initialize it.

A more valuable use case why Rust requires explicit variable initialization and not implicitly
assign default values is.

[click]
In this example, we could forget to for example assign a value in the `born` method, but with
only 2 fields, it seems a very rare error that we are going to commit, perhaps that would happen
if the `struct` would have a lot of fields. Moreover, if `u8` default value would be 0, which is
a very common value to use, and we would forget it, the result would be the same.

But imagine that while we are developing our system to monitor the humans that are born in a
hospital, we are informed that we should know their temperature because some systems will use to
fire emergencies when patiences are too low or too high.

We go to the source file where the `Human` struct is defined and we add the `temperature`, despite
we are not the ones that worked on that part of the code.

[click]
What do our changes miss?

We forgot to update the `born` method. The Rust compiler will yell us about it

[click]
This case isn't that weird, it's the opposite, it's very common. If Rust would use a 0 as a default
value for `temperature` and some monitoring system do checks before the temperature is set every X
seconds, it would fire emergency alarms, which isn't good.

---
# Exhaustive pattern matching

Rust have a powerful patter matching. It's used for vast amount of things, like conditional execution,
destructuring composite types (structs, tuples, etc), etc.

Rust has an expression called `match`, with a very nice property; it enforces exhaustive checking.

Let's see it with some examples

[click]
Let's use the same enum that we had in the previous slide and how we can perform a different operation
according to the variant that a variable has

[click][click]
This match isn't exhaustive so the compiler will yell us.

[click]
Let's fix it

[click]
And if we want to perform some of the variants, then we can use the wildcard pattern.

---
# Exhaustive pattern matching

The `match` works beyond of enums

[click]
In this example we see how the `match` expression with branches that match several values, using the
vertical line symbol (`|`)

---
# Exhaustive pattern matching

Here with ranches using the two dots (`..`)

[click]
Note that ranges include the left bound, but they don't
include the right bound unless that you use the equal symbol

---
# Exhaustive pattern matching

And an example combining them.

---
# Exhaustive pattern matching

And an example of `match` with structs

[click]
We must explicitly omit fields, otherwise the compiler complains.

[click]
We can also apply ranges on them.
See that when we have to refer to the field value, we must bind it to variable using the at (`@`)
operator.

---
# NOT Exhaustive pattern matching

In this example we can see that we use a match, which is exhaustive, but we only want to do
something when the human is sick, otherwise, we ignore it with the wildcard pattern and and
empty expression.

This is an annoying boiler plate and for this reason, Rust has some convenient pattern matching
expressions to get rid of this boilerplate.

[click]
An in other situations we would like to have some conditions to do something

[click]
Another examples is that we want to do something with a sick humans, but more constrained to the
temperature that they have.

This could be written as

[click]
Which is more expressive.

You can see that we still have to write the wildcard, otherwise the compiler yells about not
covering all the enum variants, but even with that this match isn't exhaustive and the compiler
doesn't complain. Hence the downside of the additional expressiveness is that the compiler doesn't
perform an exhaustiveness check.

Although for this case, the exhaustiveness that we get with the match expressions has the same
result because inside of the match branch we don't have conditionals that are exhaustive with all
the values that the temperature may have.

I suggest you to use this handy expressions, but don't abuse them and let the compiler yell at you
to ensure that you are not introducing a bug.

---
# NULL doesn't exist

`null` was considered to be a feature, so yeah, Rust doesn't have it.

You may wonder that in certain situations you want to express that a value is invalid or
absent.

The problem isn't the concept, is the implementation. Rust doesn't have nulls, but it has
a enum in the standard library that allows you express if a value is invalid or absent.

[click]
The `Option` enum.

The option enum is defined with a generic parameters to allow to be used with any type.
Let's see an example how to use it.

[click]
What we are doing int he `blackbox` function is returning the `Some` variant when the index isn't
out of bound, otherwise the `None` variant.

But, this doesn't compile, we get

[click]
That's because we are not returning an integer, we are returning an option.

[click]
The solution is to get the value if the options have value, so we are using something that we have
already seen. We get the value out of the enum when the variable is a `Some` variant and we do what
we want, otherwise we ignore it in this case.

Why is this a better implementation for invalid/absent values than the traditional null?
Because it helps to catch the common issues of assuming that something  isn't null when it actually
is.

---
# Error handling

Rust have a number of features for handling situations in which something goes wrong.

There are two major groups, recoverable and unrecoverable. I'm going to show examples of code about
how Rust forces you to check for errors, otherwise your code won't compile.

But let me first mention that in the group of unrecoverable errors. Rust will stop your program in
front of bugs that other programming languages, like C, treat them as undefined behavior, to avoid
possible issues as security vulnerabilities.

Now, let's move with the recoverable errors.

Rust doesn't have exceptions, instead it has the type

[click]
Yes, another enum. And as the option this enum is in the standard library and available without
having to import anything.

Observe, that the `Result` enum has 2 generic types, instead of 1 as the `Option` has. The only
thing that matters for the purpose of this talk is that the `Ok` variant has a value of a type and
the `Err` variant a value of another type.

Let's see how to use it

[click]
Does it look familiar?

I took the same example as before because it's straightforward that you see how error handling in
Rust is nothing else than working with another `Enum`.

[click]
What happened before, it happens again here

[click]
And the same as before, we have to get the value out of the `Result` type in order to access to the
integer returned by the function.

What does this way to handling errors help?

- It always forces to check if there is an error or not before accessing to the value that you
  expect.
- Function signatures that return a `Result` type makes clear that they return an error
  independently if it's documented or not.

Last and not least, Rust `Result` has several methods and an operator to propagate errors of the
same type to reduce boilerplate and make your life more pleasant.

---
# Ownership & Borrowing

Ownership is the Rust's most unique feature and has a deep implication for the rest of the
language.

I cannot go deep into it here, but I'm going to briefly present it to show you how the compiler uses
it to catch some of the most subtle bugs.

[click]
Rust use the  ownership to automatically insert the operations to free memory at the right time
guaranteeing that your code doesn't have bugs related to double free and pointers with invalid
memory addresses.

The trade-off is that we have to learn how to use the ownership system because none of the commonly
used programming languages have something like this.

As usual, the compiler will ensure that you use it and apply its rules properly, otherwise, your
code won't compile, won't execute.

Let's see an example

[click]
This code doesn't compile

[click]
Why? Because s is the owner of the `String` and `s` is created inside of scope delimited by the
curly brackets.
This code define a variable of a `String` reference in the line 1, which is assigned to the `s`
memory address at line 4, but when the scope finish, Rust will free the memory because `s` is
only accessible inside of the curly brackets and its the owner of that memory allocation.

Hence, if Rust would allow this, `sr` in the line 7 would be referencing an invalid memory address
and rather than printing `"Hi Girona"` would print something else that we cannot even know.

---
# Ownership & Borrowing

Let's see another example

Seems fine, right?

[click]
Bummer, it isn't.

What's happening here is that the `print_string` is takeing the ownership of the parameter, so `s`,
which is the owner before calling the first time `print_string` gives the ownership to `to_print`
parameter, so `s` isn't the owner any more after line 5, hence, it cannot be used in the following
line.

What we can do?

[click]
One solution is returning back the ownership.

Here, is the same as before, but the `print_string` function returns the same String, so a caller
can get back the ownership.

This is valid, but also

[click]
this one.

Note that the `s` in the line 4 is not the same `s` at line 1. Rust allows to define new variables
with the same name in the same scope, obviously invalidating the previous ones.

Or we could also do

[click]
Anyway, none of these solutions seem satisfactory. Here is where Borrowing gets into the game

---
# Ownership & Borrowing

Borrowing allows accessing to the same value from other variables without transferring changing its owner.

We borrow through references, but to fulfill with the ownership requirements, references have the
following constraints

[click][click]
We'll see in a few why Rust imposes this restrictions.

[click]
We saw this case earlier.

[click]
References are not pointers as you may know them in C or C++. You can manipulate the value that
points or assign another reference of a variable that was referencing another value, but you cannot
deliberately change the address in order to point to an arbitrary memory region.

[click]
So the solution that we expected to apply to the previous example was to pass a reference to the
`print_string` function.

To close this topic I'd like to mention that the __ownership model allows to go beyond the memory__
__management__. We can add logic when a value is destroyed, dropped using the Rust jargon, to
facilitate to the users of our types not to have to think about operations that must be
executed at that time and avoid bugs, like closing file handlers, closing network connections or
returning them to a pool, etc.

---
# Data races

Last thing that I want to show you is how Rust prevent data races when implementing concurrent
applications.

This is going to be a very short example because concurrency is an extensive topic which we could
have several talks.

Let's see the first example

[click]
First let me clarify the last 2 lines. The `join` method returns a `Result`. I'm using the Result's
`unwrap` for brevity. `unwrap` takes an error and if it's `Ok`, returns the associated value and if
it's `Err` then panics.

It's a bad practice to use `unwrap` unless, you are writing something quick and that's fine to skip
error checking or on other situations that you have a strong justification for it, but for the later
I recommend to use the `expect` method because it does the same, but you can provide a message that
will show up when the code panic because the `Result` is of the `Err` variant.

Let's focus on the overall snippet. We create two threads and we wait until they terminate. The two
vertical straight lines is the syntax that Rust uses to denote anonymous functions, a closure.

It seems fine, right?

[click]
It doesn't compile!

Threads are accessing `msg`. Rust infers how to capture `msg`, and because `println!` only needs a
reference, the closure tries to borrow `msg`. However, there is a problem: Rust doesn't know how
long the spawned thread will run, hence it doesn't know if `msg` will be valid when the threads
use it.

Let's see what we can do

[click]
We added the `move` keyword before the closures. This keyword indicates that we are changing the
owner, moving its ownership from one variable to another, in this case the closures that are
executed by the threads.

[click]
This won't work either because a value must only have an owner. This would work if would create one
of the threads, but not with 2.

At this time you see that despite this code would have been fine because we know that `msg` is
valid when the threads are executed because we are waiting for them to terminate before `msg` is
released, the Rust compile prevent us to compile this snippet because it cannot ensure that you are
doing something right, it prefers to stop you because in another scenario like this

[click]
the execution is non-deterministic and one or both threads could access to invalid memory at some
point.

Any other of the common programming languages would have allowed to do so and in with this last
example, you would have introduced a bug that you may find it when your code run in production.

Some languages may have some libraries to prevent this somehow, but that requires you to know them
rather than being something provided by the language.

you may wonder if Rust doesn't allow you to perform operations like this. The response is Yes!

[click]
We could create a copy of the `msg` and move one into the first thread and the other into the
second thread.

But this could be very inefficient in certain situations. Imagine that the `msg` is so large, for
example of some thousand megabytes. Creating a copy takes time and requires more memory.

Rust has some types in the standard library to allow performing operations like this and without
cloning the data. These types don't allow to commit bugs at the expense of wrapping our values with
them and reducing little bit the performance because they require runtime checks rather than
compile time checks.

[click]
This case is valid. `Arc` stands for "Atomically Reference Counted" and it's a thread-safe
reference-counting pointer.

In this case the `clone` method isn't cloning the data, it's incrementing the counter in one, then
Rust at runtime when the `msg` or `msg2`, owned by the threads, terminate, they decrement the
counter, and when it's 0, it knows that can release the memory because there aren't any other
reference to that memory address.

Last and not least, I would like to mention that Rust goes beyond concurrency with thread, it also
offers asynchronous programming based on futures, but that's another topic for another talk.

---
Thanks for listening

To wrap up, Rust won't save of all the bugs that you application could have, for example the ones
related to the business logic, however, it will safe at compile time of committing several subtle
ones, avoiding that they slip into production environment or your clients' computers, which is
never good or easy to spot them once they are there a part of the damage that they can cause in
certain systems.

---

And finally a bit of SPAM

La Fundació Nord, a foundation dedicated to foster the technology in the Girona province holds the
meetup group Rust Girona.

And Silicon Griona, a new community whose goal is to gather the whole Girona province technology
enthusiasts.
