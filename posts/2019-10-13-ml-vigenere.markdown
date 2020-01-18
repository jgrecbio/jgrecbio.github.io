---
title: Cracking the Vigenere Cipher with Machine Learning
---

# The Vigenere Cipher

Back in the old days, caesar cipher and other mono-subsitution ciphers
were good enough to keep anyone willing to snoop on someone's else secrets.
But arabic cryptanalysis were able to crack those by
calling linguistic statistics to the rescue. By taking
a look at the monosubsitution cipher, one can easily see
that they keep the relative proportion of each encoded letter.
It gives an edge to cryptanalysts and for century they had
the upper hand in the cat and mouse game cryptagraphers
and cryptanalysts like to play. Crytpographers tried
to make the game harder by using several technics.
Such as not just encoding letters but also creating
symbols for phonems, like ff or ion. One popular technic
was to encode a letter with several unique numbers.
The letter 'e' could then be encoded either by 12, 43,
12, 44. The number of symbol's synonims would depend on the
the relative frequency of each letter
in the language of interest. The idea is to blur the
relative letter frequency so that cryptanalysts cannot rely
on these statistics anymore. And it worked to a point. But, if
this method obfuscate one-letter statistics, it does not hide
2-letter or even 3 and 4-letter statistics. So these ciphers are
still suffering to more of the same kind of weaknesses
than the monosubstitution ciphers. So it called for a
revolution in the cryptography world.

The revolution was made during the renaissance in Italy.
One the most famous architect of the time, and its
followers designed the first poly-substitution cipher.
The idea is as simple as it is effective. Instead of
using one alphabet to encrypt letters, why not using
several at the same time, and alternating between them
respecting a certain order (the key). As usual, this
cipher was named after someone else: Mr Vigenere, a
french cryptanalyst who popularized the idea in
one of his book. He instead invented his own vigenere's
cipher variant, one much harder to crack: the autokey cipher.

To describe the vigenere cipher quickly:
* first, the two correspondants need to share the private key,
usually a word or a sentence stripped from its whitespace
* then for each letter of the key k, they will use a
shifted version of the normal alphabet starting by k
* then they will encode each letter of the plaintext
by cycling through the caesar ciphers created by the key

The beauty of this cipher is that the letter e can be encrypted
differently n times, n being the number of unique
letters in the key. Depending on what the key-encryption state
is, the encryption is different. So it blurs both 1-letter
but also 2, 3 and 4 letter statistics.

# History and Desription of its Cryptanalysis

For a long time, 3-4 centuries, the vigenere cipher was
deemed uncrackable.  A lot of cryptographer deemed it
absolutely safe and that it would never be broken.
And as you can expect, it has been broken, by two cryptographers,
independantly: Babbage and F. Both of them notice that
this cipher does not obfuscate completly the relationship
between plaintext letters. Indeed, if the word "the" is
encrypted trg under the key word "ale", it will always be
encrypted the same way when the encryption state is ale.
But one very important question to crack it was to discover
the length of the key.

Based on that fact, a cryptanalyst trying to find the key
length of the vigenere cipher could count the mean distance
between repeated 3-letters "word". The key length must then be
close of to a common factor for these average distances. But 
the process of screening the potential key length is rather
tedius and defining is not easy.

Kasiski, a german cryptanalyst devised a statistics to help
for this process, the index of coincidence. Its concept is
rather simple. If the key length is x, then every letters distant 
by a factor of x starting from a postion y must have a
distribution nearly identical to every letters with the same
distance but starting at y + 1.

The index of coincidence is the probability taht 2 randomly
chosen letters in a string are identical. In a caesar shifted
cipher, this index is close to equal to the index of the plaintext.
Indeed, if the shift is of 3, the probability of randomly chosing
e twice in a plaintext should be about the same of chosing
the letter h twice.

To get the exact formula of the index of coincidence, there is
$\binom{n}{2}$ ways of selecting to letters in a text of length n.
There are $\binom{F_i}{2}$ ways of selecting the letter i, $F_i$
being the count of the ieth letter.
$$\begin{aligned}
	IndCo(message) &= \sum\frac{F_i \choose 2}{n \choose 2} \\
	& \frac{F_i(F_i - 1)}{n(n - 1)}
\end{aligned}$$

In the englsih language the index coincidence is about 0.0685.
So a strategy to find the length, is the compute this index
for each subset of the letter distant by a common factor of
the length candidate. If it is close to 0.0685, then the length
tested has a good chance to be the length of the key. But Again,
this process is rather tedius and hard to automate by hand.

Once the key length has been found, it is rather easy to
try different shifts for each subset of the letters to find
the key.

# Logistic Regression to the Rescue

But instead of writing your own rules, why not take a large
corpus of text, encrypt each individual message under a 
random key, and then compute the potential index coincidence
for key length between 2 and 50 and use some machine learning,
like logistic regression, to detect the key length.
And again, once the key length has been detected, one could the
the distribution of each subset as input to detect with logistic
regression each of the ceasar individual shift.
