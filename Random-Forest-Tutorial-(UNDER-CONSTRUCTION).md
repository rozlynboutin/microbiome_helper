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
This tutorial is intended to teach beginners the basics of running random forest (RF) models on microbial sequencing data. Importantly, the below commands are **not the optimal settings to use for all datasets**. An important part of data analysis is exploring your data and deciding on what approaches would be best for your particular problem. However, as shall be described below, parameterizing models based upon your training and/or test set can resulting in biased models.  
  
Typically RF models are run when researchers are interested in classifying samples into categories (called **classes**) of interest based upon many different variables, such as taxa, functional categories, etc (called **features**). Random forest can also be run on quantitative data (e.g. height in cm rather than splitting samples into categorical groups like _short_ and _tall_).  
    
**Author**: Gavin Douglas
    
**First Created**: March 2017
  
**Last Edited**: March 2017
  
### Requirements
  
### Background
  
### Other Resources
* [caret](https://topepo.github.io/caret/) - an R package that wraps many machine learning and other predictive tools. There are many online tutorials on how to use caret to run RF models, such as this [short example](http://bigcomputing.blogspot.ca/2014/10/an-example-of-using-random-forest-in.html). The coursera course [Practical Machine Learning](https://www.coursera.org/learn/practical-machine-learning) focuses on this package.
* [scikit-learn](http://scikit-learn.org/stable/) - a set of useful tools and workflows for running machine learning in Python.
* [MetAML](http://segatalab.cibio.unitn.it/tools/metaml/) - a Python package based on scikit-learn produced by the Segata Lab from Trento University for running machine learning on metagenomics datasets. They have written a [tutorial](https://bitbucket.org/CibioCM/metaml/wiki/Home) and [research paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004977) for this tool.  

## Important Considerations
  
## Pre-processing
  
### Removing Rare Features
  
### Transforming Your Data
  
## Assessing Model Fit

### Permutation Test

### Receiver Operating Characteristic Curve   
  
## Identifying Important Features  
