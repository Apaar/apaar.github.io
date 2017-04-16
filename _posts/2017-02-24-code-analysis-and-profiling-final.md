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

To achive this I wrote two shell scripts, one to split the input file based on it's size and number of cores on the system maintaining th overlaps in the files.It then feeds these files along with the search string to the blastx program which is run with GNU Parallel which offloads each job on a separate core.

GNU parallel is a shell tool for executing jobs in parallel using one or more computers. A job can be a single command or a small script that has to be run for each of the lines in the input. GNU parallel can then split the input and pipe it into commands in parallel.GNU parallel makes sure output from the commands is the same output as you would get had you run the commands sequentially.

```sh

#gnu parallel
seq 0 7 | parallel ./blastx -query DNA_split_sequence0{}.fa -subject 
prot_sample1.fa -out 500k_600k_ParallelOutput{}.txt

```
Complete [DNA_Split Sript](https://github.com/Apaar/Parallel-Computing/blob/master/ncbi-blast/split_DNA.sh)

Once the script runs we run the [Split_Output Sript](https://github.com/Apaar/Parallel-Computing/blob/master/ncbi-blast/split_output.sh) script.It joins the files, removes duplicates and filters based on E-Value which are grepped from the output files as shown below.

```sh
for f in lolz$i*;do
		e=$(grep -Po 'Expect = \K.*(?=,)'  ${f})
		new_e=$(echo "scale=4;($ratio*$e)/1"|bc)
		
		t_e=10.0
		if [ $(echo "$new_e > $t_e" | bc -l) -ne 0 ]; then
			rm -rf lolz$i*
			break;
		else
			sed -i "s/Expect =.*, /Expect = $new_e , /g" ${f}
			mv -i "${f}" "yoyo$new_e"	
		fi
	done
```



