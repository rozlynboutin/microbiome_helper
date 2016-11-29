There are some alternatives to to OTU picking that can be useful for microbiome sequencing analyses. We have included some of these tools in our virtual box image, including: [DADA2](http://benjjneb.github.io/dada2/index.html), [Minimum Entropy Decomposition (MED)](http://merenlab.org/2014/11/04/med/) and [IPED](http://science.sckcen.be/en/Institutes/EHS/MCB/MIC/Bioinformatics/IPED). 


### Minimum Entropy Decomposition (MED)
    cd /home/shared/err_corr_examples/med_example
    decompose sponge-1K.fa -E sponge-sample-mapping.txt -T

Note that the "-T" option above turns of multi-threading, which might be necessary if you only have 1 CPU enabled on the virtual box. If you have more CPU to spare and you omit this option the command should run much faster. 
  
Once this job completes you'll be able to take a loop at the summary file below in FireFox.
    sponge-1K-m0.10-A0-M0-d4/HTML-OUTPUT/index.html


