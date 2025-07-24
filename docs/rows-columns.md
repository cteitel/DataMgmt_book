# Data Manipulation with R, Part 1 {#filter-select-mutate}

Much of this lesson draws from The Carpentries' [Data Analysis and Visualization in R for Ecologists](https://datacarpentry.github.io/R-ecology-lesson/index.html) workshop, which is published under a [CC-BY 4.0](https://datacarpentry.github.io/R-ecology-lesson/LICENSE.html) license.

## Objectives

* Explore the structure and content of data frames
* Understand vector types and missing data
* Understand how R assigns values to objects

## Additional reading

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 3: Data transformation, sections 3.1-3.4. Available: https://r4ds.hadley.nz/data-transform.html

## Murray, Sanchez et al. data

So far, we have used data built into R, and practiced loading data from **URLs** and **files**. In this lesson, we will go back to the data from Murray, Sanchez, et al., which is a meta-analysis of relationships between urbanization and wildlife health.



``` r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
## ✔ purrr     1.0.2     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
urban_data <- read_csv("data/raw/Murray-Sanchez_urban-wildlife.csv")
```

```
## Rows: 516 Columns: 42
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (14): TITLE, AUTHORS, JOURNAL, health, condition, toxtype, ptype, stress...
## dbl (28): study, YEAR, SAMPLE_SIZE, EFFECT_DIRECTION, pval, r, yi, vi, rlowe...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Filtering rows; creating and selecting columns

One of the most important skills for working with data in R is the ability to manipulate, modify, and reshape data. The `dplyr` and `tidyr` packages in the `tidyverse` provide some very useful functions for many common data manipulation tasks.

### Selecting columns with `select()`

The `select` function allows you to take only some some columns from your data frame. The first argument is the name of the data frame, and the following arguments are the names of the columns you want. Here, we will take only species information and record IDs from the Murray et al. dataset:


``` r
urban_species <- select(urban_data, study, host.class, aqterr, host.species)
head(urban_species)
```

```
## # A tibble: 6 × 4
##   study host.class aqterr      host.species         
##   <dbl> <chr>      <chr>       <chr>                
## 1  1236 mammals    terrestrial Trichosurus_vulpecula
## 2  1236 mammals    terrestrial Trichosurus_vulpecula
## 3  1236 mammals    terrestrial Trichosurus_vulpecula
## 4  1236 mammals    terrestrial Trichosurus_vulpecula
## 5  1236 mammals    terrestrial Trichosurus_vulpecula
## 6  1236 mammals    terrestrial Trichosurus_vulpecula
```

``` r
dim(urban_species)
```

```
## [1] 516   4
```

This function can be especially useful to clean up your data, for example when you have created intermediate columns, or when you read in a data set that has a lot of extraneous information. Notice that columns are now reordered: `host.species` was specified last within `select`, so it is now the last column.

We can also use `select` to remove columns, using the - symbol.


``` r
select(urban_data, -TITLE, -AUTHORS, -YEAR, -JOURNAL)
```

```
## # A tibble: 516 × 38
##    study health    condition toxtype ptype  stress2 SAMPLE_SIZE EFFECT_DIRECTION
##    <dbl> <chr>     <chr>     <chr>   <chr>  <chr>         <dbl>            <dbl>
##  1  1236 parasites <NA>      <NA>    Preva… <NA>             59                0
##  2  1236 parasites <NA>      <NA>    Preva… <NA>             59               -1
##  3  1236 parasites <NA>      <NA>    Richn… <NA>            278               -1
##  4  1236 condition Adjusted  <NA>    <NA>   <NA>             49                1
##  5  1236 parasites <NA>      <NA>    Inten… <NA>             16               -1
##  6  1236 parasites <NA>      <NA>    Inten… <NA>             10                1
##  7  1236 parasites <NA>      <NA>    Preva… <NA>             59                1
##  8  1236 parasites <NA>      <NA>    Preva… <NA>             59               -1
##  9  1326 toxicants <NA>      metal   <NA>   <NA>             39               -1
## 10  1326 toxicants <NA>      metal   <NA>   <NA>             39               -1
## # ℹ 506 more rows
## # ℹ 30 more variables: pval <dbl>, r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>,
## #   rupper <dbl>, yir <dbl>, ott_id <dbl>, host.species <chr>,
## #   host.class <chr>, aqterr <chr>, parasite2 <chr>, close <dbl>,
## #   nonclose <dbl>, vector <dbl>, intermediate <dbl>, routes <dbl>,
## #   mroute <chr>, mlat <dbl>, mlon <dbl>, udiff_1000 <dbl>, udiff_10000 <dbl>,
## #   uchange_1000 <dbl>, uchange_10000 <dbl>, umean_1000 <dbl>, …
```

`select()` also works with numeric vectors for the order of the columns. To select the 3rd, 4th, 5th, and 10th columns, we could run the following code:


``` r
select(urban_data, 3:5, 10)
```

```
## # A tibble: 516 × 4
##    AUTHORS          YEAR JOURNAL                          stress2
##    <chr>           <dbl> <chr>                            <chr>  
##  1 Webster et al.   2014 Wildlife Biology                 <NA>   
##  2 Webster et al.   2014 Wildlife Biology                 <NA>   
##  3 Webster et al.   2014 Wildlife Biology                 <NA>   
##  4 Webster et al.   2014 Wildlife Biology                 <NA>   
##  5 Webster et al.   2014 Wildlife Biology                 <NA>   
##  6 Webster et al.   2014 Wildlife Biology                 <NA>   
##  7 Webster et al.   2014 Wildlife Biology                 <NA>   
##  8 Webster et al.   2014 Wildlife Biology                 <NA>   
##  9 Hargitai et al.  2016 Science of the Total Environment <NA>   
## 10 Hargitai et al.  2016 Science of the Total Environment <NA>   
## # ℹ 506 more rows
```

You should be careful when using this method, since you are being less explicit about which columns you want. However, it can be useful if you have a data frame with many columns and you don’t want to type out too many names.

We can also select columns based on their properties using the `where()` function. For example, to select all character columns:


``` r
select(urban_data, where(is.character))
```

```
## # A tibble: 516 × 14
##    TITLE     AUTHORS JOURNAL health condition toxtype ptype stress2 host.species
##    <chr>     <chr>   <chr>   <chr>  <chr>     <chr>   <chr> <chr>   <chr>       
##  1 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Prev… <NA>    Trichosurus…
##  2 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Prev… <NA>    Trichosurus…
##  3 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Rich… <NA>    Trichosurus…
##  4 Ectopara… Webste… Wildli… condi… Adjusted  <NA>    <NA>  <NA>    Trichosurus…
##  5 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Inte… <NA>    Trichosurus…
##  6 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Inte… <NA>    Trichosurus…
##  7 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Prev… <NA>    Trichosurus…
##  8 Ectopara… Webste… Wildli… paras… <NA>      <NA>    Prev… <NA>    Trichosurus…
##  9 Effects … Hargit… Scienc… toxic… <NA>      metal   <NA>  <NA>    Parus_major 
## 10 Effects … Hargit… Scienc… toxic… <NA>      metal   <NA>  <NA>    Parus_major 
## # ℹ 506 more rows
## # ℹ 5 more variables: host.class <chr>, aqterr <chr>, parasite2 <chr>,
## #   mroute <chr>, COUNTRY <chr>
```

### Subsetting data with `filter()`

The `filter()` function subsets a data frame to rows that meet some criteria. For example, to get all rows where the taxon sampled was a bird:


``` r
filter(urban_data, host.class == "birds")
```

```
## # A tibble: 159 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  2  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  3  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  4  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  5  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  6  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  7  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  8  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  9  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
## 10  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
## # ℹ 149 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

