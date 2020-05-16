
<!-- README.md is generated from README.Rmd. Please edit that file -->

# RfEmpImp <a href='https://github.com/shangzhi-hong/RfEmpImp'><img src='man/figures/logo.png' align="right" height="160"/></a>

[![Lifecycle:
maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)  
An R package for random-forest-empowered imputation of missing Data

## Random-forest-based multiple imputation evolved

This is the repository for R package `RfEmpImp`, for multiple imputation
using random forests (RF).  
This R package is an implementation for the `RfPred` and `RfNode`
algorithms and currently operates under the multiple imputation
computation framework
[`mice`](https://CRAN.R-project.org/package=mice).  
The R package contains both newly proposed and improved existing
algorithms for random-forest-based multiple imputation of missing
data.  
For details of the newly proposed algorithms, please refer to:
[arXiv:2004.14823](https://arxiv.org/abs/2004.14823) (further updates
pending).

## Installation

This R package is already submitted to CRAN, but it may take a while
before the package being approved.  
Currently, interested users can install the package from GitHub:

``` r
# Install from GitHub online
if(!"remotes" %in% installed.packages()) install.packages("remotes")
remotes::install_github("shangzhi-hong/RfEmpImp")
# Install from downloaded source file
install.packages(path_to_source_file, repos = NULL, type = "source")
# Attach
library(RfEmpImp)
```

## Imputation based on predictions

### For mixed types of variables

With version `2.0.0`, the names of parameters were further simplified,
please refer to the documentation for details.  
For data with mixed types of variables, `RfEmp` method is a short cut
for using `RfPred.Emp` for continuous variables and `RfPred.Cate` for
categorical variables (of type `logical` or `factor`, etc.).  
Example:

``` r
# Prepare data
df <- nhanes
df[, c("age", "hyp")] <- lapply(X = nhanes[, c("age", "hyp")], FUN = as.factor)
# Do imputation
imp <- imp.rfemp(df)
# Do analyses
regObj <- with(imp, lm(chl ~ bmi + hyp))
# Pool analyzed results
poolObj <- pool(regObj)
# Extract estimates
res <- reg.ests(poolObj)
```

### Prediction-based imputation for continuous variables

For continuous variables, in `RfPred.Emp` method, the empirical
distribution of random forest’s out-of-bag prediction errors is used to
construct the conditional distributions of the variable under
imputation, providing conditional distributions with better quality.
Users can set `method = "rfpred.emp"` in function call to `mice` to use
it.

Also, in `RfPred.Norm` method, normality was assumed for RF prediction
errors, as proposed by Shah *et al.*, and users can set `method =
"rfpred.norm"` in function call to `mice` to use it.

### Prediction-based imputation for categorical variables

For categorical variables, in `RfPred.Cate` method, the probability
machine theory is used, and the predictions of missing categories are
based on the predicted probabilities for each missing observation. Users
can set `method = "rfpred.cate"` in function call to `mice` to use it.

## Imputation based on predicting nodes

For both continuous variables, the observations under the predicting
nodes of random forest are used as candidates for imputation.  
Two methods are now available for the `RfNode` algorithm.  
Example:

``` r
# Prepare data
df <- nhanes
df[, c("age", "hyp")] <- lapply(X = nhanes[, c("age", "hyp")], FUN = as.factor)
# Do imputation
imp <- imp.rfnode.cond(df)
# Or: imp <- imp.rfnode.prox(df)
# Do analyses
regObj <- with(imp, lm(chl ~ bmi + hyp))
# Pool analyzed results
poolObj <- pool(regObj)
# Extract estimates
res <- reg.ests(poolObj)
```

### Node-based imputation using condtional distributions

`RfNode.Cond` uses the conditional distribution formed by the prediction
nodes, i.e. the weight changes of observations caused by the
bootstrapping of random forest are considered, and uses “in-bag”
observations only. Users can set `method = "rfnode.cond"` in function
call to `mice` to use it.

### Node-based imputation using proximities

`RfNode.Prox` uses the concepts of proximity matrices of random forests,
and observations fall under the same predicting nodes are used as
candidates for imputation. Users can set `method = "rfnode.prox"` in
function call to `mice` to use it.

## Support for parallel computation

The model building for random forest is accelerated using parallel
computation powered by
[`ranger`](https://CRAN.R-project.org/package=ranger). The ranger
software package provides support for parallel computation using native
C++. In our simulations, parallel computation can provide impressive
performance boost for multiple imputation process (about 4x faster on a
quad-core laptop).

## References

1.  Hong, Shangzhi, et al. “Multiple imputation using chained random
    forests.” Preprint, submitted April 30, 2020.
    <https://arxiv.org/abs/2004.14823>.
2.  Zhang, Haozhe, et al. “Random forest prediction intervals.” The
    American Statistician (2019): 1-15.
3.  Wright, Marvin N., and Andreas Ziegler. “ranger: A Fast
    Implementation of Random Forests for High Dimensional Data in C++
    and R.” Journal of Statistical Software 77.i01 (2017).
4.  Shah, Anoop D., et al. “Comparison of random forest and parametric
    imputation models for imputing missing data using MICE: a CALIBER
    study.” American journal of epidemiology 179.6 (2014): 764-774.
5.  Doove, Lisa L., Stef Van Buuren, and Elise Dusseldorp. “Recursive
    partitioning for missing data imputation in the presence of
    interaction effects.” Computational Statistics & Data Analysis 72
    (2014): 92-104.
6.  Malley, James D., et al. “Probability machines.” Methods of
    information in medicine 51.01 (2012): 74-81.