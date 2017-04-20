---
layout: post
title: Parallelizing Grep
categories:
- blog
---

Grep, the pattern match tool found on most linux distros, allows us to search for strings, and also regex on files and directories, offering numerous options along the way. It makes use of the Boyer-Moore algorithm to search through these files, and also searches through directories for files recursively. Boyer-Moore makes use of a symbol table to compute the shifts to be made.

From what I have read grep further optimises this by(copy pasted relevent portions from [here](https://lists.freebsd.org/pipermail/freebsd-current/2010-August/019310.html)) - 

1. GNU grep also unrolls the inner loop of Boyer-Moore, and sets up the Boyer-Moore delta table entries in such a way that it doesn't need to do the loop exit test at every unrolled step. The result of this is that, in the limit, GNU grep averages fewer than 3 x86 instructions executed for each input byte it actually looks at.
2. GNU grep uses raw Unix input system calls and avoids copying data after reading it. Moreover, GNU grep avoids breaking the input into lines. Looking for newlines would slow grep down by a factor of several times, because to find the newlines it would have to look at every byte.


### Initial Exploration/GNU Parallel

One interested fact to be noted is that GNU Grep is not parallelized and does not use any form of threading.It works in serial, this could be attributed to the fact that it was first written in the 80's and to maintain compatibility with the vast number of devices that use it ,threading was never introduced.If you look online one very common solution is to use GNU Parallel.
This is the description of how it works in some of the documentation of the design choices behind GNU Parallel(copy pasted the relevant parts) -

1. The easiest way to explain what GNU parallel does is to assume that there are a number of job slots, and when a slot becomes available a job from the queue will be run in that slot. But originally GNU parallel did not model job slots in the code.

2. While the job sequence number can be computed in advance, the job slot can only be computed the moment a slot becomes available. So it has been implemented as a stack with lazy evaluation: Draw one from an empty stack and the stack is extended by one. When a job is done, push the available job slot back on the stack.

3. This implementation also means that if you re-run the same jobs, you cannot assume jobs will get the same slots. And if you use remote executions, you cannot assume that a given job slot will remain on the same remote server. This goes double since number of job slots can be adjusted on the fly

More Details can be read [here](https://www.gnu.org/software/parallel/parallel_design.html#Design-of-GNU-Parallel)

Thus, we can run grep with GNU Parallel with the command - 

{% highlight c %}
time find /home/apaar/Parallel-Computing type/test1  | parallel -k -j150% -n 1000 -m grep -h "hi" {} | cat > op
{% endhighlight %}

Here the -j150% argument instructs the shell to create 1.5 job per cores on the computer.This helps with I/O bound processes like grep.find is used to feed grep the file names and time was used only for benchmarking(results in next post).

### Exploring the GNU Grep Codebase(and a minor mess up...)

Enthusiastic about threading the algorithm of GNU Grep(i had read in a  couple of places that boyer-moore was not too hard to parallelize) , I downloaded the grep-3.0's source code from [here](https://ftp.gnu.org/gnu/grep/).Upon unpacking the source i was met by a very large amount of code ,this is mainly to make sure that grep can be compiled and cofigured for a very large number of distributions of linux and also to support a wide array of options. 

Upon looking at the /src folder ,the main was located in a file called grep.c, enthusiastic to make some small changes i added a simple "hello" in the start of to trace the flow of the program.To make the source we had to generate a auto-config and hit make.Unfortunately I hit make make install too! This resulted in the linux I was working on having it's native grep being overwritten.This resulted in a lot of system utilites like the package manager failing.I immediately tried to revert the changes and rebuild the source, but it looks like the program used to generate the auto config itself also used grep.The presense of "hello" in the output was causing it all to fail :(

Ultimately I had to go into the /bin and /sbin folders and delete the binaries(where it had been installed) and manually delete the binaries and rebuild ,the autoconfig used fallbacks which could be used in case of the system not having grep and successfully built.

Upon further analysis of the core grep code and some messing around, I felt that it would be too hard to parallelize it due to the occurance of a very large number of compatiblity options and code.In the next part we will discuss my attempts at creating a threaded barebones version on grep.

This was the gprof flat profile of grep.memchr_kwset is the function that does the comparisions and bmexec is the driver function for the whole boyer-moore search and calls memchr_kwset a very large number of times.The missing bit of data was because one of the c files was being compiled with compiler optimisation and gprof was not able to glean information cos of that. 

{% highlight c %}

Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 56.51      5.21     5.21    68403     0.08     0.09  bmexec
 32.97      8.25     3.04                             grepdesc
  7.54      8.95     0.70 52749779     0.00     0.00  memchr_kwset
  2.28      9.16     0.21                             memchr2
  0.33      9.19     0.03                             safe_read
  0.22      9.21     0.02                             mbscasecmp
  0.11      9.22     0.01    91706     0.00     0.00  fillbuf
  0.05      9.22     0.01        1     5.00     5.00  treenext
  0.00      9.22     0.00    68403     0.00     0.00  kwsexec
  
{% endhighlight c %}

