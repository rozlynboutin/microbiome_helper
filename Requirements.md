The following programs should be installed with commands accessible from the user's PATH, before trying to run any of the scripts included in this repository.

**Both pipelines**
* FastQC (optional): http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
* PEAR: http://sco.h-its.org/exelixis/web/software/pear/doc.html
 
**Metagenomics**
* Bowtie2: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
* Human pre-indexed database: ftp://ftp.ccb.jhu.edu/pub/data/bowtie2_indexes/hg19.zip
* DIAMOND (> v.0.7.0): http://ab.inf.uni-tuebingen.de/software/diamond/
* MetaPhlAn2: https://bitbucket.org/biobakery/metaphlan2
* HUMAnN: http://huttenhower.sph.harvard.edu/humann

**16S**
* FASTX toolkit (v0.0.14): http://hannonlab.cshl.edu/fastx_toolkit/download.html
* BBMap: http://sourceforge.net/projects/bbmap 
* VSEARCH: https://github.com/torognes/vsearch
* QIIME: http://qiime.org
* SortMeRNA: http://bioinfo.lifl.fr/RNA/sortmerna/
* SUMACLUST: http://metabarcoding.org/sumatra
* PICRUSt: http://picrust.github.io/picrust/

**Visualization**
* STAMP: http://kiwi.cs.dal.ca/Software/STAMP


**Databases (dropbox download links)**
* 16S chimera checking: [Bacteria_RDP_trainset15_092015.fa](https://www.dropbox.com/s/r290fbzgi216kja/Bacteria_RDP_trainset15_092015.fa?dl=1) (17 MB). This DB was originally from the Ribosome Database Project (RDP) and was parsed to include only bacteria. 
* 18S chimera checking: [Eukaryota_SILVA_123_SSURef_Nr99_tax_silva_U-replaced.fa] (https://www.dropbox.com/s/p4wnjyvatdobq5h/Eukaryota_SILVA_123_SSURef_Nr99_tax_silva_U-replaced.fasta?dl=1) (117 MB). This DB was taken from SILVA and was parsed to include only eukaryotes and all Us were converted to Ts.

* Eukaryotic OTU-picking files:
   * [gb203_pr2_all_10_28_99p_tax_Xs-fixed_poly-fixed.txt] (https://www.dropbox.com/s/bvot58v47rfc6a2/gb203_pr2_all_10_28_99p_tax_Xs-fixed_poly-fixed.txt?dl=1) (9 MB)
   * [gb203_pr2_all_10_28_99p_clean.fasta](https://www.dropbox.com/s/m1i6cdyj2hwgs2e/gb203_pr2_all_10_28_99p_clean.fasta?dl=1) (98 MB)
   * [90_Silva_111_rep_set_euk_aligned.filter.fasta](https://www.dropbox.com/s/z104yn84rgsltip/90_Silva_111_rep_set_euk_aligned.filter.fasta?dl=1) (95 MB)

* To use "run_human_filter.pl" you will need to download the appropriate [Bowtie2 index](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml). 

* To use "run_pre_humann.pl" you will need to use a [database of KEGG orthologs](https://www.dropbox.com/s/hzduqabilbrqr36/kegg.reduced.dmnd?dl=1). You may also want to [download the raw FASTA](https://www.dropbox.com/s/8mw2kqg1xjv7lwf/kegg.reduced.fasta.tar.bz2?dl=1). 

**Perl modules** 

    File::Basename Getopt::Long List::Util Parallel::ForkManager Pod::Usage