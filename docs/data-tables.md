# Creating and Designing Data Tables {#tables}

## Objectives

* Summarize and output data in tabular format
* Format tables to include necessary information for interpretation
* Output data from R in tabular form that can be used in reports and other documents

## Additional reading

Remshard, M., & Queenborough, S. A. (2023). Design of tables for the presentation and communication of data in ecological and evolutionary biology. *Ecology and Evolution, 13*(7), e10062. https://doi.org/10.1002/ece3.10062

## Tables as outputs

Data visualization in figures and other graphics is usually the most effective way of communicating information. Consider, for example, the same data displayed in a figure and in a table:


``` r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.6
## ✔ forcats   1.0.1     ✔ stringr   1.6.0
## ✔ ggplot2   4.0.1     ✔ tibble    3.3.1
## ✔ lubridate 1.9.4     ✔ tidyr     1.3.2
## ✔ purrr     1.2.1     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(palmerpenguins)
```

```
## 
## Attaching package: 'palmerpenguins'
## 
## The following objects are masked from 'package:datasets':
## 
##     penguins, penguins_raw
```

``` r
dat_summ <- penguins %>%
  filter(!is.na(sex)) %>%
  group_by(species, sex) %>%
  summarize(n = n(),
            body_mass_mean = mean(body_mass_g, na.rm = T),
            body_mass_sd = sd(body_mass_g, na.rm = T))
```

```
## `summarise()` has grouped output by 'species'. You can override using the
## `.groups` argument.
```

``` r
dat_summ %>%
  ggplot(aes(x = species, y = body_mass_mean, 
             ymin = body_mass_mean - body_mass_sd, 
             ymax = body_mass_mean + body_mass_sd,
             color = sex, size = n)) +
  geom_pointrange() +
  theme_bw() + theme(text = element_text(size = 14)) +
  scale_size("Number of pengins", range = c(0.5,2),
             breaks = c(35,55,73)) +
  scale_color_manual("Sex", breaks = c("male","female"), 
                     values = c("darkgreen", "orange")) +
  labs(x = "Species", y = "Body mass (mean +/- SD)")
```

<img src="data-tables_files/figure-html/unnamed-chunk-1-1.png" alt="" width="672" />

``` r
dat_summ
```

```
## # A tibble: 6 × 5
## # Groups:   species [3]
##   species   sex        n body_mass_mean body_mass_sd
##   <fct>     <fct>  <int>          <dbl>        <dbl>
## 1 Adelie    female    73          3369.         269.
## 2 Adelie    male      73          4043.         347.
## 3 Chinstrap female    34          3527.         285.
## 4 Chinstrap male      34          3939.         362.
## 5 Gentoo    female    58          4680.         282.
## 6 Gentoo    male      61          5485.         313.
```

From the figure, I clearly see that Gentoo penguins are heavier than the other two species, that males are heavier than females in all species, and that fewer chinstrap penguins were sampled than the other two species. It would take me a minute to get all this information from the table. However, tables are better than figures when exact values are important, or when you have a lot of information to convey. For example, I can't quite tell from the figure whether the exact same number of males and females were sampled in each species. Overall, some key applications for tables are:

* Providing exact values
* Reporting statistical outputs
* Reporting large data sets (often in an appendix)

In scientific publications, tables are often formatted in a specific way - for example, with borders and margins in a particular format. In the publishing world, most of this formatting is usually completed by the journal editorial team at the time of publication. However, if you are creating reports or other documents that will be shared without professional editing, you will need to know how to format tables in and out of R. The most typical scientific format of a table looks like this:

