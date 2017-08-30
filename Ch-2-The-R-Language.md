# Ch 2. The R Language


### 2.2 A Quick Tour of R's Capabilities Using ANOVA, SEM on Consumer Survey Data

**Assignment** uses the assignment operator "<-" to create a named object that comprises of other objects. 

**c()** denotes a vector


```r
# Testing
x <- c(2,4,6,8)
x
```

```
## [1] 2 4 6 8
```
Install some add-on packages that we will need


```r
#install.packages(c("lavaan","semPlot","corrplot","multcomp"))
```
This data set contains observations from sales and product satisfaction survey.

* It has 500 consumers' answers
* **iProdSAT:** satisfaction with a product
* **iSalesSAT:** satisfaction with sales experience
* **iProdREC:** likelihood to recommend the product
* **iSalesREC:** likelihood to recommend the salesperson
* **Segement:** numerically-coded segment

The function **`factor`** is used to encode a vector as a factor/category. For this data set, we set `Segment` to be a categorical factor variable. Observe Segment is now a factor data type:


```r
satData <- read.csv("http://goo.gl/UDv12g")
satData$Segment <- factor(satData$Segment)
head(satData)
```

```
##   iProdSAT iSalesSAT Segment iProdREC iSalesREC
## 1        6         2       1        4         3
## 2        4         5       3        4         4
## 3        5         3       4        5         4
## 4        3         3       2        4         4
## 5        3         3       3        2         2
## 6        4         4       4        5         4
```

Next we can plot the corr matrix excluding the categorical `Segment` variable in column 3 by specifying -3 in our slice of `satData`.

`corrplot.mixed(corr)`: Using mixed methods to visualize a correlation matrix.

`cor(x,y = NULL)`: computes the correlation of x and y if these are vectors.

```r
library(corrplot)  # In order to use package
corrplot.mixed(cor(satData[,-3]))
```

![](README_figs/README-unnamed-chunk-5-1.png)<!-- -->
*Observations*:

* All variables are positively correlated
* Satisfaction metrics are strongly correlated with one another (0.41)
* Recommendation metrics are strongly correlated with one another (0.46)

#### Q. Does product satisfaction differ by segment? 

We compute a mean satisfaction for each segment using the `aggregate()` function to observe sample means of product satisfaction per segment

`aggregate(x, by, data, function..)` splits the data into subsets and computes summary stats for each subset


```r
aggregate(iProdSAT ~ Segment, satData, mean)
```

```
##   Segment iProdSAT
## 1       1 3.462963
## 2       2 3.725191
## 3       3 4.103896
## 4       4 4.708075
```
*Observe*: Segment 4 has the highest level of satisfaciton while Segment 1 has the lowest level of satisfaction

#### Q. Are the differences in satisfaction statistically significant?

Perform a one-way ANOVA across the segments:

`aov(formula, data=NULL)`: fits a balanced-design anova model. Formula specifies the model


```r
satData.anova <- aov(iProdSAT ~ -1 + Segment, satData)  # why is there a -1?
summary(satData.anova)
```

```
##            Df Sum Sq Mean Sq F value Pr(>F)    
## Segment     4   8628    2157    2161 <2e-16 ***
## Residuals 496    495       1                   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
*Observe*: There are significant differences between the sample means

We plot the ANOVA model to visualize confidence intervals for mean product satisfaction per segment:

`par(..)` can be used to set or query graphical parameters. Parameters are set by specifying them as arguments to par in `tag = value` form

`mar` A numerical vector of the form `c(bottom, left, top, right)` which gives the number of lines of margin to be specified on the four sides of the plot. The default is c(5, 4, 4, 2) + 0.1. 

'glht(model)`: General linear hypotheses and multiple comparisons for parametric models, including generalized linear models, linear mixed effects models, and survival models.
  
`model`: a fitted model, for example an object returned by lm, glm, or aov etc. It is assumed that coef and vcov     methods are available for model.

```r
#install.packages("zoo")
library(multcomp)
```

```
## Loading required package: mvtnorm
```

```
## Loading required package: survival
```

```
## Loading required package: TH.data
```

```
## Loading required package: MASS
```

```
## 
## Attaching package: 'TH.data'
```

```
## The following object is masked from 'package:MASS':
## 
##     geyser
```



```r
par(mar=c(4,8,4,2)) # setting margin parameters for plot
plot(glht(satData.anova))
```

![](README_figs/README-unnamed-chunk-9-1.png)<!-- -->
*Observe*: 

* Seg 1, 2 and 3 differ modestly while Seg 4 is much more satisfied than the others
* Seg 1 has a wider confidence interval than the other segments

##### Likert Rating Scales:

X-axis represents a **Likert rating scale** ranging from 1 to 7 for product satisfaction. "Likert scales are survey questions that offer a range of answer options — from one extreme attitude to another, like “extremely likely” to “not at all likely.” Typically, they include a moderate or neutral midpoint.

Likert scales (named after their creator, American social scientist Rensis Likert) are quite popular because they are one of the most reliable ways to measure opinions, perceptions, and behaviors.""

Src: https://www.surveymonkey.com/mp/likert-scale/

##### Structural Equation Models

Many marketing analysts are interested in SEM's and R has multiple pkgs to fit SEMs. "Attitudes, opinions and personality traits are important drivers of consumer behavior, but they are latent constructs and marketing researchers cannot actually observe them or measure them directly. We can only make inferences about them from what we can observe, responses to questionnaire items, for example. Measuring latent constructs is challenging and we must also incorporate estimates of measurement error into our models. SEM excels at both of these tasks."

