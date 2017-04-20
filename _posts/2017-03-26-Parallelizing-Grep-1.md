---
layout: post
title: Parallelizing Grep - 1
categories:
- blog
---


After the mishaps with GNU Grep I decided to give up on modifying the official codebase and tried to write my own version of grep.It takes a file as input and does simple string matching on it in a parallelized manner.To take advantage of parallelism , we will be using the pThreads library in C++.Since the search results are line based, we can divide the file to be searched into chunks line based and pass it to threads to be searched.In this post we will be discussing that code(can be found [here](https://github.com/Apaar/Parallel-Computing/tree/master/grep)).


### Solution

My first approch was to figure out a way of separating the file into chunks so as to make it easier to offload a section of the search onto each core.For this I used mmap along with a simple structure.After a file had been mapped in memory along with the shared memory flag to allow for each of the threads to be able to access the file's contents in memory.But this approach does require the system to have sufficient memory to load almost the whole file into memory at once, this is because despite load on use used mmap ,all of the threads will be using differnet portions of the file at once.Once the file is loaded into memory.This is the code that does the same.

{% highlight c %}

   int fd = open(filename, O_RDONLY);
   fstat(fd, &mapstat) < 0)
   map = mmap(NULL, mapstat.st_size, PROT_READ, MAP_SHARED, fd, 0);
   if (map == MAP_FAILED) {
      perror(filename);
      goto out2;
   }
   atexit(free_map);

{% endhighlight c %}

### Splitting File into Chunks for Each Thread

Once the file is loaded into memory we read the number of cores on the system and divide the file into chunks , storing the base address,and length of each blob of memory into a array of structures.Each of these will be passed to each thread for them to search simultaneously.I also had to take care of the case of the last chunk/blob which would be of uneven size.
The following code was used to spawn the threads, notice the ellements of the array of structures being passed to each thread along wiht a handler function which we will discuss next.

{% highlight c %}
   for (i = 0; i < num_cpus; i++) {
      pthread_create(&threads[i], NULL, &run_blob, (void *)&blob[i]);
   }

   for (i = 0; i < num_cpus; i++) {
      pthread_join(threads[i], NULL);
   }
{% endhighlight c %} 

### Handler for pThread

The handler function uses strchr to load each line within a while loop from the base address to the length.The memem fuction is used to do efficient substring matching within the line with the search string as shown below and all occurances are printed as they are found.Contents of the loop are shown below.

{% highlight c %}
   while (offset < blob->len) {
      next = strchr(line, '\n');
      if(!next) {
         break;
      }

      linelen = next - line + 1;

      if (memmem(line, linelen - 1, opt.input, opt.input_len)) {
            printf("%.*s", (int)linelen, line);         
      }

      offset += linelen;
      line = next + 1;
   }
{% endhighlight c %} 

The result was a fairly parallelized version of grep that took input as a filename and a search string and outputs the line of occurance of the string, but it lacks all the features and options of the original grep.Next I will be benchmarking both approaches discussed and comparing with the original grep.

---

### Benchmarks/Results

The three approaches being benchmarked are GNU Grep, GNU Grep along with GNU parallel and my threaded barebones grep.
I used a shell script to generate a 5.6GiB text file which repeatedly had the same content of one 3MB file.
{% highlight c %}
for i in {1..20}; do cat test test > test1 && mv test1 test; done
{% endhighlight c %}
The system being used was a Quad core i5 with clock speed 2.8GHz and 12 GB of RAM(my laptop)

![Benchmark](/assets/benchmark.png)


GNU Grep + Parallel was a bit slower than my threaded version mainly due to the overhead of options etc that are in GNU Grep, my version only does simple string matching and not much processing.I also wrote a shell script to recursively grep through a directory structure.




