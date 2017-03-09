# Please note that this page is currently being written.
 
**Author**: Gavin Douglas
    
**First Created**: 7 March 2017
  
**Last Edited**: 8 March 2017

* [Introduction](#introduction)  
    * [Requirements](#requirements)  
    * [Other Resources](#other-resources)  
* [Background](#background)   
    * [Classification and Regression Tree Methods](#classification-and-regression-tree-methods)  
    * [Out-of-bag Error](#out-of-bag-error)  
    * [Variable Importance](#variable-importance)  
    * [Proximities](#proximities) 
* [Load Packages and Read in Data](#load-packages-and-read-in-data)    
* [Pre-processing](#pre-processing)  
    * [Removing Rare Features](#removing-rare-features)  
    * [Transforming Your Data](#transforming-your-data)   
* [Assessing Model Fit](#assessing-model-fit)  
    * [Permutation Test](#permutation-test)  
    * [Receiver Operating Characteristic Curve](#receiver-operating-characteristic-curve)   
* [Identifying Important Features](#identifying-important-features)  

## Introduction
This tutorial is intended to teach beginners the basics of running random forest (RF) models on microbial sequencing data. Users may also be interested in [MetAML](http://segatalab.cibio.unitn.it/tools/metaml/), which implements RF along with other machine learning techniques with a simple workflow for metagenomic data. Importantly, the below commands are **not the best practices for all datasets**. An important part of data analysis is determining what approaches would be best for your particular problem. However, as shall be described below, it's easy to create biased models if you select parameters based on observations you made while exploring your data.   
    
Often when running machine learning tools you would split up your samples into a training set (~80%) to build your model and a test set (~20%) to evaluate your model. Partitioning allows an estimate of the model performance to be inferred on different data than was used to train the model. When sample sizes are large enough researchers sometimes create a third partition, the validation set, which is used to select (or _tune_) parameters for the model. However, often in microbial datasets sample sizes are small (e.g. often in the range of 10-50 samples), in which case alternative approaches need to be taken. Cross-validation, where different subsets of your data are systematically split into training and test sets to estimate model performance, is a family of approaches to get around this problem (a few examples are shown in this [blog post](http://appliedpredictivemodeling.com/blog/2014/11/27/vpuig01pqbklmi72b8lcl3ij5hj2qm)). Often this actually isn't necessary for RF models given the internal measure of out-of-bag error which is calculated for each model (described below). 
  
The below tutorial will focus on a dataset of 40 samples, where partitioning the data into training and test sets would not be helpful. [Operational taxonomic unit (OTU)](https://en.wikipedia.org/wiki/Operational_taxonomic_unit) relative abundances are the features of this dataset. All samples have inflammation scores (IS) associated with them (ranging from 0-1). Samples with IS >= 0.5 were grouped as _inflamed_, while those with a value < 0.5 were put in the _control_ group. This means that each sample has an inflammation category and a quantitative value associated with it, which will allow us to try running RF classification and regression models using the same features. Note that this example dataset was simulated for this tutorial.    

### Requirements
* Example dataset  
* [R](https://www.r-project.org/) (this tutorial was run with version 3.3.2)  
* [RStudio](https://www.rstudio.com/) (recommended if running R commands yourself)  
* [randomForest R package](https://cran.r-project.org/web/packages/randomForest/index.html)  
* **Several other R packages described below**
  
### Other Resources
* Original RF paper: [Leo Breiman. 2001. Random Forests. Machine Learning 45(1):5-32.](https://link.springer.com/article/10.1023%2FA%3A1010933404324)
* Paper describing the randomForest R package (includes a great description of the algorithm and helpful examples): [Andy Liaw and Matthew Wiener. 2002. Classification and Regression by randomForest. R News 2(3):18-22.](http://www.bios.unc.edu/~dzeng/BIOS740/randomforest.pdf)
* [scikit-learn](http://scikit-learn.org/stable/): a set of useful tools and workflows for running machine learning in Python.
* [MetAML](http://segatalab.cibio.unitn.it/tools/metaml/): a Python package based on scikit-learn produced by the Segata Lab from Trento University for running machine learning on metagenomics datasets. They have written a [tutorial](https://bitbucket.org/CibioCM/metaml/wiki/Home) and [research paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004977) for this tool.  
* [caret](https://topepo.github.io/caret/): an R package that wraps many machine learning and other predictive tools. There are many online tutorials on how to use caret to run RF models, such as this [short example](http://bigcomputing.blogspot.ca/2014/10/an-example-of-using-random-forest-in.html). The coursera course [Practical Machine Learning](https://www.coursera.org/learn/practical-machine-learning) focuses on this package.

## Background

RF models are run when researchers are interested in classifying samples into categories (called **classes**) of interest based upon many different variables, such as taxa, functional categories, etc (called **features**). RF can also be used to regress features against quantitative data (e.g. height in cm rather than splitting samples into categorical groups like _short_ and _tall_).  

Sometimes RF is run to perform feature selection on a dataset. This can be useful when there are thousands of features and you'd like to reduce the number to a less complex subset. However, it's important to realize that you need to validate selected features on independent data. [Click here to see a simple demonstration for why this is important.](https://github.com/mlangill/microbiome_helper/wiki/Random-Forest-Feature-Selection-Caution)  
        
### Classification and Regression Tree Methods
  
### Out-of-bag Error  
  
### Variable Importance   

### Proximities  
 
## Load Packages and Read in Data    

You'll need to load R packages for this tutorial (which require a number of dependencies as well):
  
    library("randomForest")
    library("plyr") # for the "arrange" function
    library("rfUtilities") # to test model significance
    library("caret") # to get leave-one-out cross-validation accuracies and also contains the nearZeroVar function 
  
Often setting your working environment simplifies working in R:
  
    setwd("/Path/to/my/RF_tutorial/")

After downloading the tutorial OTU table and map file from the above links you can read them into R:  
  
    otu_table <- read.table("otu_table_RF_tutorial.txt", sep="\t", header=T, row.names=1, stringsAsFactors=FALSE, comment.char="")  
    metadata <- read.table("metadata_RF_tutorial.txt", sep="\t", header=T, row.names=1, stringsAsFactors=TRUE, comment.char="")
   
Now that we have the data read in R it's a good idea to explore the data a bit. A couple of basic things that we can do is double-check that the object has the expected number of rows and columns ("dim" returns the number of rows first followed by the number of columns):
    
    dim(otu_table)
    [1] 1000   40
    
    dim(metadata)
    [1] 40  2
    
So there are 40 samples in each file, 1000 rows in the OTU table (each corresponding to a different OTU), and 2 columns of metadata.  
  
It's also a good idea to look at summaries of your data. This is a little more difficult for the OTU table since there are so many columns, but it's easy for the metadata file (using the "str" and "summary" functions):  
  
    str(metadata)
    'data.frame':	40 obs. of  2 variables:
    $ IS   : num  0.76 0.71 0.86 0.72 0.53 0.96 0.97 0.85 0.66 0.55 ...
    $ state: Factor w/ 2 levels "control","inflamed": 2 2 2 2 2 2 2 2 2 2 ...

    summary(metadata)
    IS                   state   
    Min.   :0.0100       control :20  
    1st Qu.:0.2175       inflamed:20  
    Median :0.4950                
    Mean   :0.5012                
    3rd Qu.:0.7625                
    Max.   :0.9700     
  
These steps are important to make sure you're working with the right data and that it's in the format you're expecting. However, it's also important to avoid over-exploring your data so that you don't bias your analyses.  
  
## Pre-processing
* One of the most important steps in machine learning
* Often the difference between the best performing model isn't the choice of algorithm, but simply how the data is pre-processed
* Possible to transform the data using a dimension reduction technique like principal component analysis, but the downside is that it's much harder to interpret
* RF doesn't make any assumptions about how the data is distributed so it often is not necessary to transform your data
* However, reducing noise in your input data will improve model performance. An easy way to do this is to throw out features that are rare or have very low variance across samples. Any cut-offs used at this step are slightly arbitrary and would depend on the dataset.
    
### Removing Rare Features  
  
A basic way to decide which features are unlikely to be informative is if they are non-zero in only a small number of samples. It's a good idea to take a look at this distribution for all features to help choose a reasonable cut-off.  
  
To quickly figure out and plot how many non-zero values each OTU has:  
  
    otu_nonzero_counts <- apply(otu_table, 1, function(y) sum(length(which(y > 0))))
    hist(otu_nonzero_counts, breaks=100, col="grey", main="", ylab="Number of OTUs", xlab="Number of Non-Zero Values")
  
**HISTOGRAM GOES HERE**
  
This histogram shows a typical distributions for OTU tables: most OTUs are rare and found in only a few samples. These OTUs are less likely to help our model so we can throw them out. Typically researchers discard OTUs that are zero in greater than 75-90% of samples although these cut-offs are somewhat arbitrary. This cut-off could be optimized if you had an independent dataset or partition of your data that you could use as a validation set.  

This R function removes features that are non-zero less than a specified proportion of the time:  
  
    remove_rare <- function( table , cutoff_pro ) {
      row2keep <- c()
      for ( i in 1:nrow(table) ) {
        row_nonzero <- length( which( table[ i , ]  > 0 ) ) 
        cutoff <- ceiling( cutoff_pro * ncol(table) )
        if ( row_nonzero > cutoff ) {
          row2keep <- c( row2keep , i)
        }
      }
      return( table [ row2keep , , drop=F ])
    }
  
Based on the above histogram I removed OTUs that have non-zero values in <= 20% of samples:  
  
    otu_table_rare_removed <- remove_rare(table=otu_table, cutoff_pro=0.2)
  
How many OTUs are left?  
  
    dim(otu_table_rare_removed)
    [1] 647  40

647 OTUs are remaining, which means that 35% of the OTUs were excluded.  
  
We might also be interested in excluding OTUs that don't vary much across samples. This usually isn't different from excluding OTUs based on their number of non-zero values. However, this can be useful for other types of feature tables. You may be interested in trying out the _nearZeroVar()_ function from the caret R package to perform this filtering.    
  
### Transforming Your Data
  
  

## Assessing Model Fit

### Permutation Test

### Receiver Operating Characteristic Curve   
  
## Identifying Important Features  
