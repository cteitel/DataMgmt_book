# Complex Data Structures in R {#lists}

## Objectives

* Become familiar with non-tabular data structures (lists) in R
* Understand the uses of lists and nested data frames
* Be able to create and manipulate lists and nested data frames


## Additional reading

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 23: Hierarchical data. Available: https://r4ds.hadley.nz/rectangling.html

## What is non-tabular data?

So far, we have worked with tabular data only - that is, we have a table where 
each column has the same number of rows (and vice versa), and the data in each 
column has the same structure (all numeric, all character, etc.). In R, these columns
also have limited types of data they can accommodate (again: numeric, character,
factor, etc.). Not all data meets these requirements, which means there are many
more data structures out there. For example, spatial data (which we will cover
soon) contains embedded metadata about the coordinate reference system, and each
data point might be a polygon rather than simply a point with X and Y coordinates.
You can also imagine a case where you might want to have multiple datasets relating
to one another, without duplicating all the information (e.g., associating individual
traits to hourly monitoring data without repeating the individual traits over
and over in your data frame), or where you might want to store multiple models
in a certain order. 

So far, we could only do any of this by creating a lot of separate objects in
the environment - for example, a different named object for each of 10 models. 



## Lists

A list is a hierarchical data structure that can contain any number of data types.
Here's an example:


``` r
l1 <- list("element1", 1, TRUE, c("this","is","a","vector"),
           data.frame(num = c(1,2), lett = c("a","b")))
l1
```

```
## [[1]]
## [1] "element1"
## 
## [[2]]
## [1] 1
## 
## [[3]]
## [1] TRUE
## 
## [[4]]
## [1] "this"   "is"     "a"      "vector"
## 
## [[5]]
##   num lett
## 1   1    a
## 2   2    b
```

Here, our list has four elements, each of which is a different data type or length
(character, numeric, logical, a character vector, and a data frame). Because printing the list
can take up space very quickly, we can summarize the contents of a list with the 
`str()` function:


``` r
str(l1)
```

```
## List of 5
##  $ : chr "element1"
##  $ : num 1
##  $ : logi TRUE
##  $ : chr [1:4] "this" "is" "a" "vector"
##  $ :'data.frame':	2 obs. of  2 variables:
##   ..$ num : num [1:2] 1 2
##   ..$ lett: chr [1:2] "a" "b"
```

Lists can also be named:


``` r
l1 <- list(chr = "element1", int = 1, log = TRUE, chr_v = c("this","is","a","vector"),
           df = data.frame(num = c(1,2), lett = c("a","b")))
str(l1)
```

```
## List of 5
##  $ chr  : chr "element1"
##  $ int  : num 1
##  $ log  : logi TRUE
##  $ chr_v: chr [1:4] "this" "is" "a" "vector"
##  $ df   :'data.frame':	2 obs. of  2 variables:
##   ..$ num : num [1:2] 1 2
##   ..$ lett: chr [1:2] "a" "b"
```

To extract elements from a list, we can use square brackets or names. If using square
brackets, a single bracket will return a *list* and double brackets will return
the element in its own structure:


``` r
l1[1]
```

```
## $chr
## [1] "element1"
```

``` r
l1[[1]]
```

```
## [1] "element1"
```

The behavior might seem inconvenient, but means that we can subset a list using
single brackets, but extract its elements using double brackets:


``` r
l1[c(1,2)] # Returns a list with the first two elements
```

```
## $chr
## [1] "element1"
## 
## $int
## [1] 1
```

``` r
l1[[c(1,2)]] # Does not work
```

```
## Error in l1[[c(1, 2)]]: subscript out of bounds
```

Using a name works just like using double brackets.


``` r
l1$chr
```

```
## [1] "element1"
```

Finally, lists are designed to be hierarchical - in other words, each element of
a list can also be a list (and so on and so forth).


``` r
l2 <- list(
  l1 = list(v1 = 1, v2 = 2, v2 = "yes"),
  l2 = list(v1 = 3, v2 = "element2")
)
str(l2)
```

