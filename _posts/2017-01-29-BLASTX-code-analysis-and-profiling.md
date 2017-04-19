---
layout: post
title: BLASTX - Code Analysis and Profiling
categories:
- blog
---

Many a time we have to deal with the situation of having to analyse code written by someone else in order to debug errors or improve efficency via methods such as parallelizing or identifying bottlenecks in the code.It is imperitive that we fully understand the code flow, analyse the portions of the code that are most time consuming and work on them.
There are various ways in which we can approch this problem.One is via static code analysis via call graphs and another is Profiling tools like Valgrind/Callgrind and GProf.In this post we will be discussing some of these tools along with some use cases(blastx) and examples.

---

**NCBI-BLAST**

To demostrate these tools we will be working with the NCBI(National Center for Biotechnology Information) implemenation of the BLAST(Basic Local Alignment Search Tool) algorithm.We will specifically be dealing with the blastx variant of this algorithm.It is an algorithm for comparing primary biological sequence information, such as the amino-acid sequences of proteins or the nucleotides of DNA sequences. A BLAST search enables a researcher to compare a query sequence with a library or database of sequences, and identify library sequences that resemble the query sequence above a certain threshold.

---

**GProf**

We will first be exploring GProf which is a Dynamic Code Analysis tool.Gprof is a tool that computes the time spent by each function/routine in a program. It also follows the calls along the edges of a call graph and if there are any cycles, the calls into the cycle share time taken by the cycle.

Several forms of output are available from the analysis -

	1) The flat profile tells us how much time was spend in a single routine cumulatively and also gives us the number of times it was called. It is handy to identify which function is burning the most time.

	2) The call graph is also displayed along with the number of times calls were made in the graph. It can be used to identify and reduce the number of calls to expensive functions/sub-routines.

	3) The annotated source listing is a modified cpy of the code where each line of code also has the number of times it was executed.

For blastx I had to change couple of flags in their Makefile because they were stripping the executable which results in loss of the symbol table unpon creation of executable.A gmon.out file is generated which upon using the command gprof "executable name" gmon.out > analysis.txt, the following output was produced.

{% highlight c %}

Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  Ts/call  Ts/call  name    
 72.97     44.69    44.69                             s_ShiftWin1
 14.48     53.56     8.87                             BlastAaWordFinder
  3.80     55.89     2.33                             s_BlastSmallAaScanSubject
  2.68     57.53     1.64                             Blast_SemiGappedAlign
  1.19     58.26     0.73                             s_AddWordHitsCore
  0.82     58.76     0.50                             s_SequenceGetRange
  0.77     59.23     0.47                             Blast_ReadAaComposition
  0.73     59.68     0.45                             BSearchContextInfo

{% endhighlight %}

The output clearly shows the ShiftWin_1 consumes majority of the time for the current input case.Upon further analysis/reading, Shiftwin happens to be the function that shifts the window accross which string comparisions are taking place.

---

**Doxygen**

Now we will be trying to use a Dynamic code analysis tool known as Doxygen, it is normally used to generate documentation for large projects.We can also use it to automatically generate and analyse the code structure of larger undocumented codes. It can also visulize the relations between the sub-routines in code using dependecy graphs, inheritance diagrams, and collaberation diagrams.

Running doxygen on the blastx codebase which spans a few million lines of code will give us a clear idea about the purpose and functionality of ShiftWin1.

![Call graph](/assets/shiftwin.png)



From the above graph we can clearly see the functions which call ShiftWin1 and who it calls.ShiftWin1 is a recursive fuction which is used to shift the comparision frame by one every time a possible match is found.

---

**Valgrind**

Valgrind is a suite of tools for debugging and profiling programs. There are three tools: a memory error detector, a time profiler, and a space profiler.For profiling purposes, the time profilier along with the space profilier are quite usefull.Valgrind is a form of dynamic profilier .ie it runs and profilies while the executable is run.It increases the time of execution quite a bit.

---

**ShiftWin_1**

Through this analysis we identified the function ShiftWin_1 to be consuming majority of the time in the program.Unfortunately on further reading and research i realised modifying it to produce a speed gain may be beyond me in a timely manner.Thus, I focused my efforts on another interesting solution that has been discussed in the next post :)
An interesting point to note was that when I found CUDA code for the BLAST shifwin_1 was the function that has been converted and used to achieve a almost 30X speedup on the existing implementation.



