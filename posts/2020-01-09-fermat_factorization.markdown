---
title: The Fermat's Factorization attack
---

The fermat's factorization method
=================================

As we've seen in the last post, the selection of the public exponent
does have a security impact. But many more parameters of the RSA
cryptosystem holds a big impact on the security. Actually the selection
of the prime number $p$ and $q$, essential to compute the private key
and the trap door, is also a delicate process if done wrong opens to
some severe security flaws.

Indeed if $p$ and $q$ are roughly equal to one another, then there exist
an efficient algorithm to factorize $n$ in polynomial time. This
factorization method is called after its inventor: Fermat. This
factorization algorithm relies on simple math: any number $c\times d$
can be written as:

$$\begin{aligned}
        &(\frac{c + d}{2})^2 - (\frac{c - d}{2})^2 \\
        = &\frac{c^2 + 2cd + 4}{4} - \frac{c^2 - 2cd + 4}{4} \\
        = & cd
\end{aligned}$$

Here is the pseudo-code for the fermat's factorization:
```
FermatFactor(N): // N should be odd
	a ←  ceiling(sqrt(N))
	b2 ← a*a - N
	repeat until b2 is a square:
		a ← a + 1
		b2 ← a*a - N
	return a - sqrt(b2) // or a + sqrt(b2)
```

This algorithm has complexity of $O(|p - q|/4n^{1/2})$. If
$|p - q| \leq n^{1/4}$, then the factorization of n become quite
trivial.

Which is rather simple to write in haskell through recursion:

```haskell
fermatFactorization :: Integer -> Integer -> (Integer, Integer, Integer)
fermatFactorization a n = let b2 = a * a - n
                           in if isSquare b2
                                 then (a - integerSquareRoot n,  -- count the number of iterations
                                       a - integerSquareRoot b2, -- p
                                       a + integerSquareRoot b2) -- q, p < q
                                 else fermatFactorization (a + 1) n
```

With this code let just test for $n=316033277426326097045474758505704980910037958719395560565571239100878192955228495343184968305477308460190076404967552110644822298179716669689426595435572597197633507818204621591917460417859294285475630901332588545477552125047019022149746524843545923758425353103063134585375275638257720039414711534847429265419$.
For this very long composite number, my function based on the fermat
factorization is able to find p and q in 697852 iterations.\
$p = 17777324810733646969488445787976391269105128850805128551409042425916175469483806303918279424710789334026260880628723893508382860291986009694703181381742497$
$q = 17777324810733646969488445787976391269105128850805128551409042425916175469168770593916088768472336728042727873643069063316671869732507795155086000807594027$.
So this simple program is able to find the factors of $n$ in a matter of milliseconds.

Relevance of the attack
=======================

But when $p$ and $q$ are very large number, like in the case of 1024bits
long RSA keys, $|p - q$ is very small is an event very unlikely to happen.
So, if that attack is mathematically plausible, is it relevant in real-life ?
But in real life, cryptanalysts rely more on cryptgraphic errors of usage
or implementation than of fatal design flaws of the used cryptographic schemes.
For example, it is in fact much easier to distribute a RSA faulty app to the public,
making its user believe that their communications are private, and
then use the knowledge of your under-the-sleeve primes to snoop on the
communications. Another example, would a cryptographer apprentice who would
decide to select $p$ and $q$ with the following method.

```
generate_p_q(key_size):
    repeat until p.is_prime():
        p <- random(2^(key_size), 2^(key_size + 1))
    repeat until q.is_prime():
        q <- p + 2
    return (p, q)
```

This implementation will invevitably creates primes with very small differences,
which will be subject to femat's factorization method. It is another example
of the principle that you should not roll your own cryptography primitives.

Conclusion
==========

In the previous example the number of iterations to find $p$ and $q$
was rather low, but even though the differences between the two
primes is low, the amount of computations required with this
naive algorithm may be very important.
For example for $n = 2462649746477364143454082050368305440553491900304388646893610847386194301369924385009730987303651345060031438478297733694036327257723431468649220444397635127530301992505638291521092898714917678389314956983918603221732358628680256253537449204312287724750669208856634711056863315465220853759428826555838536733$,
it requires 26365624027328 iterations, so my little haskell programm would probably takes months
to solve it. I could try to distribute the computation over many processors, but it
would still take a lot of time and cost a few thousand bucks.
There are some tricks to speed up the fermat's factorization,
which will be the subject of a later post.
