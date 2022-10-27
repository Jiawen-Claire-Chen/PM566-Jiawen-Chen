lab9
================
Jiawen
2022-10-26

``` r
install.packages("microbenchmark")
```

    ## Installing package into '/Users/chenjiawen/Library/R/arm64/4.2/library'
    ## (as 'lib' is unspecified)

``` r
library(microbenchmark)
```

\#PROBLEM 2create a n x k matrix

``` r
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda)) #rpois: retern random numbers from pois, k means how many numbers should be returned; rbind,r menas rows, bind menas put them together; lambda means the mean 
  
  return(x)
}
f1 <- fun1(1000,4)
mean(f1)
```

    ## [1] 4.0335

``` r
fun1alt <- function(n = 100, k = 4, lambda = 4) {
  # YOUR CODE HERE
  x <- matrix( rpois(n*k, lambda), ncol=4)
  return(x)
}
f1 <- fun1alt(5000,4)
# Benchmarking
microbenchmark::microbenchmark(
  fun1(),
  fun1alt()
)
```

    ## Warning in microbenchmark::microbenchmark(fun1(), fun1alt()): less accurate
    ## nanosecond times to avoid potential integer overflows

    ## Unit: microseconds
    ##       expr     min      lq      mean   median       uq     max neval
    ##     fun1() 160.474 177.243 183.34708 180.7075 186.1195 218.202   100
    ##  fun1alt()  11.603  12.259  22.40076  12.6280  13.2225 956.899   100

\#way of creating matrix

``` r
d <- matrix(1:16,ncol=4)
d
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    5    9   13
    ## [2,]    2    6   10   14
    ## [3,]    3    7   11   15
    ## [4,]    4    8   12   16

``` r
diag(d) #take diag elements
```

    ## [1]  1  6 11 16

``` r
d[2]
```

    ## [1] 2

``` r
d[2,1]
```

    ## [1] 2

``` r
d[c(1,6,11,16)]
```

    ## [1]  1  6 11 16

``` r
cbind(1:4,1:4)
```

    ##      [,1] [,2]
    ## [1,]    1    1
    ## [2,]    2    2
    ## [3,]    3    3
    ## [4,]    4    4

``` r
d[cbind(1:4,1:4)]
```

    ## [1]  1  6 11 16

\#Problem 3: Find the column max (hint: Checkout the function
max.col()).

``` r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
M <- matrix(runif(12), ncol=4) #run random numbers from uniform distribution 
M
```

    ##           [,1]      [,2]        [,3]      [,4]
    ## [1,] 0.1137034 0.6233794 0.009495756 0.5142511
    ## [2,] 0.6222994 0.8609154 0.232550506 0.6935913
    ## [3,] 0.6092747 0.6403106 0.666083758 0.5449748

``` r
# Find each column's max value, 2 means column/1 means row, max means return the maximum number
fun2 <- function(x) {
  apply(x, 2, max)
}
fun2(M)
```

    ## [1] 0.6222994 0.8609154 0.6660838 0.6935913

``` r
fun2alt <- function(x) {
  # YOUR CODE HERE, t(x) means flip matrix
  idx <- max.col(t(x))  #return the location
  x[cbind(idx,1:4)] #go back to the x matrix and return the value of location
  }
fun2alt(M)
```

    ## [1] 0.6222994 0.8609154 0.6660838 0.6935913

``` r
x <- matrix(rnorm(1e4), nrow=10) #run random numbers from normal distribution 1e4=10000
# Benchmarking
microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x)
)
```

    ## Unit: microseconds
    ##        expr     min       lq      mean   median       uq      max neval
    ##     fun2(x) 466.990 481.8935 547.06546 494.0705 556.3085 1924.007   100
    ##  fun2alt(x)  62.566  65.3130  78.62078  67.3220  70.3355 1102.162   100

``` r
# Bootstrap of an OLS
my_stat <- function(d) coef(lm(y ~ x, data=d)) #return coef from the regression 

# DATA SIM
set.seed(1)
n <- 500; R <- 1e4 #n is the sample size, R is tne number of resamplingwe do

x <- cbind(rnorm(n)); y <- x*5 + rnorm(n) #y is the output of regression, rnorm(n) is noise 
```

\#problem 4

``` r
library(parallel)
my_boot <- function(dat, stat, R, ncpus = 1L) {
  
  # Getting the random indices
  n <- nrow(dat)
  idx <- matrix(sample.int(n, n*R, TRUE), nrow=n, ncol=R)
 
  # Making the cluster using `ncpus`
  # STEP 1: GOES HERE
  # 1. CREATING A CLUSTER
library(parallel)
cl <- makePSOCKcluster(4)    
# 2. PREPARING THE CLUSTER
clusterSetRNGStream(cl, 123) # Equivalent to `set.seed(123)`

# 3. DO YOUR CALL
  # STEP 2: GOES HERE
clusterExport(cl,c("stat", "dat", "idx"), envir = environment())
    # STEP 3: THIS FUNCTION NEEDS TO BE REPLACES WITH parLapply
  ans <- parLapply(cl, seq_len(R), function(i) {
    stat(dat[idx[,i], , drop=FALSE])
  })
  # Coercing the list into a matrix
  ans <- do.call(rbind, ans)
  
  # STEP 4: GOES HERE
  
  ans
  
}
# Checking if we get something similar as lm
ans0 <- confint(lm(y~x)) #regress confidence interval 
ans1 <- my_boot(dat = data.frame(x, y), my_stat, R = R, ncpus = 2L)

# You should get something like this
t(apply(ans1, 2, quantile, c(.025,.975)))
```

    ##                   2.5%      97.5%
    ## (Intercept) -0.1386903 0.04856752
    ## x            4.8685162 5.04351239

``` r
ans0
```

    ##                  2.5 %     97.5 %
    ## (Intercept) -0.1379033 0.04797344
    ## x            4.8650100 5.04883353

\#problem 4:

``` r
library(parallel)
R
```

    ## [1] 10000
