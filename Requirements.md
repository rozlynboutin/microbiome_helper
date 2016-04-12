The following programs should be installed with commands accessible from the user's PATH, before trying to run any of the scripts included in this repository.

**Both pipelines**
* FastQC (v0.11.2) (optional): http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
* PEAR: http://sco.h-its.org/exelixis/web/software/pear/doc.html
 
**Metagenomics**
* Bowtie2: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
* Human pre-indexed database: ftp://ftp.ccb.jhu.edu/pub/data/bowtie2_indexes/hg19.zip
* DIAMOND (> v.0.7.0): http://ab.inf.uni-tuebingen.de/software/diamond/
* MetaPhlAn2: https://bitbucket.org/biobakery/metaphlan2
* HUMAnN: http://huttenhower.sph.harvard.edu/humann

**16S**
* FASTX toolkit (v0.0.14): http://hannonlab.cshl.edu/fastx_toolkit/download.html
* BBMap (v35.59): http://sourceforge.net/projects/bbmap 
* USEARCH (v6.1.544): http://www.drive5.com/usearch/
* QIIME (v1.9): http://qiime.org
* SortMeRNA: http://bioinfo.lifl.fr/RNA/sortmerna/
* SUMACLUST: http://metabarcoding.org/sumatra
* PICRUSt: http://picrust.github.io/picrust/

**Visualization**
* STAMP: http://kiwi.cs.dal.ca/Software/STAMP


**Databases (dropbox download links)**
* 16S chimera checking: [Bacteria_RDP_trainset15_092015.udb](https://www.dropbox.com/s/8qr42doaez48oc3/Bacteria_RDP_trainset15_092015.udb?dl=0) (70 MB). This DB was originally from the Ribosome Database Project (RDP) and was parsed to include only bacteria. Note that all .udb files on this page were created with the usearch61 -makeudb_usearch command.

**Eukaryotic OTU-picking files, as discussed [here]**
* [Silva_119_rep_set90_aligned_18S.fna](https://www.dropbox.com/s/cw77k375ayaqh0n/Silva_119_rep_set90_aligned_18S.fna?dl=0)
* [Silva_119_rep_set99_18S_taxonomy_7_levels.txt](https://www.dropbox.com/s/lj44kqjx3u1u2ei/Silva_119_rep_set99_18S_taxonomy_7_levels.txt?dl=0)
* [Silva_119_rep_set99_18S.fna](https://www.dropbox.com/s/muq4up1lp5al5pz/Silva_119_rep_set99_18S.fna?dl=0)