Now we are introducing logical statements with the `==` sign! `==` means "is equal to". It is very different from `=`, which specifies arguments in functions (and can be used as an assignment operator, but that is not recommended). A few other useful operators and functions for filtering are:

* `!=`, which means "not equal to." `!` means "not" and can be applied to many logical statements.
* `%in%`, which asks if the left hand side is found anywhere in the vector on the right side.
* `is.na`, which asks if the value is missing.
* `>`, `>=`, `<`, and `<=`, which filter based on the values of numeric columns.


``` r
filter(urban_data, host.class != "mammals")
```

```
## # A tibble: 321 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  2  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  3  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  4  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  5  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  6  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  7  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  8  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  9  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
## 10  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
## # ℹ 311 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

``` r
filter(urban_data, host.class %in% c("birds","fish"))
```

```
## # A tibble: 251 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  2  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  3  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  4  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  5  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  6  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  7  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  8  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  9  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
## 10  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
## # ℹ 241 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

``` r
filter(urban_data, is.na(pval))
```

```
## # A tibble: 253 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1236 Ectoparas… Webste…  2014 Wildli… condi… Adjusted  <NA>    <NA>  <NA>   
##  2  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  3  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  4  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  5  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
##  6  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
##  7  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
##  8  1480 Environme… Garcia…  2005 Bullet… toxic… <NA>      metal   <NA>  <NA>   
##  9  1869 Great Tit… Torne-…  2013 J Orni… stress <NA>      <NA>    <NA>  other  
## 10  1903 halogenat… Olsson…  1999 Scienc… condi… Raw       <NA>    <NA>  <NA>   
## # ℹ 243 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

``` r
filter(urban_data, YEAR >= 2010)
```

```
## # A tibble: 422 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  2  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  3  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Rich… <NA>   
##  4  1236 Ectoparas… Webste…  2014 Wildli… condi… Adjusted  <NA>    <NA>  <NA>   
##  5  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  6  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  7  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  8  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  9  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
## 10  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
## # ℹ 412 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

