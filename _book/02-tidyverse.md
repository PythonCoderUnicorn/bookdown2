# Tidyverse 

The tidyverse has its own book called *R for Data Science* by Hadley Wickham & Garrett Grolemund (available online). This section will discuss a few functions out of the library that are very useful to know.

## Tidy Data

Data Transformation and Introduction to **Tidyverse** library. This section is by **RLadies Freiburg** Divya Seernani. 
Tidy data is data that has data or values for each column (variable) and each row (observation). The paradigm: 
`import` -> `tidy` -> [Transform <-> Visualize <-> Model] -> Communicate

### dplyr library

Inside the **tidyverse** library is the dplyr library which is helpful when creating new variables, renaming or reordering observations, selecting specific observations and calculating summary statistics among many other functions.


The dataset for this section will be the Indian Census (2011), dataset is available [here](https://github.com/nishusharma1608/India-Census-2011-Analysis/blob/master/india-districts-census-2011.csv), *Remember to click on 'raw' data format first to load in data, you should see white background and black text CSV data, copy the url link in order to read in the data*. 


The tidyverse library brings along other libraries and when you load tidyverse you will a bunch of warnings in the console, these are no concern to you, basically telling you that one library hides another for function use. To turn off warnings in your Rmd file, inside the code chunk {r} set warning to false `{r, warning= FALSE, message= FALSE}`. 
s

```r
# step 1 - install tidyverse library
# install.packages('tidyverse')  # comment-out to run 
library(tidyverse)

# read in the data
india_census = read.csv('https://raw.githubusercontent.com/nishusharma1608/India-Census-2011-Analysis/master/india-districts-census-2011.csv')
```


## section 2

<!-- Don't miss Table \@ref(tab:nice-tab). -->

<!-- ```{r nice-tab, tidy=FALSE} -->
<!-- knitr::kable( -->
<!--   head(pressure, 10), caption = 'Here is a nice table!', -->
<!--   booktabs = TRUE -->
<!-- ) -->
<!-- ``` -->