SEM is suited for causal analysis especially when there's MTC in the data set. It can be used on social media data, transactional data, economic data, and etc. SEM helps us observe latent segments of consumers with different perceptions or attributes (aka Driver Segmentation) or latent variables within the data set 

Src: http://www.kdnuggets.com/2017/03/structural-equation-modeling.html

"SEM, is a very general, chiefly linear, chiefly cross-sectional statistical modeling technique. Factor analysis, path analysis and regression all represent special cases of SEM. In SEM, interest usually focuses on latent constructs--abstract psychological variables like "intelligence" or "attitude toward the brand"--rather than on the manifest variables used to measure these constructs. Measurement is recognized as difficult and error-prone. By explicitly modeling measurement error, SEM users seek to derive unbiased estimates for the relations between latent constructs. To this end, SEM allows multiple measures to be associated with a single latent construct."
Src 2: http://www2.gsu.edu/~mkteer/sem.html

#### Q. Do latent variables affect satisfaction or likelihood-to-recommend?

By fitting an SEM to the satisfaction data, we can define a model with latent vars for both satisfaction and recommendation. The SAT latent var is manifested in the two satisfaction metrics while the REC latent var is manifested in the two recommendation metrics. 

As marketers, we wish to understand, is the latent REC var affected by the latent SAT var?

```r
satModel <- "SAT =~ iProdSAT + iSalesSAT 
             REC =~ iProdREC + iSalesREC
             REC ~ SAT "
# line 1: Latent SAT var is observed as items iProdSAT and iSalesSAT
# line 2: Latent REC var is observated as items iProdREC and iSalesREC
# line 3: RECommendation varies with SATisfaction
```

Now we fit the model to the data using `lavaan` package:

```r
library(lavaan)
```

```
## This is lavaan 0.5-23.1097
```

```
## lavaan is BETA software! Please report any bugs.
```

```r
sat.fit <- cfa(satModel, data=satData)
summary(sat.fit, fit.m=TRUE)
```

```
## lavaan (0.5-23.1097) converged normally after  31 iterations
## 
##   Number of observations                           500
## 
##   Estimator                                         ML
##   Minimum Function Test Statistic                2.319
##   Degrees of freedom                                 1
##   P-value (Chi-square)                           0.128
## 
## Model test baseline model:
## 
##   Minimum Function Test Statistic              278.557
##   Degrees of freedom                                 6
##   P-value                                        0.000
## 
## User model versus baseline model:
## 
##   Comparative Fit Index (CFI)                    0.995
##   Tucker-Lewis Index (TLI)                       0.971
## 
## Loglikelihood and Information Criteria:
## 
##   Loglikelihood user model (H0)              -3040.385
##   Loglikelihood unrestricted model (H1)      -3039.225
## 
##   Number of free parameters                          9
##   Akaike (AIC)                                6098.769
##   Bayesian (BIC)                              6136.701
##   Sample-size adjusted Bayesian (BIC)         6108.134
## 
## Root Mean Square Error of Approximation:
## 
##   RMSEA                                          0.051
##   90 Percent Confidence Interval          0.000  0.142
##   P-value RMSEA <= 0.05                          0.347
## 
## Standardized Root Mean Square Residual:
## 
##   SRMR                                           0.012
## 
## Parameter Estimates:
## 
##   Information                                 Expected
##   Standard Errors                             Standard
## 
## Latent Variables:
##                    Estimate  Std.Err  z-value  P(>|z|)
##   SAT =~                                              
##     iProdSAT          1.000                           
##     iSalesSAT         1.067    0.173    6.154    0.000
##   REC =~                                              
##     iProdREC          1.000                           
##     iSalesREC         0.900    0.138    6.528    0.000
## 
## Regressions:
##                    Estimate  Std.Err  z-value  P(>|z|)
##   REC ~                                               
##     SAT               0.758    0.131    5.804    0.000
## 
## Variances:
##                    Estimate  Std.Err  z-value  P(>|z|)
##    .iProdSAT          0.706    0.088    7.994    0.000
##    .iSalesSAT         0.793    0.100    7.918    0.000
##    .iProdREC          0.892    0.129    6.890    0.000
##    .iSalesREC         0.808    0.107    7.533    0.000
##     SAT               0.483    0.097    4.970    0.000
##    .REC               0.516    0.115    4.505    0.000
```
*Observe*: the model fits the data well with a Comparative Fit Index (CFI) ~ 1 . See Ch. 10

We can visualize the SEM using the `semPlot` package in order to create a structural model. A **structural model** includes path loadings for a model and the estimated coefficient between latent vars.

```r
#install.packages(c("lme4","car","psych", "ggplot2","htmlwidgets","data.table","pkgconfig"))
```


```r
library(semPlot)
semPaths(sat.fit, what="est",
         residuals=FALSE, intercepts=FALSE, nCharNodes=9)
```

```
## Warning in qgraph(Edgelist, labels = nLab, bidirectional = Bidir, directed
## = Directed, : The following arguments are not documented and likely not
## arguments of qgraph and thus ignored: loopRotation; residuals; residScale;
## residEdge; CircleEdgeEnd
```

![](README_figs/README-unnamed-chunk-13-1.png)<!-- -->
*Observe*:

* Each proposed latent var is highly loaded (contingent) on its observed (manifested) survey items. (1.0 and 1.7 for SAT, 1.0 and .90 for REC)
* Customers' latent satisfaction (SAT) is shown to have a strong association or relationship with their likelihood to recommend (REC) with an estimated coefficient of 0.76. See Ch. 10 FMI.