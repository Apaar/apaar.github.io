---
layout: post
title: Parallelizing Grep
categories:
- blog
---

Grep is a frequently used command line tool built into most distuibutions of linux, it allows us to easily search for string, regex , files and data within a directory structure containing many files or just a single file.It comes with a very wide range of options allowing for everything from displaying counts to finding inverted matches.It uses the Boyer–Moore string search algorithm to recursively search through directories and the files containted within them.Boyer-Moore is highly efficient as it matches the first letter of the string and depending on wether it is a match or not uses a symbol table to compute the jump to be made.

From what I have read grep further optimises this by - 

1. GNU grep also unrolls the inner loop of Boyer-Moore, and sets up the Boyer-Moore delta table entries in such a way that it doesn't need to do the loop exit test at every unrolled step. The result of this is that, in the limit, GNU grep averages fewer than 3 x86 instructions executed for each input byte it actually looks at.
2. GNU grep uses raw Unix input system calls and avoids copying data after reading it. Moreover, GNU grep avoids breaking the input into lines. Looking for newlines would slow grep down by a factor of several times, because to find the newlines it would have to look at every byte.


### Initial Exploration

One interested fact to be noted is that GNU Grep is not parallelized and does not use any form of threading.It works in serial, this could be attributed to the fact that it was first written in the 80's and to maintain compatibility with the vast number of devices that use it ,threding was never introduced.If you look online one very common solution is to use GNU Parallel.This is the description of how it works in some of the documentation of the design choices behind GNU Parallel

1. The easiest way to explain what GNU parallel does is to assume that there are a number of job slots, and when a slot becomes available a job from the queue will be run in that slot. But originally GNU parallel did not model job slots in the code.

2. While the job sequence number can be computed in advance, the job slot can only be computed the moment a slot becomes available. So it has been implemented as a stack with lazy evaluation: Draw one from an empty stack and the stack is extended by one. When a job is done, push the available job slot back on the stack.

3. This implementation also means that if you re-run the same jobs, you cannot assume jobs will get the same slots. And if you use remote executions, you cannot assume that a given job slot will remain on the same remote server. This goes double since number of job slots can be adjusted on the fly

More Details can be read [here](https://www.gnu.org/software/parallel/parallel_design.html#Design-of-GNU-Parallel)

Thus, we can run grep with GNU Parallel with the command - 

{% highlight c %}
find . -type f | parallel -j+1 grep mypattern
{% endhighlight %}

Here the -j+1 argument instructs the shell to create on job more than the cores on the computer.This helps with I/O bound processes like grep.

Upon running the parallel version against the normal GNU grep i achived around 4-5 times speed up on a 2GB file containing Housefly DNA

### Exploring the GNU Parallel Codebase

Enthusiastic about threading the algorithm of GNU parallel(i had read ina  couple of places that uyer-moore was not too hard to parallelize) , I downloaded the grep-3.0's source code from gnu.org
###

