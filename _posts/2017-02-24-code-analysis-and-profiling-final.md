---
layout: post
title: Code Analysis and Profiling - Final
categories:
- blog
---

As discussed in the previous post we achieved an almost 7 times speed up on the baseline NCBI Blast algorithm by simply applying the concept of SIMD or Single Instruction Multiple Data to this algorithm.

Here's what Wikipedia has to say about SIMD :

>Single instruction, multiple data (SIMD), is a class of parallel computers in Flynn's taxonomy. It describes computers with multiple processing elements that perform the same operation on multiple data points simultaneously. Thus, such machines exploit data level parallelism, but not concurrency: there are simultaneous (parallel) computations, but only a single process (instruction) at a given moment. 

Since it is a string match/search algorithm we could simply split up the input file and run it through the program multiple times to get the same output.

But there would be two major issues - 

- The may be matches across splits ie. half the string is in split one and the other half in split two.
- BLAST is a heuristic algorithm and uses a "E" Value to do the filtering.This value is directly proportional to the size of input.


To overcome these issues we first make sure the splits of the input file overlap by the size of the protein sequence being searched for,this will result in 2 duplicate matches but those are easy to filter out.
The E-Value,since it is directly proportional to the size of input we just multiple it by the number of splits and do another round of filtering via the script itself.
