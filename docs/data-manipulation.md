# Data Manipulation with R, Part 2 {#manipulation}

Much of this lesson draws from The Carpentries' [Data Analysis and Visualization in R for Ecologists](https://datacarpentry.github.io/R-ecology-lesson/index.html) workshop, which is published under a [CC-BY 4.0](https://datacarpentry.github.io/R-ecology-lesson/LICENSE.html) license.

## Objectives

* Be able to use a split-apply-combine approach to summarize data sets
* Understand how to augment datasets, both by adding observations and by adding variables
* Understand different types of joins and relates

## Additional reading

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 3: Data transformation, sections 3.5-3.7. Available: https://r4ds.hadley.nz/data-transform.html

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 5: Data tidying. Available: https://r4ds.hadley.nz/data-transform.html

## Grouping and summarizing data

One of the most common tasks in data exploration is summarizing data. For example, we might want to know the mean of a response variable across a treatment, or how many samples were taken on a given day, and so on. `dplyr` allows us to easily summarize data by combining the functions `group_by()` and `summarize()`. For example:


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

``` r
urban_data %>%
  group_by(host.class) %>%
  summarize(mean_r = mean(r, na.rm=T), n = n())
```

```
## # A tibble: 5 × 3
##   host.class     mean_r     n
##   <chr>           <dbl> <int>
## 1 birds         -0.141    159
## 2 fish          -0.216     92
## 3 herpetofauna  -0.137     27
## 4 invertebrates -0.400     43
## 5 mammals        0.0232   195
```

We end up with a mean value of *r* for each host group and the number of records in that group. A common function you will use when grouping is `n()`, which gives the current group size. Other common summary functions include:

* `n_distinct()`, which gives the number of unique values in a group and takes a column name as its argument;
* quantitative summaries, like `mean()`, `median()`, `sd()`, and `sum()`;


``` r
urban_data %>%
  group_by(host.class) %>%
  summarize(mean_r = mean(r, na.rm=T), n = n(), n_studies = n_distinct(study))
```

```
## # A tibble: 5 × 4
##   host.class     mean_r     n n_studies
##   <chr>           <dbl> <int>     <int>
## 1 birds         -0.141    159        46
## 2 fish          -0.216     92         8
## 3 herpetofauna  -0.137     27         8
## 4 invertebrates -0.400     43        10
## 5 mammals        0.0232   195        35
```

## Combining and augmenting data sets

Often, we have multiple data sets that relate to each other in one way or another, or we want to combine multiple sources of the same data. For example, we might have separate data sheets from multiple field seasons and want to combine them. Or, we might have one data table that describes characteristics of plots and one that describes characteristics of trees, and we want to add the plot information for each tree. 

### Binding rows and columns

Adding ("binding") rows and columns is the easiest way to add data to a data frame. For example, if we have two separate data sets, we can add more rows with `bind_rows`:


``` r
ds1 <- data.frame(plot = 1:10, observer = "Me", n_obs = sample(1:100, 10), date = "2024-04-01")
ds2 <- data.frame(plot = 1:10, observer = "You", n_obs = sample(1:100, 10), date = "2025-01-01")

ds_full <- bind_rows(ds1, ds2)
nrow(ds_full)
```

```
## [1] 20
```

``` r
head(ds_full)
```

```
##   plot observer n_obs       date
## 1    1       Me    92 2024-04-01
## 2    2       Me    60 2024-04-01
## 3    3       Me    49 2024-04-01
## 4    4       Me    43 2024-04-01
## 5    5       Me     1 2024-04-01
## 6    6       Me    30 2024-04-01
```

``` r
tail(ds_full)
```

```
##    plot observer n_obs       date
## 15    5      You    62 2025-01-01
## 16    6      You    42 2025-01-01
## 17    7      You    22 2025-01-01
## 18    8      You    56 2025-01-01
## 19    9      You    92 2025-01-01
## 20   10      You     8 2025-01-01
```

`bind_rows` looks for column names to know how to match up the data. This is great when your columns are in different orders, but be careful with column names:


``` r
ds1 <- data.frame(plot = 1:10, observer = "Me", n_obs = sample(1:100, 10))
ds2 <- data.frame(Plot = 1:10, obs = "You", N = sample(1:100, 10))

ds_full <- bind_rows(ds1, ds2)
nrow(ds_full)
```

```
## [1] 20
```

``` r
head(ds_full)
```

```
##   plot observer n_obs Plot  obs  N
## 1    1       Me    76   NA <NA> NA
## 2    2       Me    34   NA <NA> NA
## 3    3       Me    78   NA <NA> NA
## 4    4       Me    11   NA <NA> NA
## 5    5       Me    38   NA <NA> NA
## 6    6       Me    63   NA <NA> NA
```

We can also add more information with `bind_cols`, though joins (below) are usually a better way to add columns to a data set. `bind_cols` is also useful for creating new data frames from vectors:


``` r
plots <- c(1:10)
observers <- rep("Me", 10)
N <- sample(1:100, 10)

ds1 <- bind_cols(plot = plots, observer = observers, n_obs = N)
```

### Joins

There are several types of join. The most common types are left join, inner join, or full join. To understand this terminology, consider this: whenever you are joining two tables, the first table you mention (the one to which you’re joining) is called the left table, whereas the second table (the one you’re joining to the first) is called the right table. With a left join, you keep all the records in the left table and add information from the right table whenever there’s a matching row. A full join means that you retain all rows from both tables, matching them whenever possible. An inner join means that you only retain the rows that match between the two tables.

