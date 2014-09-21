














Preprocessing & transformation
========================================================
  
  Following adjustments were made on the raw training data to reduce number of predictors:
  - remove first (order) predictor
- remove cvtd_timestamp-predictor - suspiciously performing and not interpretable
- remove all predictors that are blanks/NA in more than 50% cases (these predictors are blanks/NA in the testing sample, so no information should be lost)

```
for (i in 1:n_col) {
  for (j in 1:n_lin) {
    if (is.na(data_train_raw[j,i])) {
      na_count[i] = na_count[i]+1 }
    else if (data_train_raw[j,i] == "") {
      blanks_count[i] = blanks_count[i]+1}
  }
}
leave_out <- blanks_count+na_count < (n_lin/2)
data_train <- data_train_raw[,leave_out]
```

After these adjustments 57 predictors remain.


Measuring predictors importance
========================================================
  
  On the remaining predictors random tree is estimated to assign importance to these variables.

```
tree_model <- train(data_train$classe ~ ., method="rpart", data = data_train)
```

Estimated tree on the whole training sample has folowing form:
  
  
  ```r
  rpart.plot(tree_model$finalModel)
  ```
  
  ![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-1.png) 



```r
zero_importance <- varImp(tree_model)$importance$Overall==0
important_vars <- row.names(varImp(tree_model)$importance)[(1-zero_importance) == 1]
```


Predictors with Gini importance equal to zero are excluded from the model:
  
  Following 13 predictors remain:
  
  ```r
  names(data_train_2)
  ```
  
  ```
  ##  [1] "raw_timestamp_part_1" "num_window"           "roll_belt"           
  ##  [4] "yaw_belt"             "total_accel_belt"     "accel_belt_z"        
  ##  [7] "magnet_belt_y"        "accel_arm_x"          "magnet_arm_x"        
  ## [10] "magnet_dumbbell_y"    "magnet_dumbbell_z"    "roll_forearm"        
  ## [13] "pitch_forearm"        "classe"
  ```


k-fold cross validation - TREE (k = 10)
========================================================
  
  To estimate the tree model accuracy k-fold cross validation is applied with 10 random disjunctive testing samples from the original train sample.

```
test_sample_index[1,] <-sample(1:dim(data_train_2)[1],size=dim(data_train_2)[1]/10,replace=F)
train_sample_index[1,] <- setdiff(1:dim(data_train_2)[1],test_sample_index[1,])
rest_of_index <- c(1:dim(data_train_2)[1])

for (i in 2:10) {
  rest_of_index <- setdiff(rest_of_index,test_sample_index[i-1,])
  test_sample_index[i,] <-sample(rest_of_index,size=dim(data_train_2)[1]/10,replace=F)
  train_sample_index[i,] <- setdiff(1:dim(data_train_2)[1],test_sample_index[i,])
}
```
On each cross validation sample accuracy is estimated as 
```
accuracy[i]<- sum(prediction==test_set$classe)/dim(test_set)[1]

```
We receive following values:
  
  ```r
  accuracy
  ```
  
  ```
  ##  [1] 0.3751 0.5902 0.5362 0.5240 0.5367 0.4878 0.3649 0.4954 0.5571 0.5688
  ```

Random forests
========================================================
  
  With 13 predictors, random forest can be applied with quite reasonable calculation time.

```
forest_model <- train(data_train_2$classe ~ ., method="rf", data = data_train_2)
```



k-fold cross validation - RANDOM FOREST (k=3)
========================================================
  
  For the Random forest k-fold cross validation, only 3 samples are taken due to larger time demand of the algorithm. The rest is the same as with the tree k-fold cross validation.

Results are considerably beter than the tree. 

```r
accuracy2 
```

```
## [1] 0.5725 0.5734 0.5679
```

Accuracy estimation is:
  
  ```r
  mean(accuracy2)
  ```
  
  ```
  ## [1] 0.5713
  ```

Random forest is applied to the testing sample.

```r
summary(forest_model)
```

```
##                 Length Class      Mode     
## call                4  -none-     call     
## type                1  -none-     character
## predicted       19622  factor     numeric  
## err.rate         3000  -none-     numeric  
## confusion          30  -none-     numeric  
## votes           98110  matrix     numeric  
## oob.times       19622  -none-     numeric  
## classes             5  -none-     character
## importance         13  -none-     numeric  
## importanceSD        0  -none-     NULL     
## localImportance     0  -none-     NULL     
## proximity           0  -none-     NULL     
## ntree               1  -none-     numeric  
## mtry                1  -none-     numeric  
## forest             14  -none-     list     
## y               19622  factor     numeric  
## test                0  -none-     NULL     
## inbag               0  -none-     NULL     
## xNames             13  -none-     character
## problemType         1  -none-     character
## tuneValue           1  data.frame list     
## obsLevels           5  -none-     character
```
