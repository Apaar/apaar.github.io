---
layout: post
title: Parallelizing Grep - 1
categories:
- blog
---

After the mishaps with GNU Grep I decided to give up on modifying the official codebase and tried to write my own version of grep.It takes a file as input and does simple string matching on it in a parallelized manner.This was written in C and pThreads were used to achive parallelization.In this post we will be discussing that code(can be found [here](https://github.com/Apaar/Parallel-Computing/tree/master/grep)).

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

The result was a fairly parallelized version of grep that took input as a filename and a search string and outputs the line of occurance of the string, but it lacks all the features and options of the original grep.In the next blog post I will be benchmarking both approaches discussed and comparing with the original grep.






