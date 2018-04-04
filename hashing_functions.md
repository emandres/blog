# Hashing Functions

## What is a hashing function?

A hashing function is a one-way function that takes some input and returns a deterministic output. The output is often referred to as a digest, a hash code, or simply a hash.

## Why do I need one?

Hashing function have many uses, but in cryptography we're most interested in a few properties<sup>[1](https://en.wikipedia.org/wiki/Cryptographic_hash_function)</sup>.

1. Deterministic - the same input always returns the same output
2. Fast - they can be computed relatively quickly (although, in many applications slowness can be a desirable trait).
3. One-way - it is infeasible to reproduce the input given the output.
4. Chaotic - a small change in the input will produce an output so different that it is impossible to correlate the two by looking at outputs alone.
5. Uniqueness - it is infeasible to find two inputs that produce the same output.

In cryptography, hashing functions are often used as message authentication code (MAC). MACs that are produced by a hashing function are often referred to as HMACs.

So what is a MAC, and what is it good for? Let's say I'm sending a message.

    Give $10 to Bob

If an attacker were to intercept the message, they could change it in flight.

    Give $1000 to Bob
    Give $10 to Eve

To prevent this we can try appending a MAC.

    { "message": "Give $10 to Bob", mac: "12345" }

But now, the attacker just has to modify the mac and the message will look legitimate.

    { "message": "Give $10 to Eve", mac: "6789" }

To prevent this, we can add a secret when we calculate the MAC.

    // secret = puppies
    { "message": "Give $10 to Bob", mac: "28573" }

Unless the attacker knows the secret, it will be incredibly difficult to contrive a message that matches the given MAC (see property 5). Please note that, while this may seem like a simple security scheme to implement, there are multiple ways in which naive implementations can be exploited. Always use crypto code that has been built and vetted by security experts. If you're looking in your languages standard libraries, the functions will often be called something like `HMACAlgorithm`. For instance, the .Net framework has a class called [`HMACSHA256`](https://msdn.microsoft.com/en-us/library/system.security.cryptography.hmacsha256(v=vs.110).aspx) under the `System.Security.Cryptography` namespace. This algorithm uses the `SHA-2 256` algorithm to compute an HMAC.

Another usage of hashing algorithms is in password verification, but that's another blog post.

## How do they work?

Let's imagine a simplistic hashing function.

    LENGTH_HASH(x) = len(x)

That is, given some input `x`, we're going to return the of the input in bytes. This brings us to one of the desirable traits of a good hashing function: uniqueness. With this simplistic hashing function, many inputs – an infinite number, actually – return the same result.

    LENGTH_HASH('foo') = 3
    LENGTH_HASH('bar') = 3

Depending on our use case, this might be good enough, but not for cryptographic uses. Let's consider something a little more complex.

    SUM_HASH(x) =
        LET sum = 0
        FOR c IN x
            sum += c
        RETURN sum

So, lets say that our input is restricted to uppercase alpha characters, with characters assigned values starting at 1 (i.e. `A=1`, `B=2`, ...). Then we have some more varied input

    SUM_HASH('ABC') = 1 + 2 + 3 = 6
    SUM_HASH('BCD') = 2 + 3 + 4 = 9

We've improved our output a bit. But what if we transpose some of the characters?

    SUM_HASH('ABC') = 6
    SUM_HASH('CBA') = 6

Remember, we're going for unique output, so this slightly less naive hashing function isn't going to cut it, either.

Another desirable trait for a hashing function is opacity. In other words, given the output, it should be difficult to produce the input.


## Which algorithm should I use?