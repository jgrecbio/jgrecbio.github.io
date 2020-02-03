---
title: The Wiener Attack
---

*This post assume that the reader is familiar with
the RSA cryptosystem.*

As we gleaned about on some of the previous posts,
RSA can be used to encrypt stuff, but also to sign
documents. In that case, it can be interesting to
optimize the computations to limit the cost of 
modular exponentiation of the private exponent $d$.
This need can be extremely important if the hardware
in charge of the signature is not very powerful, as
it is often the case. As we have seen in the
[previous post](./2020-01-18-rsa-crt.html),
computations can be speed up with the CRT algorithm
but this approach is not without some
associated risks. So, imagine an apprentice cryptographer
Bob, who just noticed that in order  to reduce the number ofcomputations on the
signing side, it is possible to choose a small value for $d$.
And voila! But this actually makes the RSA signing scheme
very insecure, as it opens to an attack with the help of continued
fractions. This attack is coined as the Wiener's attack
and were published some 30 years ago.

So I will start here to describe what are continued
fractions, their link to the famous Euclid's algorithm
to compute the GCD and how to attack RSA with that
knowledge.

Continued Fractions and the Euclid algorithm
============================================

Let $a/b$ be a rational. Let $r_0 = a$ and $r_1 = b$ and start
the euclidean division:

$$\begin{aligned}
		&r_0 = q_0r_1 + r_2 \\
		\implies &\frac{r_0}{r_1} = q_0 + \frac{r_2}{r_1} \\
		\implies &\frac{a}{b} = r_1 + \frac{1}{\frac{r_1}{r_2}}
\end{aligned}$$

And we can also perform the euclidean division on $r_1$ by $r_2$.
Which will develop this continued fraction further.
$$\begin{aligned}
		\implies &\frac{a}{b} = q_0 + \cfrac{1}{q_1 + \cfrac{r_3}{r_2}}
