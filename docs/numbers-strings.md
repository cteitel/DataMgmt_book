# Numbers, Characters, and Factors in R {#nums-chrs}

## Objectives

* Perform basic functions on numeric and integer objects and columns
* Manipulate character/string objects and columns
* Create and convert factors

In this lesson and the following one about dates and times, we will learn basic principles on simple R objects, then apply them to columns of data frames to build on your knowledge of data management (`mutate`, etc.).

## Additional reading

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 13: Numbers. Available: https://r4ds.hadley.nz/numbers.html

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 14: Strings. Available: https://r4ds.hadley.nz/strings.html

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 16: Factors. Available: https://r4ds.hadley.nz/factors.html

## Numbers: integers and numeric variables

In R, numbers come in two flavors: integers and double (both called "numeric"). Integers are non-decimal, negative or non-negative numbers. Although integers are, to some extent, a subset of all numbers, they sometimes act differently when coding. For example, some functions only take integers as their inputs.


``` r
num1 <- 10
class(num1)
```

```
## [1] "numeric"
```

``` r
num2 <- as.integer(10)
class(num2)
```

```
## [1] "integer"
```

``` r
num3 <- 10L
class(num3)
```

```
## [1] "integer"
```

Performing math on integers will sometimes automatically convert them to double:


``` r
class(num2/3)
```

```
## [1] "numeric"
```

You might expect R to return `NA` if you try to convert a decimal into an integer, but instead it truncates:


``` r
as.integer(2.7)
```

```
## [1] 2
```

The distinction between integers and numeric objects becomes most important when using them as arguments in functions. For example, the function `round()` takes an argument `digits` telling it how many digits to round to. For example:


``` r
round(12.2918, digits = 0)
```

```
## [1] 12
```

``` r
round(12.2918, digits = 3)
```

```
## [1] 12.292
```

``` r
round(12.2918, digits = -1)
```

```
## [1] 10
```

In contrast, `round(12.2918, digits = 1.2)` doesn't make any sense (though R will still return a value, in this case without an error message). This can become important when using objects/variables as arguments in functions:


``` r
round_val <- 1
round(12.2918, digits = round_val)
```

```
## [1] 12.3
```

``` r
# That was too big, let's make it smaller
round_val2 <- round_val/2
round(12.2918, digits = round_val2)
```

```
## [1] 12.3
```

With both integers and numeric variables, we can perform basic arithmetic:


``` r
12 + 10 + 4
```

```
## [1] 26
```

``` r
12 - 10 - 4
```

```
## [1] -2
```

``` r
12 * 10 * 4
```

```
## [1] 480
```

``` r
vals <- c(12,10,4)
sum(vals)
```

```
## [1] 26
```

``` r
prod(vals)
```

```
## [1] 480
```

``` r
diff(vals) #Nope, this is not the same as the arithmetic above. What is it doing?
```

```
## [1] -2 -6
```

You can use most mathematical operations in R on numbers or vectors of numbers. A few examples include:


``` r
sqrt()
mean()
median()
abs()
```

### Parsing numbers

