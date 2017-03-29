Author: Gavin Douglas  
Last Updated: 29 March 2017  
  
## Introduction  
  
[GNU Parallel](https://www.gnu.org/software/parallel/) is a helpful tool for threading repetitive commands. This is especially useful in bioinformatics where often we want to run exactly the same command on a large batch of files. GNU Parallel has extensive documentation and can give users sophisticated control. Below I'm just going to demonstrate how a user would run the basic commands to run many jobs in simultaneously.  
  
If you plan on running GNU Parallel on a server with many CPUs you should note that it will likely be more efficient to run fewer jobs if they are spending most of their time on input/output operations (i.e. I/O bound). So if you want to run many gzip, cp, or similar commands in parallel it will likely be better to set "--load" to a lower percentage then we have set below.  
    
### Downloading the data  
  
I downloaded example sequences from the [Protein Data Bank sequences](http://www.rcsb.org/pdb/home/home.do) and generated a blast database. You'll need to download [this database and the tutorial data](https://www.dropbox.com/s/v7twasg0fro45x0/parallel_example.tar.gz?dl=1) (45 MB) if you want to run the examples commands below. This data is just a toy example and isn't meant to demonstrate the best way to run blastp!    
  
You can download the data with this command: 
   
    wget https://www.dropbox.com/s/v7twasg0fro45x0/parallel_example.tar.gz?dl=1 -O parallel_example.tar.gz  
  
And decompress the data with this command:  
  
    tar xfvz parallel_example.tar.gz  
  
Enter this folder:  
  
    cd parallel_example
  
The commands below assume you are within this decompressed directory.    
  
### Resources  
  
Remember to cite the paper if you use this tool: [GNU Parallel: The Command-Line Power Tool](https://www.usenix.org/system/files/login/articles/105438-Tange.pdf)
    
This [detailed forum post](https://www.biostars.org/p/63816/) is really useful and shows some more sophisticated examples. 
    
Also, be sure to get some [GNU parallel merch](https://gnuparallel.threadless.com/) to get in the mood for some serious multi-threading.  
  
## Unique Options  
You'll need to understand what the below syntax stands for to understand the tutorial commands. 
  
* The file name: **{}**  
* The file name with the extension removed: **{.}**  
    e.g. test.fa would become test  
* To indicate that everything that follows should be read in from the command line: **:::**   
    e.g. "parallel gzip ::: *" means to gzip all files in the current working directory, while "parallel gzip *" wont work. You need to include ":::".  
  
There are many other possible options for GNU Parallel as well, which you can read about [here](https://www.gnu.org/software/parallel/man.html).  

## Running blastp commands with GNU parallel  
  
To demonstrate how to run basic commands in parallel we'll be running blastp on a number of different FASTA files (see above for how to download them) simultaneously. Note that blastp supports multi-threading (e.g. a single job can be split over multiple CPUs), but this isn't the case for all programs.   
  
Often in bioinformatics we want to repeat a command over a large number of files. In this example there are 10 files we want to run blastp on with the same options. With the below _parallel_ command we'll run blastp on two files at a time with 1 thread allocated for each file.   
  
    mkdir blastp_outfiles    
    
    parallel --eta -j 2 --load 80% --noswap 'blastp -db pdb_blast_db_example/pdb_seqres.txt -query {} -out blastp_outfiles/{.}.out -evalue 0.0001 -word_size 7 -outfmt "6 std stitle staxids sscinames" -max_target_seqs 10 -num_threads 1' ::: test_seq*.fas
    
You should find that only four query sequences actually match anything in the database.  
    
Again, the options being given to _parallel_ is everything before the single-quotes. The command inside the single-quotes contains options for blastp only, which were chosen just for this example (one to note is "-num_threads" which is the number of threads to use for each blastp command). 
  
The options we passed to _parallel_ are:  
* "--eta": Shows the estimated time remaining to run all jobs.  
* "-j 2" (or "--jobs 2"): The number of commands to run at the same time, which in this case was set to 2.
* "--load 80%": The maximum CPU load at which new jobs will not be started. So in this case we are specifying that jobs can be run simultaneously up to 80% of the CPUs being run. This is more important when you are dealing with larger numbers of long-running commands.  
* "--noswap": New jobs won't be started if there is both swap-in and swap-out activity. In other words, new jobs won't be started if the server is under heavy memory load such that information needs to be removed from memory before new information can be stored.  

You can put options you want to use every time in a global configuration file here: /etc/parallel/config. Command-line inputs take precedence over these options.  
   
### Piping a list of input files     
  
Note that at the end of the _parallel_ command we used the ":::" syntax to indicate the input files. You can also pipe ("|") in input files to _parallel_ rather than using the ":::" syntax. If you're not familiar with piping in Linux then you should look-up an online tutorial like the one [here](http://ryanstutorials.net/linuxtutorial/piping.php). Piping input files can be a little easier to understand due to the simpler syntax. This _parallel_ command runs the same jobs as the earlier example:  
  
    mkdir blastp_outfiles2  
    
    ls test_seq*.fas | parallel --eta -j 2 --load 80% --noswap 'blastp -db pdb_blast_db_example/pdb_seqres.txt -query {} -out blastp_outfiles2/{.}.out -evalue 0.0001 -word_size 7 -outfmt "6 std stitle staxids sscinames" -max_target_seqs 10 -num_threads 1'  
  
### Piping commands from a file  
  
You can also pipe lines of a file to _parallel_. As an example I will use a simple bash loop to write the commands we ran above to a file and then input these commands line-by-line to _parallel_. For this example writing a bash loop is much more complicated then just running the commands using either of the methods shown above, but I'll show it anyway since it could be useful in other contexts. 

Make new output folder:
  
    mkdir blastp_outfiles3  

Bash loop to produce the commands:  
  
    for f in test_seq*.fas  
    do  
    out=${f/.fas/.out};  
    echo "blastp -db pdb_blast_db_example/pdb_seqres.txt -query $f -out blastp_outfiles3/$out -evalue 0.0001 -word_size 7 -outfmt \"6 std stitle staxids sscinames\" -max_target_seqs 10 -num_threads 1" >> blastp_cmds.txt    
    done   
  
Cat file of commands and pipe to _parallel_:  
  
    cat blastp_cmds.txt | parallel --eta -j 2 --load 80% --noswap '{}'  
  
In this case since the whole command is being input we can just refer to the input itself with '{}'.  
  