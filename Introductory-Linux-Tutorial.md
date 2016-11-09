# **This page is still under construction**

__Author:__ Gavin Douglas  
__First created:__ 1 June 2016  
__Last updated:__ 9 Nov 2016 

This quiz is designed to make sure that you understand command-line Linux basics. If you're having trouble you should try to google the problem (or look at the "man" page for a particular command) before clicking for the answer. Note that the answer shown in most cases is just one possible method.  

Although I will briefly review some Linux commands, this is not intended to be an exhaustive overview. If you're totally new to Linux I would recommend starting with a general tutorial, such as this one: http://korflab.ucdavis.edu/bootcamp.html. 

These commands are not covered below, but you should also become familiar with them or something similar for every-day work in Linux:  
* "df --si" - shows disk usage in human-readable format
* "du -sh (file or folder)" - shows size in human-readable format
* "history" - prints a large number of your last commands (usually ~1000) to screen. You can pipe this with grep (see below) to quickly see particular past commands you're interested in.  
* "screen" (with -d and -r options) - allows you to return to shells you previously had open (helps jobs from being interrupted!).
* "top" - shows all processes along with CPU and memory usage (you can customize this to just show your jobs, sort by memory and to show full commands).  

This is just a small number of commands that I find useful, but there are many others!

## Working with folders and files

1. Go to your home directory and make a new folder called "tmp". Go into this folder.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        cd # with no target this command should bring you to your home directory 
        mkdir tmp
        cd tmp
    </code></pre></details>

2. Make a new file called "test" and put "a	b	c" on the first line (Note this is tab, NOT space delimited). Looking at the "echo" command's manual page with "man echo" might be helpful.
  
    This question is related to knowing the difference between these input and output operators, which are commonly mis-understood by beginners: ">", ">>" , "2>" and "2>>".

    ">" is used to direct output to a file, as used above:  
        command > out.txt

    ">>" is used to _append_ output to a file (i.e. it wont be overwritten):  
        command >> out.txt    
 
    "2>" is used to direct error output to a file:  
        command 2> log.txt

    "2>>" is used to append error output to a file:  
        command 2>> log.txt

    The output written with ">" is called standard output (or "stdout"), while "2>" writes to standard error ("stderr"). Both stdout and stderr are printed to the screen by default when you run a command.

    There are other ways to re-direct output, but these are the most common ones you'll likely need.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        echo -e "a\tb\tc" > test
    </code></pre></details>

3. With one command, append "d	e	f" (tab delimited) to the file (do this without opening "test" up in any text editor). 
    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        echo -e "d\te\tf" >> test
    </code></pre></details>

4. Make a copy of "test" called "test_copy".

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        cp test test_copy
    </code></pre></details>

5. Rename "test_copy" to "test2". Delete "test". 

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        mv test_copy test2
        rm test
    </code></pre></details>

6. Use wget to download the file found at this link: https://www.dropbox.com/s/gta5voeugfmkv14/test_table.txt?dl=1. Use the -O option to specify a new output name to be "test_table.txt" (otherwise it will end in "?dl=1").

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        wget https://www.dropbox.com/s/gta5voeugfmkv14/test_table.txt?dl=1 -O test_table.txt
    </code></pre></details>

7. Figure out how many lines are in this file (you might get a weird result...)

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        $ wc -l test_table.txt
        0
    </code></pre></details>

    If you look at the file using the "less" command (hit "q" to quit less), you should see a bunch of ^M characters (it may not look this way depending on your system). This is a special character that means "carriage-return", which is required at the end of lines in some formats. It's good to be aware that files might contain special characters that cause problems on different systems, since this is a common problem when uploading files from your local PC to a Linux server.

8. There are a couple of ways to fix this file, but one is by using _sed_, which is a program that is used to transform text as it is read in from an input "stream" (usually a file, but it could be the output of a different function). You can read about sed with this command: _man sed_. Use sed to replace all "^M" characters with "\n" (newline) characters. The trick is that to type the ^M character correctly on the command-line you'll need to hold CTRL and then hit the v and m keys (you wont even be able to copy and paste the below answer).

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        sed -i 's/^M/\n/g' test_table.txt 
    </code></pre></details>


9. Figure out the number of lines in the file after replace carriage-returns with newlines.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        $ wc -l test_table.txt
        984
    </code></pre></details>

10. Cut out the 2nd, 4th and 6th columns into a new file called "test_table_col_cut.txt".

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        cut -f 2,4,6 test_table.txt  > test_table_col_cut.txt
    </code></pre></details>

11. Using the "grep" command (you will need to look at the options with "man grep" and be aware that the regular expression for tab is usually "/t") figure out the # of lines in test_table_col_cut.txt that match "\t1\t".

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        grep -Pc "\t1\t" test_table_col_cut.txt 
        # The -P option allows for Perl regular expressions. 
        # This option may not be available depending on your version of grep 
        # in which case you could type in the special tab character ("^I") directly, 
        # similar to how we did above for ^M.
    </code></pre></details>

12. Using "head", "less" and a pipe ("|") look at ONLY the first 5 lines of "test_table_col_cut.txt" with less.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        head -n 5 test_table_col_cut.txt | less 
        # You will need to hit q to exit less
    </code></pre></details>

13. Using the _awk_ command, print out all rows of test_table_col_cut.txt where the first column has a value greater than 
10. You should not print the header line, which you can avoid by using an option in awk or by piping the output of the "tail" command (the opposite of "head") to awk, similar to Q13.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        tail -n +2 test_table_col_cut.txt  | awk '{ if ( $1 > 10 ) { print $0 } }'  > test_table_col_cut_first10.txt
    </code></pre></details>

14. Create 10 folders called "parent_i" (where the i is an integer ranging from 1 to 10). In each of these folders also create a folder called "child_i" (where the last i is the same integer from the parent folder's name). This can be done with a bash loop. You will likely need to check out a bash tutorial to answer this question.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        for i in {1..10}; do parent="parent_"$i; child="child_"$i; mkdir -p $parent/$child; done
    </code></pre></details>