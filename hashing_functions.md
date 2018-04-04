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

    { "message": "Give $10 to Bob" }

If an attacker were to intercept the message, they could change it in flight.

    { "message": "Give $1000 to Bob" }
    { "message": "Give $10 to Eve" }

To prevent this we can try appending a MAC.

    { "message": "Give $10 to Bob", mac: "12345" }

But now, the attacker just has to modify the mac and the message will look legitimate.

    { "message": "Give $10 to Eve", mac: "6789" }

To prevent this, we can add a secret when we calculate the MAC. The sender will hash the message together with the secret to produce an HMAC, and then the receiver will verify that they can calculate the same HMAC when they receive the message.

    // secret = "puppies"
    { "message": "Give $10 to Bob", hmac: "28573" }

Unless the attacker knows the secret, it will be incredibly difficult to contrive a message that matches the given MAC (see property 5).

    // secret = ??? - idk let's try 'foobar'
    { "message": "Give $100 to Eve", hmac: "56184" }

Please note that, while this may seem like a simple security scheme to implement, there are multiple ways in which naive implementations can be exploited. **Always use crypto code that has been built and vetted by security experts.**

If you're looking in your language's standard libraries, the functions will often be called something like `HMACAlgorithm`. For instance, the .Net framework has a class called [`HMACSHA256`](https://msdn.microsoft.com/en-us/library/system.security.cryptography.hmacsha256(v=vs.110).aspx) under the `System.Security.Cryptography` namespace. This algorithm uses the [`SHA-2 256`](https://en.wikipedia.org/wiki/SHA-2) algorithm to compute an HMAC.

Another usage of hashing algorithms is in password verification, but that's another blog post.

## How do they work?

Let's imagine a simplistic hashing function.

    LENGTH_HASH(x) = len(x)

That is, given some input `x`, we're going to return the of the input in bytes. This brings us back to property 5: uniqueness. With this simplistic hashing function, many inputs produce the same output.

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

We also want to make it difficult to figure out the input given the output. While it's non-trivial, we can deduce a lot about the input given its sum. If we assume that all letters are equally likely, then we expect that the value of the average character is 13 (median of the range 1-26). If we have a value around 1300, we could make a deduction that the original input was about 100 characters long. That might not seem like a lot of information if you're doing it by hand, but if you can prioritize what strings your cracking algorithm is going to try first based on length, you might save a lot of time.

This solution isn't very chaotic either. If I change my input from `ABC` to `ABD`, the input changes from 6 to 7. If I hash `ABE`, I get 8. I'm seeing a trend here....

So in summary, our `SUM_HASH` algorithm gets a 2/5. It is deterministic, and fast. Unfortunately, the remaining three properties are important enough that we can't use it for cryptography.

Clearly, this is an area where it is better to lean on the experts. Devising a cryptographic hashing function is a difficult process involving mathematics far beyond the abilities of the average sofware engineer. I remind you again,
**always use crypto code that has been built and vetted by security experts.**

## Which algorithm should I use?

If in doubt, go for the SHA-2 family of hashing algorithms. SHA-2 comes in six different variants, but it mostly boils down to how long you would like the hash to be. SHA-2 256 produces a 256-bit digest, while SHA-2 512 produces – you guessed it – a 512-bit digest. There are also 224 and 384 bit variants. SHA-2 has the advantage of having been around for a while (first published in 2001), so there is wide support for them, and that age implies a certain level of battle-hardening.

SHA-3 is the up-and-coming algorithm which aims to address some of the weaknesses of SHA-2. It was published in 2015, so it's not as widely supported. However, SHA-3 is a [NIST standard](https://csrc.nist.gov/Projects/Hash-Functions), so its security characteristics are well understood and should be safe for production use.

You should avoid using MD5 and SHA-1 for cryptographic uses. These algorithms have well-known vulnerabilities that disqualify them for use in secure scenarios.