# Practical Machine Learning Course Project

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is possible 
to collect a large amount of data about personal activity relatively inexpensively. 
One thing that people regularly do is quantify how much of a particular activity they do, 
but they rarely quantify how well they do it. The goal of this project is to predict 
the manner in which they did the exercise. In this project is used data from accelerometers on the belt, 
forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and 
incorrectly in 5 different ways.

Loading of data

The training data is here:
```{r}
trainUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
training <- read.csv(url(trainUrl))
```
and the test data is here:

```{r}
testUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv" 
testing <- read.csv(url(testUrl))
```
Examine what kind of data is.

```{r}
dim(training)
dim(testing)
head(training)
head(testing)
```

Cleaning of data

There are a lot of NA values. That kind of columns is removed and 
also the first 7 columns is removed, because they are name and timestamps.
Only columns that are used in testing case plus classe is considered.

```{r}
dataCol <- names(testing[,colSums(is.na(testing)) == 0])[8:59]
training <- training[,c(dataCol,"classe")] 
testing  <- testing[,c(dataCol,"problem_id")]

```
Because there is a lot of rows (19622) and train will be taken so long time, 
is considered only 10% of rows.  
Some libraries have also to be loaded.

```{r}
library(caret)
library(gbm)
library(AppliedPredictiveModeling)
library(randomForest)
library(plyr)
```
```{r} 
set.seed(123)
indT = createDataPartition(training$classe, p = 0.1, list = FALSE)
training = training[ indT,]

```

Cross validation

The trainig set is divided into two data set, 25% of the data for testing.

```{r}
set.seed(456)
inTrain = createDataPartition(training$classe, p = 3/4, list = FALSE)
train = training[ inTrain,]
test  = training[-inTrain,]

```

First using a random forest ("rf") and next boosted trees ("gbm") 

```{r,echo=FALSE}
mod_rf <- train(classe ~ ., data=train, method="rf", verbose = FALSE)
mod_gbm <- train(classe ~ ., data=train, method="gbm", verbose = FALSE)


pred_rf <- predict(mod_rf, test)
pred_gbm <- predict(mod_gbm, test)

```
Accuracy

```{r}
confusionMatrix(pred_rf, test$classe)
confusionMatrix(pred_gbm, test$classe)

```
Because accuracy of "rf" 0.9325 is higher than accuracy of "gbm" 0.9121 , 
the model mod_rf is used for predicting.

submit
```{r}
testingPred <- predict(mod_rf, testing)
testingPred

```