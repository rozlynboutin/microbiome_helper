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

### Other Resources
* Original RF paper: [Leo Breiman. 2001. Random Forests. Machine Learning 45(1):5-32.](https://link.springer.com/article/10.1023%2FA%3A1010933404324)
* Paper describing the randomForest R package (includes a great description of the algorithm and helpful examples): [Andy Liaw and Matthew Wiener. 2002. Classification and Regression by randomForest. R News 2(3):18-22.](http://www.bios.unc.edu/~dzeng/BIOS740/randomforest.pdf)
* [scikit-learn](http://scikit-learn.org/stable/): a set of useful tools and workflows for running machine learning in Python.
* [MetAML](http://segatalab.cibio.unitn.it/tools/metaml/): a Python package based on scikit-learn produced by the Segata Lab from Trento University for running machine learning on metagenomics datasets. They have written a [tutorial](https://bitbucket.org/CibioCM/metaml/wiki/Home) and [research paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004977) for this tool.  
* [caret](https://topepo.github.io/caret/): an R package that wraps many machine learning and other predictive tools. There are many online tutorials on how to use caret to run RF models, such as this [short example](http://bigcomputing.blogspot.ca/2014/10/an-example-of-using-random-forest-in.html). The coursera course [Practical Machine Learning](https://www.coursera.org/learn/practical-machine-learning) focuses on this package.

## Background

RF models are run when researchers are interested in classifying samples into categories (called **classes**) of interest based upon many different variables, such as taxa, functional categories, etc (called **features**). RF can also be run on quantitative data (e.g. height in cm rather than splitting samples into categorical groups like _short_ and _tall_).  

Sometimes RF is run to perform feature selection on a dataset. This can be useful when there are thousands of features and you'd like to reduce the number to a less complex subset. However, it's important to realize that you need to validate selected features on independent data. [Click here to see a simple demonstration for why this is important.](https://github.com/mlangill/microbiome_helper/wiki/Random-Forest-Feature-Selection-Caution)  
        
### Classification and Regression Tree Methods
  
### Out-of-bag Error  
  
### Variable Importance   

### Proximities  

## Pre-processing
  
### Removing Rare Features
  
### Transforming Your Data
  
## Assessing Model Fit

### Permutation Test

### Receiver Operating Characteristic Curve   
  
## Identifying Important Features  
