We use the readFilter.pl script to wrap two useful tools:

* [FASTX-Toolkit](http://hannonlab.cshl.edu/fastx_toolkit/)
* [BBMap]https://sourceforge.net/projects/bbmap/)

FASTX-Toolkit is used to filter out reads with a given proportion of sites under a set quality threshold and also reads less than a specified length. BBMap is then used to filter out reads that do not match either the forward or reverse primer sequences (which should be at each end of the read).

Here is an example of how this script is run:

    readFilter.pl -q 30 -p 90 -l 400 stitched_reads/*.assembled.*

This command would filter out reads that have a quality score less than 30 in at least 90% of sites. Any reads less than 400 bp or that do not match the default primer sequences are also removed.  

Note that by default the primer sequences checked for are ACGCGHNRAACCTTACC and TTGYACWCACYGCCCGT for the 5' and 3' primers respectively (note that is the reverse complement of the 3' primer). You should check what primer sequences were used for your data and use the "-f" and "-r" options as described below. You can only check for the forward primer by setting by using the "--primer_check" option. If you don't want to check for primer sequences than you can set the oligos to scan for to be "N", which will match all sequences. 

This script assumes FASTX-Toolkit is in the user's PATH. You will likely need to point the program to where the jar files for BBMap are installed with the "--bbmap" option (it assumes the jar files are in /usr/local/prg/bbmap by default). 

By default the numbers (and %) of reads filtered out at each step are reported in "readFilter_log.txt".

Options: 

* -h, --help <br>
   Displays the entire help documentation.

* -v, --version <br>
   Displays script version and exits.

* -o, --out_dir <file> <br> 
   Output directory for filtered fastq files. Default is "filtered_reads".

* -thread <# of CPUs> <br>
   Using this option without a value will use all CPUs on machine, while giving it a value will limit to that many CPUs. Without option only one CPU is used.

* -log <file> <br>
   The location to write the log file.

* -q, --min_quality <#> <br>
   Minimum base quality.

* -p, --percent <#> <br>
   Minimum percent of bases per read that pass quality cut-off

* -l, --min_length <#> <br>
   Minimum read length.

* -f, --forward <oligo> <br>
   Forward primer to match at beginning of all reads (IUPAC format, default: ACGCGHNRAACCTTACC).

* -r, --reverse <oligo> <br>
   Reverse primer to match at end of all reads (IUPAC format, default: TTGYACWCACYGCCCGT, which is the reverse complement of the primer ACGGGCRGTGWGTRCAA).

* -b, --bbmap <PATH> <br>
   BBMap directory containing sh files (default: /usr/local/prg/bbmap).

* -pc, --primer_check <[both|forward]> <br>
   Either "both" or "forward", indicating whether to check both forward (5') and reverse (3') primer sequences or only the forward primer respectively (default: both).