\end{aligned}$$
It is possible to continue this process until one of the $r_i$'s
equal zero. An attentive reader probably recognized the
algorithm of Euclid for finding the GCD of $a$ and $b$. As
the $r_i$'s are strictly decreasing, this proces is garanted
to terminate in a finite number of steps, moreover it can be
proven that for any integer $a$, $b$, this algorithm runs
in at most $O(2 \times log_2(v + 1)$, which is in polynomial
time.

The sequence of quotients in continued fraction is denoted
$[a_0, a_1, \ldots, ...]$ as shown below:
$$
\begin{aligned}
	\frac{a}{b}=a_0+\cfrac{1}{a_1+\cfrac{1}{a_2+\cfrac{1}{a_3+\cfrac{1}{a_4+\cfrac{1}{a_5+\cfrac{1}{a_6+\cfrac{1}{a_7+\cfrac{1}{a_8+\cdots}}}}}}}} = [a_0, \cdots]
\end{aligned}
$$

Computing the continued fraction from the division of
two integers is then fairly simple:
```haskell
contFrac :: (Integer, Integer) -> [Integer] -> [Integer]
contFrac (a, b) as  = let (q, r) = a `divMod` b
                       in if r == 0
                             then as ++ [q]
                             else contFrac (b, r) (as ++ [q])
```

To close this section, let define what is a convergent.
Let $[a_0, \cdots, a_n]$ be a continued fractions equal
to $a/b$. Let $p_m/q_m$ be a convergent of $a/b$ if
$p_m/q_m$ is equal to $[a_0, \cdots, a_m]$ for $m \leq n$.

It can be proven relatively easily through recursion that
for $m \leq n$ the convergents $p_m/q_n$ of $a/b$ is:
$$\begin{aligned}
p_m &= a_m * p_{m - 1} + p_{m _ 2} \\
q_m &= a_m * q_{m - 1} + q_{m - 2}
\end{aligned}$$
With $p_0 = a_0$, $q_0 = 1$, $p_1 = a_0a_1 + 1$ and $q_1 =a_1$.


To compute the convergents is not difficult either:
```haskell
convergents :: [Integer]
               -> [(Integer, Integer)]
               -> [(Integer, Integer)]
convergents [] cs = cs
convergents (an:as) cs = let (pn1, qn1) = head cs
                             (pn2, qn2) = head . tail $ cs
                             pn = an * pn1 + pn2
                             qn = an * qn1 + qn2
                          in convergents as ((pn, qn):cs)
```


Continued Fractions and Number Approximation
============================================

But that may be beautiful, but why all the fuss about
continued fractions in the context of RSA (beside the
RSA context, infinite continued fractions can be used
to represent irrational numbers such as $\pi$, the 
euler constant $e$, which also yields a way to compute
their approximation, this is the way Archidemes computed
$\pi$ as $22/7$) ? It all comes from the Legendre theorem,
which states that if:
$$\begin{aligned}
	|x - \frac{a}{b}| \leq \frac{1}{2b^2}
\end{aligned}$$
Then $a/b$ must be equal to a convergent of $x$. So imagine
that Marvin --- an attacker --- is able to compute a rational approximate $a/b$
of $d$ from the public key $(e, n)$. And that $|d - a/b| \leq 1/2b^2$.
Then it provides us a way to compute $d$, by testing the
convergents of$a/b$ as a candidate for $d$, and then decrypt
any message encrypted with such a weak key. Indeed the Euclid
algorithm applied together with continued fractions, would
offer us a polynomial method to find $d$. But it is not
as simple as it is not as easy to find an estimate of the
private exponent $d$.

The Wiener's Attack
===================

This is the idea behind the Wiener attack but it does not
work by finding an estimate of $d$ but instead something close
enough. 

We know that:
$$\begin{aligned}
	&ed \equiv 1 \mod{n} \\
\iff &ed - 1 = k\phi(n), k \in \mathbb{Z} \\
\iff &ed - k\phi(n) = 1 \\
\iff &\frac{ed}{\phi(n)} - k = \frac{1}{\phi(n)} \\
\iff &\frac{e}{\phi(n)} - \frac{k}{d} = \frac{1}{d\phi(n)}
\end{aligned}$$

Now by convention assume, that $q$ is the smaller factor
of the modulus $n$. Therefore
$q < p \implies q^2 < p \times q < n \implies q < n^0.5$.
$$\begin{aligned}
	&\phi(n) = (p - 1) (q - 1) = n - (p + q) + 1 \\
\iff & n - \phi(n) = p + q - 1 < 2q + q - 1 < 3q < 3\sqrt{n} \\\end{aligned}$$

Armed with that:
$$\begin{aligned}
|\frac{e}{n} - \frac{k}{d}| &= |\frac{ed - kn}{nd}| \\
							&= \frac{1 + k\phi(n) - k(\phi(n) + p + q - 1}{nd} \\
							&= \frac{1 - k(p + q - 1)}{nd} \\
							&= \frac{1 + k(\phi(n) - n)}{nd} \\
						    &< \frac{3k\sqrt{n}}{dn} \\
							&= \frac{3k}{d\sqrt{n}}
\end{aligned}$$

Or we know that $k < d$, so $3k < 3d$. Now let assume that:
$$\begin{aligned}
d < \frac{n^0.25}{3} 
\implies 3d^2 < dn^025 
&\implies \frac{1}{3d^2} > \frac{1}{n^0.25} > |\frac{e}{n} - \frac{k}{d}| \\
&\implies |\frac{e}{n} - \frac{k}{d}| < \frac{1}{d^2}
\end{aligned}$$

Which by Legendre theorem proves that $k/d$ is a convergent of $e/n$.
So once $k/d$ has was retrieved, what to do?

Now we have a rational $a/b$ equal to $k/d$ so:
$$\begin{aligned}
	\frac{b}{a} &= \frac{d}{k} \\
	\frac{be}{a} &= \frac{ed}{k} \\
				&= \frac{k\phi(n) + 1}{k} \\
				&= \phi(n) + \frac{1}{k}
\end{aligned}$$
Or $1/k$ is inferior to 1, so by rounding, we can get $\phi(n)$
which is course sufficient to find $d$ from the public exponent $e$.

Conclusion
==========

This attack really amazed me, as I would not have imagined that
one could foment one from continued fractions (which
are about rational and irrational numbers) to find a weakness
in the RSA cryptosystem. To me the link between the two has to
be about the Euclid's algorithm rooted in the continued
fraction concept. Nonetheless this attack warns us not to use small
value of $d$ to speed up the decryption process of RSA, even
though we may have incentives to do so.

To conclude, there are improvements of this attack which may
be the subject of following posts.