We can also filter using multiple criteria simultaneously using `&` (AND) and `|` (OR):


``` r
filter(urban_data, host.class != "mammals" & !is.na(pval))
```

```
## # A tibble: 133 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  2  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
##  3  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  4  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  5  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  6  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  7  1426 Egrets as… Boncom…  2003 Archiv… toxic… <NA>      metal   <NA>  <NA>   
##  8  1934 Helminth … Rzad e…  2014 Helmin… paras… <NA>      <NA>    Prev… <NA>   
##  9  2166 Influence… Goulso…  2012 Functi… paras… <NA>      <NA>    Prev… <NA>   
## 10  2166 Influence… Goulso…  2012 Functi… paras… <NA>      <NA>    Prev… <NA>   
## # ℹ 123 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

``` r
filter(urban_data, ptype == "Prevalence" | SAMPLE_SIZE > 100)
```

```
## # A tibble: 218 × 42
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  2  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  3  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Rich… <NA>   
##  4  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  5  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  6  1869 Great Tit… Torne-…  2013 J Orni… stress <NA>      <NA>    <NA>  other  
##  7  1934 Helminth … Rzad e…  2014 Helmin… paras… <NA>      <NA>    Prev… <NA>   
##  8  1955 high freq… Bagagl…  2003 Medica… paras… <NA>      <NA>    Prev… <NA>   
##  9  2129 infection… Robard…  2008 Parasi… paras… <NA>      <NA>    Inte… <NA>   
## 10  2129 infection… Robard…  2008 Parasi… paras… <NA>      <NA>    Inte… <NA>   
## # ℹ 208 more rows
## # ℹ 32 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

Can you describe what is being included and excluded in each of these statements?

Use the `urban_data` data to make a data frame with data from 2000 through 2010.



### Making new columns with `mutate()`

