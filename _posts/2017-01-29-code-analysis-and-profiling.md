---
layout: post
title: Code Analysis and Profiling
categories:
- blog
---

Many a time we have to deal with the situation of having to analyse code written by someone else in order to debug errors or improve efficency via methods such as parallelizing or identifying bottlenecks in the code.It is imperitive that we fully understand the code flow, analyse the portions of the code that are most time consuming and work on them.
There are various ways in which we can approch this problem.One is via static code analysis via call graphs and another is Profiling tools like Valgrind/Callgrind and GProf.In this post we will be discussing some of these tools along with some use cases(blastx) and examples.

---

We will first be exploring GProf which is a Dynamic Code Analysis tool.For blastx I had to change couple of plags in their Makefile because they were stripping the executable which results in loss of the symbol table unpon creation of executable.A gmon.out file is generated which upon using the command gprof "executable name" gmon.out > analysis.txt, the following output was produced.

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

Now we will be trying to use a Dynamic code analysis tool known ad DOxygen, it is also used to generate documentation for large projects.Doxygen provided us with data about parameters being passed ot functions and tracing the call graph.
This gives us a clear idea about the purpose and functionality of ShiftWin1.

![Call graph](/assets/shiftwin.png)

