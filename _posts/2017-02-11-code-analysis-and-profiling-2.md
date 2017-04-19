---
layout: post
title: BLASTX - Parallelization
categories:
- blog
---

In this post we will be discussing a possible solution to improving the efficiency of the blastx program discussed in the previous post.

### Algorithm

Here is a relevant brief of the algorithm from wikipedia - 

>BLAST for Basic Local Alignment Search Tool is an algorithm for comparing primary biological sequence information, such as the amino-acid sequences of proteins or the nucleotides of DNA sequences. A BLAST search enables a researcher to compare a query sequence with a library or database of sequences, and identify library sequences that resemble the query sequence above a certain threshold.


>The BLAST algorithm is a heuristic search method that seeks words of length W (default = 3 in blastp) that score at least T when aligned with the query and scored with a substitution matrix. Words in the database that score T or greater are extended in both directions in an attempt to find a locally optimal ungapped alignment or HSP (high scoring pair) with a score of at least S or an E value lower than the specified threshold. HSPs that meet these criteria will be reported by BLAST, provided they do not exceed the cutoff value specified for number of descriptions and/or alignments to report.

![Algorithm](/assets/BLAST_algorithm.gif)

---

### Possible Solution

The algorithm's implementation distributed by NCBI is what we are working on.One feature that was interestingly lacking was the fact that it did not support multiple cores and did not even multithread.A very simple but effective solution would be to utilise the concept of SIMD in this case

Here's what Wikipedia has to say about SIMD :

>Single instruction, multiple data (SIMD), is a class of parallel computers in Flynn's taxonomy. It describes computers with multiple processing elements that perform the same operation on multiple data points simultaneously. Thus, such machines exploit data level parallelism, but not concurrency: there are simultaneous (parallel) computations, but only a single process (instruction) at a given moment.

Since it is a string match/search algorithm we could simply split up the input file and run it through the program multiple times to get the same output.

---

### Details and Issues

The input to the program is a protein database which consists of a long string of ACTG characters and alignment takes place against a smaller protien or amino acid.Since it is simple alignment the larger input can be split up into multiple smaller dna sequences and searched on.Each of these input files can be run on a separate processor of a multicore machine improving time efficiency.We will be writing a simple shell script to take the split up input file and spawn a process for each core and combine the results in the end.

But there would be two major issues - 

- The may be matches across splits ie. half the string is in split one and the other half in split two.
- BLAST is a heuristic algorithm and uses a "E" Value to do the filtering.This value is directly proportional to the size of input.

To overcome these issues we first make sure the splits of the input file overlap by the size of the protein sequence being searched for,this will result in 2 duplicate matches but those are easy to filter out.
The E-Value,since it is directly proportional to the size of input we just multiply it by the number of splits and do another round of filtering via the script itself.

---

### Solution Implementation

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

To achieve this I wrote two shell scripts, one to split the input file based on it's size and number of cores on the system maintaining th overlaps in the files.It then feeds these files along with the search string to the blastx program which is run with GNU Parallel which offloads each job on a separate core.

GNU parallel is a shell tool for executing jobs in parallel using one or more computers. A job can be a single command or a small script that has to be run for each of the lines in the input. GNU parallel can then split the input and pipe it into commands in parallel.GNU parallel makes sure output from the commands is the same output as you would get had you run the commands sequentially.

```sh

#gnu parallel
seq 0 7 | parallel ./blastx -query DNA_split_sequence0{}.fa -subject 
prot_sample1.fa -out 500k_600k_ParallelOutput{}.txt

```
Complete [DNA_Split Sript](https://github.com/Apaar/Parallel-Computing/blob/master/ncbi-blast/split_DNA.sh)

Once the script runs we run the [Split_Output Sript](https://github.com/Apaar/Parallel-Computing/blob/master/ncbi-blast/split_output.sh) script.It joins the files, removes duplicates and filters based on E-Value which are grepped from the output files as shown below.

```sh
for f in loop$i*;do
		e=$(grep -Po 'Expect = \K.*(?=,)'  ${f})
		new_e=$(echo "scale=4;($ratio*$e)/1"|bc)
		
		t_e=10.0
		if [ $(echo "$new_e > $t_e" | bc -l) -ne 0 ]; then
			rm -rf loop$i*
			break;
		else
			sed -i "s/Expect =.*, /Expect = $new_e , /g" ${f}
			mv -i "${f}" "file_$new_e"	
		fi
done
```

---

### Results

Here are the results gained upon running the script - 

![Benchmarks](/assets/benchmarks.png)

This benchmark was run on a computer with a i7-4820 which had 4 physical cores along with 20GB of RAM.
The scale of time is in minutes and a speedup of around 0.9 times the number of cores available was achieved on the baseline NCBI Blast algorithm by simply noticing the lack of threading and applying the concept of SIMD or Single Instruction Multiple Data to this algorithm.



