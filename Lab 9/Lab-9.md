---
title: "Lab 9"
author: "Juehan Wang"
date: "10/29/2021"
output:
    html_document:
      toc: yes 
      toc_float: yes 
      keep_md: yes
    github_document:
      keep_html: true
      html_preview: false
always_allow_html: true
---





# Learning goals

In this lab, you are expected to learn/put in practice the following skills:

* Evaluate whether a problem can be parallelized or not.

* Practice with the parallel package.

* Use Rscript to submit jobs

* Practice your skills with Git.

# Problem 1: Think

Give yourself a few minutes to think about what you just learned. List three examples of problems that you believe may be solved using parallel computing, and check for packages on the HPC CRAN task view that may be related to it.

# Problem 2: Before you

The following functions can be written to be more efficient without using parallel:

1. This function generates a n x k dataset with all its entries distributed poission with mean lambda.


```r
fun1 <- function(n = 100, k = 4, lambda = 4) {
  set.seed(1029)
  x <- NULL
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda))
  x
}
fun1(5,10)
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    1    3    2    4    4    2    3    5    5     4
## [2,]    2    4    3    5    3    4    1    4    1     5
## [3,]    4    8    2    3    1    4    5    5    5     4
## [4,]    7    3    5    1    4    9    5    2    6     5
## [5,]    2    1    2    6    3    5    4    4    3    10
```

```r
fun1alt <- function(n = 100, k = 4, lambda = 4) {
  set.seed(1029)
  x <- matrix(rpois(n*k, lambda), nrow=n, ncol=k, byrow = TRUE)
  x
}
fun1alt(5,10)
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    1    3    2    4    4    2    3    5    5     4
## [2,]    2    4    3    5    3    4    1    4    1     5
## [3,]    4    8    2    3    1    4    5    5    5     4
## [4,]    7    3    5    1    4    9    5    2    6     5
## [5,]    2    1    2    6    3    5    4    4    3    10
```

```r
# Benchmarking
microbenchmark::microbenchmark(
  fun1(n = 1000),
  fun1alt(n = 1000)
)
```

```
## Unit: microseconds
##               expr      min       lq      mean   median       uq       max
##     fun1(n = 1000) 4898.150 5246.873 7663.2969 5968.443 8239.108 19033.816
##  fun1alt(n = 1000)  126.154  136.517  180.2863  151.559  166.884  2291.489
##  neval cld
##    100   b
##    100  a
```

2. Find the column max (hint: Checkout the function max.col()).


```r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
x <- matrix(rnorm(1e4), nrow=10)

# Find each column's max value
fun2 <- function(x) {
  apply(x, 2, max)
}

fun2alt <- function(x) {
  # Position of the max value per row of x.
  idx <- max.col(t(x))
  # Get the actual max value
  # x[cbind(1,15)] ~ x[1,15]
  # want to access x[1,16], x[4,1]
  # x[rbind(c(1,16),c(4,1))]
  # want to access x[4,16], x[4,1]
  # x[rbind(4,c(16,1))]
  x[cbind(idx, 1:ncol(x))]
}

# Do we get the same?
all(fun2(x) == fun2alt(x))
```

```
## [1] TRUE
```

```r
x <- matrix(rnorm(5e4), nrow=10)
# Benchmarking
microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x), unit = "relative"
)
```

```
## Unit: relative
##        expr      min       lq     mean   median       uq      max neval cld
##     fun2(x) 12.01304 9.268255 8.314419 9.659178 8.586624 2.857046   100   b
##  fun2alt(x)  1.00000 1.000000 1.000000 1.000000 1.000000 1.000000   100  a
```

# Problem 3: Parallelize everyhing

We will now turn our attention to non-parametric bootstrapping. Among its many uses, non-parametric bootstrapping allow us to obtain confidence intervals for parameter estimates without relying on parametric assumptions.

The main assumption is that we can approximate many experiments by resampling observations from our original dataset, which reflects the population.

This function implements the non-parametric bootstrap:


```r
my_boot <- function(dat, stat, R, ncpus = 1L) {
  
  # Getting the random indices
  n <- nrow(dat)
  idx <- matrix(sample.int(n, n*R, TRUE), nrow=n, ncol=R)
 
  # Making the cluster using `ncpus`
  # STEP 1: GOES HERE
  # STEP 2: GOES HERE
  
    # STEP 3: THIS FUNCTION NEEDS TO BE REPLACES WITH parLapply
  ans <- lapply(seq_len(R), function(i) {
    stat(dat[idx[,i], , drop=FALSE])
  })
  
  # Coercing the list into a matrix
  ans <- do.call(rbind, ans)
  
  # STEP 4: GOES HERE
  
  ans
  
}
```

1. Use the previous pseudocode, and make it work with parallel. Here is just an example for you to try:


```r
# Bootstrap of an OLS
my_stat <- function(d) coef(lm(y ~ x, data=d))

# DATA SIM
set.seed(1)
n <- 500; R <- 1e4

x <- cbind(rnorm(n)); y <- x*5 + rnorm(n)

# Checking if we get something similar as lm
ans0 <- confint(lm(y~x))
ans1 <- my_boot(dat = data.frame(x, y), my_stat, R = R, ncpus = 2L)

# You should get something like this
t(apply(ans1, 2, quantile, c(.025,.975)))
```

```
##                   2.5%      97.5%
## (Intercept) -0.1386903 0.04856752
## x            4.8685162 5.04351239
```

```r
##                   2.5%      97.5%
## (Intercept) -0.1372435 0.05074397
## x            4.8680977 5.04539763
ans0
```

```
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.04883353
```

```r
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.04883353
```

2. Check whether your version actually goes faster than the non-parallel version:


```r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 1L))
```

```
##    user  system elapsed 
##   3.015   0.060   3.126
```

```r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 2L))
```

```
##    user  system elapsed 
##   2.786   0.035   2.857
```

# Problem 4: Compile this markdown document using Rscript

Once you have saved this Rmd file, try running the following command in your terminal:

Rscript --vanilla -e 'rmarkdown::render("/Users/juehanwang/Desktop/566/Lab/PM566-fall2021-labs/Lab 9/Lab 9.Rmd")' &

Where [full-path-to-your-Rmd-file.Rmd] should be replace with the full path to your Rmd file… :).
