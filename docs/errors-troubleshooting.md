# Troubleshooting in R {#troubleshooting}

This lesson draws heavily from materials from [WILD 6900: Tools for Reproducible Science](https://ecorepsci.github.io/reproducible-science/), Spring 2021, Utah State University, by Dr. Simona Picardi, also published under a [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/) license.

## Objectives

* Interpret common error messages encountered in R
* Understand basic steps to troubleshoot code
* Build awareness of resources available for troubleshooting

## Additional reading

Hadley Wickham, Mine Çetinkaya-Rundel, and Garrett Grolemund. R for Data Science (2e). Chapter 8: Workflow: getting help. Available: https://r4ds.hadley.nz/workflow-help.html

## Introduction

By this point, you have probably encountered a few error and warning messages in R. These can be frustrating! But no matter how good you get at programming, you will always make mistakes, whether these are typos or fundamental problems with your code. Troubleshooting is a fundamental programming skill. 

Computers only do what we *tell* them to do, not what we *want* them to do. Sometimes, an error appears because the computer doesn't understand what we told it, and sometimes (usually worse, because it is harder to detect), we accidentally tell it to do the wrong thing. R is famous for having cryptic error messages, but learning just a few key points can help you understand what the computer is saying back to you ("Wait, I didn't get that, can you say it another way?").

## How to interpret error messages

Error messages can be frustrating but they can be an important ally in diagnosing problems in your code. Some error messages (the good ones) are clear and point you to the problem quite explicitly. Others are cryptic, but they are so typical of  certain situations that they end up working as pretty good hints of what is wrong. The worst kind of error is the undescriptive, rare one that gives you no real clue where the problem is. You will encounter all of these. 

R provides three main types of outputs in the console:

* **Errors**, which indicate that a piece of code had a fatal flaw and did not run. These need to be fixed before you can move on.
* **Warnings**, which indicate that a piece of code ran, but that it *might* not have done what you want it to. When you get a warning, you should usually check the output to make sure it makes sense.
* **Messages**, which tell you something about what is going on under the hood. These don't require your attention.

### Locating the problematic piece of code

RStudio helps you identify syntax errors by placing little red marks on the left of your script next to the line numbers. However, these marks only warn you of syntax errors: missing commas, missing parentheses or quotes, etc. They cannot warn you about improper use of a function or wrong data dimensions.   
  
The best way to isolate a problematic piece of code is to step through the script line by line until the error occurs. Then, there can either be a problem in that line itself or there can be a mistake in the lines that lead up to that and produced the objects that went as input into that line. So, the first thing to do before you try to dig deeper in the error message is checking that what went in input into that line actually looks like it's supposed to. 

### Isolating the error

In general, isolating the problem is the first step in troubleshooting, but even within the same line it's useful to run each piece of code that can be run independently to check that it's giving the expected output. For example, if I am running the following code:


``` r
df <- data.frame(numbers = 1:5,
                 letters = c("A", "B", "C", "D", "E"),
                 animals = c("cat", "dog", "lion", "cheetah", "giraffe"))
```

I can run each piece to look at it and double check it:


``` r
1:5
```

```
## [1] 1 2 3 4 5
```


``` r
c("A", "B", "C", "D", "E")
```

```
## [1] "A" "B" "C" "D" "E"
```


``` r
c("cat", "dog", "lion", "cheetah", "giraffe")
```

```
## [1] "cat"     "dog"     "lion"    "cheetah" "giraffe"
```

### Deciphering error messages

If the input looks good, it's time to start deciphering the error message. Here are some examples of common errors and what they usually mean:


``` r
test <- 1:10
sum(tets)
```

```
## Error: object 'tets' not found
```

An object can be "not found" when you misspelled its name or when you didn't run your code in the correct order and you haven't created it yet. 

Here's another common one:


``` r
df <- data.frame(numbers = 1:5,
                 letters = c("A", "B", "C", "D", "E"))

df$animals <- c("cat", "dog", "lion", "giraffe")
```

```
## Error in `$<-.data.frame`(`*tmp*`, animals, value = c("cat", "dog", "lion", : replacement has 4 rows, data has 5
```

Whenever you see an error of the form `replacement has... data has...` you know
that you're trying to plug in the wrong number of values into an object. In this
case, I tried to add a column with four elements to a data frame that has five
rows. Notice that this error is a bit more specific than the previous example
because it gives us an indication of what line failed. This is not particularly
useful in this case where we only ran one line, but when you're troubleshooting 
a function or a larger piece of code that works as one it's useful to know at
which point within it the error occurred.  

Bottom line: use the first component of the error message (the one after 
`Error in`) to identify *where* the error occurred, and use the second component
(the one after the `:`) to understand *what* went wrong. 

### Syntax errors 
  
Perhaps the most classic of error messages is this:


``` r
vec <- c(1, 4, 6, 7, 8))
```

```
## Error in parse(text = input): <text>:1:24: unexpected ')'
## 1: vec <- c(1, 4, 6, 7, 8))
##                            ^
```

Make sure you take advantage of RStudio's highlighting tool to check if you
closed all the parentheses you opened. If you move your blinking cursor to an opening (or
closing) parenthesis (or bracket), RStudio will highlight the closing (or 
opening) one. 

Another classic:


``` r
df[df$numbers > 4]
```

```
## Error in `[.data.frame`(df, df$numbers > 4): undefined columns selected
```

This is most likely the symptom that we forgot a comma inside the brackets and 
therefore the indexing isn't working correctly. Remember, square brackets give
you the element of a vector, but for a data frame, R needs to know the row *and*
column to look for. To fix this:


``` r
df[df$numbers > 4,]
```

```
##   numbers letters
## 5       5       E
```

### Errors from using package functions

In some situations, you will get an error message that will point you to a line
of code that you haven't written. That is most likely because a function that
you are calling ran into an internal error, and the code you're seeing in the
error message is inside that function. In that case, it's a good idea to focus
on that function and try to figure out why it's not working like expected. This 
mostly happens when using functions from a package other than `base`.  

Another common problem when using functions from packages is that a function can 
exist in two packages, have the same name but take in a completely different set 
of arguments and do different things. If you use one of these functions 
thinking you're calling it from package A and it's coming from package B instead, 
chances are the inputs you gave it do not work. When a function with the same
name exists in more than one package among those you have loaded, R assumes you
are trying to use the function from the package you loaded most recently. This 
is why the order in which you load packages matter. Another (the best) solution
is to use the syntax `package::function` to specify which package you're calling
the function from. This removes any ambiguity. 

### Errors without error messages

Sometimes, your code runs fine, but it produces unexpected results. These are harder to detect - most quantitative scientists I know have a story of thinking they found a really novel and exciting result, only to find that they had accidentally left out half their data, sorted it in the wrong order, or some other small coding error. Some tips for avoiding these problems:

* Print and check intermediate products of your code and their properties. If you filter a data frame, how many rows does the new data frame have? How does that line up with your expectations?
* Check for missing values throughout the process. Many functions silently filter out missing values, but this can produce some unexpected results.


## How to look for help

### R documentation

The first thing to check when a function is not working like expected is to 
look up the documentation and double check that you provided all the necessary
input in the right format:


``` r
?mean
```

### Google

Googling error messages can be extremely helpful, but it also takes some practice. You are unlikely to find much useful by just copying and pasting your error message into a search bar. Instead, think about which elements of the error message are likely to be general, rather than specific to your code. For example, if the error message says,
 
 > Error in data.frame(numbers = 1:5, letters = c("A", "B", "C", "D", "E"),  : 
 > arguments imply differing number of rows: 5, 4`
 
 the content of the data frame is specific to my situation, and so are the 
 numbers at the end. I need to remove those and only include the following in 
 my Google search:
 
 > Error in data.frame arguments imply differing number of rows

Other tips:

* Always add "in R" at the end of your search
* If you have identified what function the error comes from, add the name of the
function to your search
* If the problematic function comes from a package, add the name of the package
to your search

Most often, Googling will take you to StackOverflow...

### StackOverflow et al.

In most cases, somebody has probably already run into your problem and posted about on StackOverflow or similar websites. This gets less true if you are using a new or uncommon package, but usually the difficult part is to be able to apply the answers from these forums to your problem. The specifics of the dataset the other person is using may not be exactly identical to yours. This is where thinking outside the box and drawing parallels between your case and somebody else's case becomes critical. For instance, say that I run the following code and get an error:


``` r
vec <- 1:10

mat <- matrix(NA, 5, 5)

mat[1, ] <- vec
```

```
## Error in mat[1, ] <- vec: number of items to replace is not a multiple of replacement length
```

If I go and Google the error, [this](https://stackoverflow.com/questions/38738347/why-do-i-get-number-of-items-to-replace-is-not-a-multiple-of-replacement-length) is the first post I find on StackOverflow about it. This person is trying to do a different thing than me:
they are trying to assign new values to a column in a data frame based on another
column. Their problem is that, because they only want to replace the values that
are `NA`, the slots to replace are fewer than the total length of the column.
Therefore, there are too many items for too few spaces. In my case, I am not 
subsetting the target column, so what I'm doing is a bit different. However, 
what both situations have in common is that the number of spaces to be filled 
is not the same as the number of items to fill them with. So now I know that my
problem must be that I have too many (or too few) values to fit in my column. 

<div class="figure" style="text-align: center">
<img src="images/debugging.jpg" alt="Artwork by Allison Horst, and thanks to Simona Picardi for including this in her course materials" width="80%" />
<p class="caption">(\#fig:debugging)Artwork by Allison Horst, and thanks to Simona Picardi for including this in her course materials</p>
</div>

<!-- ## How to troubleshoot a loop 

Consider the following loop:


``` r
my_list <- list(11, 4, TRUE, 98, FALSE, "yellow", 34, FALSE, TRUE, "dog")

for (i in 1:length(my_list)) {
  
  item <- my_list[[i]]
  res <- item + 10

}
```

```
## Error in item + 10: non-numeric argument to binary operator
```

Something is wrong with this loop because I am getting an error. But how do I 
know where the error happened? The first trick to know is that loops work 
through the indexes you provide as `i` in order (first 1, then 2, and so on, 
until it gets to the end, which in this case is `length(my_list)`). Each time
it runs through an iteration, the loop will assign to `i` the current value; 
thus, `i` is an object in your environment. If one iteration returns an error, 
the loop stops. So the value of `i` after the loop stops will tell you which was
the element that produced the error:


``` r
i
```

```
## [1] 6
```

In this case, the sixth iteration is the one that failed. Now that we know this,
the second trick to know is that we can step through the loop one line at a time 
while having `i = 6` and see where the error occurs:


``` r
i <- 6

item <- my_list[[i]] # this line returns no error

res <- item + 10 # this is the culprit!
```

```
## Error in item + 10: non-numeric argument to binary operator
```

In this case, the problem was that item 6 of my list is a character string and
I'm asking R to do math on it. I was able to do math on elements 1 through 5
because they were either numbers or logical (which can be coerced to numbers
like we saw in Chapter \@ref(intro-to-r).)  

Note that this is a useful trick to use not only to troubleshoot loops when 
something goes wrong, but also to verify that the loop is functioning correctly 
on a single item before you pull the trigger on the whole thing. You can set 
`i <- 1` and step through the loop line by line to do a test run. 

## How to troubleshoot a function

In Chapter \@ref(intro-to-r) we have seen how to write functions and also how to
use `lapply` to run the same function on a list of objects. Functions can be 
as simple or as complicated as we need them to be, but the more complex they get
the more likely it is it will take some trial and error to get them to work 
correctly. We need some troubleshooting tools!  

Unlike loops, functions do not save any objects to the global environment other
than the final result (wrapped in the `return` statement). In a loop, every 
intermediate product (as well as the index `i`) are saved to the global 
environment at each iteration. Functions, instead, work by creating their own
environment where temporary objects are saved. Then, once the final output is
ready to be returned, the function saves that and only that to the global 
environment. Consider the following function:


``` r
fun_test <- function(x) {
  
  y <- x * 2
  z <- y + 10
  return(z)

}

(output <- fun_test(1))
```

```
## [1] 12
```

The intermediate object `y` is never saved to our global environment. The `z`
object is the one we're returning, and it gets saved to the environment with the
name we give it (`output`). To troubleshoot or verify what's going on inside of
the function, we need to "move" the process into our global environment. This 
means that we need to define an `x` object in our global environment and then
step through the code inside the curly braces manually:


``` r
(x <- 1)
```

```
## [1] 1
```

``` r
(y <- x * 2)
```

```
## [1] 2
```

``` r
(z <- y + 10)
```

```
## [1] 12
```

Now let's make an example with `lapply`. Say that we want to apply our function
to the list we used in the loop above:


``` r
my_list <- list(11, 4, TRUE, 98, FALSE, "yellow", 34, FALSE, TRUE, "dog")

lapply(my_list, fun_test)
```

```
## Error in x * 2: non-numeric argument to binary operator
```

Here's the error again. This time, we don't get the luxury of looking at the
iteration index to get a hint on where things went wrong. So what we can do
instead is step through the list elements one at a time: 


``` r
x <- my_list[[1]] # first we assign the first element of the list to x

# then we step through the code inside the function
(y <- x * 2)
```

```
## [1] 22
```

``` r
(z <- y + 10)
```

```
## [1] 32
```

This went smoothly, so that must not have been the problematic item. If we keep
going with the other elements, eventually we find the culprit:


``` r
x <- my_list[[6]] 

(y <- x * 2)
```

```
## Error in x * 2: non-numeric argument to binary operator
```

``` r
(z <- y + 10)
```

```
## [1] 32
```

``` r
x
```

```
## [1] "yellow"
```

And there it is: element #6 is a character string and we get an error because
we are trying to do math on it. 
-->

<!-- ## Reproducible examples  -->

<!-- In the unlikely event that you run into an error that no human has posted on  -->
<!-- StackOverflow before, you can post a request for help. It is best to do so by -->
<!-- sharing a reproducible example. A reproducible example is a standalone script -->
<!-- that allows someone else to reproduce your problem on their computer. Without  -->
<!-- it, all people can do is read your code and *imagine* what the output of each  -->
<!-- step should look like, and it's hard to diagnose errors that way. To write a  -->
<!-- good reproducible example, you need to provide the following: -->

<!-- * Required packages: it's good practice to load these at the top of your script -->
<!-- so people can quickly see if they need to install anything before running the  -->
<!-- code; -->
<!-- * Data: if you can reproduce your problem using one of the built-in datasets  -->
<!-- available in R, you can use that. Otherwise, you can use the function `dput` to -->
<!-- generate code to recreate your current dataset and then copy-paste it into your -->
<!-- script: -->

<!-- ```{r, eval = TRUE, echo = TRUE} -->
<!-- df <- data.frame(numbers = 1:5, -->
<!--                  letters = c("A", "B", "C", "D", "E"), -->
<!--                  animals = c("cat", "dog", "lion", "cheetah", "giraffe")) -->

<!-- dput(df) -->
<!-- ``` -->

<!-- ```{r, eval = TRUE, echo = TRUE} -->
<!-- df <- structure(list(numbers = 1:5,  -->
<!--                      letters = c("A", "B", "C", "D", "E"),  -->
<!--                      animals = c("cat", "dog", "lion", "cheetah", "giraffe")),  -->
<!--                 class = "data.frame", row.names = c(NA, -5L)) -->
<!-- ``` -->

<!-- * Code: the code that you want to get help with; -->
<!-- * R environment: you can obtain information on your current session environment  -->
<!-- using the `sessionInfo` function. Copy-paste the output as a comment into your  -->
<!-- reproducible example script: -->

<!-- ```{r, eval = TRUE, echo = TRUE} -->
<!-- sessionInfo() -->
<!-- ``` -->


## Other troubleshooting tips

* The rubber duck: The rubber duck is a method of debugging code introduced by software engineers. 
It consists of explaining code, line by line, to a rubber duck. While this debugging technique might sound silly, it does work for many people: saying out loud what the code is supposed to do in each step forces us to identify details that might not have been clear in our minds, or might not have gotten from our minds to the script. 
* Step away: Like many other problems, sometimes a short break will clear your mind and help you re-focus. The problem might become clear when you come back to it
* AI/LLM tools: ChatGPT and other LLMs can be great troubleshooting buddies, especially if you ask them to explain why you were getting your error in the first place. But be careful - sometimes they'll change other parts of your code along the way.

## Some final wisdom

<div class="figure" style="text-align: center">
<img src="https://imgs.xkcd.com/comics/error_code.png" alt="You can always try this. (https://xkcd.com/1024/)" width="80%" />
<p class="caption">(\#fig:unnamed-chunk-20)You can always try this. (https://xkcd.com/1024/)</p>
</div>