Another common task is making new columns using values in existing columns. For example, we we can create a new column that measures gross domestic product (GDP, the `gdpbill` column) length in tens of billions instead of billions of dollars:


``` r
urban_data <- mutate(urban_data, gdpbill10 = gdpbill/10)
```

`mutate` can also create multiple columns at once, separated by a comma:


``` r
urban_data <- mutate(urban_data, gdpbill10 = gdpbill10/10,
                      years_since_1998 = YEAR - 1998) #not the best column name - a little long
# To view the new columns, we will select them, 
# because this data frame has too many columns to print them all
# and our new columns are placed at the end
select(urban_data, gdpbill, gdpbill10, YEAR, years_since_1998)
```

```
## # A tibble: 516 × 4
##    gdpbill gdpbill10  YEAR years_since_1998
##      <dbl>     <dbl> <dbl>            <dbl>
##  1    926       9.26  2014               16
##  2    926       9.26  2014               16
##  3    926       9.26  2014               16
##  4    926       9.26  2014               16
##  5    926       9.26  2014               16
##  6    926       9.26  2014               16
##  7    926       9.26  2014               16
##  8    926       9.26  2014               16
##  9    130.      1.30  2016               18
## 10    130.      1.30  2016               18
## # ℹ 506 more rows
```

We can also use `mutate` to modify values in existing columns. For example, we see that the transmission mode columns are coded as integers instead of logical:


``` r
unique(urban_data$close)
```

```
## [1]  1  0 NA
```

To replace `1`s with `TRUE`s and `0`s with `FALSE`s, we can use the `as.logical` function:


``` r
urban_data_close <- mutate(urban_data, close = as.logical(close))
unique(urban_data_close$close)
```

```
## [1]  TRUE FALSE    NA
```

Note that this works because R assumes by default that `0` means `FALSE` and `1` means `TRUE.` We will get into `if` and `ifelse` statements later, which would allow you to convert more types of values. Another way to do the same thing would be using a logical statement:


``` r
urban_data_lgl <- mutate(urban_data, nonclose = (nonclose == 1))
unique(urban_data_lgl$nonclose)
```

```
## [1]  TRUE    NA FALSE
```

As with `filter`, we can use `mutate` to create multiple columns at once:


``` r
urban_data_lgl2 <- mutate(urban_data, close = as.logical(close),
                      nonclose = as.logical(nonclose))
```

Finally, we can also apply exactly the same function to multiple columns:


``` r
urban_data_lgl <- mutate(urban_data, across(c(close, nonclose, vector, intermediate), as.logical))
```

Note that this is a little more complicated! The first argument is still the data set (`urban_data`), but the second argument is now a vector of column names, enclosed within `c()`, and the function `across` applies the function to multiple columns. The second argument to `across` is the function that is applied to all the columns.

Another common task is to replace NAs with another value, or to replace certain character strings (e.g., `""`) with `NA`s. For example, for studies with no difference in urbanization across the study area (the `udiff_1000` column), we might want to replace `0` with a missing value. Here, we can use a combination of `mutate` with the `na_if` function from `tidyr`:


``` r
mutate(urban_data, udiff_1000 = na_if(udiff_1000, 0))
```