```
## List of 2
##  $ l1:List of 3
##   ..$ v1: num 1
##   ..$ v2: num 2
##   ..$ v2: chr "yes"
##  $ l2:List of 2
##   ..$ v1: num 3
##   ..$ v2: chr "element2"
```

This property makes lists useful for containing data that is inherently hierarchical,
for example a taxonomy (here, of hominids):


``` r
hominid_taxonomy <- list(
  Ponginae = list(
    Pongo = c("pygmaeus","abelii","tapanuliensis")
  ),
  Homininae = list(
    Gorilla = c("gorilla", "beringei"),
    Pan = c("paniscus", "troglodytes"),
    Homo = c("sapiens")
  )
)
hominid_taxonomy
```

```
## $Ponginae
## $Ponginae$Pongo
## [1] "pygmaeus"      "abelii"        "tapanuliensis"
## 
## 
## $Homininae
## $Homininae$Gorilla
## [1] "gorilla"  "beringei"
## 
## $Homininae$Pan
## [1] "paniscus"    "troglodytes"
## 
## $Homininae$Homo
## [1] "sapiens"
```

Here, if I wanted to extract the species names within the genus *Homo*, I would use
one of the following:


``` r
hominid_taxonomy$Homininae$Homo
```

```
## [1] "sapiens"
```

``` r
hominid_taxonomy[["Homininae"]][["Homo"]]
```

```
## [1] "sapiens"
```

## Nested data frames (list-columns)

We can combine lists and data frames in `tidyverse` by making a list into a column.
To do so, the list must have the same number of elements as there are rows in the
data frame, so that each row corresponds to one element of the list. Here's an example:


``` r
dat <- tibble(
  ID = c("A", "B", "C"),
  ID_num = c(1, 2, 3),
  list_info = list(list("note", "note"), list("note"), list("note", "note", "note"))
)
dat
```

```
## # A tibble: 3 × 3
##   ID    ID_num list_info 
##   <chr>  <dbl> <list>    
## 1 A          1 <list [2]>
## 2 B          2 <list [1]>
## 3 C          3 <list [3]>
```

You can see that we now have a tibble with three columns, one of which is a list.

This data structure is very useful under two scenarios: (1) the data itself is nested
and you want to perform operations separately on the nested and non-nested portions, 
and (2) you want to store models for subsets of data. For example, if I have information
for 100 different species but want to analyze them separately, I could group my
data by species such that I have a list-column with all the data for that species,
then loop across that list to run my analysis. This is much easier than having 100
data objects and 100 model objects (one for each species).

If the list-column contains a data frame, we can also easily nest and unnest the data.
For example, for the urban wildlife data, I might want to nest my data by the type of 
health metric measured. To do so, I use the `nest()` function in `tidyr` (part
of `tidyverse`):


``` r
urban_data <- read_csv(here("data/raw/Murray-Sanchez_urban-wildlife.csv"))
nrow(urban_data)
```

```
## [1] 516
```

``` r
urban_data_bymetric <- nest(urban_data, data = -health)
nrow(urban_data_bymetric)
```

```
## [1] 4
```

``` r
urban_data_bymetric
```

```
## # A tibble: 4 × 2
##   health    data               
##   <chr>     <list>             
## 1 parasites <tibble [194 × 41]>
## 2 condition <tibble [60 × 41]> 
## 3 toxicants <tibble [189 × 41]>
## 4 stress    <tibble [73 × 41]>
```

Now, I have created a data frame with four rows, one for each health metric, and a 
column called `data` that contains a list of tibbles, each of which contains all
the other columns corresponding to that health metric. The first argument in `nest()` 
indicates the data frame to work with, while the second indicates which columns 
to nest; in this case, we want to nest all columns *except* `health`. This is 
conceptually similar to using `group_by()`. 

To remove the list-column, we can use `unnest()`:


``` r
urban_data2 <- unnest(urban_data_bymetric, data)
```

The first argument in `unnest()`indicates the data frame to work with, while the 
second indicates which columns to unnest (in theory, you could have multiple list-
columns in your data set).

There are two related functions, `unnest_wider()` and `unnest_longer()`, that
you can also explore, which allow you to unnest lists (above, our list-column 
was a data frame, which made it more obvious how to unnest it).

