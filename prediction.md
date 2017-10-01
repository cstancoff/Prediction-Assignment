Introduction
------------

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement – a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, your goal
will be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different ways. More information is
available from the website here:
<http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har>
(see the section on the Weight Lifting Exercise Dataset).

Six young health participants were asked to perform one set of 10
repetitions of the Unilateral Dumbbell Biceps Curl in five different
fashions:

-   A - exactly according to the specification
-   B - throwing the elbows to the front
-   C - lifting the dumbbell only halfway
-   D - lowering the dumbbell only halfway
-   E - throwing the hips to the front

Data Import and Exploratory Analysis
------------------------------------

Read the csv file containing the data into R.

    # Read the data
    # The data for this project come from this source:
    # http://groupware.les.inf.puc-rio.br/har. If you use the document you create for
    # this class for any purpose please cite them as they have been very generous in
    # allowing their data to be used for this kind of assignment.

    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    download.file(fileUrl, destfile="../pml-training.csv", method="curl")
    data <- tbl_df(read.csv("../pml-training.csv"))

    dim(data)

    [1] 19622   160

Read the data for the 20 question quiz.

    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    download.file(fileUrl, destfile="../pml-testing.csv", method="curl")
    quiz <- tbl_df(read.csv("../pml-testing.csv"))

Data Selection
--------------

Many of the columns have no data in them and some of them are irrelevant
to predict how the bicep curl was performed, so I reduced the data to
only columns that I think will be useful for prediction. Because they
give information on orientation I selected the roll, pitch, and yaw for
all 4 sensors.

    data_subset <- data[c("roll_belt","pitch_belt","yaw_belt",
                           "roll_arm","pitch_arm","yaw_arm",
                           "roll_dumbbell","pitch_dumbbell","yaw_dumbbell",
                           "roll_forearm","pitch_forearm","yaw_forearm",
                           "classe")]

After reducing the data the next step was to break the data into
training and testing sets.

    # Parition the data into training and testing sets
    inTrain <- createDataPartition(y=data_subset$classe,p=0.60,list=FALSE)
    training <- data_subset[inTrain,]
    testing <- data_subset[-inTrain,]

Predicting with Trees
---------------------

I first tried predicting with trees.

    # Train the model
    fit_rpart <- train(classe~., data=training, method="rpart")
    fit_rpart

    CART 

    11776 samples
       12 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 

    No pre-processing
    Resampling: Bootstrapped (25 reps) 
    Summary of sample sizes: 11776, 11776, 11776, 11776, 11776, 11776, ... 
    Resampling results across tuning parameters:

      cp          Accuracy   Kappa     
      0.04621500  0.4143055  0.21467610
      0.04912197  0.4034891  0.19554297
      0.11497390  0.3412765  0.08817957

    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was cp = 0.046215.

    # Perform the prediction
    pred <- predict(fit_rpart, testing)
    cm_rpart <- confusionMatrix(testing$classe,pred)
    cm_rpart

    Confusion Matrix and Statistics

              Reference
    Prediction    A    B    C    D    E
             A 2060    0  167    0    5
             B 1035    0  483    0    0
             C  658    0  710    0    0
             D  775    0  511    0    0
             E  304    0  485    0  653

    Overall Statistics
                                              
                   Accuracy : 0.4363          
                     95% CI : (0.4253, 0.4473)
        No Information Rate : 0.6159          
        P-Value [Acc > NIR] : 1               
                                              
                      Kappa : 0.2553          
     Mcnemar's Test P-Value : NA              

    Statistics by Class:

                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.4263       NA  0.30136       NA  0.99240
    Specificity            0.9429   0.8065  0.88015   0.8361  0.89023
    Pos Pred Value         0.9229       NA  0.51901       NA  0.45284
    Neg Pred Value         0.5062       NA  0.74591       NA  0.99922
    Prevalence             0.6159   0.0000  0.30028   0.0000  0.08386
    Detection Rate         0.2626   0.0000  0.09049   0.0000  0.08323
    Detection Prevalence   0.2845   0.1935  0.17436   0.1639  0.18379
    Balanced Accuracy      0.6846       NA  0.59075       NA  0.94132

Because the accuracy of this method was 43.627326% I decided to try
another method.

Predicting with Random Forests
------------------------------

The next method I tried was to predict using random forests. I used 3
10-fold cross-validations to re-sample the data.

    # Train the model
    fit_rf <- train(classe~., data=training,
                    method="rf",trControl=trainControl(method="cv"), number=3)
    fit_rf

    Random Forest 

    11776 samples
       12 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 

    No pre-processing
    Resampling: Cross-Validated (10 fold) 
    Summary of sample sizes: 10598, 10599, 10598, 10598, 10599, 10599, ... 
    Resampling results across tuning parameters:

      mtry  Accuracy   Kappa    
       2    0.9829306  0.9784089
       7    0.9838651  0.9795931
      12    0.9795345  0.9741178

    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 7.

    # Perform the prediction
    pred_rf <- predict(fit_rf, testing)
    cm_rf <- confusionMatrix(testing$classe,pred_rf)
    cm_rf

    Confusion Matrix and Statistics

              Reference
    Prediction    A    B    C    D    E
             A 2223    7    0    2    0
             B    9 1482   21    5    1
             C    0    7 1347   13    1
             D    2    1   10 1271    2
             E    0    2    6    7 1427

    Overall Statistics
                                              
                   Accuracy : 0.9878          
                     95% CI : (0.9851, 0.9901)
        No Information Rate : 0.2847          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9845          
     Mcnemar's Test P-Value : NA              

    Statistics by Class:

                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9951   0.9887   0.9733   0.9792   0.9972
    Specificity            0.9984   0.9943   0.9968   0.9977   0.9977
    Pos Pred Value         0.9960   0.9763   0.9846   0.9883   0.9896
    Neg Pred Value         0.9980   0.9973   0.9943   0.9959   0.9994
    Prevalence             0.2847   0.1911   0.1764   0.1654   0.1824
    Detection Rate         0.2833   0.1889   0.1717   0.1620   0.1819
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9967   0.9915   0.9850   0.9885   0.9974

This model has an accuracy of 98.7764466%.

Conclusion
----------

The random forest method created the best model. It has high accuracy of
98.7764466% and therefore a low out of sample error of 1.2235534%
