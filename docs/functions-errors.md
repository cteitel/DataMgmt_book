# Writing Your Own Functions {#functions}

## Objectives

* Understand when custom functions are useful
* Build simple custom functions with multiple arguments
* Efficiently handle potential errors when writing functions and loops

## Additional reading

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 25: Functions. Available: https://r4ds.hadley.nz/functions.html

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 16: Function documentation. Available: https://r-pkgs.org/man.html



## Why write your own functions?

We have used a lot of different functions in this class already. Some are built into R, like `mean()`,
and some come with packages, like `mutate()` in `tidyverse`. It might feel like there are already an
almost-infinite number of functions out there. Why would you ever need to write your own? A few reasons:

* Much like loops, functions can help automate a repeating task. If you have a series of functions
that you often use in order, combining them into a single function can be useful.
* Instead of copying and pasting sections of code, you can write and modify a single function,
all at once, making it less likely that you will have an error in one of your many copy-paste
replicates.
* Functions can exist in a separate file for you to load later. This is especially useful if they
are long and complex; unlike a loop, you don't have to take up space in your working code with a
long series of data processing or modeling tasks. You can also use the same function in multiple 
projects.
* You can share code for just a function with a colleague, instead of having to send them your
entire script to show how to get one task done.

## How do functions work?

You've been on the "user" end of functions many times, so you know that they have a **name**, 
they take a series of **arguments** as input, and they give you an **output**. Beyond that, 
there's not much to know. This is the basic structure of a function:


``` r
name <- function(arugments){
  output <- code
  return(output)
}
```

We would then use this function like this: `name(arguments)`.

For example, for some reason there is no function built into R to calculate the mode (i.e.,
the most common element in a vector/series). (`mode()` tells you the internal storage mode of 
an object, not the value that occurs the most often.) I could write a function that does that:


``` r
mode_calc <- function(vect){
  unique_vals <- sort(unique(vect)) #Get unique values in the vector
  value_counts <- tabulate(as.factor(vect)) #Count the number of occurrence of each value
  max_count <- max(value_counts) #Identify the max number of occurrence
  mode <- unique_vals[which(value_counts == max_count)] #Identify the unique value(s) that have the max number of occurrences
  return(mode)
}
```

If we test this:


``` r
test_values <- c(1, 1, 1, 2, 2, 2, 2)
modal_value <- mode_calc(test_values)
modal_value
```

```
## [1] 2
```

Functions can also take multiple arguments. For example, we might want to be able to remove `NA`s
from the function with an `na.rm` argument like we have in `mean()` and `median()`, so that with
the presence of any `NA` values, the function returns `NA`:


``` r
mode_calc2 <- function(vect, na.rm){
  if(na.rm == FALSE & any(is.na(vect))) return(NA) #Return NA if needed
  unique_vals <- sort(unique(vect)) #Get unique values in the vector
  value_counts <- tabulate(as.factor(vect)) #Count the number of occurrence of each value
  max_count <- max(value_counts) #Identify the max number of occurrence
  mode <- unique_vals[which(value_counts == max_count)] #Identify the unique value(s) that have the max number of occurrences
  return(mode)
}
```


``` r
test_values <- c(1, 1, 1, 2, 2, 2, 2, NA)
modal_value <- mode_calc2(test_values, na.rm = FALSE)
modal_value
```

```
## [1] NA
```

``` r
modal_value <- mode_calc2(test_values, na.rm = TRUE)
modal_value
```

```
## [1] 2
```

Finally, you can also provide **defaults** to function arguments, which are specified just
like you see them on the help pages of other functions. We did not specify a default for either
`vect` or `na.rm` in our `mode_calc2` function. This means that the function will throw an 
error if we fail to specify either argument:


``` r
modal_value <- mode_calc2(test_values)
```

```
## Error in mode_calc2(test_values): argument "na.rm" is missing, with no default
```

We can fix this but providing a default value:


``` r
mode_calc2 <- function(vect, na.rm = FALSE){
    ...
}
```

Although functions can take multiple *arguments*, they can only produce a single *output*. 
If you want multiple pieces of information, you will have to combine them into a single argument, 
whether that's a data frame, a vector, or a list. The output of a function is specified 
within `return()`. This isn't strictly required, but otherwise the function will output
whatever it last ran, which is not always what you want (see the `na.rm` example above).

## Documenting functions