<div class="figure" style="text-align: center">
<img src="images/table-format.png" alt="A common data table format" width="80%" />
<p class="caption">(\#fig:unnamed-chunk-2)A common data table format</p>
</div>

Notice that the table includes border lines above and below the header row and a border line below the table, but no vertical borders or horizontal borders within the data area. The header row is in **bold** but nothing else is. This formatting is relatively straightforward to achieve in word processing programs but can also be achieved (with a little more of a learning curve) directly in R.

## Writing tables to files

Although the final formatting of a table might be done in a word processor, we first need to get the data out of R. We covered some of this in our lesson on [reading and writing data](#importexport). Since a CSV file will open easily in a spreadsheet program like Excel, you can simply:

1. save your table as a CSV, 
2. open it in Excel,
3. copy that table to Word, and 
4. add borders and alignment. 

However, since one benefit of using a scripting language like R is that you can easily update your analyses, it is helpful to pre-format the table as much as possible before outputting. If we outputted `data_summ` as it is now, our fourth step would also require rounding or truncating numbers and renaming columns. Every time we updated the analysis we would need to update these as well, making this process more time-consuming and, to some degree, less reproducible; for example, there would be no record of the translation between column names in our data.frame and column names in the data presented. Ideally, you would do all of this rounding and renaming before writing your data to a CSV:


``` r
dat_summ_out <- dat_summ %>%
  rename(`No. birds` = n, 
         `Mean body mass (g)` = body_mass_mean,
         `SD body mass` = body_mass_sd) %>%
  mutate(across(where(is.numeric), round))
dat_summ_out
```

```
## # A tibble: 6 × 5
## # Groups:   species [3]
##   species   sex    `No. birds` `Mean body mass (g)` `SD body mass`
##   <fct>     <fct>        <dbl>                <dbl>          <dbl>
## 1 Adelie    female          73                 3369            269
## 2 Adelie    male            73                 4043            347
## 3 Chinstrap female          34                 3527            285
## 4 Chinstrap male            34                 3939            362
## 5 Gentoo    female          58                 4680            282
## 6 Gentoo    male            61                 5485            313
```

``` r
write_csv(dat_summ_out, "outputs/penguin_summary.csv")
```

You may have noticed in other exercises that column names with spaces can be specified inside tick marks (```). 

*What other formatting details might you want to consider before writing data to a file?*

## An R package for exporting to Word: `flextable`

The method above is useful if you have a table or two to place in a report. What if you have a lot? The amount of extra time it takes to reformat each (and update every time something changes) might make it worth investing in a more automated solution. The [`flextable`](https://ardata-fr.github.io/flextable-book/) package provides just that. Although there is a bit of a learning curve to using the package, the time investment pays off if you produce a lot of tables and want to be able to export them pre-formatted to Word (or HTML, or PowerPoint, or PDF). 

To export to Word or PowerPoint, you will also need the [`officer`](https://davidgohel.github.io/officer/) package, which provides functions to interface R with Microsoft Office.

There are plenty of other packages to help you with formatting tables; for example, the [`gt`](https://gt.rstudio.com/articles/gt.html) package supports creating beautiful, flexible, and complex tables (like those with nesting structures) and interfaces nicely with the tidyverse. It can't yet export all your formatting to Word though, which is likely something you will need to do, so for now we will focus on `flextable`. Also note that `flextable` is a really flexible package with a lot of options, and this reading covers only what are likely to be your most-used options. Check out the [excellent documentation](https://ardata-fr.github.io/flextable-book/) for more details.


``` r
install.packages("flextable")
install.packages("officer")
```


``` r
library(flextable)
```

```
## 
## Attaching package: 'flextable'
```

```
## The following object is masked from 'package:purrr':
## 
##     compose
```

``` r
library(officer)
```

### The structure of a flextable

A flextable has three parts: a *header*, *body*, and a *footer*. It's unusual to use a footer, but this is where you might put footnotes. The *header* is, by default, the column names, but could include multiple levels (for example, if column types are grouped and labeled - see below). 

### Formatting a flextable

Most of the functions in `flextable` are used for formatting your table; saving it to Office is just the last step. Formatting functions can change:

* text (font, face, justification, etc.)
* cells (colors, borders)
* table organization (shared headers)
* size and layout (column widths, row heights, etc.)

First, we have to convert our table into a `flextable` object:


``` r
ft <- flextable(dat_summ_out)
print(ft)
```

```
## a flextable object.
## col_keys: `species`, `sex`, `No. birds`, `Mean body mass (g)`, `SD body mass` 
## header has 1 row(s) 
## body has 6 row(s) 
## original dataset sample: 
## 'data.frame':	6 obs. of  5 variables:
##  $ species           : Factor w/ 3 levels "Adelie","Chinstrap",..: 1 1 2 2 3 3
##  $ sex               : Factor w/ 2 levels "female","male": 1 2 1 2 1 2
##  $ No. birds         : num  73 73 34 34 58 61
##  $ Mean body mass (g): num  3369 4043 3527 3939 4680 ...
##  $ SD body mass      : num  269 347 285 362 282 313
```

The table above shows you the default design of a flextable. Luckily, this pretty close to the format we want! It includes horizontal borders, left-aligns text columns, and right-aligns non-text columns. However, you can customize most of this. Like `ggplot2`, `flextable` comes with some built-in themes. For example `theme_zebra()` removes cell borders and adds alternating shading:


``` r
theme_zebra(ft)
```

```{=html}
<div class="tabwid"><style>.cl-42be2d1c{}.cl-42b855d6{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42b855ea{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42b9cbc8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42b9cbc9{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42b9dc4e{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42b9dc4f{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42b9dc50{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42b9dc58{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42b9dc62{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42b9dc63{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42be2d1c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42b9dc4e"><p class="cl-42b9cbc8"><span class="cl-42b855d6">species</span></p></th><th class="cl-42b9dc4e"><p class="cl-42b9cbc8"><span class="cl-42b855d6">sex</span></p></th><th class="cl-42b9dc4f"><p class="cl-42b9cbc9"><span class="cl-42b855d6">No. birds</span></p></th><th class="cl-42b9dc4f"><p class="cl-42b9cbc9"><span class="cl-42b855d6">Mean body mass (g)</span></p></th><th class="cl-42b9dc4f"><p class="cl-42b9cbc9"><span class="cl-42b855d6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42b9dc50"><p class="cl-42b9cbc8"><span class="cl-42b855ea">Adelie</span></p></td><td class="cl-42b9dc50"><p class="cl-42b9cbc8"><span class="cl-42b855ea">female</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">73</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">3,369</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42b9dc62"><p class="cl-42b9cbc8"><span class="cl-42b855ea">Adelie</span></p></td><td class="cl-42b9dc62"><p class="cl-42b9cbc8"><span class="cl-42b855ea">male</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">73</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">4,043</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42b9dc50"><p class="cl-42b9cbc8"><span class="cl-42b855ea">Chinstrap</span></p></td><td class="cl-42b9dc50"><p class="cl-42b9cbc8"><span class="cl-42b855ea">female</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">34</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">3,527</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42b9dc62"><p class="cl-42b9cbc8"><span class="cl-42b855ea">Chinstrap</span></p></td><td class="cl-42b9dc62"><p class="cl-42b9cbc8"><span class="cl-42b855ea">male</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">34</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">3,939</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42b9dc50"><p class="cl-42b9cbc8"><span class="cl-42b855ea">Gentoo</span></p></td><td class="cl-42b9dc50"><p class="cl-42b9cbc8"><span class="cl-42b855ea">female</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">58</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">4,680</span></p></td><td class="cl-42b9dc58"><p class="cl-42b9cbc9"><span class="cl-42b855ea">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42b9dc62"><p class="cl-42b9cbc8"><span class="cl-42b855ea">Gentoo</span></p></td><td class="cl-42b9dc62"><p class="cl-42b9cbc8"><span class="cl-42b855ea">male</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">61</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">5,485</span></p></td><td class="cl-42b9dc63"><p class="cl-42b9cbc9"><span class="cl-42b855ea">313</span></p></td></tr></tbody></table></div>
```

You can see examples of available themes [on the `flextable` webpage](https://ardata-fr.github.io/flextable-book/define-visual-properties.html#available-themes).

To format tables beyond these default themes, you will need to use the `style()` function. Within `style()`, you specify:

* the rows (argument `i`) and columns (argument `j`) you want to format
* the part of the table you want to format (argument `part`): "header", "body", "footer", or "all"
* information about what formatting to apply (argument `pr_t` for text, `pr_p` for paragraph formatting, and `pr_c` for cells). 

Since `style()` is mainly used to specify the selection (where, what, and how do you want to modify your table?), it also needs the actual formatting to be enclosed in a function. This can feel unwieldy, but just takes a little practice. 

These are easier to understand in an example:


``` r
ft <- flextable(dat_summ_out)
print(ft)
```

```
## a flextable object.
## col_keys: `species`, `sex`, `No. birds`, `Mean body mass (g)`, `SD body mass` 
## header has 1 row(s) 
## body has 6 row(s) 
## original dataset sample: 
## 'data.frame':	6 obs. of  5 variables:
##  $ species           : Factor w/ 3 levels "Adelie","Chinstrap",..: 1 1 2 2 3 3
##  $ sex               : Factor w/ 2 levels "female","male": 1 2 1 2 1 2
##  $ No. birds         : num  73 73 34 34 58 61
##  $ Mean body mass (g): num  3369 4043 3527 3939 4680 ...
##  $ SD body mass      : num  269 347 285 362 282 313
```

``` r
# Change header text to bold
ft %>%
  style(part = "header", pr_t = fp_text(bold = TRUE))
```

```{=html}
<div class="tabwid"><style>.cl-42d4cdba{}.cl-42d1b5da{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42d1b5e4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42d2f77e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42d2f77f{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42d30ad4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d30ad5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d30ade{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d30adf{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d30ae0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d30ae1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42d4cdba'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42d30ad4"><p class="cl-42d2f77e"><span class="cl-42d1b5da">species</span></p></th><th class="cl-42d30ad4"><p class="cl-42d2f77e"><span class="cl-42d1b5da">sex</span></p></th><th class="cl-42d30ad5"><p class="cl-42d2f77f"><span class="cl-42d1b5da">No. birds</span></p></th><th class="cl-42d30ad5"><p class="cl-42d2f77f"><span class="cl-42d1b5da">Mean body mass (g)</span></p></th><th class="cl-42d30ad5"><p class="cl-42d2f77f"><span class="cl-42d1b5da">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">Adelie</span></p></td><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">female</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">73</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">3,369</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">Adelie</span></p></td><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">male</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">73</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">4,043</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">Chinstrap</span></p></td><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">female</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">34</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">3,527</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">Chinstrap</span></p></td><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">male</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">34</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">3,939</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">Gentoo</span></p></td><td class="cl-42d30ade"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">female</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">58</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">4,680</span></p></td><td class="cl-42d30adf"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d30ae0"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">Gentoo</span></p></td><td class="cl-42d30ae0"><p class="cl-42d2f77e"><span class="cl-42d1b5e4">male</span></p></td><td class="cl-42d30ae1"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">61</span></p></td><td class="cl-42d30ae1"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">5,485</span></p></td><td class="cl-42d30ae1"><p class="cl-42d2f77f"><span class="cl-42d1b5e4">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-42d96e4c{}.cl-42d674d0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42d7af76{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-42d7c0ec{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d7c0ed{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42d7c0ee{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42d96e4c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42d7c0ec"><p class="cl-42d7af76"><span class="cl-42d674d0">species</span></p></th><th class="cl-42d7c0ec"><p class="cl-42d7af76"><span class="cl-42d674d0">sex</span></p></th><th class="cl-42d7c0ec"><p class="cl-42d7af76"><span class="cl-42d674d0">No. birds</span></p></th><th class="cl-42d7c0ec"><p class="cl-42d7af76"><span class="cl-42d674d0">Mean body mass (g)</span></p></th><th class="cl-42d7c0ec"><p class="cl-42d7af76"><span class="cl-42d674d0">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">Adelie</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">female</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">73</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">3,369</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">Adelie</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">male</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">73</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">4,043</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">Chinstrap</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">female</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">34</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">3,527</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">Chinstrap</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">male</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">34</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">3,939</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">Gentoo</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">female</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">58</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">4,680</span></p></td><td class="cl-42d7c0ed"><p class="cl-42d7af76"><span class="cl-42d674d0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42d7c0ee"><p class="cl-42d7af76"><span class="cl-42d674d0">Gentoo</span></p></td><td class="cl-42d7c0ee"><p class="cl-42d7af76"><span class="cl-42d674d0">male</span></p></td><td class="cl-42d7c0ee"><p class="cl-42d7af76"><span class="cl-42d674d0">61</span></p></td><td class="cl-42d7c0ee"><p class="cl-42d7af76"><span class="cl-42d674d0">5,485</span></p></td><td class="cl-42d7c0ee"><p class="cl-42d7af76"><span class="cl-42d674d0">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-42de09ac{}.cl-42db0360{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42dc3d2a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42dc3d2b{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42dc4d9c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4d9d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4da6{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4da7{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4da8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4da9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4db0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42dc4dba{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42de09ac'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42dc4d9c"><p class="cl-42dc3d2a"><span class="cl-42db0360">species</span></p></th><th class="cl-42dc4d9c"><p class="cl-42dc3d2a"><span class="cl-42db0360">sex</span></p></th><th class="cl-42dc4d9d"><p class="cl-42dc3d2b"><span class="cl-42db0360">No. birds</span></p></th><th class="cl-42dc4d9d"><p class="cl-42dc3d2b"><span class="cl-42db0360">Mean body mass (g)</span></p></th><th class="cl-42dc4d9d"><p class="cl-42dc3d2b"><span class="cl-42db0360">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42dc4da6"><p class="cl-42dc3d2a"><span class="cl-42db0360">Adelie</span></p></td><td class="cl-42dc4da6"><p class="cl-42dc3d2a"><span class="cl-42db0360">female</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">73</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">3,369</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42dc4da8"><p class="cl-42dc3d2a"><span class="cl-42db0360">Adelie</span></p></td><td class="cl-42dc4da8"><p class="cl-42dc3d2a"><span class="cl-42db0360">male</span></p></td><td class="cl-42dc4da9"><p class="cl-42dc3d2b"><span class="cl-42db0360">73</span></p></td><td class="cl-42dc4da9"><p class="cl-42dc3d2b"><span class="cl-42db0360">4,043</span></p></td><td class="cl-42dc4da9"><p class="cl-42dc3d2b"><span class="cl-42db0360">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42dc4da6"><p class="cl-42dc3d2a"><span class="cl-42db0360">Chinstrap</span></p></td><td class="cl-42dc4da6"><p class="cl-42dc3d2a"><span class="cl-42db0360">female</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">34</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">3,527</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42dc4da8"><p class="cl-42dc3d2a"><span class="cl-42db0360">Chinstrap</span></p></td><td class="cl-42dc4da8"><p class="cl-42dc3d2a"><span class="cl-42db0360">male</span></p></td><td class="cl-42dc4da9"><p class="cl-42dc3d2b"><span class="cl-42db0360">34</span></p></td><td class="cl-42dc4da9"><p class="cl-42dc3d2b"><span class="cl-42db0360">3,939</span></p></td><td class="cl-42dc4da9"><p class="cl-42dc3d2b"><span class="cl-42db0360">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42dc4da6"><p class="cl-42dc3d2a"><span class="cl-42db0360">Gentoo</span></p></td><td class="cl-42dc4da6"><p class="cl-42dc3d2a"><span class="cl-42db0360">female</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">58</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">4,680</span></p></td><td class="cl-42dc4da7"><p class="cl-42dc3d2b"><span class="cl-42db0360">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42dc4db0"><p class="cl-42dc3d2a"><span class="cl-42db0360">Gentoo</span></p></td><td class="cl-42dc4db0"><p class="cl-42dc3d2a"><span class="cl-42db0360">male</span></p></td><td class="cl-42dc4dba"><p class="cl-42dc3d2b"><span class="cl-42db0360">61</span></p></td><td class="cl-42dc4dba"><p class="cl-42dc3d2b"><span class="cl-42db0360">5,485</span></p></td><td class="cl-42dc4dba"><p class="cl-42dc3d2b"><span class="cl-42db0360">313</span></p></td></tr></tbody></table></div>
```

The help pages for `fp_text`, `fp_par`, and `fp_cell` will show you all the various options and what they are called.

It is also possible to specify rows with selection functions:


``` r
# Shade cells for Adelie penguins
ft %>%
  style(i = ~species == "Adelie" , part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-42e3ddf0{}.cl-42e0fa36{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42e21614{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42e21615{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42e22514{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e2251e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e2251f{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e22520{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e22528{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e22529{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e2252a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e22532{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42e3ddf0'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42e22514"><p class="cl-42e21614"><span class="cl-42e0fa36">species</span></p></th><th class="cl-42e22514"><p class="cl-42e21614"><span class="cl-42e0fa36">sex</span></p></th><th class="cl-42e2251e"><p class="cl-42e21615"><span class="cl-42e0fa36">No. birds</span></p></th><th class="cl-42e2251e"><p class="cl-42e21615"><span class="cl-42e0fa36">Mean body mass (g)</span></p></th><th class="cl-42e2251e"><p class="cl-42e21615"><span class="cl-42e0fa36">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42e2251f"><p class="cl-42e21614"><span class="cl-42e0fa36">Adelie</span></p></td><td class="cl-42e2251f"><p class="cl-42e21614"><span class="cl-42e0fa36">female</span></p></td><td class="cl-42e22520"><p class="cl-42e21615"><span class="cl-42e0fa36">73</span></p></td><td class="cl-42e22520"><p class="cl-42e21615"><span class="cl-42e0fa36">3,369</span></p></td><td class="cl-42e22520"><p class="cl-42e21615"><span class="cl-42e0fa36">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e2251f"><p class="cl-42e21614"><span class="cl-42e0fa36">Adelie</span></p></td><td class="cl-42e2251f"><p class="cl-42e21614"><span class="cl-42e0fa36">male</span></p></td><td class="cl-42e22520"><p class="cl-42e21615"><span class="cl-42e0fa36">73</span></p></td><td class="cl-42e22520"><p class="cl-42e21615"><span class="cl-42e0fa36">4,043</span></p></td><td class="cl-42e22520"><p class="cl-42e21615"><span class="cl-42e0fa36">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e22528"><p class="cl-42e21614"><span class="cl-42e0fa36">Chinstrap</span></p></td><td class="cl-42e22528"><p class="cl-42e21614"><span class="cl-42e0fa36">female</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">34</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">3,527</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e22528"><p class="cl-42e21614"><span class="cl-42e0fa36">Chinstrap</span></p></td><td class="cl-42e22528"><p class="cl-42e21614"><span class="cl-42e0fa36">male</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">34</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">3,939</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e22528"><p class="cl-42e21614"><span class="cl-42e0fa36">Gentoo</span></p></td><td class="cl-42e22528"><p class="cl-42e21614"><span class="cl-42e0fa36">female</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">58</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">4,680</span></p></td><td class="cl-42e22529"><p class="cl-42e21615"><span class="cl-42e0fa36">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e2252a"><p class="cl-42e21614"><span class="cl-42e0fa36">Gentoo</span></p></td><td class="cl-42e2252a"><p class="cl-42e21614"><span class="cl-42e0fa36">male</span></p></td><td class="cl-42e22532"><p class="cl-42e21615"><span class="cl-42e0fa36">61</span></p></td><td class="cl-42e22532"><p class="cl-42e21615"><span class="cl-42e0fa36">5,485</span></p></td><td class="cl-42e22532"><p class="cl-42e21615"><span class="cl-42e0fa36">313</span></p></td></tr></tbody></table></div>
```

There are also built-in functions that don't require you to use `style()`, including `font()`, `bold()`, `align()`, and so on - but you can always do these within `style()`, too.


``` r
ft %>%
  #make header row bold
  bold(part = "header", bold = T) %>%
  #italicize species names
  italic(part = "body", j = 1, italic = T)
```

```{=html}
<div class="tabwid"><style>.cl-42e87aae{}.cl-42e5c3cc{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42e5c3d6{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42e5c3d7{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42e6dbe0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42e6dbe1{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42e6e9a0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e6e9a1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e6e9aa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e6e9b4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e6e9b5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42e6e9b6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42e87aae'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42e6e9a0"><p class="cl-42e6dbe0"><span class="cl-42e5c3cc">species</span></p></th><th class="cl-42e6e9a0"><p class="cl-42e6dbe0"><span class="cl-42e5c3cc">sex</span></p></th><th class="cl-42e6e9a1"><p class="cl-42e6dbe1"><span class="cl-42e5c3cc">No. birds</span></p></th><th class="cl-42e6e9a1"><p class="cl-42e6dbe1"><span class="cl-42e5c3cc">Mean body mass (g)</span></p></th><th class="cl-42e6e9a1"><p class="cl-42e6dbe1"><span class="cl-42e5c3cc">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d6">Adelie</span></p></td><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d7">female</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">73</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">3,369</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d6">Adelie</span></p></td><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d7">male</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">73</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">4,043</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d6">Chinstrap</span></p></td><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d7">female</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">34</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">3,527</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d6">Chinstrap</span></p></td><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d7">male</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">34</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">3,939</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d6">Gentoo</span></p></td><td class="cl-42e6e9aa"><p class="cl-42e6dbe0"><span class="cl-42e5c3d7">female</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">58</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">4,680</span></p></td><td class="cl-42e6e9b4"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42e6e9b5"><p class="cl-42e6dbe0"><span class="cl-42e5c3d6">Gentoo</span></p></td><td class="cl-42e6e9b5"><p class="cl-42e6dbe0"><span class="cl-42e5c3d7">male</span></p></td><td class="cl-42e6e9b6"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">61</span></p></td><td class="cl-42e6e9b6"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">5,485</span></p></td><td class="cl-42e6e9b6"><p class="cl-42e6dbe1"><span class="cl-42e5c3d7">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-42ed330a{}.cl-42ea77dc{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42eb9d60{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42eb9d61{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42ebaaee{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ebaaef{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42ed330a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">species</span></p></th><th class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">sex</span></p></th><th class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">No. birds</span></p></th><th class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">Mean body mass (g)</span></p></th><th class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">Adelie</span></p></td><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">female</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">73</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">3,369</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">Adelie</span></p></td><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">male</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">73</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">4,043</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">Chinstrap</span></p></td><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">female</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">34</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">3,527</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">Chinstrap</span></p></td><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">male</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">34</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">3,939</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">Gentoo</span></p></td><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">female</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">58</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">4,680</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">Gentoo</span></p></td><td class="cl-42ebaaee"><p class="cl-42eb9d60"><span class="cl-42ea77dc">male</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">61</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">5,485</span></p></td><td class="cl-42ebaaef"><p class="cl-42eb9d61"><span class="cl-42ea77dc">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-42f20c68{}.cl-42ef5fcc{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42f07420{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42f0742a{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42f0826c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08276{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08277{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08278{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08279{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08280{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08281{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08282{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f08283{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f0828a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f0828b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f0828c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42f20c68'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42f0826c"><p class="cl-42f07420"><span class="cl-42ef5fcc">species</span></p></th><th class="cl-42f08276"><p class="cl-42f07420"><span class="cl-42ef5fcc">sex</span></p></th><th class="cl-42f08277"><p class="cl-42f0742a"><span class="cl-42ef5fcc">No. birds</span></p></th><th class="cl-42f08277"><p class="cl-42f0742a"><span class="cl-42ef5fcc">Mean body mass (g)</span></p></th><th class="cl-42f08278"><p class="cl-42f0742a"><span class="cl-42ef5fcc">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42f08279"><p class="cl-42f07420"><span class="cl-42ef5fcc">Adelie</span></p></td><td class="cl-42f08280"><p class="cl-42f07420"><span class="cl-42ef5fcc">female</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">73</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">3,369</span></p></td><td class="cl-42f08282"><p class="cl-42f0742a"><span class="cl-42ef5fcc">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f08279"><p class="cl-42f07420"><span class="cl-42ef5fcc">Adelie</span></p></td><td class="cl-42f08280"><p class="cl-42f07420"><span class="cl-42ef5fcc">male</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">73</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">4,043</span></p></td><td class="cl-42f08282"><p class="cl-42f0742a"><span class="cl-42ef5fcc">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f08279"><p class="cl-42f07420"><span class="cl-42ef5fcc">Chinstrap</span></p></td><td class="cl-42f08280"><p class="cl-42f07420"><span class="cl-42ef5fcc">female</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">34</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">3,527</span></p></td><td class="cl-42f08282"><p class="cl-42f0742a"><span class="cl-42ef5fcc">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f08279"><p class="cl-42f07420"><span class="cl-42ef5fcc">Chinstrap</span></p></td><td class="cl-42f08280"><p class="cl-42f07420"><span class="cl-42ef5fcc">male</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">34</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">3,939</span></p></td><td class="cl-42f08282"><p class="cl-42f0742a"><span class="cl-42ef5fcc">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f08279"><p class="cl-42f07420"><span class="cl-42ef5fcc">Gentoo</span></p></td><td class="cl-42f08280"><p class="cl-42f07420"><span class="cl-42ef5fcc">female</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">58</span></p></td><td class="cl-42f08281"><p class="cl-42f0742a"><span class="cl-42ef5fcc">4,680</span></p></td><td class="cl-42f08282"><p class="cl-42f0742a"><span class="cl-42ef5fcc">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f08283"><p class="cl-42f07420"><span class="cl-42ef5fcc">Gentoo</span></p></td><td class="cl-42f0828a"><p class="cl-42f07420"><span class="cl-42ef5fcc">male</span></p></td><td class="cl-42f0828b"><p class="cl-42f0742a"><span class="cl-42ef5fcc">61</span></p></td><td class="cl-42f0828b"><p class="cl-42f0742a"><span class="cl-42ef5fcc">5,485</span></p></td><td class="cl-42f0828c"><p class="cl-42f0742a"><span class="cl-42ef5fcc">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-42f61312{}.cl-42f374f4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42f48420{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42f48421{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42f4924e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f4924f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f49250{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f49258{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f49259{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f4925a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f4925b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f49262{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f49263{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f4926c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42f61312'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42f4924e"><p class="cl-42f48420"><span class="cl-42f374f4">species</span></p></th><th class="cl-42f4924e"><p class="cl-42f48420"><span class="cl-42f374f4">sex</span></p></th><th class="cl-42f4924f"><p class="cl-42f48421"><span class="cl-42f374f4">No. birds</span></p></th><th class="cl-42f4924f"><p class="cl-42f48421"><span class="cl-42f374f4">Mean body mass (g)</span></p></th><th class="cl-42f4924f"><p class="cl-42f48421"><span class="cl-42f374f4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42f49250"><p class="cl-42f48420"><span class="cl-42f374f4">Adelie</span></p></td><td class="cl-42f49258"><p class="cl-42f48420"><span class="cl-42f374f4">female</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">73</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">3,369</span></p></td><td class="cl-42f4925a"><p class="cl-42f48421"><span class="cl-42f374f4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f49250"><p class="cl-42f48420"><span class="cl-42f374f4">Adelie</span></p></td><td class="cl-42f49258"><p class="cl-42f48420"><span class="cl-42f374f4">male</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">73</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">4,043</span></p></td><td class="cl-42f4925a"><p class="cl-42f48421"><span class="cl-42f374f4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f49250"><p class="cl-42f48420"><span class="cl-42f374f4">Chinstrap</span></p></td><td class="cl-42f49258"><p class="cl-42f48420"><span class="cl-42f374f4">female</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">34</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">3,527</span></p></td><td class="cl-42f4925a"><p class="cl-42f48421"><span class="cl-42f374f4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f49250"><p class="cl-42f48420"><span class="cl-42f374f4">Chinstrap</span></p></td><td class="cl-42f49258"><p class="cl-42f48420"><span class="cl-42f374f4">male</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">34</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">3,939</span></p></td><td class="cl-42f4925a"><p class="cl-42f48421"><span class="cl-42f374f4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f49250"><p class="cl-42f48420"><span class="cl-42f374f4">Gentoo</span></p></td><td class="cl-42f49258"><p class="cl-42f48420"><span class="cl-42f374f4">female</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">58</span></p></td><td class="cl-42f49259"><p class="cl-42f48421"><span class="cl-42f374f4">4,680</span></p></td><td class="cl-42f4925a"><p class="cl-42f48421"><span class="cl-42f374f4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f4925b"><p class="cl-42f48420"><span class="cl-42f374f4">Gentoo</span></p></td><td class="cl-42f49262"><p class="cl-42f48420"><span class="cl-42f374f4">male</span></p></td><td class="cl-42f49263"><p class="cl-42f48421"><span class="cl-42f374f4">61</span></p></td><td class="cl-42f49263"><p class="cl-42f48421"><span class="cl-42f374f4">5,485</span></p></td><td class="cl-42f4926c"><p class="cl-42f48421"><span class="cl-42f374f4">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-42fa1e8a{}.cl-42f777d4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42f88c1e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42f88c28{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42f89a7e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a7f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a88{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a89{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a8a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a92{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a9c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42f89a9d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-42fa1e8a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42f89a7e"><p class="cl-42f88c1e"><span class="cl-42f777d4">species</span></p></th><th class="cl-42f89a7e"><p class="cl-42f88c1e"><span class="cl-42f777d4">sex</span></p></th><th class="cl-42f89a7f"><p class="cl-42f88c28"><span class="cl-42f777d4">No. birds</span></p></th><th class="cl-42f89a7f"><p class="cl-42f88c28"><span class="cl-42f777d4">Mean body mass (g)</span></p></th><th class="cl-42f89a7f"><p class="cl-42f88c28"><span class="cl-42f777d4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42f89a88"><p class="cl-42f88c1e"><span class="cl-42f777d4">Adelie</span></p></td><td class="cl-42f89a88"><p class="cl-42f88c1e"><span class="cl-42f777d4">female</span></p></td><td class="cl-42f89a89"><p class="cl-42f88c28"><span class="cl-42f777d4">73</span></p></td><td class="cl-42f89a89"><p class="cl-42f88c28"><span class="cl-42f777d4">3,369</span></p></td><td class="cl-42f89a89"><p class="cl-42f88c28"><span class="cl-42f777d4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">Adelie</span></p></td><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">male</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">73</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">4,043</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">Chinstrap</span></p></td><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">female</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">34</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">3,527</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">Chinstrap</span></p></td><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">male</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">34</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">3,939</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">Gentoo</span></p></td><td class="cl-42f89a8a"><p class="cl-42f88c1e"><span class="cl-42f777d4">female</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">58</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">4,680</span></p></td><td class="cl-42f89a92"><p class="cl-42f88c28"><span class="cl-42f777d4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42f89a9c"><p class="cl-42f88c1e"><span class="cl-42f777d4">Gentoo</span></p></td><td class="cl-42f89a9c"><p class="cl-42f88c1e"><span class="cl-42f777d4">male</span></p></td><td class="cl-42f89a9d"><p class="cl-42f88c28"><span class="cl-42f777d4">61</span></p></td><td class="cl-42f89a9d"><p class="cl-42f88c28"><span class="cl-42f777d4">5,485</span></p></td><td class="cl-42f89a9d"><p class="cl-42f88c28"><span class="cl-42f777d4">313</span></p></td></tr></tbody></table></div>
```

You can set some of these parameters as defaults with `set_flextable_defaults()`, so you don't have to define them every time for a series of tables.


``` r
set_flextable_defaults(
  font.color = "black",
  font.family = "Times",
  border.color = "darkgrey",
  theme_fun = "theme_vanilla")

flextable(penguins[c(1,50,100), 1:3])
```

```{=html}
<div class="tabwid"><style>.cl-4300c8ca{}.cl-42fe3420{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42fe3421{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-42ff4626{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42ff4627{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-42ff5382{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff5383{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff5384{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff538c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff538d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff538e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff538f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-42ff5390{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-4300c8ca'><thead><tr style="overflow-wrap:break-word;"><th class="cl-42ff5382"><p class="cl-42ff4626"><span class="cl-42fe3420">species</span></p></th><th class="cl-42ff5382"><p class="cl-42ff4626"><span class="cl-42fe3420">island</span></p></th><th class="cl-42ff5383"><p class="cl-42ff4627"><span class="cl-42fe3420">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-42ff5384"><p class="cl-42ff4626"><span class="cl-42fe3421">Adelie</span></p></td><td class="cl-42ff5384"><p class="cl-42ff4626"><span class="cl-42fe3421">Torgersen</span></p></td><td class="cl-42ff538c"><p class="cl-42ff4627"><span class="cl-42fe3421">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ff538d"><p class="cl-42ff4626"><span class="cl-42fe3421">Adelie</span></p></td><td class="cl-42ff538d"><p class="cl-42ff4626"><span class="cl-42fe3421">Dream</span></p></td><td class="cl-42ff538e"><p class="cl-42ff4627"><span class="cl-42fe3421">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-42ff538f"><p class="cl-42ff4626"><span class="cl-42fe3421">Adelie</span></p></td><td class="cl-42ff538f"><p class="cl-42ff4626"><span class="cl-42fe3421">Dream</span></p></td><td class="cl-42ff5390"><p class="cl-42ff4627"><span class="cl-42fe3421">43.2</span></p></td></tr></tbody></table></div>
```

### Flextable layout

In the example above, I renamed our summary table within `mutate` and rounded numbers to make things easier when exporting to a csv, so there would be less to change manually. But `flextable` provides options for changing header labels and editing cell content so you don't have to deal with using tick marks (or if you want to use your summary table elsewhere, later, and want column names without spaces).


``` r
ft <- dat_summ %>%
  flextable() %>%
  set_header_labels(n = "No. birds", 
                    body_mass_mean = "Mean body mass (g)",
                    body_mass_sd =  "SD body mass")
ft
```

```{=html}
<div class="tabwid"><style>.cl-43081300{}.cl-43055ab6{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-43055ac0{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-4306786a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-43067874{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-430686c0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686c1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686c2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686c3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686ca{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686cb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686cc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430686cd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-43081300'><thead><tr style="overflow-wrap:break-word;"><th class="cl-430686c0"><p class="cl-4306786a"><span class="cl-43055ab6">species</span></p></th><th class="cl-430686c0"><p class="cl-4306786a"><span class="cl-43055ab6">sex</span></p></th><th class="cl-430686c1"><p class="cl-43067874"><span class="cl-43055ab6">No. birds</span></p></th><th class="cl-430686c1"><p class="cl-43067874"><span class="cl-43055ab6">Mean body mass (g)</span></p></th><th class="cl-430686c1"><p class="cl-43067874"><span class="cl-43055ab6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-430686c2"><p class="cl-4306786a"><span class="cl-43055ac0">Adelie</span></p></td><td class="cl-430686c2"><p class="cl-4306786a"><span class="cl-43055ac0">female</span></p></td><td class="cl-430686c3"><p class="cl-43067874"><span class="cl-43055ac0">73</span></p></td><td class="cl-430686c3"><p class="cl-43067874"><span class="cl-43055ac0">3,368.836</span></p></td><td class="cl-430686c3"><p class="cl-43067874"><span class="cl-43055ac0">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">Adelie</span></p></td><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">male</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">73</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">4,043.493</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">Chinstrap</span></p></td><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">female</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">34</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">3,527.206</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">Chinstrap</span></p></td><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">male</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">34</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">3,938.971</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">Gentoo</span></p></td><td class="cl-430686ca"><p class="cl-4306786a"><span class="cl-43055ac0">female</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">58</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">4,679.741</span></p></td><td class="cl-430686cb"><p class="cl-43067874"><span class="cl-43055ac0">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430686cc"><p class="cl-4306786a"><span class="cl-43055ac0">Gentoo</span></p></td><td class="cl-430686cc"><p class="cl-4306786a"><span class="cl-43055ac0">male</span></p></td><td class="cl-430686cd"><p class="cl-43067874"><span class="cl-43055ac0">61</span></p></td><td class="cl-430686cd"><p class="cl-43067874"><span class="cl-43055ac0">5,484.836</span></p></td><td class="cl-430686cd"><p class="cl-43067874"><span class="cl-43055ac0">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-430d291c{}.cl-4309ea9a{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-4309eaa4{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-430b9a34{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-430b9a3e{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-430ba7b8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7b9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7c2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7c3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7c4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7c5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7cc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-430ba7cd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-430d291c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-430ba7b8"><p class="cl-430b9a34"><span class="cl-4309ea9a">species</span></p></th><th class="cl-430ba7b8"><p class="cl-430b9a34"><span class="cl-4309ea9a">sex</span></p></th><th class="cl-430ba7b9"><p class="cl-430b9a3e"><span class="cl-4309ea9a">No. birds</span></p></th><th class="cl-430ba7b9"><p class="cl-430b9a3e"><span class="cl-4309ea9a">Mean body mass (g)</span></p></th><th class="cl-430ba7b9"><p class="cl-430b9a3e"><span class="cl-4309ea9a">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-430ba7c2"><p class="cl-430b9a34"><span class="cl-4309eaa4">Adelie</span></p></td><td class="cl-430ba7c2"><p class="cl-430b9a34"><span class="cl-4309eaa4">female</span></p></td><td class="cl-430ba7c3"><p class="cl-430b9a3e"><span class="cl-4309eaa4">73</span></p></td><td class="cl-430ba7c3"><p class="cl-430b9a3e"><span class="cl-4309eaa4">3369</span></p></td><td class="cl-430ba7c3"><p class="cl-430b9a3e"><span class="cl-4309eaa4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">Adelie</span></p></td><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">male</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">73</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">4043</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">Chinstrap</span></p></td><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">female</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">34</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">3527</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">Chinstrap</span></p></td><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">male</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">34</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">3939</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">Gentoo</span></p></td><td class="cl-430ba7c4"><p class="cl-430b9a34"><span class="cl-4309eaa4">female</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">58</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">4680</span></p></td><td class="cl-430ba7c5"><p class="cl-430b9a3e"><span class="cl-4309eaa4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-430ba7cc"><p class="cl-430b9a34"><span class="cl-4309eaa4">Gentoo</span></p></td><td class="cl-430ba7cc"><p class="cl-430b9a34"><span class="cl-4309eaa4">male</span></p></td><td class="cl-430ba7cd"><p class="cl-430b9a3e"><span class="cl-4309eaa4">61</span></p></td><td class="cl-430ba7cd"><p class="cl-430b9a3e"><span class="cl-4309eaa4">5485</span></p></td><td class="cl-430ba7cd"><p class="cl-430b9a3e"><span class="cl-4309eaa4">313</span></p></td></tr></tbody></table></div>
```

You can also merge adjacent rows or columns with the same values. This is especially useful for grouped tables like ours:


``` r
ft <- ft %>%
  merge_v(j = "species")
```

To edit column sizes, use the `width()` option:


``` r
ft <- ft %>%
  width(width = 1)
ft
```

```{=html}
<div class="tabwid"><style>.cl-43122a16{}.cl-430f79ba{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-430f79bb{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-43109656{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-43109657{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-4310a47a{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a47b{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a47c{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a484{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a485{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a486{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a487{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-4310a48e{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-43122a16'><thead><tr style="overflow-wrap:break-word;"><th class="cl-4310a47a"><p class="cl-43109656"><span class="cl-430f79ba">species</span></p></th><th class="cl-4310a47a"><p class="cl-43109656"><span class="cl-430f79ba">sex</span></p></th><th class="cl-4310a47b"><p class="cl-43109657"><span class="cl-430f79ba">No. birds</span></p></th><th class="cl-4310a47b"><p class="cl-43109657"><span class="cl-430f79ba">Mean body mass (g)</span></p></th><th class="cl-4310a47b"><p class="cl-43109657"><span class="cl-430f79ba">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-4310a47c"><p class="cl-43109656"><span class="cl-430f79bb">Adelie</span></p></td><td class="cl-4310a47c"><p class="cl-43109656"><span class="cl-430f79bb">female</span></p></td><td class="cl-4310a484"><p class="cl-43109657"><span class="cl-430f79bb">73</span></p></td><td class="cl-4310a484"><p class="cl-43109657"><span class="cl-430f79bb">3,368.836</span></p></td><td class="cl-4310a484"><p class="cl-43109657"><span class="cl-430f79bb">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-4310a485"><p class="cl-43109656"><span class="cl-430f79bb">male</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">73</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">4,043.493</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-4310a485"><p class="cl-43109656"><span class="cl-430f79bb">Chinstrap</span></p></td><td class="cl-4310a485"><p class="cl-43109656"><span class="cl-430f79bb">female</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">34</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">3,527.206</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-4310a485"><p class="cl-43109656"><span class="cl-430f79bb">male</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">34</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">3,938.971</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-4310a487"><p class="cl-43109656"><span class="cl-430f79bb">Gentoo</span></p></td><td class="cl-4310a485"><p class="cl-43109656"><span class="cl-430f79bb">female</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">58</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">4,679.741</span></p></td><td class="cl-4310a486"><p class="cl-43109657"><span class="cl-430f79bb">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-4310a487"><p class="cl-43109656"><span class="cl-430f79bb">male</span></p></td><td class="cl-4310a48e"><p class="cl-43109657"><span class="cl-430f79bb">61</span></p></td><td class="cl-4310a48e"><p class="cl-43109657"><span class="cl-430f79bb">5,484.836</span></p></td><td class="cl-4310a48e"><p class="cl-43109657"><span class="cl-430f79bb">313.1586</span></p></td></tr></tbody></table></div>
```

### Exporting a flextable

You have five format options for exporting a `flextable`: html, docx, image, rtf, and pptx. Just use save_as_* for each of these, followed by a file name *with the proper extension*:


``` r
save_as_docx(ft, path = "outputs/flextable_penguins.docx")
```


<div class="figure" style="text-align: center">
<img src="images/penguins_table_word.png" alt="Just like the Viewer version!" width="50%" />
<p class="caption">(\#fig:unnamed-chunk-17)Just like the Viewer version!</p>
</div>

<!-- ## An R package for table design: `gt` -->

<!-- The method above is useful if you have a table or two to place in a report. What if you have a lot? The amount of extra time it takes to reformat each (and update every time something changes) might make it worth investing in a more automated solution. The [`gt`](https://gt.rstudio.com/articles/gt.html) package provides just that. It is very flexible, allowing you to create complex, nested tables. Although there is a bit of a learning curve to using the package, the time investment pays off if you produce a lot of tables. -->

<!-- Here's an example of our penguin data in `gt` format: -->

<!-- ```{r} -->
<!-- library(gt) -->
<!-- gt(dat_summ_out) -->
<!-- ``` -->

<!-- The formatting might be a little unexpected, because it's different than what we saw before. Now, instead of repeating species across sex, the table is nested by species. `gt` knew to do this because our tibble/data frame was grouped by species: -->

<!-- ```{r} -->
<!-- class(dat_summ_out) -->
<!-- groups(dat_summ_out) -->
<!-- ``` -->

<!-- By default `dplyr` maintains the groups in `group_by()` even after the summary is done. We could ungroup the table, in which case, our output would change: -->

<!-- ```{r} -->
<!-- dat_summ_out %>% -->
<!--   ungroup() %>% -->
<!--   gt() -->
<!-- ``` -->

<!-- This output is an image. You could copy it into a document, but that it wouldn't be embedded in the same way a table is. Instead, we can use the `gtsave()` function with a given file ending to save our output in a format for a word processor. -->

<!-- ```{r} -->
<!-- dat_summ_out %>% -->
<!--   gt() %>% -->
<!--   gtsave("outputs/penguin_bm_summary.docx") -->
<!-- ``` -->

<!-- ```{r, fig.cap="The default gtable format in Word", fig.align='center', out.width='80%', echo = FALSE, eval = TRUE} -->
<!-- knitr::include_graphics("images/gtable-out.png") -->
<!-- ``` -->

<!-- Now this format is readable by Word; if you click in the table, the usual formatting options will appear. Next, let's work on formatting the table to look exactly like the manually created one. To start off, we'll create an object for our table so we can modify it: -->

<!-- ```{r} -->
<!-- gt_penguins <- gt(ungroup(dat_summ_out)) -->
<!-- ``` -->

<!-- I usually write my documents in Times New Roman, so first I'll change the font using the `opt_table_font()` function, which defines fonts for the entire table: -->

<!-- ```{r} -->
<!-- gt_penguins <- opt_table_font(gt_penguins, font = "Times") -->
<!-- gt_penguins -->
<!-- ``` -->

<!-- Now I will change the header row to bold font. This time I use the `tab_style()` function, since I only want to change properties for specific cells, not the entire table: -->

<!-- ```{r} -->
<!-- gt_penguins <- tab_style(gt_penguins,  -->
<!--                          style = cell_text(weight = "bold"),  -->
<!--                          location = cells_column_labels()) -->
<!-- gt_penguins -->
<!-- ``` -->

<!-- Looking in the help funciton for `tab_style()`, I learned that to the `style` argument I need to specify a function like `cell_text()` or `cell_fill()`, then the options within that function. The `location` argument indicates which cells to edit. Again, this should be a function starting with `cells_*`, which is essentially a selection argument. The help functions will be useful to you here to find the cells you are looking for. -->

<!-- Now I will work on the borders. This workflow is similar to the one I used above. First, I remove all the borders using an `opt_*` function, then I add borders back where I want them using `tab_style()`: -->

<!-- ```{r} -->
<!-- gt_penguins <- gt_penguins %>% -->
<!--   opt_table_lines(extent = "none") %>% -->
<!--   tab_style(style = cell_borders(sides = c("top", "bottom")), -->
<!--             location = cells_column_labels()) %>% -->
<!--   tab_style(style = cell_borders(sides = "bottom"), -->
<!--             location = cells_body(rows = nrow(dat_summ_out))) #add bottom border to the last row -->
<!-- gt_penguins -->
<!-- ``` -->

<!-- All that's left is the alignment. I want to left-align my character columns and right-align my numeric columns. -->

<!-- ```{r} -->
<!-- gt_penguins <- gt_penguins %>% -->
<!--   tab_style(style = cell_text(align = "left"), -->
<!--             location = list(cells_column_labels(columns = c("Species","Sex")), -->
<!--                             cells_body(columns = c("Species","Sex")))) -->
<!-- gt_penguins -->
<!-- ``` -->

<!-- Notice that because I wanted to do this to both the body cells and the column labels, I had to specify both of those and combine them as a `list`. -->

<!-- Finally, let's do the same thing, but for the grouped data. Now, we need to deal with design choices related to the groups, but first I will change the font: -->

<!-- ```{r} -->
<!-- gt_penguins <- dat_summ_out %>% -->
<!--   gt() %>% -->
<!--   opt_table_font(font = "Times") %>% -->
<!--   tab_style(style = cell_text(weight = "bold"),  -->
<!--                          location = cells_column_labels())  -->
<!-- gt_penguins  -->
<!-- ``` -->

<!-- Now, notice that the species are on their own line. I typically would include them as another column, so I'll do that using the strategy I found on [this Stack Overflow page](https://stackoverflow.com/questions/76260847/how-can-i-put-the-groups-in-an-extra-column-in-a-gt-table): -->

<!-- ```{r} -->
<!-- gt_penguins <- gt_penguins %>% -->
<!--   tab_options(row_group.as_column = TRUE) -->
<!-- gt_penguins  -->
<!-- ``` -->

<!-- Now I can use the same lines and alignment as I did before: -->

<!-- ```{r} -->
<!-- gt_penguins %>% -->
<!--   tab_style(style = cell_text(align = "left"), -->
<!--             location = list(cells_column_labels(columns = "Sex"), -->
<!--                             cells_body(columns = "Sex"))) %>% -->
<!--   opt_table_lines(extent = "none") %>% -->
<!--   tab_style(style = cell_borders(sides = c("top", "bottom")), -->
<!--             location = cells_column_labels()) %>% -->
<!--   tab_style(style = cell_borders(sides = "bottom"), -->
<!--             location = cells_body(rows = nrow(dat_summ_out)))  -->
<!-- ``` -->

<!-- Oops, that's not what I wanted. I guess `cells_column_labels()` only gives me named columns, whereas the species column no longer has a name. Trying again: -->

<!-- ```{r} -->
<!-- gt_penguins <- gt_penguins %>% -->
<!--   tab_style(style = cell_text(align = "left"), -->
<!--             location = list(cells_column_labels(columns = "Sex"), -->
<!--                             cells_body(columns = "Sex"))) %>% -->
<!--   opt_table_lines(extent = "none") %>% -->
<!--   tab_style(style = cell_borders(sides = c("top", "bottom")), -->
<!--             location = list(cells_column_labels(),  -->
<!--                             cells_stubhead())) %>% -->
<!--   tab_style(style = cell_borders(sides = "bottom"), -->
<!--             location = list(cells_body(rows = nrow(dat_summ_out)), -->
<!--                             cells_row_groups(groups = n_distinct(dat_summ_out$Species))))  -->
<!-- gt_penguins -->
<!-- ``` -->

<!-- The **stub** is the area to the left in a table that contains row labels, row group labels, and/or summary labels; in this case it is the species group. The `cells_stubhead()` function finds the header of these cells, so in this case the top-leftmost cell of the table. Similarly, `cells_row_groups()` finds all the grouping rows; I just want the last one, so I use the number of species groups present (`n_distinct(dat_summ_out$Species`)). -->

<!-- This last example highlights one of the challenges of `gt`: there are so many options that they can be hard to find. The [documentation](https://gt.rstudio.com/reference/index.html) is very helpful. Also, once you understand the basic structure of the functions, you can try using autofill to suggest other options. For example, if you know you want to select a cell type but don't know what it's called, try typing in `?cell_` and checking out your options. -->

<!-- Additional features of `gt` include: -->

<!-- * Footnotes -->
<!-- * Embedded title and subtitle -->
<!-- * Grouped columns -->
<!-- * ... -->

<!-- A note: at the time of writing, formatting is not preserved perfectly across output formats in `gt` (e.g., what you see in the Viewer is not exactly what Word shows). [This fix is in progress](https://github.com/rstudio/gt/issues/1098). -->
