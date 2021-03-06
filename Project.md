# Project



## The goal of this project is to use data collected from devices such as Jawbone Up, Nike FuelBand, and Fitbit to predict whether the user was performing the exercise correctly. 

```r
suppressMessages(suppressWarnings(library(tidyverse)))
suppressMessages(suppressWarnings(library(caret)))
suppressMessages(suppressWarnings(library(randomForest)))
suppressMessages(suppressWarnings(library(gbm)))
download.file(url = 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv', destfile = 'traindata.csv')
download.file(url = 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv', destfile = 'testdata.csv')
traindata <- read.csv('traindata.csv')
testdata <- read.csv('testdata.csv')
```


## This section deals with cleaning the data. Many of the original columns in traindata have a large majority of the values equal to NA. Others have a large majority equal to an empty string. As these columns are almost devoid of all information, I am removing them from use in fitting the model.

```r
responsevariable <- 'classe'
napercvector <- c()
blankvector <- c()
for(i in 1:ncol(traindata))
{
  naperc <- sum(is.na(traindata[,i]))/nrow(traindata)
  blanks <- sum(traindata[,i] == '')/nrow(traindata)
  if(is.na(blanks)) blanks <- 0
  napercvector <- c(napercvector, naperc)
  blankvector <- c(blankvector,blanks)
}
napercbycol <- data.frame(Column = names(traindata), napercvector, blankvector)
columnstokeep <- napercbycol$Column[which(napercbycol$napercvector < .95 & napercbycol$blankvector < .95)] %>% as.character()
traindata <- traindata[,which(names(traindata) %in% columnstokeep)] %>% select(-X)
testdata <- testdata[,which(names(testdata) %in% columnstokeep)] %>% select(-X)
```


### This section builds a random forest using the caret package and 5 fold cross validation. Cross validation was used to assess the predictive power of the model on holdout datasets. This was repeated 5 times to avoid overfitting.

```r
traincontrol <- caret::trainControl(method = 'cv', number = 5, savePredictions = 'final')
rfmodel <- caret::train(classe~., data = traindata, method = 'rf', trControl = traincontrol, tuneLength = 1)
preds <- caret::predict.train(rfmodel, newdata = testdata)
```

