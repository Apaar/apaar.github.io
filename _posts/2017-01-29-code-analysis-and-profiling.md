---
layout: post
title: Code Analysis and Profiling
categories:
- blog
---

Many a time we have to deal with the situation of having to analyse code written by someone else in order to debug errors or improve efficency via methods such as parallelizing or identifying bottlenecks in the code.It is imperitive that we fully understand the code flow, analyse the portions of the code that are most time consuming and work on them.
There are various ways in which we can approch this problem.One is via static code analysis via call graphs and another is Profiling tools like Valgrind/Callgrind and GProf.In this post we will be discussing some of these tools along with some use cases(blastx) and examples.

---

**NCBI-BLAST**

To demostrate these tools we will be working with the NCBI(National Center for Biotechnology Information) implemenation of the BLAST(Basic Local Alignment Search Tool) algorithm.We will specifically be wokring with the blastx variant of this algorithm.It is an algorithm for comparing primary biological sequence information, such as the amino-acid sequences of proteins or the nucleotides of DNA sequences. A BLAST search enables a researcher to compare a query sequence with a library or database of sequences, and identify library sequences that resemble the query sequence above a certain threshold.

---

**GProf**

We will first be exploring GProf which is a Dynamic Code Analysis tool.Gprof calculates the amount of time spent in each routine. Next, these times are propagated along the edges of the call graph. Cycles are discovered, and calls into a cycle are made to share the time of the cycle.

Several forms of output are available from the analysis -

	1) The flat profile shows how much time your program spent in each function, and how many times that function was called. If you simply want to know which functions burn most of the cycles, it is stated concisely here.

	2) The call graph shows, for each function, which functions called it, which other functions it called, and how many times. There is also an estimate of how much time was spent in the subroutines of each function. This can suggest places where you might try to eliminate function calls that use a lot of time.

	3) The annotated source listing is a copy of the program’s source code, labeled with the number of times each line of the program was executed.

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

Now we will be trying to use a Dynamic code analysis tool known ad Doxygen, it is also used to generate documentation for large projects.Doxygen provided us with data about parameters being passed ot functions and tracing the call graph.
This gives us a clear idea about the purpose and functionality of ShiftWin1.

![Call graph](/assets/shiftwin.png)