Sometimes, when you import numeric data, it doesn't read in correctly and ends up as a string/character instead of as a number. This is usually because there are typos or oddities in the raw data. The `readr` `tidyverse` package provides the `parse_number()` function to help with these issues. 


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
nums_in <- c("10 days", "2 days", "<1 days", "190d")
readr::parse_number(nums_in)
```

```
## [1]  10   2   1 190
```

But, please don't enter data this way! For example, if the data were in different units, `parse_number` would not know this. When we go into strings below we will learn about another way to deal with these irregularities in data entry.

### Precision and comparing numbers

In the previous lesson, we learned about equalities (`==` and `!=`). These can be used for numbers, too:


``` r
num <- 4/2
num == 2
```

```
## [1] TRUE
```

However, there is a limited degree of precision in doubles, which can only take up a fixed amount of memory. For example, numbers with inifite decimal places (e.g., $\sqrt{2}$, $\pi$) are not infinite.


``` r
sqrt2 <- sqrt(2)
sqrt2
```

```
## [1] 1.414214
```

``` r
two <- sqrt(2)^2
two
```

```
## [1] 2
```

``` r
two == 2
```

```
## [1] FALSE
```

Instead, you can use `dplyr::near()`, which uses the system's default precision to determine equality:


``` r
dplyr::near(two, 2)
```

```
## [1] TRUE
```

## Characters

Characters (or "strings") seem simple on the surface: to some extent, they are exactly what they appear to be. But, dealing with data inputs and outputs can become a lot easier when you know how to manipulate strings automatically. The `tidyverse` package called `stringr` provides a lot of very useful functions for this. Many of these are present in base R, but with slightly less intuitive function names and syntax.

### Creating strings and using special characters

We have used strings before: they are just characters enclosed in quotes:


``` r
string1 <- "string"
string_vect <- c("hello", "I", "am", "a", "vector")
```

To create a quotes within a string, use a combination of single quotes (`''`) and  double quotes (`""`):


``` r
"I said, 'I did not know you could do this'"
'"You totally can", she said, "and it works either way"'
""Just don't try this""
```

```
## Error in parse(text = input): <text>:3:3: unexpected symbol
## 2: '"You totally can", she said, "and it works either way"'
## 3: ""Just
##      ^
```

If you forget to close a quote, you won't get an error: instead, you’ll see `+` and won't be able to do anything else, because all your code is being entered as part of the string:

```
> "I forgot to close my quotation marks
+ 
+ 
```

If you can’t figure out which quote to close, press Escape to cancel.

Some characters act unusually within strings, usually because they also code for something else. For example, the backslash `\` is the escape character, meaning that R will now ignore the programmatic meaning of whatever comes after it. This can be useful, for example if you want to include a single quotation mark within a string:


``` r
quotestring <- "This is a single quotation mark: \", but I made it work!"
quotestring
```

```
## [1] "This is a single quotation mark: \", but I made it work!"
```

``` r
cat(quotestring)
```

```
## This is a single quotation mark: ", but I made it work!
```

Backslashes will be helpful when you want to insert special characters, for example Greek letters in figure and table legends, or non-English characters. We can do this by looking up the Unicode character codes. For example the Unicode character for beta is U+03B2. This is a more advanced topic, but is useful to keep in mind:


``` r
"\u03b2"
```

```
## [1] "β"
```

But if you want to include a backslash in your string, you then need to escape *that*:


``` r
cat("Backslash \\")
```

```
## Backslash \
```

You probably noticed that we used `cat()` here instead of `print()` and that this changed the way the string showed up. This is because `cat()` prints text to the screen, while `print()` (and just the object name alone) show the actual object. It's a subtle difference but is important in some cases.

Sometimes, we might want to create a string out of multiple strings. For example, we might have two vectors of strings that we want to combine:


``` r
colors <- c("yellow", "orange", "purple")
fruits <- c("banana", "tangerine", "blueberry")
str_c(colors, fruits, sep = " ")
```

```
## [1] "yellow banana"    "orange tangerine" "purple blueberry"
```

``` r
# Equivalent "base R" way of doing this:
#paste(colors, fruits)
```

We've put together the corresponding authors and years. We can also do this with a single value:


``` r
str_c("This is a", colors, fruits, ".", sep = " ")
```

```
## [1] "This is a yellow banana ."    "This is a orange tangerine ."
## [3] "This is a purple blueberry ."
```

If we are picky, we could nest our functions to get rid of the space before each period:


``` r
str_c("This is a ", str_c(colors, fruits, sep = " "), ".")
```

```
## [1] "This is a yellow banana."    "This is a orange tangerine."
## [3] "This is a purple blueberry."
```

### Manipulating strings with `stringr`

Sometimes, character strings exist but not in the form we want, or we want to create new strings from old ones. For example, take our urban wildlife data set:


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
head(urban_data$host.species)
```

