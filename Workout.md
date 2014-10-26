

# &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Analysis of Weight Lifting Excersize

<br>
<br>
### Introduction

The dataset contains measurements from several detectors placed on the arm, forearm and belt of a person involved in a workout session. Several thousand recording have been made during correct and 4 incorrect ways of doing a certain excersize. Based on those instanteneous measurements it is then predicted if the person is doing and excersize properly or, if not, which one of the most common mistakes are being made.

### Covariates analysis

Below is a graph of the most distinctive variables.

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

On top left in can be seen that "gyros_belt_z" is a very useful variable to pick out classes D or E of the exercise, because for others this value is very close to zero. That's to be expected, as your hand is only supposed to move in two planes if the excercise is done correctly.

Bottom left, "gyros_belt_y", allows to pick out classes B and C with a relatively high probability if measured value is low.

Shown on the top right, "roll_arm" can allow to distinguish A and B classes from the rest. However, the abosulte values are a bit different for each of the participants, resulting in an almost even spread of the values if looked at all 6 people at the same, so to correctly make use of this variable, it should somehow be decided what group the measurement that is being predicted belongs to. 

A higly covariated "yaw_arm" variable is added to do that.

The last one shown, on the bottom right, "magnet_dumbbell_x", also has large differences between measurements, it's absolute value (since it is negative for some participants) can potentially help distinguish A, C and D classes, with even higher chance of it being A, although varying slightly across participants. "magnet_dumbbell_y" also shows the same trend and can therefore be added to increase accuracy.


This would results to a model with 6 covariates, but a couple more are added for an increased accuracy:

```r
modfit <- randomForest(classe ~ gyros_belt_z + gyros_belt_y + roll_arm + yaw_arm + 
    magnet_dumbbell_x + magnet_dumbbell_y + accel_arm_y + pitch_forearm + yaw_forearm + 
    roll_dumbbell, ntree = 40, data = training)
```

Which gives a consedarably low Out-of-Bag error rate of only 3.92%. The confusion matrix is shown below:

```
##      A    B    C    D    E class.error
## A 5477   46   14   35    8     0.01846
## B  108 3521  100   39   29     0.07269
## C   13   76 3286   32   15     0.03974
## D   11   12  108 3059   26     0.04882
## E    8   41   33   28 3497     0.03050
```

From which it can be seen that there is a very low chance of false negative - that is, classifying your execution of an exercise as incorrect when in fact everything was ok. It would be reasonable to assume that it is very important to have this error as the lowest, thus not allowing a person to mistakenly learn an improper way of doing an excersize.  

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

Using more than 40 trees barely decreases error rates (drops to about 5% with 500 trees), and that will most likely introduce a lot of overfitting, which is undesirable.

When trained on a subset of a training data set (the one correct answers are known to), the following are the results:

```r
ind <- sample(seq(1, 19622), 9811)  # random indices to spit
trains <- training[ind, ]
tests <- training[-ind, ]
trains <- trains[sample(nrow(trains)), ]  # shuffle rows
tests <- tests[sample(nrow(tests)), ]  # shuffle rows
subfit <- randomForest(classe ~ gyros_belt_z + gyros_belt_y + roll_arm + yaw_arm + 
    magnet_dumbbell_x + magnet_dumbbell_y + accel_arm_y + pitch_forearm + yaw_forearm + 
    roll_dumbbell, ntree = 40, data = trains)
pred <- predict(subfit)
confusionMatrix(pred, trains$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2742   84   10   14    8
##          B   39 1607   47    9   24
##          C   14   83 1552   81   27
##          D   27   25   29 1503   21
##          E   10   34   12   22 1787
## 
## Overall Statistics
##                                         
##                Accuracy : 0.937         
##                  95% CI : (0.932, 0.942)
##     No Information Rate : 0.289         
##     P-Value [Acc > NIR] : < 2e-16       
##                                         
##                   Kappa : 0.92          
##  Mcnemar's Test P-Value : 2.75e-11      
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.968    0.877    0.941    0.923    0.957
## Specificity             0.983    0.985    0.975    0.988    0.990
## Pos Pred Value          0.959    0.931    0.883    0.936    0.958
## Neg Pred Value          0.987    0.972    0.988    0.985    0.990
## Prevalence              0.289    0.187    0.168    0.166    0.190
## Detection Rate          0.279    0.164    0.158    0.153    0.182
## Detection Prevalence    0.291    0.176    0.179    0.164    0.190
## Balanced Accuracy       0.976    0.931    0.958    0.955    0.974
```

### Results

Overall, the model gives small errors: 7% overall, and very low probability of incorrectly identifying an exercize as being carried out improperly in any way - 2.5%. Based on a single measurement only, but the above model can be updated to "adapt" during the execrsize to the person, because accuracy can be greatly increased by processing not only several measurements, but also its variances, as it can say a lot, when mean/median values for some of variables are known. And simply having several measurements to produce out a single outcome vastly reduces the chance of error to being, essentially, negligible.