```
## # A tibble: 516 × 44
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  2  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  3  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Rich… <NA>   
##  4  1236 Ectoparas… Webste…  2014 Wildli… condi… Adjusted  <NA>    <NA>  <NA>   
##  5  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  6  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  7  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  8  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  9  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
## 10  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
## # ℹ 506 more rows
## # ℹ 34 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

Finally, a very useful function for creating new columns is `if_else()` (or the base R version, `ifelse`). This is a simplified version of the `if` statements that we will learn about later in the class. Again makes a vector as a function of another vector, this time conditionally. I provide a conditional statement ("if") and an alternative ("else"). The example above is essentially a streamlined ifelse function; the following code does the same thing:


``` r
mutate(urban_data, udiff_1000 = if_else(udiff_1000 == 0, NA, udiff_1000))
```

```
## # A tibble: 516 × 44
##    study TITLE      AUTHORS  YEAR JOURNAL health condition toxtype ptype stress2
##    <dbl> <chr>      <chr>   <dbl> <chr>   <chr>  <chr>     <chr>   <chr> <chr>  
##  1  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  2  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  3  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Rich… <NA>   
##  4  1236 Ectoparas… Webste…  2014 Wildli… condi… Adjusted  <NA>    <NA>  <NA>   
##  5  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  6  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Inte… <NA>   
##  7  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  8  1236 Ectoparas… Webste…  2014 Wildli… paras… <NA>      <NA>    Prev… <NA>   
##  9  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
## 10  1326 Effects o… Hargit…  2016 Scienc… toxic… <NA>      metal   <NA>  <NA>   
## # ℹ 506 more rows
## # ℹ 34 more variables: SAMPLE_SIZE <dbl>, EFFECT_DIRECTION <dbl>, pval <dbl>,
## #   r <dbl>, yi <dbl>, vi <dbl>, rlower <dbl>, rupper <dbl>, yir <dbl>,
## #   ott_id <dbl>, host.species <chr>, host.class <chr>, aqterr <chr>,
## #   parasite2 <chr>, close <dbl>, nonclose <dbl>, vector <dbl>,
## #   intermediate <dbl>, routes <dbl>, mroute <chr>, mlat <dbl>, mlon <dbl>,
## #   udiff_1000 <dbl>, udiff_10000 <dbl>, uchange_1000 <dbl>, …
```

Here, the first argument is the condition: I am asking whether `udiff_1000` is equal to zero. The second argument is the output if the condition is true: in this place I would like the function to return `NA`. The second argument is the output if the condition is false: in this case I would like it to return the original value of the vector/column, i.e., `udiff_1000`.

Some simpler examples:


``` r
# Create a vector
species <- c("elephant", "snow leopard", "indian leopard", "sloth bear", "bengal tiger")
# Check whether these animals are cats
if_else(species %in% c("snow leopard", "indian leopard", "bengal tiger"), "cat" , "not a cat")
```

```
## [1] "not a cat" "cat"       "cat"       "not a cat" "cat"
```

``` r
# Create a vector
nums <- c(50, 194, 281, 92)
# Check if each element is even or odd
if_else(nums %% 2 == 0,"even","odd")
```

```
## [1] "even" "even" "odd"  "even"
```

Notice that here, `%%` returns the remainder of a division, so numbers divided by two with a remainder of zero are even.

The main difference between the `tidyverse's` `if_else` and base R's `ifelse` is that `if_else` preserves factor levels and other types (for example, dates, which we will learn about soon).


``` r
# Unlike `ifelse()`, `if_else()` preserves types
# This example comes from the help page for if_else
x <- factor(c("a", "i", "e", "f", "g", "c", "i", "b"))
ifelse(x %in% c("a", "b", "c"), x, NA)
```

```
## [1]  1 NA NA NA NA  3 NA  2
```

``` r
if_else(x %in% c("a", "b", "c"), x, NA)
```

```
## [1] a    <NA> <NA> <NA> <NA> c    <NA> b   
## Levels: a b c e f g i
```

### Sorting data tables

In many cases, the order of data in table is not important; the information is the same regardless of its row number. Sorting and ordering tables can be useful for filtering them and for calculating some columns. The `dplyr` function `arrange()` will sort a data frame by a column:


``` r
head(select(urban_data, study, AUTHORS, r))
```

```
## # A tibble: 6 × 3
##   study AUTHORS              r
##   <dbl> <chr>            <dbl>
## 1  1236 Webster et al.  0     
## 2  1236 Webster et al. -0.229 
## 3  1236 Webster et al. -0.0770
## 4  1236 Webster et al.  0.200 
## 5  1236 Webster et al. -0.301 
## 6  1236 Webster et al.  0.063
```

