Practical Machine Learning - Pediction Assignment Writeup
---------------------------------------------------------

Background:

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement - a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, your goal
will be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different ways.

Install packages for the project
--------------------------------

    library(tidyverse)

    ## -- Attaching packages ---------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 2.2.1     v purrr   0.2.4
    ## v tibble  1.4.2     v dplyr   0.7.4
    ## v tidyr   0.8.0     v stringr 1.3.1
    ## v readr   1.1.1     v forcats 0.3.0

    ## -- Conflicts ------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    library(caret)

    ## Loading required package: lattice

    ## 
    ## Attaching package: 'caret'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     lift

    library(randomForest)

    ## randomForest 4.6-14

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

Load Data (pml-training.csv and pml-testing.csv)
------------------------------------------------

The data for this project come from this source:
<http://groupware.les.inf.puc-rio.br/har>

    url.trainData <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    url.testData  <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    training      <- read.csv(url(url.trainData), na.strings = c("NA", "", "#DIV0!"))
    testing       <- read.csv(url(url.testData), na.strings = c("NA", "", "#DIV0!"))
    set.seed(23)

Cleansing the data
------------------

    train.noNA <- training[colSums(!is.na(training))>(nrow(training)*.9)]
    test       <- testing[colSums(!is.na(testing))>(nrow(testing)*.9)]

Remove the first 7 columns because they are not needed for calculation.
-----------------------------------------------------------------------

    train.cleansese <- train.noNA[,-c(1:7)]

Removing highly correlated predictors. Get only numeric columns
---------------------------------------------------------------

    num.cols <-sapply(train.cleansese,is.numeric)

Create a correlation matrix
---------------------------

    cor.data <- cor(train.cleansese[,num.cols])

Identify correlated predictors for removal
------------------------------------------

    high.cor <- findCorrelation(cor.data, cutoff = .75)
    train.cor <- train.cleansese[,-high.cor]
    train.cor$classe <- as.factor(train.cor$classe)

Split the data into training (75%) and testing (25%) datasets to be used for Cross Validation
---------------------------------------------------------------------------------------------

    inTrain <- createDataPartition(y=train.cor$classe,p=0.75,list=FALSE)
    training <- train.cor[inTrain,]
    testing <- train.cor[-inTrain,]

Plot the training dataset
-------------------------

    qplot(classe, data=training,  main = "Activity Classes (Training Data)")

![](Prediction_Assignment_Writeup_files/figure-markdown_strict/unnamed-chunk-9-1.png)

Train the model using RandomForest
----------------------------------

    modelfit.rf <- randomForest(classe ~.,training)
    pred.rf <- predict(modelfit.rf,testing)

View the Confusion Matrix
-------------------------

Based on the Confusion Matrix summary, Sensitivity is .9993 and
Specificity is .9991 the balance accuracy is .9992

    cmatrix <- confusionMatrix(pred.rf,testing$classe)
    print(cmatrix)

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1394    3    0    0    0
    ##          B    1  943    7    0    0
    ##          C    0    2  841   17    0
    ##          D    0    0    7  786    0
    ##          E    0    1    0    1  901
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.992           
    ##                  95% CI : (0.9891, 0.9943)
    ##     No Information Rate : 0.2845          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.9899          
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9993   0.9937   0.9836   0.9776   1.0000
    ## Specificity            0.9991   0.9980   0.9953   0.9983   0.9995
    ## Pos Pred Value         0.9979   0.9916   0.9779   0.9912   0.9978
    ## Neg Pred Value         0.9997   0.9985   0.9965   0.9956   1.0000
    ## Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
    ## Detection Rate         0.2843   0.1923   0.1715   0.1603   0.1837
    ## Detection Prevalence   0.2849   0.1939   0.1754   0.1617   0.1841
    ## Balanced Accuracy      0.9992   0.9958   0.9895   0.9880   0.9998

Plot model fit
--------------

    plot(modelfit.rf)

![](Prediction_Assignment_Writeup_files/figure-markdown_strict/unnamed-chunk-12-1.png)
\#\# Final prediction using the test dataset (pml-testing.csv)

    final.pred <- predict(modelfit.rf, test, type = "class")
    print(final.pred)

    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
    ##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
    ## Levels: A B C D E

Plot the Final predictiion
--------------------------

    qplot(final.pred, main = "Activity Classes (Test Data)")

![](Prediction_Assignment_Writeup_files/figure-markdown_strict/unnamed-chunk-14-1.png)