It's a good idea to define the arguments and outputs of functions for your future self
and any other future users. You can do this with comments or with a special documentation
called "roxygen", which package developers use. Roxygen comment lines always start with #' , 
(the usual # for a comment, followed immediately by a single quote '). They are nice because
they create automatic highlighting and text colors, but also because you can use them to
create actual help pages, if you want to! (See the "Additional reading" section above.)

Whether you use roxygen or not, you want to define:

1. The purpose of the function ("Description" under the help pages)
2. The function arguments
3. The function's outputs
4. Any useful examples

Again, with our `mode_calc` function:


``` r
#' Determine the mode of a vector
#' 
#' @param vect A vector (numeric, integer, character, date, or factor)
#' @param na.rm Logical; remove NAs?
#' @returns A vector of the same type as vect
#' @examples
#' mode_calc2(c(1,1,2), na.rm = TRUE)
mode_calc2 <- function(vect, na.rm = FALSE){
  if(na.rm == FALSE & any(is.na(vect))) return(NA) #Return NA if needed
  unique_vals <- sort(unique(vect)) #Get unique values in the vector
  value_counts <- tabulate(as.factor(vect)) #Count the number of occurrence of each value
  max_count <- max(value_counts) #Identify the max number of occurrence
  mode <- unique_vals[which(value_counts == max_count)] #Identify the unique value(s) that have the max number of occurrences
  return(mode)
}
```

## Error handling

### Adding error and warning messages

In both loops and functions (and maybe even more so in loops), you will sometimes end up with
arguments that just don't work and need an error or warning. We are accustomed to using functions 
that show error and warning messages; for example, if we provide a character vector to `mean()`:


``` r
mean(c("this","is","a","character","vector"))
```

```
## Warning in mean.default(c("this", "is", "a", "character", "vector")): argument
## is not numeric or logical: returning NA
```

```
## [1] NA
```

We can make this happen for our own functions, too. For example, the `tabulate` function, 
doesn't work well on numeric vectors of length 1 because it assumes we want a sequence of numbers
of that length. We can warn users about this using the `warning()` function:


``` r
mode_calc2 <- function(vect, na.rm = FALSE){
  if(length(vect)==1){
    warning("Vector has only 1 non-NA value; results may be unexpected")
  }
  if(na.rm == FALSE & any(is.na(vect))) return(NA) #Return NA if needed
  unique_vals <- sort(unique(vect)) #Get unique values in the vector
  value_counts <- tabulate(as.factor(vect)) #Count the number of occurrence of each value
  max_count <- max(value_counts) #Identify the max number of occurrence
  mode <- unique_vals[which(value_counts == max_count)] #Identify the unique value(s) that have the max number of occurrences
  return(mode)
}
mode_calc2(10)
```

```
## Warning in mode_calc2(10): Vector has only 1 non-NA value; results may be
## unexpected
```

```
## [1] 10
```

Alternatively, we could make this an error using the `stop()` function:


``` r
mode_calc2 <- function(vect, na.rm = FALSE){
  if(length(na.omit(vect))==1){
    stop("Vector has only 1 non-NA value")
  }
  if(na.rm == FALSE & any(is.na(vect))) return(NA) #Return NA if needed
  unique_vals <- sort(unique(vect)) #Get unique values in the vector
  value_counts <- tabulate(as.factor(vect)) #Count the number of occurrence of each value
  max_count <- max(value_counts) #Identify the max number of occurrence
  mode <- unique_vals[which(value_counts == max_count)] #Identify the unique value(s) that have the max number of occurrences
  return(mode)
}
mode_calc2(10)
```

```
## Error in mode_calc2(10): Vector has only 1 non-NA value
```

It's also important to note that if you look at your environment, you will *not* see objects
called `unique_vals`, `value_counts`, and so on. These are created only inside the function.

### Errors and warnings

On the other hand, sometimes an internal function will throw an error that we don't
care about, but it will stop our loop or function from working. This is especially frustrating
in a loop because the loop will quit partway through, just when you walked away to let your 
code run for a while. Luckily, there is a single function that helps with this: `tryCatch()`. 
The function takes a little getting used to, but with some practice (and copying and pasting)
from other code) it can streamline your workflow significantly.

Let's say that, using my function above, I wrote a loop to take the mode of a series of numbers.
First, I'm creating a list, which is just a single object containing multiple vectors. Notice that
one of them (the third vector) has only one element, so we know already that `mode_calc2` will
produce an error:


``` r
vectors <- list(
  sample(10, 10, replace = T),
  sample(10, 5, replace = T),
  sample(10, 1, replace = T),
  sample(10, 10, replace = T),
  sample(10, 10, replace = T),
  sample(10, 5, replace = T),
  sample(10, 10, replace = T)
)
vectors[[3]]
```

```
## [1] 6
```

``` r
mode_calc2(vectors[[3]])
```

```
## Error in mode_calc2(vectors[[3]]): Vector has only 1 non-NA value
```

If I want the function return something instead of an error, I can enclose the function 
causing the problem within `tryCatch()`:


``` r
tryCatch(mode_calc2(vectors[[3]]), error = function(e) NA)
```

```
## [1] NA
```

In our usual use, `tryCatch()` takes two arguments: first, a function (or series of functions,
which we will get to), and second, the `error` argument, which itself has to be a function. 
In most cases, you can write a function like the one I wrote above, that just returns a single value
(in this case, `NA`).

This becomes especially useful because if I loop through this list, my loop will quit at the third 
element, which you see because I have two outputs before my error message:


``` r
for(i in 1:length(vectors)){
  vect <- vectors[[i]]
  mode_val <- mode_calc2(vect)
  print(mode_val)
}
```

```
## [1] 4
## [1] 1
```

```
## Error in mode_calc2(vect): Vector has only 1 non-NA value
```

I can once again enclose my function within `tryCatch()`:


``` r
for(i in 1:length(vectors)){
  vect <- vectors[[i]]
  mode_val <- tryCatch({
    mode_calc2(vect)
  }, error = function(e) NA)
  print(mode_val)
}
```

```
## [1] 4
## [1] 1
## [1] NA
## [1] 7 9
## [1] 10
## [1] 2
## [1] 5 6
```

This brings up another point about `tryCatch()`: you want it to enclose the function, but not 
the assignment. This is because in this case the output of `tryCatch()` is `NA`, so we want to
assign `mode_val` *to the output of `tryCatch()`*. Using curly brackets allows us to include as many lines
of code as we want within the `tryCatch()` function, which can be helpful if you are using a series of 
functions to create a single argument. This takes some practice, but try to remember two key
elements:

1. The `error` argument to `tryCatch()` needs to be a function.
2. You want to assign the output of `tryCatch()` to an object in most cases.

Happy functioning!




