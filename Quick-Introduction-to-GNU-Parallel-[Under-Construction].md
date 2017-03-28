## Introduction  
  
[GNU Parallel](https://www.gnu.org/software/parallel/) is a helpful tool for threading repetitive commands. This is especially useful in bioinformatics where often we want to run exactly the same command on a large batch of files. GNU Parallel has extensive documentation and can give users sophisticated control. Below I'm just going to demonstrate how a user would run the basic commands to run many jobs in parallel.  
    
### Downloading the data  
  
You'll need to download the [tutorial data](https://www.dropbox.com/s/glb161j05vyxx4v/parallel_examples.tar.gz?dl=1) if you want to run the examples commands below.  
  
You can download the data with this command: 
   
    wget https://www.dropbox.com/s/glb161j05vyxx4v/parallel_examples.tar.gz?dl=1 -O parallel_examples.tar.gz  
  
And decompress the data with this command:  
  
    tar xfvz parallel_examples.tar.gz  
  
The commands below assume you are within this decompressed directory.    
  
### Resources  
  
Remember to cite the paper if you use this tool: [GNU Parallel: The Command-Line Power Tool](https://www.usenix.org/system/files/login/articles/105438-Tange.pdf)
    
This [detailed forum post](https://www.biostars.org/p/63816/) is really useful and shows some more sophisticated examples. 
    
Also, be sure to get some [GNU parallel merch](https://gnuparallel.threadless.com/) to get in the mood for some serious multi-threading.  
  
## Unique Variables  
You'll need to understand what the below syntax stands for to understand the tutorial commands. 
  
* The file name: **{}**  
* The file name with the extension removed: **{.}**  
    e.g. test.fa would become test  
* To indicate that everything that follows should be read in from the command line: **:::**   
    e.g. "parallel gzip ::: *" means to gzip all files in the current working directory, while "parallel gzip *" wont work. You need to include ":::".  

## Running blastp in parallel  
  
To demonstrate how to run basic commands in parallel we'll be running blastp on a number of different FASTA files (see above for how to download them) simultaneously. Note that blastp supports multi-threading (e.g. a single job can be split over multiple CPUs), but this isn't the case for all programs.   
  
Often in bioinformatics we want to repeat a command over a large number of files. In this example there are 10 files we want to run blastp on with the same options. With the below command we'll run blastp on two files at a time with 4 threads allocated for each file.   
  
    parallel -j 2 'blastp -db nr -query {} -out {.}.out -evalue 0.0001 -outfmt "6 std stitle staxids sscinames" -max_target_seqs 10 -num_threads 4' ::: T4protein_*_randSplit.fas &
    
Note that everything within the single-quotes (') are example blastp options, which you could change to whatever you needed. Also note that two different options indicate the number of threads: "-j" is a parallel option while "-num_threads" is a blastp option. Again      
  

