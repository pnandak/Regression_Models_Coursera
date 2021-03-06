## Impact of Transmission on MPG  

### Executive Summary

Motor Trend magazine is interested in exploring the relationship between a set of variables and miles per gallon (MPG) of a collection of cars. This analysis answers two questions:

    - "Is an automatic or manual transmission better for MPG ?"
    - "Quantify the MPG difference between automatic and manual transmissions"

Considering a model that includes weight, acceleration and transmission, we can say that automatic cars have 2.94 miles per galon (MPG) more than manual cars.  

### Exploratory Data Analysis

The data was extracted from the 1974 Motor Trend US magazine, and comprises fuel consumption and 10 aspects of automobile design and performance for 32 automobiles (1973-74 models).

We compare MPG for the automatic and manual transmission using Student t test: Our null hypothesis (H0) is "there is no difference in MPG between transmission" and our alternative hypothesis (Ha) is "automatic transmission have lower fuel consumption than manual"  


```r
t.test(mpg ~ am, mtcars, alternative="less")$p.value
```

```
## [1] 0.0006868192
```

The p-value is lower than 0.05, the result is significant and the null hypothesis can be rejected.
We can say that, when we assume that all other variables are same for automatic and manual transmission, automatic transmission is better than manual transmission for MPG.

But, as shown in Figure 2 in Appendix, this is not the case: some variables of the dataset (like weight, for example) doesn't have the same distribution for automatic and manual transmission.  


```r
data(mtcars)

round(sort(cor(mtcars)["mpg", -1]), 3)
```

```
##     wt    cyl   disp     hp   carb   qsec   gear     am     vs   drat 
## -0.868 -0.852 -0.848 -0.776 -0.551  0.419  0.480  0.600  0.664  0.681
```

Comparing the correlation of the caracteristic of a car with MPG, we notice that 3 variables are highly correlated with MPG (>.8): wt, cyl and disp.
Model selection

We build a first model based on Simple Linear Regression.


```r
mtcars$am  <- factor(mtcars$am, levels=c(0,1), labels=c("automatic", "manual"))

fit.simple <- lm(mpg ~ am, mtcars)
summary(fit.simple)$adj.r.squared
```

```
## [1] 0.3384589
```


The adjusted $R^2$ value indicates that the model explains only 34% of the variations. It's a very low value.


```r
summary(fit.simple)$coefficients
```

```
##              Estimate Std. Error   t value     Pr(>|t|)
## (Intercept) 17.147368   1.124603 15.247492 1.133983e-15
## ammanual     7.244939   1.764422  4.106127 2.850207e-04
```


This model tells us that changing from automatic to manual transmission causes a 7.245 increase in MPG.

We then use the Stepwise Algorithm (step-by-step selection) to select a better model (keeping am variable in the model):


```r
data(mtcars)
mtcars$am  <- factor(mtcars$am, levels=c(0,1), labels=c("automatic", "manual"))

fit.step <- step(lm(mpg~., mtcars), trace=0, scope=list(lower=~am), direction="both")
summary(fit.step)$call
```

```
## lm(formula = mpg ~ wt + qsec + am, data = mtcars)
```

The best model proposed by Stepwise includes the weight (wt) and the "1/4 mile time" (qsec) of the cars, in addition to transmission (am), to explain fuel consumption (MPG).


```r
summary(fit.step)$adj.r.squared
```

```
## [1] 0.8335561
```


The adjusted $R^2$ is 0.8336 which means that the model explains 83% of the variation.  

### Model comparison and Residuals analysis

We then compare the model proposed by Stepwise with our first model using ANOVA.


```r
anova(fit.simple, fit.step)[2,6] #p-value
```

```
## [1] 1.550495e-09
```


The p-value is very low: we can then reject the null hypothesis (i.e. "Model are equals") and claim that the model proposed by the Stepwise algorithm is better than our first simple model.
The Figure 3 (in Appendix) is a residual plot of the selected model. Residuals seems to be uncorrelated with the fit, independent and (almost) identically distributed with mean zero.  

### Results


```r
summary(fit.step)$coefficients
```

```
##              Estimate Std. Error   t value     Pr(>|t|)
## (Intercept)  9.617781  6.9595930  1.381946 1.779152e-01
## wt          -3.916504  0.7112016 -5.506882 6.952711e-06
## qsec         1.225886  0.2886696  4.246676 2.161737e-04
## ammanual     2.935837  1.4109045  2.080819 4.671551e-02
```


Given the coefficients of our model, we can say that automatic cars have lower fuel consumption than manual cars: they have 2.94 miles per galon (MPG) more than manual cars. This value can be obtained when we consider the weight (wt) and the "1/4 mile time" (qsec) variables of the cars of our dataset.  

### Appendix  

### Figure 1: MPG by Transmission


```r
data(mtcars)
mtcars$am  <- factor(mtcars$am, levels=c(0,1), labels=c("automatic", "manual"))
plot(mpg~am, mtcars, main="MPG by Transmission",
     xlab="Transmission", 
     ylab="Fuel consumption (in miles per gallon)")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 


### Figure 2: Scatterplot Matrix


```r
data(mtcars)
pairs(mtcars[, c("mpg", "wt", "qsec", "am")], 
      panel = panel.smooth, 
      main = "Motor Trends cars", col = 3 + (mtcars$am))
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 


Cars with automatic transmission are plot in green.
Cars with manual transmission are plot in blue.  

### Figure 3: Residual plot for the selected model without and with interactions


```r
par(mfrow=c(2,2))
plot(fit.step)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 

```r
par(mfrow=c(1,1))
```


