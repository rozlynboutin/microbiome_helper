There are some alternatives to to OTU picking that can be useful for microbiome sequencing analyses. We have included some of these tools in our virtual box image, including: [DADA2](http://benjjneb.github.io/dada2/index.html), [Minimum Entropy Decomposition (MED)](http://merenlab.org/2014/11/04/med/) and [IPED](http://science.sckcen.be/en/Institutes/EHS/MCB/MIC/Bioinformatics/IPED). Below is a quick overview of how you can try these tools out on our virtual box. For details you should check out each tool's website linked to above. Also, if you use any of these tools be sure to cite them! 

### DADA2 
DADA2 takes in a directory of FASTQ files and models and corrects errors found within the reads. This approach allows exact sequence variants to be inferred, which is an alternative to OTU clustering. This is the approach that [QIIME 2](https://qiime2.org/) (which is still in alpha phase) is moving towards. There is a [tutorial](http://benjjneb.github.io/dada2/tutorial.html) available for DADA2 that you can walk through. You'll be able to run this in RStudio or using command-line R in our virtual box. 
  
Besides the DADA2 website linked to above, you can check out the DADA2 paper for details:
_Callahan et al. 2016. DADA2: High-resolution sample inference from Illumina amplicon data. Nature Methods 13:581-583_.

### Minimum Entropy Decomposition (MED)
    cd /home/shared/err_corr_examples/med_example
    decompose sponge-1K.fa -E sponge-sample-mapping.txt -T

Note that the "-T" option above turns of multi-threading, which might be necessary if you only have 1 CPU enabled on the virtual box. If you have more CPU to spare and you omit this option the command should run much faster. 
  
Once this job completes you'll be able to take a loop at the summary file below in FireFox.  
  
    sponge-1K-m0.10-A0-M0-d4/HTML-OUTPUT/index.html


### IPED

**Step 1**

    /usr/local/prg/IPED/IPED_V1.run _F /home/shared/err_corr_examples/IPED_example/sample.forward.fastq _R /home/shared/err_corr_examples/IPED_example/sample.reverse.fastq _o /home/shared/err_corr_examples/IPED_example/step1_out_

**Step 2**
  
Note that you'll need to replace "TMP_DIRECTORY" below with the large number which is the name of the folder in the Step 1 directory (e.g. for me it was "952567556015314", but it will be different each time you run it).

    mkdir step2_out

    /usr/local/prg/IPED/IPED_V1.run _f /home/shared/err_corr_examples/IPED_example/sample.fasta _n /home/shared/err_corr_examples/IPED_example/sample.names _c /home/shared/err_corr_examples/IPED_example/step1_out_IPED_Final/TMP_DIRECTORY/sample.forward.trim.contigs.fasta _q /home/shared/err_corr_examples/IPED_example/step1_out_IPED_Final/TMP_DIRECTORY/sample.forward.contigs.qual _o /home/shared/err_corr_examples/IPED_example/step2_out