## Applying functions across lists (`map`)

Storing data as a list makes sense in many cases, but it is much more powerful
when we can apply functions (actions) across the list. 

The `map()` function in `tidyverse` (equivalent to `lapply()` for the base R users
among you) works similarly to `across()`, but instead of applying to columns in
a data frame, it applies to elements in a list. The functions enclosed in `map()`
often become more complex that those you might use in `across()` because they
can apply to data frames or lists, not just vectors.

For example, let's say we wanted to know the number of data points in each of our
health metric groups. We could acheive this by using `map()` on the list-column:


``` r
map(urban_data_bymetric$data, ~nrow(.x))
```

```
## [[1]]
## [1] 194
## 
## [[2]]
## [1] 60
## 
## [[3]]
## [1] 189
## 
## [[4]]
## [1] 73
```

Notice that `map()` always returns a list. However, there are other forms of `map()`
that create a vector, which you can use if your function will return something
that makes sense as a vector. These functions include `map_dbl()` for numeric,
`map_int()` for integers, `map_chr()` for characters, and `map_lgl()` for logical:


``` r
map_int(urban_data_bymetric$data, ~nrow(.x))
```

```
## [1] 194  60 189  73
```

To make this even more powerful, we can integrate it with `mutate` to add a new column
to our data set:


``` r
urban_data_bymetric <- urban_data_bymetric %>%
  mutate(n_obs = map_int(data, ~nrow(.x)))
urban_data_bymetric
```

```
## # A tibble: 4 × 3
##   health    data                n_obs
##   <chr>     <list>              <int>
## 1 parasites <tibble [194 × 41]>   194
## 2 condition <tibble [60 × 41]>     60
## 3 toxicants <tibble [189 × 41]>   189
## 4 stress    <tibble [73 × 41]>     73
```

Here, we created a new column, `n_obs`, by mapping across the `data` list-column and
pulling out the number of rows. Of course, this would have easily been doable with
`group_by()` and `summarize()`, but we would have "lost" the rest of the data,
at least temporarily, in doing so.

## Saving lists and nested data frames

So far, we have been able to save all our data as a .csv. CSVs are tabular, and
so was our data. But opening a list outside of R is harder, so how can we save and
communicate these types of data?

One way to save data is as an RDS object. This is a compressed data format that 
will preserve all the features of your data; when you read it back in, everyting
will remain the same (for example, if you manually set factor levels). It can also
accomodate pretty much any data structure, including lists and nested data frames.
To create an RDS object, use the `saveRDS()` function and include the .rds file
extension:


``` r
saveRDS(urban_data_bymetric, here("data/clean/urban_data_nested.rds"))
```

While RDS objects are great for your own uses (for example, to save intermediate
data products), they're not great for sharing because they can't easily be opened
by anything other than R. For these cases, you are usually best off saving separate
.csv files (for example, one for each health metric) in a new folder that contains
them all. 


``` r
dir.create(here("data/clean/nested_urban_data/")) # make a new directory for the files
for(i in 1:nrow(urban_data_bymetric)){
  file_name <- str_c(urban_data_bymetric$health[i], ".csv") #create a new name for the file
  dat <- urban_data_bymetric$data[[i]] # extract the data to save
  write_csv(dat, here("data/clean/nested_urban_data/", file_name)) # save!
}
dir(here("data/clean/nested_urban_data/")) # Check that files have been created
```

Then, when you want to read them back in, you can use `map()` to re-create a list
or tibble:


``` r
files <- dir(here("data/clean/nested_urban_data/")) # Get all file names
health <- str_remove(files, ".csv") # Extract metric names from file names
urban_data2 <- tibble(
  health,
  data = map(files, ~read_csv(here("data/clean/nested_urban_data/", .x)))
)
urban_data2 
```

```
## # A tibble: 4 × 2
##   health    data                 
##   <chr>     <list>               
## 1 condition <spc_tbl_ [60 × 41]> 
## 2 parasites <spc_tbl_ [194 × 41]>
## 3 stress    <spc_tbl_ [73 × 41]> 
## 4 toxicants <spc_tbl_ [189 × 41]>
```
