## Practical Machine Learning - Classifying Exercise

## Background (from Coursera)
The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases.

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

## Project Setup
```{r setup, warning=FALSE, message=FALSE}
library(knitr)
library(caret)
library(randomForest)
library(gbm)
library(gam)
```

## Loading Data

```{r results='hide'}
testing_df <- read.table("pml-testing.csv")
training_df <- read.csv("pml-training.csv")
dim(training_df)
dim(testing_df)
str(training_df)
```

## Cleaning Data

Upon initial investigation it appears that there are several incomplete values in many of the columns. It is important that we set all incomplete data values to na values using na.strings when loading the data. This will make cleaning the data a much smoother process.

```{r}
training_df <- read.csv("pml-training.csv", na.strings = c("", "NA", "#DIV/0!"))
testing_df <- read.csv("pml-testing.csv", na.strings = c("", "NA", "#DIV/0!"))
```
Now that all incomplete data points are set to NA we need to clean the data.
```{r results='hide'}
training_clean_na <- training_df[, colSums(is.na(training_df)) == 0]
testing_clean_na <- testing_df[, colSums(is.na(training_df)) == 0]
str(training_clean_na)
```
Next we cut out the remaining unnecessary variables.
```{r results='hide'}
near_zero_var <- nearZeroVar(training_clean_na, saveMetrics = "TRUE")
near_zero_var <- nearZeroVar(testing_clean_na, saveMetrics = "TRUE")

training_clean_nzv <- training_clean_na[,near_zero_var$nzv == FALSE]
testing_clean_nzv <-  testing_clean_na[,near_zero_var$nzv == FALSE]

training_clean_final <- training_clean_nzv[, -c(1:6)]
testing_clean_final <- testing_clean_nzv[, -c(1:6,59)]
```

## Setting up the training, testing, vaildation data

```{r results='hide'}
trainingpartion <- createDataPartition(training_clean_final$classe, p = .6, list = FALSE)
train <- training_clean_final[trainingpartion, ]
train_valid <- training_clean_final[-trainingpartion, ]
```

## Fitting Models
Below my process was to fit two tree methods Random Forest (rf) and Boosting(gbm). From there I wanted to see if I could get the best of both features and combine preditors using the gam package. The final results show that the random forest performed the best out of all three models. We will use this to predict the classe in the test set.

```{r results='hide', message=FALSE, warning=FALSE}
fit_cntrl <- trainControl(method = "cv", number = 5, verboseIter = TRUE)

fit_1 <- train(classe~., data = train, method = "rf", prox = TRUE, trControl = fit_cntrl)

fit_2 <- train(classe~., data = train, method = "gbm", trControl = fit_cntrl, verbose = FALSE)

prediction_train_test_1 <- predict(fit_1, train_valid)
prediction_train_test_2 <- predict(fit_2, train_valid)

combodf <- data.frame(prediction_train_test_1, prediction_train_test_2, classe = train_valid$classe)

fit_3 <- train(classe~., method = "gam", data = combodf)
prediction_train_test_3 <- predict(fit_3, train_valid)
```

```{r message=FALSE, warning=FALSE}
## Figure 1 - Random Forest Results
confusionMatrix(train_valid$classe, prediction_train_test_1)
## Figure 2 - Boosting Results
confusionMatrix(train_valid$classe, prediction_train_test_2)
## Figure 3 - Combined Results
confusionMatrix(train_valid$classe, prediction_train_test_3)
```

## Final Prediction
Now we use our model to predict the classe from the testing set.

```{r}
testing_predict <- predict(fit_1, testing_clean_final)
testing_predict
```
