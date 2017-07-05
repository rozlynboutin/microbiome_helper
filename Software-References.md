# Under Construction

This page aims to list many useful bioinformatic tools for microbial ecology and related areas. This is not an exhaustive list, but should be useful to newcomers to the field.

Some other lists you may be interested in are:
* [Wikipedia list of RNA-seq bioinformatic tools](https://en.wikipedia.org/wiki/List_of_RNA-Seq_bioinformatics_tools) - many of which are the same or similar tools that are commonly used in microbial ecology.

Some of the below tools are highlighted by asterisks, which correspond to:
```
* Tools that are in the Microbiome Helper virtual box.
** Tools that we are looking into and may add soon.
```

If you there are tools you think we are missing from this list then please let us know! 

# Toolkits
* mothur* ([website](https://www.mothur.org/), [paper](http://aem.asm.org/content/75/23/7537.full))
* QIIME* ([website](http://qiime.org/), [paper](http://www.nature.com/nmeth/journal/v7/n5/full/nmeth.f.303.html))
* QIIME2 - alpha release ([website](https://qiime2.org/))

# Pre-processing

## Quality Control  
* FastQC* ([website](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)) - Java program that provides simple graphical user interface (GUI) or command-line options to quickly generate summary graphs and tables describing raw sequence files.  
* fastqp ([website](https://github.com/mdshw5/fastqp)) - Python alternative to FastQC that produces similar summaries.  

## Read filtering and/or trimming  
* BBDuk* ([website](https://sourceforge.net/projects/bbmap/), [helpful seqanswers thread](http://seqanswers.com/forums/showthread.php?t=41057)) - Java program implemented in the BBMap package, which includes bioinformatic tools for a number of diverse purposes. BBDuk's focus is on filtering, trimming, or masking reads based on matches to user-specified sequences/kmers, but can also be used for other purposes such as filtering reads by length and quality cut-off.
* FASTX-Toolkit* ([website](http://hannonlab.cshl.edu/fastx_toolkit/)) - Many different C and C++ command-line programs for filtering and transforming sequence files. 

## Chimera filtering  
* CATCh ([website](http://science.sckcen.be/en/Institutes/EHS/MCB/MIC/Bioinformatics/CATCh), [paper](http://aem.asm.org/content/81/5/1573.full))
* ChimeraSlayer
* UCHIME
* VSEARCH* ([website](https://github.com/torognes/vsearch), [paper](https://peerj.com/articles/2584/)) - open-source implementation of the UCHIME algorithm.

## Error correction 
* AmpliconNoise
* DADA2
* Deblur