```
## [1] "Trichosurus_vulpecula" "Trichosurus_vulpecula" "Trichosurus_vulpecula"
## [4] "Trichosurus_vulpecula" "Trichosurus_vulpecula" "Trichosurus_vulpecula"
```

The `host.species` variable includes both the genus and species. What if we just want the genus? To possible functions come in here: `str_split()` and its cousin, `str_split_fixed()`


``` r
splitting_species <- str_split(urban_data$host.species, pattern = "_")
head(splitting_species)
```

```
## [[1]]
## [1] "Trichosurus" "vulpecula"  
## 
## [[2]]
## [1] "Trichosurus" "vulpecula"  
## 
## [[3]]
## [1] "Trichosurus" "vulpecula"  
## 
## [[4]]
## [1] "Trichosurus" "vulpecula"  
## 
## [[5]]
## [1] "Trichosurus" "vulpecula"  
## 
## [[6]]
## [1] "Trichosurus" "vulpecula"
```

Now we have a list (oops, we haven't gotten to those yet!) of vectors. Each element of the list now has the genus and species separated. In many cases, you can use `str_split_fixed()`, which will instead return a matrix. It just requires that each element has the same number of elements in the output:


``` r
splitting_species <- str_split_fixed(urban_data$host.species, pattern = "_", n = 2)
head(splitting_species)
```

```
##      [,1]          [,2]       
## [1,] "Trichosurus" "vulpecula"
## [2,] "Trichosurus" "vulpecula"
## [3,] "Trichosurus" "vulpecula"
## [4,] "Trichosurus" "vulpecula"
## [5,] "Trichosurus" "vulpecula"
## [6,] "Trichosurus" "vulpecula"
```

The complication here is that we could imagine a case where we have subspecies sometimes and not others:


``` r
species <- c("Anas acuta", "Anas crecca", "Anas crecca carolinensis")
str_split_fixed(species, " ", 2)
```

```
##      [,1]   [,2]                 
## [1,] "Anas" "acuta"              
## [2,] "Anas" "crecca"             
## [3,] "Anas" "crecca carolinensis"
```

``` r
str_split_fixed(species, " ", 3)
```

```
##      [,1]   [,2]     [,3]          
## [1,] "Anas" "acuta"  ""            
## [2,] "Anas" "crecca" ""            
## [3,] "Anas" "crecca" "carolinensis"
```

We can also change capitalization using functions like `str_to_lower()`, `str_to_upper()`, and `str_to_sentence()`.


``` r
str_to_lower(species)
```

```
## [1] "anas acuta"               "anas crecca"             
## [3] "anas crecca carolinensis"
```

Most of `stringr`'s functions start with `str_`, so if you're looking for a new function for something you want to do, you can type in `str_` and get some suggestions:

<div class="figure" style="text-align: center">
<img src="images/stringr_funs.png" alt="Autofill for stringr functions" width="80%" />
<p class="caption">(\#fig:unnamed-chunk-25)Autofill for stringr functions</p>
</div>

## Factors: character/integer hybrids

Factors are used for categorical variables, i.e. variables that have a known set of possible values. Some common categorical variables are months of the year and days of the week; in scientific studies, site names and individual IDs are other common applications of factors. Factors are useful not only because they limit the number of values a variable can take (and provide errors/warnings when that is violated) but also because they can be sorted in non-alphabetical/non-numerical order. These possible values are called `levels` by R. For example:


``` r
survey_months <- c("January","February","April","May","July","August")
sort(survey_months)
```

```
## [1] "April"    "August"   "February" "January"  "July"     "May"
```

``` r
survey_months_fct <- factor(survey_months, levels = survey_months)
survey_months_fct
```

```
## [1] January  February April    May      July     August  
## Levels: January February April May July August
```

``` r
survey_months_fct2 <- factor(survey_months, levels = month.name) #month.name is conveniently built into R
survey_months_fct2
```

```
## [1] January  February April    May      July     August  
## 12 Levels: January February March April May June July August ... December
```

Not all levels need to appear in a vector, but the vector cannot take any values not in the levels:


``` r
survey_months_fct2[2] <- "Tuesday"
```

```
## Warning in `[<-.factor`(`*tmp*`, 2, value = "Tuesday"): invalid factor level,
## NA generated
```

``` r
survey_months_fct2
```

```
## [1] January <NA>    April   May     July    August 
## 12 Levels: January February March April May June July August ... December
```

If you ever need to get the the set of levels, you can do so with `levels()`. This is sometimes helpful after you have filtered your data but all levels are still present:


``` r
levels(survey_months_fct2)
```

```
##  [1] "January"   "February"  "March"     "April"     "May"       "June"     
##  [7] "July"      "August"    "September" "October"   "November"  "December"
```

Under the hood, R considers factors to be integers, corresponding to their levels. For example:


``` r
as.integer(survey_months_fct)
```

```
## [1] 1 2 3 4 5 6
```

``` r
as.integer(survey_months_fct2)
```

```
## [1]  1 NA  4  5  7  8
```

Remember this if you ever have a factor (for example, site number!) that otherwise appears as an integer:


``` r
site_ids <- c(1, 3, 4, 9, 12, 15)
site_ids_fct <- as.factor(site_ids)
site_ids_fct
```

```
## [1] 1  3  4  9  12 15
## Levels: 1 3 4 9 12 15
```

``` r
as.integer(site_ids_fct)
```

```
## [1] 1 2 3 4 5 6
```

One way to go get the actual site IDs back from this factor is to go through a character type, because the character no longer has the levels associated with it:


``` r
as.character(site_ids_fct)
```

```
## [1] "1"  "3"  "4"  "9"  "12" "15"
```

``` r
as.integer(as.character(site_ids_fct))
```

```
## [1]  1  3  4  9 12 15
```

<!-- ## Exercise 5 -->

<!-- We will work with the urban wildlife health data to practice manipulating factors, strings, and numbers in data frames. -->

<!-- ```{r, eval = F, echo = T} -->
<!-- # Load the tidyverse -->
<!-- library(tidyverse) -->

<!-- # Read in the urban wildlife dataset -->
<!-- urban_data <- read_csv("data/raw/Murray-Sanchez_urban-wildlife.csv") -->

<!-- # Create a new column that includes both host type and host habitat -->
<!-- urban_data <- mutate(urban_data, class_hab = str_c(___, ___, sep = ___)) -->

<!-- # OPTIONAL CHALLENGE: create a new column with the full citation,  -->
<!-- # including the date in parentheses -->
<!-- urban_data <- mutate(urban_data, citation = ___) -->

<!-- # Create a new column in urban_data that calculates  -->
<!-- # number of years since the study -->
<!-- urban_data <- mutate(urban_data, years_ago = ___) -->

<!-- # Create a new column in urban_data  -->
<!-- # with the absolute value of the effect size -->
<!-- urban_data <- mutate(urban_data, r_abs = ___(r)) -->

<!-- # Create a new column in urban_data -->
<!-- # that tells you whether the p-value is sigificant at alpha=0.05 -->
<!-- urban_data <- mutate(urban_data, sig = ___) -->

<!-- # Filter the data to parasites only -->
<!-- para_data <- filter(urban_data, ___) -->

<!-- # With the parasite-only data, change the "parasite2" column  -->
<!-- # to remove the redundant "parasite" word -->
<!-- para_data <- mutate(para_data, parasite2 = str_remove(___, ___)) -->

<!-- # Convert the parasite2 column to a factor -->
<!-- para_data <- mutate(para_data, ___) -->

<!-- # What is the order of the levels of the parasite2 factor? -->
<!-- levels(para_data$___) -->

<!-- # What is the mean effect size in the entire data set? -->
<!-- mean(___) -->

<!-- # What is the mean effect size in the parasite-only data set? -->
<!-- mean(___) -->

<!-- ``` -->
