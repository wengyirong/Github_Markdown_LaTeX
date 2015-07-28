# Using Inverse Probability Weighting with SVMs to Address Confounding
### by Kristin A. Linn
### June 25, 2015

Here we provide an example of how to implement inverse probability weighting with SVMs to address confounding.  The basic setup is that we have feature, class label pairs of the form (![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_03.png)) for each subject, i=1,...,n, where ![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_04.png) and ![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_05.png) for all ![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_06.png). We wish to train a SVM to predict y given x. As an example, y might be an indicator of disease/control group and x might be a vectorized image containing voxel values or volumes of regions across the brain. However, the additional feature vector ![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_07.png) observed for all subjects confounds the relationship between x and y. For example, a might contain covariates such as age and sex.  In the presence of confounding by a, inverse probability weighting is used to recover an estimate of the target classifier, which is the SVM classifier that would have been estimated had there been no confounding by a.



```r
set.seed (1)
```

We use the package 'rPython' to access libSVM (https://www.csie.ntu.edu.tw/~cjlin/libsvm/) through scikit learn (http://scikit-learn.org/stable/). The file fit_svm.py contains a python function that implements a linear kernel SVM with subject-level weights and a grid search to tune the cost parameter, C.



```r
library(MASS)
library(rPython)
```

```
## Loading required package: RJSONIO
```

```r
python.load("/Users/kalinn/GitHub/IPW-SVM/fit_svm.py")
```

```
## Warning in file(con, "r"): cannot open file '/Users/kalinn/GitHub/IPW-SVM/
## fit_svm.py': No such file or directory
```

```
## Error in file(con, "r"): cannot open the connection
```

## Generate data for example

We generate data such that the confounders, a1 and a2, affect both the features, x1 and x2, as well as the class labels, d.



```r
# Total in confounded sample
n = 200
# Number of noise features
k = 10
# a1 and a2 are confounders
a1 = runif (n, 0, 1)
a2 = rbinom(n, 1, .5)
# d is a vector of class labels
ld = -1 + a1 + a2 + rnorm(n, 0, .5)
d = 1*(exp(ld)/(1+exp(ld))>.5)
# covariance structure for features
# x1 and x2 are  
covmat = matrix (c (2, .5, .5, 2), 2, 2)
errs = mvrnorm (n, mu=rep (0, 2), Sigma=covmat)
# x1 and x2 are features
x1mean = 5 - 2*d - .5*a1
x2mean = -3*a1 + .5*a2 - .5*d*(a1 + .5*a2 + .25*a1*a2) 
x1 = scale(x1mean + errs[,1])
x2 = scale(x2mean + errs[,2])
noise = matrix (rnorm(n*k), n, k)
features = data.frame(x1=x1, x2=x2, noise=noise)
```

## Estimate the inverse probability weights

Here, we estimate the weights by fitting a logistic regression of class (d) on confounders (a1 and a2). However, more flexible methods can be substituted here to obtain estimates of the weights. All that is needed is an estimate of ![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_08.png) for each subject, i=1,...,n.


```r
# Fit the model
lr.fit = glm(d~a1+a2, family=binomial)
# Obtain predicted values of pr(d=1 | a1, a2) for each subject
lr.predict = lr.fit![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_09.png)S_i = pr(d_i=1 | a1_i, a2_i)![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_10.png)M_i = pr(d_i=1)$ as well as corresponding estimates \hat{S}_i = \hat{pr}(d_i=1 | a1_i, a2_i) and \hat{M}_i = \hat{pr}(d_i=1), Then, stabilized weights and their corresponding estimates are defined, respectively, as:
![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_01.png)
and fun
![latex equation](https://rawgit.com/muschellij2/Github_Markdown_LaTeX/master/eq_no_02.png)
## Train the inverse probability weighted SVM (IPW-SVM)


```r
# For tuning the cost parameter
cost = 10^c(-3:3)
# rPython needs a matrix with no column names
features = as.matrix(features)
colnames(features) = NULL
# rPython is picky about inputs!
ipweights = as.numeric (as.character (ipweights))
# Here we input the full data as both the training and test sets, 
# but in a real application we might split the original data into
# training and validation sets or perform cross-validation.
train.svm = python.call("fit_ipw_svm", features, d, features, ipweights, cost)
```

```
## Error in python.exec(python.command): name 'fit_ipw_svm' is not defined
```

## Return parameters of interest


```r
# IPW-SVM intercept of the linear decision rule
train.svm[[1]]
```

```
## Error in eval(expr, envir, enclos): object 'train.svm' not found
```

```r
# IPW-SVM weights of the linear decision rule
train.svm[[2]]
```

```
## Error in eval(expr, envir, enclos): object 'train.svm' not found
```

```r
# Class predictions for the test set
train.svm[[3]]
```

```
## Error in eval(expr, envir, enclos): object 'train.svm' not found
```