``` r
r_ordered <- arrange(urban_data, r)
head(select(r_ordered, study, AUTHORS, r))
```

```
## # A tibble: 6 × 3
##   study AUTHORS                 r
##   <dbl> <chr>               <dbl>
## 1  3913 Zapata et al.      -0.997
## 2  3913 Zapata et al.      -0.984
## 3   521 Abu El-Saad et al. -0.982
## 4   851 Bilandzic et al.   -0.979
## 5   521 Abu El-Saad et al. -0.977
## 6   521 Abu El-Saad et al. -0.975
```

`arrange()` can take multiple column names, in which case the data frame is sorted first by the first column. Then, if any values are duplicated in that column, the rows are sorted within that group by the second column, and so on:


``` r
author_ordered <- arrange(urban_data, AUTHORS)
head(select(author_ordered, study, AUTHORS, r))
```

```
## # A tibble: 6 × 3
##   study AUTHORS                 r
##   <dbl> <chr>               <dbl>
## 1   521 Abu El-Saad et al. -0.922
## 2   521 Abu El-Saad et al. -0.963
## 3   521 Abu El-Saad et al. -0.378
## 4   521 Abu El-Saad et al. -0.900
## 5   521 Abu El-Saad et al. -0.974
## 6   521 Abu El-Saad et al. -0.952
```

``` r
author_r_ordered <- arrange(urban_data, AUTHORS, r)
head(select(author_r_ordered, study, AUTHORS, r))
```

```
## # A tibble: 6 × 3
##   study AUTHORS                 r
##   <dbl> <chr>               <dbl>
## 1   521 Abu El-Saad et al. -0.982
## 2   521 Abu El-Saad et al. -0.977
## 3   521 Abu El-Saad et al. -0.975
## 4   521 Abu El-Saad et al. -0.974
## 5   521 Abu El-Saad et al. -0.963
## 6   521 Abu El-Saad et al. -0.952
```

And the base R function `sort` will order a vector:


``` r
sort(x)
```

```
## [1] a b c e f g i i
## Levels: a b c e f g i
```

## Pipes

What if you want to do a bunch of operations in order, for example filtering your data and creating new columns. The most basic way to do this is to create intermediate objects:


``` r
urban_data_mamm <- filter(urban_data, host.class == "mammals")
urban_data_mamm <- mutate(urban_data_mamm, across(c(close, nonclose, vector, intermediate), as.logical))
```

You could also nest your functions, but as you can see this gets unwieldy quickly:


``` r
urban_data_mamm <- filter(mutate(urban_data_mamm, across(c(close, nonclose, vector, intermediate), as.logical)), host.class == "mammals")
```

`tidyverse` provides a powerful way to tie a squence of actions together: the pipe (`%>%`). Pipes can initially be intimidating (what are those `%` doing?!) and they are by no means necessary to use, but they can make your code neat and easier to understand. 


``` r
urban_data_mamm <- urban_data %>%
  filter(host.class == "mammals") %>%
  mutate(across(c(close, nonclose, vector, intermediate), as.logical))
```

You can read the pipe as *then*: take the urban data, *then* take only rows for mammals, *then* convert the transmission mode columns to logical. The pipe takes the output of the previous line and feeds it as input into the next one. Notice that you don't have to repeat the name of the data object,because the data is whatever the pipe is feeding into the function. There are 
several advantages to using pipes compared to traditional syntax. First, by 
using a pipe in the example above, we avoided saving intermediate objects (e.g., 
`no_pkey`) to the environment: we only saved the final result we wanted. Second,
we typed less. Third, our code is more readable because the syntax of our code
reflects the logical structure of what we are doing. The shortcut for inserting a pipe
is `Ctrl + Shift + M` on Windows and `Cmd + Shift + M` on Mac.



