---
layout: post
title: Code Analysis and Profiling - 2
categories:
- blog
---

In this post we will be discussing a very interesting solution to improving the efficiency of the blastx program discussed in the previous post.BLAST for Basic Local Alignment Search Tool is an algorithm for comparing primary biological sequence information, such as the amino-acid sequences of proteins or the nucleotides of DNA sequences. A BLAST search enables a researcher to compare a query sequence with a library or database of sequences, and identify library sequences that resemble the query sequence above a certain threshold.The algorithm's implementation distributed by NCBI is what we are working on.One feature that was interestingly lacking was the fact that it did not support multiple cores and spawned a very limited number of threads.

---

The input to the program is a protien database which consists of a long string of ACTG characters and alignment takes place against a smaller protien or amino acid.Since it is simple alignment the larger input can be split up into multiple smaller dna sequences and searched on.Each of these input files can be run on a separate processor of a multicore machine improving time efficency.We will be writing a simple shell script to take the split up input file and spawn a process for each core and combine the results in the end.

---



