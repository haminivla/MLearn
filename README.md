---
title: "Practical Machine Learning Project"
author: "Alvin"
date: "Wednesday, March 04, 2015"
output: html_document
---
## 1. Introduction  
The emergence of Human Activity Recognition (HAR) devices has encouraged development of applications that could measure the quality of executing an activity. This is particularly useful for measuring the correct posture in some sports related activities in order to prevent injuries.  
  
This project focus on using data collected from HAR devices to predict if an activity (dumbbell lifting) is performed in an expected manner.     
    
For a more comprehensive report with figures in HTML format, please refer to this [link]()  
  
## 2. Setup   
Six human subjects were asked to perform dumbbell lifting in five different ways.  
A. Proper and correct execution of dumbbell lifting  
B. Throwing the elbows to the front  
C. Lifting the dumbbell only halfway  
D. Lowering the dumbbell only halfway  
E. Throwing the hips to the front  
  
Four HAR devices were attached to sense the movement of the subjects and dumbbell in the following locations:  
- A gyro sensor on the Dumbbell  
- A glove sensor worn around the Wrist  
- An arm-band sensor worn around the Arm  
- A belt sensor worn around Waist  
  
The subjects performed one set of 10 repetitions for each of the five different ways mentioned above and the data is captured in a CSV.  
  
## 3. Exploratory Data Analysis
The dataset contains **19622** observations and **160** variables. 
  
```
pmltrain <- read.csv("pml-training.csv", header=T)
```
  
### 3.1 Filtering
Studying the data, there are some variables which have no value or NAs on most of the observations because these variables contain the max, var, stddev, min, etc. for a set of repetitions. Since the amount of data collected in these variables are minor compared to the entire dataset, let's filter off these variables. In addition, the first seven columns (id, names, timestamps, repetition window indicators) also does not contain any interesting data relevant for analysis and will be filtered.  
  
```
x <- as.numeric()  
for(i in 1:ncol(pmltrain)) { if(pmltrain[1,i]=="" | is.na(pmltrain[1,i])) x <- c(x, i) }  
pmltrain <- pmltrain[-x]; pmltrain <- pmltrain[-c(1:7)]  
```
  
## 4. Data Slicing  
The dataset will be split into 60% (training) 40% (testing) partitions. The prediction model is trained using the training partition to achieve a satisfactory result before applying to the testing partition. The seed is set so that the same results can be reproduced.   
  
```
set.seed(7)  
inTrain <- createDataPartition(y=pmltrain$classe, p=0.6, list=F)  
training <- pmltrain[inTrain,]  
testing <- pmltrain[-inTrain,]  
```
  
## 5. Model Fitting  
After comparing a few prediction models, the model picked is Random Forest algorithm as this is the most accurate model. To improve performance of the model processing, the data is preprocessed within caret 'train' function using Principal Components Analysis (PCA). **10 fold cross validation** is performed within caret 'train' function to improve the accuracy.  
  
```
library(caret)  
set.seed(7)  
modelFit <- train(classe ~ ., data=training, preProcess=c("pca"), trControl=trainControl(method="cv"), method="rf")  
```
  
### 5.1 Out of Sample Error
Based on the results, the optimal model was selected based on Accuracy. This optimal model uses mtry of 2 and yields an accuracy of **96.77%** sampling on the training data.  
  
The model is then applied to the testing data to predict the 'classe' results. The accurary for **Out of Sample Error** is **97.46%**. This is better than our training data and proves that there is no overfitting.  
  
```
## Using the model to predict on testing data excluding the 'classe' column  
ptest <- predict(modelFit, testing[,-53])  
confusionMatrix(ptest, testing$classe)  
```  
  
## 6. Final Prediction - Validation  
The validation dataset which contains 20 test cases is loaded from the CSV.  
The same process of data filtering and tidying must be performed on the validation data set before predicting the results.    
  
```
## Read validation data set from file
validation <- read.csv("pml-testing.csv", header=T)  
  
## Perform data filtering and tidying as per training/testing data set
x <- as.numeric()  
for(i in 1:ncol(validation)) { if(validation[1,i]=="" | is.na(validation[1,i])) x <- c(x, i) }  
validation <- validation[-x]; validation <- validation[-c(1:7)]  
  
## Predict on validation data set excluding the last column which is problem_id
pred <- predict(modelFit, validation[,-53])  
```
  
***
Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. Qualitative Activity Recognition of Weight Lifting Exercises. Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human '13) . Stuttgart, Germany: ACM SIGCHI, 2013.  
  
[Read more](http://groupware.les.inf.puc-rio.br/har#ixzz3Tf8PCS1m)