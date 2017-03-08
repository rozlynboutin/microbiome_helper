Random Forest is often used to select a subset of features that are the best classifiers. This application is especially helpful for large datasets with thousands of features since it helps reduce the noise, complexity and running time of subsequent analyses. It's important to realize that you will get a ranked list of features returned even if the model is totally uninformative. You need to validate those features on an independent dataset (or independent partition of your data) to be be confident in them. Otherwise you might simply be modelling noise in your training data. I demonstrate this simple point below using the [randomForest R package](https://cran.r-project.org/web/packages/randomForest/index.html).
  
The below commands were run in RStudio using R v3.3.2. You will need the randomForest and plyr R packages if you want to run these commands yourself.

Load required packages:
  
    library("randomForest")
    library("plyr") # for "arrange" function

Set seed so that the analysis is reproducible:
  
    set.seed(441)

To simulate a toy example OTU table I started out by making a table filled with 0s with 40 rows (samples) and 1000 columns (features):

    random_table <- data.frame( matrix(0, nrow=40 , ncol=1000) )
    
For each sample I generated 1000 values from a Poisson distribution with lambda=0.3. This a crude way to get a random present/absence pattern across the table. You shouldn't use these commands if you want to simulate realistic OTU tables.
   
    for(r in 1:nrow(random_table)) {
        random_table[r,] <- rpois(1000, lambda=0.3)*sample(x=2:1000, size=1000, replace=TRUE)
    }
  
Transform into relative abundance so that each row sums to 100:  
  
    random_table_rel <- sweep(random_table, 1, rowSums(random_table), '/')*100  
  
Add a column called "class" which is the samples randomly split into group "A" and "B":  
  
    random_table_rel$class <- NA  
    groupA <- sample(1:40, size = 20, replace = FALSE)  
    random_table_rel[groupA , "class"] <- "A"  
    random_table_rel[-groupA , "class"] <- "B"  
    random_table_rel$class <- as.factor(random_table_rel$class)  
    
Generate a random forest model that uses all the features to predict the "class" (which is the last column in the dataframe):
    
    toy_example_RF_mod <- randomForest( x=random_table_rel[,1:(ncol(random_table_rel)-1)] , y=random_table_rel[ , ncol(random_table_rel)] , ntree=501, importance = TRUE )

To see the model summary:
  
    toy_example_RF_mod

When I ran this model I got an out-of-bag error of 55%, which is poor. But what does the distribution of variable importance look like?  
  
To sort and plot the features by their variable importance:
  
    toy_example_RF_mod_imp <- as.data.frame( toy_example_RF_mod$importance )
    toy_example_RF_mod_imp$features <- rownames( toy_example_RF_mod_imp )
    toy_example_RF_mod_imp_sorted <- arrange( toy_example_RF_mod_imp  , desc(MeanDecreaseAccuracy)  )
    barplot(toy_example_RF_mod_imp_sorted$MeanDecreaseAccuracy, ylab="Variable Importance")

Based on this distribution it looks like a small subset of features are useful for classification. How well would a random forest model perform based on only the top features?
  
Get the subset of 10 top features:  
  
    random_table_rel_top <- random_table_rel[ , toy_example_RF_mod_imp_sorted[1:10,"features"] ]  
  
Re-run random forest with just this subset:  
  
    toy_example_RF_mod_top <- randomForest( x=random_table_rel_top , y=random_table_rel[ , ncol(random_table_rel)] , ntree=501, importance = TRUE )  
  
    toy_example_RF_mod_top

Which returns an out-of-bag error rate of 15%, which is much better. However, in this case we know that the input features are totally random and **we are overfitting the model and simply fitting noise**. In other words, you can't validate your model using the same data you used for training. For real datasets you would need an independent dataset/partition to validate the selected features.  
      