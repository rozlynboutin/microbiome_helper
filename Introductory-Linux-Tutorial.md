# **This page is still under construction**

__Author:__ Gavin Douglas  
__First created:__ 1 June 2016  
__Last updated:__ 27 Oct 2016 

This quiz is designed to make sure that you understand command-line Linux basics. If you're having trouble you should try to google the problem (or look at the "man" page for a particular command) before clicking for the answer. Note that the answer shown in most cases is just one possible method.  

Although I will briefly review some Linux commands, this is not intended to be an exhaustive overview. If you're totally new to Linux I would recommend starting with a general tutorial, such as this one: http://korflab.ucdavis.edu/bootcamp.html. 

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

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        echo -e "a\tb\tc" > test
    </code></pre></details>

3. With one command, append "d	e	f" (tab delimited) to the file (do this without opening "test" up in any text editor). 

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

    There are other ways to re-direct, but these are the most common ones you'll likely need.

    <details> 
      <summary>Click here for answer</summary>
    <pre><code>
        echo -e "d\te\tf" >> test
    </code></pre></details>




4. Make a copy of "test" called "test_copy".

5. Rename "test_copy" to "test2". Delete "test". 

6. Use wget to download the file found at this link: https://www.dropbox.com/s/jnqbx4nzkjkx6we/test_OTU_table.txt?dl=1
(you will probably have to rename it to remove "?dl=1" from the end of the downloaded file)

7. Figure out how many lines are in this file (you might get a weird result...)

8. Look at the file using the "less" command (hit "q" to quit less), do you see a bunch of weird ^M characters? If so, what is the likely reason the file is messed up? 

### Note the string replacement below should be done with the text editor "vim"
9. If you do see the ^M characters then you can get around this problem using vim, which is great for string replacement (see: http://vim.wikia.com/wiki/Search_and_replace). As a test example, replace all "IWK" strings with "SCIENCEISGREAT" ( :%s/IWK/SCIENCEISGREAT/g ). What happens when "g" is omitted from that command?

10. Now replace all ^M characters using this command in vim: %s/^M/^M/g
The tricky part is you can't just copy and paste that since ^M is actually a single character. You need to hold down CTRL and then hit the v key and then the m key. 

The ^M characters should be replaced with proper newlines (i.e. line breaks) after this step.

11. Figure out the number of lines in the file again.

12. Cut out the 2nd, 4th and 6th columns into a new file called "test_OTU_table_col_cut.txt".

13. Using the "grep" command (you will need to look at the options with "man grep" and be aware that the special character for tab is usually "/t") figure out the # of lines in test_OTU_table_col_cut.txt that match "	1	".

14. Using "head", "less" and a pipe ("|") look at ONLY the first 5 lines of "test_OTU_table_col_cut.txt" with less.

15. Using the "awk" command, print out all rows of test_OTU_table_col_cut.txt where the first column has a value greater than 10. Write this output to a new file called "test_OTU_table_col_cut_first10.txt". Note that the header should not be printed since, which you can avoid by using an option in awk or by piping the output of the "tail" command to awk, similar to question 14.

16. By using the "sed" command (this is another way of doing string replacements that you might see), do the opposite of question 9, i.e. replace "SCIENCEISGREAT" with "IWK".

17. Create 10 folders called "parent_i" (where the i is an integer ranging from 1 to 10). In each of these folders also create a folder called "child_i" (where the last i is the same integer from the parent folder's name). This can be done with 1 line of bash code, but multiple commands (you can separate commands with ";" instead of return in bash).  