# Please note that this page is currently being written.
* [Introduction](#introduction)  
    * [Requirements](#requirements)  
    * [General Background](#background)  
    * [Other Resources](#other-resources)  
* [Important Considerations](#important-considerations)  
* [Pre-processing](#pre-processing)  
    * [Removing Rare Features](#removing-rare-features)  
    * [Transforming Your Data](#transforming-your-data)   
* [Assessing Model Fit](#assessing-model-fit)  
    * [Permutation Test](#permutation-test)  
    * [Receiver Operating Characteristic Curve](#receiver-operating-characteristic-curve)   
* [Identifying Important Features](#identifying-important-features)  
  
## Introduction
This tutorial is intended to teach beginners the basics of running random forest (RF) models on microbial sequencing data. Importantly, the below commands are **not the optimal settings to use for all datasets**. An important part of data analysis is exploring your data and deciding on what approaches would be best for your particular problem. However, as shall be described below, parameterizing models based upon your training and/or test set can result in biased models.  
  
Typically RF models are run when researchers are interested in classifying samples into categories (called **classes**) of interest based upon many different variables, such as taxa, functional categories, etc (called **features**). Random forest can also be run on quantitative data (e.g. height in cm rather than splitting samples into categorical groups like _short_ and _tall_).  
    
Ideally when running machine learning tools you would split up your samples into a training set (~80%) to build your model and a test set (~20%) to evaluate your model. This allows   

The below tutorial will consider two scenarios,
**Author**: Gavin Douglas
    
**First Created**: March 2017
  
**Last Edited**: March 2017
  
### Requirements
  
### Background
  
### Other Resources
* Original random forest paper: [Leo Breiman. 2001. Random Forests. Machine Learning 45(1):5-32.](https://link.springer.com/article/10.1023%2FA%3A1010933404324)
* Paper describing the randomForest R package (includes a great description of the algorithm and helpful examples): [Andy Liaw and Matthew Wiener. 2002. Classification and Regression by randomForest. R News 2(3):18-22.](http://www.bios.unc.edu/~dzeng/BIOS740/randomforest.pdf)
* [scikit-learn](http://scikit-learn.org/stable/): a set of useful tools and workflows for running machine learning in Python.
* [MetAML](http://segatalab.cibio.unitn.it/tools/metaml/): a Python package based on scikit-learn produced by the Segata Lab from Trento University for running machine learning on metagenomics datasets. They have written a [tutorial](https://bitbucket.org/CibioCM/metaml/wiki/Home) and [research paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004977) for this tool.  
* [caret](https://topepo.github.io/caret/): an R package that wraps many machine learning and other predictive tools. There are many online tutorials on how to use caret to run RF models, such as this [short example](http://bigcomputing.blogspot.ca/2014/10/an-example-of-using-random-forest-in.html). The coursera course [Practical Machine Learning](https://www.coursera.org/learn/practical-machine-learning) focuses on this package.

## Important Considerations
  
## Pre-processing
  
### Removing Rare Features
  
### Transforming Your Data
  
## Assessing Model Fit

### Permutation Test

### Receiver Operating Characteristic Curve   
  
## Identifying Important Features  
