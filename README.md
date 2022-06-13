# summer
A line-order-independent checksum generator


# Intention

Order-independent hashing (line by line) of a file to determine if there are any lines missing.


# Use-case

You have two files, and you want to make sure they have the same content, but
the order of the lines in the file don't matter for your purposes, and they are
big enough that you don't want to sort either (let alone both) of them.

More specifically, say you have a process that takes the same input data, and
produces the same data on the output, but where the order of the output doesn't
matter, and isn't guaranteed by the process.  For example, a program that generates
a 'dot' file [https://en.wikipedia.org/wiki/DOT_%28graph_description_language%29] 
might produce lines in different orders, but still describe the same graph.

For context, the files being big enough you don't want to sort implies that the 
files are significantly larger than RAM, as unix 'sort' is wicked fast, and faster 
than this program for large files, hands down.  The reason is that our hash 
algorithm has significant setup and output costs, and these overwhelm the memory 
benefits by multiples.  Put another way: probably only look to this tool for files 
100s of gigabytes or larger, where our O(n) algorithm can start to compete with 
sort's O(n log n) algorithm.

The cryptographic hash function that performs best (so far) is sha1.  Security is discussed
in another section below.


# Security

The checksum output by this program should NOT be used in a security-sensitive
context.  It is vulnerable to collision (see: the use of SHA1) in the base case, 
and the combination of line checksums is done with two different algorithms that
each have a relatively trivial way to cancel out the effect of a single line
(either a duplicate line in the case of xor, or an additive inverse for the
addition operation).

That said, is it obviously bad?  ... No, ish?  The attack is to generate two
lines whose SHA1 hashes are additive inverses of each other.  You can then
include 2N of each line and have no impact on the output checksum.  Probably
hard, but don't bet against the NSA.

The theory of using both xor and addition is that the trivial attack offered
by the xor algorithm (sending the same line 2N times) is obviated by the other
addition algorithm, and likewise for the additive inverse attack (a bit more
challenging to pull off for a cryptographic hash).

The security tradeoffs
are probably overwhelmed by the intended function of this program (non-
security contexts, probably?)


# Multiple processors

There is a version that supports multiple processes to permit better usage of
available CPU to improve runtime.

It is probably still possible to improve the overflow add function for speed.


# Testing  

Take a multi-column file, run the algorithm on it, sort it on each 
column into differen files, run the algorithm on those outputs. should all 
match.


# Interesting quirks

 - If we are going to check for line-unique-ness, how to do so? (fights against
   someone using duplicate lines to defeat the xor operation) Options:
   * Hash table of line-hashes. (possibly shorter than entire file?, puts us
     the same category as 'sort' for memory usage)
   * Bloom filter of line-hashes (but what to do when you get a hit?)
     + Keep N bloom filters of line hashes, that let you divide the file into
       2^(N-1) segments for a subsequent search for a duplicate of a found
       collision?
     + Combine this with recording the location of the collision, and it is
       possible to significantly reduce the amount of data that must be re-
       read to determine if there is a duplicate line, or just a hash
       collision.


