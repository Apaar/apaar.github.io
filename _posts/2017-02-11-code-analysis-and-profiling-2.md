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

The Script had to take care of some basic feature - 

1. Had the script file chop the DNA into segments
2. Run each part on Individual cores simultaneously 
3. This Process contains additional overheads like 
	- Splitting It perfectly with overlaps to ensure regularity is maintained while searching each file
	- Some redundant operations are being performed, but negligible for larger input
	- Tailoring back the output files together to ensure output is readable and consistent with the expected output
4. After processing, a script stitches all the output files together
5. But there is a E-value which is proportional to the size of dna inputted which needs to be modified
6. The modified E -Value is then used to filter out some more of the output hits
7. The remaining hits are tailored back to give effective output


---

Here are the results gained upon running the scripts - 

![Benchmarks](/assets/benchmarks.png)