Let's practice with the Murray et al. data. First, we need to download some extra data to add. The EltonTraits database provides foraging attributes and body mass for birds and mammals. These might be interesting, for example to see if species' responses to urbanization vary depending on their body size or foraging traits. These data can be downloaded from [Figshare](https://figshare.com/collections/EltonTraits_1_0_Species-level_foraging_attributes_of_the_world_s_birds_and_mammals/3306933).


``` r
elton <- read_csv("data/raw/EltonTraits/BirdFuncDat.csv")
```

```
## Rows: 10205 Columns: 40
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (17): SpecID, PassNonPass, IOCOrder, BLFamilyLatin, BLFamilyEnglish, Tax...
## dbl (23): BLFamSequID, Diet-Inv, Diet-Vend, Diet-Vect, Diet-Vfish, Diet-Vunk...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

``` r
urban_birds <- filter(urban_data, host.class == "birds")
```

This can be a little tricky: EltonTraits and the Murray et al. database use different taxonomies, so there are a few birds in `urban_birds` that aren't in EltonTraits. To really use these data we would need to correct those discrepancies, but for now this will help illustrate the different types of joins. 


``` r
# Which birds are missing?
urban_birds %>%
  filter(!host.species %in% elton$Scientific) %>%
  select(host.species)
```

```
## # A tibble: 9 × 1
##   host.species     
##   <chr>            
## 1 Abrornis_inornata
## 2 Kieneria_aberti  
## 3 Kieneria_aberti  
## 4 Kieneria_aberti  
## 5 Kieneria_aberti  
## 6 Kieneria_aberti  
## 7 Kieneria_aberti  
## 8 Kieneria_aberti  
## 9 Kieneria_aberti
```

Left joins are probably the most common type you will use, because they help add new information to a reference data set.


``` r
urban_traits <- left_join(urban_birds, elton, by = c("host.species" = "Scientific"))
ncol(urban_birds)
```

```
## [1] 42
```

``` r
ncol(urban_traits)
```

```
## [1] 81
```

The `by` argument is the second-trickiest part of a join (the trickiest part is making sure it did what you expected). Here, we specify that we want to match up values in the column `host.species` from the first data frame with values in the column `Scientific` in the second data set. The `tidyverse` join functions will automatically use columns with the same names if you don't specify `by`. If you specify a single character, it will look for columns with that name in both data sets. Finally, you can specify multiple columns (for example, if you had a study design with `block` nested within `plot` and you wanted to match both).

An inner join would be similar, but would eliminate those species that are not in EltonTraits, because they are not present in both data sets:


``` r
urban_traits <- inner_join(urban_birds, elton, by = c("host.species" = "Scientific"))
nrow(urban_birds)
```

```
## [1] 159
```

``` r
nrow(urban_traits)
```

```
## [1] 150
```

An full join would be helpful if you want to include all the effect sizes and all the trait information in a single data set, regardless of whether you have the other information:


``` r
urban_traits <- full_join(urban_birds, elton, by = c("host.species" = "Scientific"))
ncol(urban_traits)
```

```
## [1] 81
```

``` r
nrow(urban_traits)
```

```
## [1] 10334
```

## Pivoting between wide and long format

We talked about wide and long formats when discussing data entry and spreadsheets, but it is still useful to know how to transform your data sets once they get into R. Why? Well, sometimes you will have a data set that isn't set up in long form. Other times, wide form is useful for visualization or making tables.

### Long to wide

Let's start with a data set in long form: one observation per row, one variable per column. The urban wildlife data set follows this structure. Let's say we want to know how many studies we have per year and health metric:


``` r
urban_data %>%
  group_by(health, YEAR) %>%
  summarize(n = n())
```

```
## `summarise()` has grouped output by 'health'. You can override using the
## `.groups` argument.
```

```
## # A tibble: 54 × 3
## # Groups:   health [4]
##    health     YEAR     n
##    <chr>     <dbl> <int>
##  1 condition  1999     6
##  2 condition  2002     1
##  3 condition  2003     3
##  4 condition  2005     2
##  5 condition  2008     5
##  6 condition  2010    10
##  7 condition  2011     4
##  8 condition  2012    13
##  9 condition  2013     1
## 10 condition  2014     3
## # ℹ 44 more rows
```

To present this in a paper, we might consider transforming it to wide form:


``` r
urban_data %>%
  group_by(health, YEAR) %>%
  summarize(n = n()) %>%
  pivot_wider(id_cols = YEAR, names_from = health, values_from = n, values_fill = 0)
```

```
## `summarise()` has grouped output by 'health'. You can override using the
## `.groups` argument.
```

```
## # A tibble: 20 × 5
##     YEAR condition parasites stress toxicants
##    <dbl>     <int>     <int>  <int>     <int>
##  1  1999         6         0      0         0
##  2  2002         1         1      2         0
##  3  2003         3         1      0         5
##  4  2005         2         1      0         5
##  5  2008         5        13      4         4
##  6  2010        10         5      0        36
##  7  2011         4        44      4         0
##  8  2012        13        20     15        15
##  9  2013         1         6      8        18
## 10  2014         3        46      9         0
## 11  2016        11        26     12         7
## 12  2017         1         6      6        61
## 13  1998         0         3      0         0
## 14  2000         0         2      1         0
## 15  2001         0         1      0         0
## 16  2004         0        11      1         1
## 17  2006         0         1      0         0
## 18  2009         0         1      8         0
## 19  2015         0         6      2        27
## 20  2007         0         0      1        10
```

Here we use the `pivot_wider()` function from the `tidyr` package (part of the `tidyverse`). Here `id_cols` tells the function which variable to use to define a row in the new table. `names_from` defines the column in the long-form data that will become the column names, and `values_from` defines the column in the long-form data that will be used to fill those columns. Finally, you don't always need `values_fill`, but when an id-name combination is missing in the original data, this tells the funciton what to include in the new table (by default, it will be `NA`).

Pivoting data frames takes some practice. One of the most common pitfalls in going from wide to long form is that each id-name combination needs to have a unique value. For example, let's say we first summarized the data by both year and host habitat:


``` r
urban_data %>%
  group_by(health, YEAR, aqterr) %>%
  summarize(n = n()) %>%
  pivot_wider(id_cols = YEAR, names_from = health, values_from = n)
```

```
## `summarise()` has grouped output by 'health', 'YEAR'. You can override using
## the `.groups` argument.
```

```
## Warning: Values from `n` are not uniquely identified; output will contain list-cols.
## • Use `values_fn = list` to suppress this warning.
## • Use `values_fn = {summary_fun}` to summarise duplicates.
## • Use the following dplyr code to identify duplicates.
##   {data} |>
##   dplyr::summarise(n = dplyr::n(), .by = c(YEAR, health)) |>
##   dplyr::filter(n > 1L)
```

```
## # A tibble: 20 × 5
## # Groups:   YEAR [20]
##     YEAR condition parasites stress    toxicants
##    <dbl> <list>    <list>    <list>    <list>   
##  1  1999 <int [1]> <NULL>    <NULL>    <NULL>   
##  2  2002 <int [1]> <int [1]> <int [1]> <NULL>   
##  3  2003 <int [1]> <int [1]> <NULL>    <int [1]>
##  4  2005 <int [1]> <int [1]> <NULL>    <int [1]>
##  5  2008 <int [1]> <int [1]> <int [1]> <int [1]>
##  6  2010 <int [2]> <int [1]> <NULL>    <int [2]>
##  7  2011 <int [1]> <int [1]> <int [1]> <NULL>   
##  8  2012 <int [2]> <int [1]> <int [2]> <int [2]>
##  9  2013 <int [1]> <int [1]> <int [2]> <int [1]>
## 10  2014 <int [2]> <int [2]> <int [1]> <NULL>   
## 11  2016 <int [2]> <int [2]> <int [2]> <int [1]>
## 12  2017 <int [1]> <int [1]> <int [1]> <int [2]>
## 13  1998 <NULL>    <int [1]> <NULL>    <NULL>   
## 14  2000 <NULL>    <int [1]> <int [1]> <NULL>   
## 15  2001 <NULL>    <int [1]> <NULL>    <NULL>   
## 16  2004 <NULL>    <int [2]> <int [1]> <int [1]>
## 17  2006 <NULL>    <int [1]> <NULL>    <NULL>   
## 18  2009 <NULL>    <int [1]> <int [1]> <NULL>   
## 19  2015 <NULL>    <int [1]> <int [1]> <int [1]>
## 20  2007 <NULL>    <NULL>    <int [1]> <int [2]>
```

You see this gave us a warning message and that the new columns are lists. This is because some of the cells in this wide data frame now have more than one element (i.e., years where both aquatic and terrestrial hosts had data for a given health metric). We can fix this by including `aqterr` as either and ID or a names column:


``` r
urban_data %>%
  group_by(health, YEAR, aqterr) %>%
  summarize(n = n()) %>%
  pivot_wider(id_cols = c(YEAR, aqterr), names_from = health, values_from = n, values_fill = 0) %>%
  head()
```

```
## `summarise()` has grouped output by 'health', 'YEAR'. You can override using
## the `.groups` argument.
```

```
## # A tibble: 6 × 6
## # Groups:   YEAR [6]
##    YEAR aqterr      condition parasites stress toxicants
##   <dbl> <chr>           <int>     <int>  <int>     <int>
## 1  1999 aquatic             6         0      0         0
## 2  2002 terrestrial         1         1      2         0
## 3  2003 terrestrial         3         1      0         5
## 4  2005 terrestrial         2         1      0         5
## 5  2008 terrestrial         5        13      4         0
## 6  2010 aquatic             8         0      0         6
```

``` r
urban_data %>%
  group_by(health, YEAR, aqterr) %>%
  summarize(n = n()) %>%
  pivot_wider(id_cols = YEAR, names_from = c(health, aqterr), values_from = n, values_fill = 0) %>%
  head()
```

```
## `summarise()` has grouped output by 'health', 'YEAR'. You can override using
## the `.groups` argument.
```

```
## # A tibble: 6 × 9
## # Groups:   YEAR [6]
##    YEAR condition_aquatic condition_terrestrial parasites_terrestrial
##   <dbl>             <int>                 <int>                 <int>
## 1  1999                 6                     0                     0
## 2  2002                 0                     1                     1
## 3  2003                 0                     3                     1
## 4  2005                 0                     2                     1
## 5  2008                 0                     5                    13
## 6  2010                 8                     2                     5
## # ℹ 5 more variables: parasites_aquatic <int>, stress_aquatic <int>,
## #   stress_terrestrial <int>, toxicants_terrestrial <int>,
## #   toxicants_aquatic <int>
```

In the second case, we end up with the health and habitat data being sandwiched together into new column names.

### Wide to long

In the other direction, we sometimes end up with data that is wide and needs to be long. For example, let's say we have a data set with counts by site across years:


``` r
survey_data <- sample(0:100, 10*12, replace = T) %>% 
  matrix(nrow = 10, ncol = 12) %>%
  as.data.frame() %>%
  setNames(str_c("site",1:12)) %>%
  bind_cols(year = 2000:2009) %>% 
  select(year, everything())
survey_data
```

```
##    year site1 site2 site3 site4 site5 site6 site7 site8 site9 site10 site11
## 1  2000    51    69    32    75    21    34    81    79    30     93     77
## 2  2001    26    11    11    28    62     1    76    36    24     24     74
## 3  2002    61    91    15    76    53    15    59    27    25     22     92
## 4  2003    92     1    84    82    60    94    67    93    35      7     69
## 5  2004    17    83    90    52    49    87    33    93    61     11      2
## 6  2005     7    11    16    83    21    18    45    36    37     42     70
## 7  2006    13    87    47    92    70    83     7    28    93     26     41
## 8  2007   100    81    17    47    25    76    62    22     3     16     95
## 9  2008    55    30    54    29    84    43    52    95    42     34      3
## 10 2009    67    81    35    92    49    77    87    86    86     29     20
##    site12
## 1      95
## 2      64
## 3      40
## 4      23
## 5      18
## 6       4
## 7      18
## 8      29
## 9      47
## 10     38
```

(Side note: this is an example of how to simulate data to test code. We can go into this in more detail later in the course.)


``` r
survey_data %>%
  pivot_longer(cols = -year, names_to = "site", values_to = "count")
```

```
## # A tibble: 120 × 3
##     year site   count
##    <int> <chr>  <int>
##  1  2000 site1     51
##  2  2000 site2     69
##  3  2000 site3     32
##  4  2000 site4     75
##  5  2000 site5     21
##  6  2000 site6     34
##  7  2000 site7     81
##  8  2000 site8     79
##  9  2000 site9     30
## 10  2000 site10    93
## # ℹ 110 more rows
```

Here, we use the `pivot_longer()` function, also from `tidyr` and identify the columns that we want to pivot with `cols`. Because we want to get values from all the columns *except* the `year` column, we use a minus sign (-) to indicate that. We could also list all the columns (`c(site1, site2, site3)`), but this would be unwieldy. We also specify the column names of the new columns we are creating: one for what were the column names in the old wide data frame (`names_to`) and one for what were the values. 

One common challenge with pivoting longer is when we have a data frame that has more columns than the ones we want to include in the long data. For example, let's say our survey data also included information about the observer:


``` r
survey_data <- mutate(survey_data, observer = sample(c("A","B","C"), nrow(survey_data), replace = T))
survey_data %>%
  pivot_longer(cols = -year, names_to = "site", values_to = "count")
```

```
## Error in `pivot_longer()`:
## ! Can't combine `site1` <integer> and `observer` <character>.
```

Here, we get an error because `pivot_longer()` is trying to make a single column ("count") that includes both numbers and a character. We can get around this by removing the observer from the data before pivoting or, if we want that information, including it in our column specification:


``` r
survey_data %>%
  select(-observer) %>%
  pivot_longer(cols = -year, names_to = "site", values_to = "count") %>%
  head()
```

```
## # A tibble: 6 × 3
##    year site  count
##   <int> <chr> <int>
## 1  2000 site1    51
## 2  2000 site2    69
## 3  2000 site3    32
## 4  2000 site4    75
## 5  2000 site5    21
## 6  2000 site6    34
```

``` r
survey_data %>%
  pivot_longer(cols = -c(year, observer), names_to = "site", values_to = "count") 
```

```
## # A tibble: 120 × 4
##     year observer site   count
##    <int> <chr>    <chr>  <int>
##  1  2000 C        site1     51
##  2  2000 C        site2     69
##  3  2000 C        site3     32
##  4  2000 C        site4     75
##  5  2000 C        site5     21
##  6  2000 C        site6     34
##  7  2000 C        site7     81
##  8  2000 C        site8     79
##  9  2000 C        site9     30
## 10  2000 C        site10    93
## # ℹ 110 more rows
```




