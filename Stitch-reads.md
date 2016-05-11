We use [PEAR](http://sco.h-its.org/exelixis/web/software/pear/) to stitch pair-end reads together.

Our run_pear.pl wrapper script can simply be used like so:

    run_pear.pl raw_data/*

Where "raw_data" is a folder that contains FASTQs with either "\_R1\_" and "\_R2\_" in their names (alternatively just "_1" and "_2" in the filenames will work). 

Four FASTQs are created for each sample: 
* assembled reads (typically ~98% of reads, although this can depend on your data type / quality)
* discarded reads (e.g. reads that are all Ns)
* unassembled reads, 1 FASTQ for each direction (we discard these for downstream analyses)

This script generates a summary of the percents of assembled, discarded and unassembled reads for each sample (by default: "pear_summary_log.txt").

Options:
*    -o, --out_dir <file> <br>
        The name of the output directory to place all PEAR output files.

*    -p, --parallel [<# of proc>] <br>
        Using this option without a value will use all CPUs on machine, while
        giving it a value will limit to that many CPUs. Without option only
        one CPU is used.

*    -g, --gzip_output <br>
        Gzip the PEAR output files.

*    -f, --full_log <file> <br>
        The location to write the PEAR full log file. Default is
        "pear_full_log.txt"

*    -s, --summary_log <file> <br>
        The location to write the PEAR summary log file. Default is
        "pear_summary_log.txt"

*    -h, --help <br>
        Displays the entire help documentation.