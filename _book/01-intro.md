# Hello RLadies

All chapters of **RLadies** are condensed under RLadies Global and each chapter will try to showcase the topic by as many RLadies chapters as possible without redundancy. 

## StaRt here

This section is about starting in RStudio using R. This starter code is from **RLadies Freiburg** repo. Starting out with using base R which entails the use of the `$` dollar sign, which is used to grab a specific column from a dataset.

The dataset you will use is from a library called *datasets* which is a R package that contains various datasets to be used for your learning needs. Using either a simple .R file or Rmd file with a `{r}` code chunk, enter the code below for basic descriptive statistics.


```r
#--- step one is to install the library package
# install.packages('datasets')    # comment out to run
library(datasets)                 # load library

# the specific dataset we want is called ChickWeight
data("ChickWeight")               # function to pull out specific dataset

head(ChickWeight)                 # see first 6 rows of dataset
#>   weight Time Chick Diet
#> 1     42    0     1    1
#> 2     51    2     1    1
#> 3     59    4     1    1
#> 4     64    6     1    1
#> 5     76    8     1    1
#> 6     93   10     1    1
```


### An unnumbered section {-}

Chapters and sections are numbered by default. To un-number a heading, add a `{.unnumbered}` or the shorter `{-}` at the end of the heading, like in this section.
