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
<div class="tabwid"><style>.cl-c7d718c2{}.cl-c7d04fa6{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7d04fb0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7d22eb6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7d22ec0{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7d241d0{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7d241da{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7d241db{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7d241e4{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7d241e5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7d241e6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c7d718c2'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c7d241d0"><p class="cl-c7d22eb6"><span class="cl-c7d04fa6">species</span></p></th><th class="cl-c7d241d0"><p class="cl-c7d22eb6"><span class="cl-c7d04fa6">sex</span></p></th><th class="cl-c7d241da"><p class="cl-c7d22ec0"><span class="cl-c7d04fa6">No. birds</span></p></th><th class="cl-c7d241da"><p class="cl-c7d22ec0"><span class="cl-c7d04fa6">Mean body mass (g)</span></p></th><th class="cl-c7d241da"><p class="cl-c7d22ec0"><span class="cl-c7d04fa6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c7d241db"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">Adelie</span></p></td><td class="cl-c7d241db"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">female</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">73</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">3,369</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7d241e5"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">Adelie</span></p></td><td class="cl-c7d241e5"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">male</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">73</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">4,043</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7d241db"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">Chinstrap</span></p></td><td class="cl-c7d241db"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">female</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">34</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">3,527</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7d241e5"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">Chinstrap</span></p></td><td class="cl-c7d241e5"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">male</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">34</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">3,939</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7d241db"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">Gentoo</span></p></td><td class="cl-c7d241db"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">female</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">58</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">4,680</span></p></td><td class="cl-c7d241e4"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7d241e5"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">Gentoo</span></p></td><td class="cl-c7d241e5"><p class="cl-c7d22eb6"><span class="cl-c7d04fb0">male</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">61</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">5,485</span></p></td><td class="cl-c7d241e6"><p class="cl-c7d22ec0"><span class="cl-c7d04fb0">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-c7ecd00e{}.cl-c7e9ebe6{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7e9ebf0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7eb16e2{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7eb16e3{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7eb26e6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7eb26e7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7eb26e8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7eb26f0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7eb26f1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7eb26f2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c7ecd00e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c7eb26e6"><p class="cl-c7eb16e2"><span class="cl-c7e9ebe6">species</span></p></th><th class="cl-c7eb26e6"><p class="cl-c7eb16e2"><span class="cl-c7e9ebe6">sex</span></p></th><th class="cl-c7eb26e7"><p class="cl-c7eb16e3"><span class="cl-c7e9ebe6">No. birds</span></p></th><th class="cl-c7eb26e7"><p class="cl-c7eb16e3"><span class="cl-c7e9ebe6">Mean body mass (g)</span></p></th><th class="cl-c7eb26e7"><p class="cl-c7eb16e3"><span class="cl-c7e9ebe6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">Adelie</span></p></td><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">female</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">73</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">3,369</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">Adelie</span></p></td><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">male</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">73</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">4,043</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">Chinstrap</span></p></td><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">female</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">34</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">3,527</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">Chinstrap</span></p></td><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">male</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">34</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">3,939</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">Gentoo</span></p></td><td class="cl-c7eb26e8"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">female</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">58</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">4,680</span></p></td><td class="cl-c7eb26f0"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7eb26f1"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">Gentoo</span></p></td><td class="cl-c7eb26f1"><p class="cl-c7eb16e2"><span class="cl-c7e9ebf0">male</span></p></td><td class="cl-c7eb26f2"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">61</span></p></td><td class="cl-c7eb26f2"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">5,485</span></p></td><td class="cl-c7eb26f2"><p class="cl-c7eb16e3"><span class="cl-c7e9ebf0">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-c7f16196{}.cl-c7ee6de2{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7efa4a0{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-c7efb558{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7efb562{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7efb563{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c7f16196'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c7efb558"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">species</span></p></th><th class="cl-c7efb558"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">sex</span></p></th><th class="cl-c7efb558"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">No. birds</span></p></th><th class="cl-c7efb558"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Mean body mass (g)</span></p></th><th class="cl-c7efb558"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Adelie</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">female</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">73</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">3,369</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Adelie</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">male</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">73</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">4,043</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Chinstrap</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">female</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">34</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">3,527</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Chinstrap</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">male</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">34</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">3,939</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Gentoo</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">female</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">58</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">4,680</span></p></td><td class="cl-c7efb562"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7efb563"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">Gentoo</span></p></td><td class="cl-c7efb563"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">male</span></p></td><td class="cl-c7efb563"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">61</span></p></td><td class="cl-c7efb563"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">5,485</span></p></td><td class="cl-c7efb563"><p class="cl-c7efa4a0"><span class="cl-c7ee6de2">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-c7f5ed9c{}.cl-c7f2f740{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7f42b92{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7f42b9c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7f43c72{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c7c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c7d{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c7e{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c7f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c80{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c86{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f43c87{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c7f5ed9c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c7f43c72"><p class="cl-c7f42b92"><span class="cl-c7f2f740">species</span></p></th><th class="cl-c7f43c72"><p class="cl-c7f42b92"><span class="cl-c7f2f740">sex</span></p></th><th class="cl-c7f43c7c"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">No. birds</span></p></th><th class="cl-c7f43c7c"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">Mean body mass (g)</span></p></th><th class="cl-c7f43c7c"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c7f43c7d"><p class="cl-c7f42b92"><span class="cl-c7f2f740">Adelie</span></p></td><td class="cl-c7f43c7d"><p class="cl-c7f42b92"><span class="cl-c7f2f740">female</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">73</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">3,369</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f43c7f"><p class="cl-c7f42b92"><span class="cl-c7f2f740">Adelie</span></p></td><td class="cl-c7f43c7f"><p class="cl-c7f42b92"><span class="cl-c7f2f740">male</span></p></td><td class="cl-c7f43c80"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">73</span></p></td><td class="cl-c7f43c80"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">4,043</span></p></td><td class="cl-c7f43c80"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f43c7d"><p class="cl-c7f42b92"><span class="cl-c7f2f740">Chinstrap</span></p></td><td class="cl-c7f43c7d"><p class="cl-c7f42b92"><span class="cl-c7f2f740">female</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">34</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">3,527</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f43c7f"><p class="cl-c7f42b92"><span class="cl-c7f2f740">Chinstrap</span></p></td><td class="cl-c7f43c7f"><p class="cl-c7f42b92"><span class="cl-c7f2f740">male</span></p></td><td class="cl-c7f43c80"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">34</span></p></td><td class="cl-c7f43c80"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">3,939</span></p></td><td class="cl-c7f43c80"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f43c7d"><p class="cl-c7f42b92"><span class="cl-c7f2f740">Gentoo</span></p></td><td class="cl-c7f43c7d"><p class="cl-c7f42b92"><span class="cl-c7f2f740">female</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">58</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">4,680</span></p></td><td class="cl-c7f43c7e"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f43c86"><p class="cl-c7f42b92"><span class="cl-c7f2f740">Gentoo</span></p></td><td class="cl-c7f43c86"><p class="cl-c7f42b92"><span class="cl-c7f2f740">male</span></p></td><td class="cl-c7f43c87"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">61</span></p></td><td class="cl-c7f43c87"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">5,485</span></p></td><td class="cl-c7f43c87"><p class="cl-c7f42b9c"><span class="cl-c7f2f740">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-c7fb6402{}.cl-c7f89ab0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7f9c656{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7f9c660{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7f9d5f6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d5f7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d5f8{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d600{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d601{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d602{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d60a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7f9d60b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c7fb6402'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c7f9d5f6"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">species</span></p></th><th class="cl-c7f9d5f6"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">sex</span></p></th><th class="cl-c7f9d5f7"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">No. birds</span></p></th><th class="cl-c7f9d5f7"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">Mean body mass (g)</span></p></th><th class="cl-c7f9d5f7"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c7f9d5f8"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">Adelie</span></p></td><td class="cl-c7f9d5f8"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">female</span></p></td><td class="cl-c7f9d600"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">73</span></p></td><td class="cl-c7f9d600"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">3,369</span></p></td><td class="cl-c7f9d600"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f9d5f8"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">Adelie</span></p></td><td class="cl-c7f9d5f8"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">male</span></p></td><td class="cl-c7f9d600"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">73</span></p></td><td class="cl-c7f9d600"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">4,043</span></p></td><td class="cl-c7f9d600"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f9d601"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">Chinstrap</span></p></td><td class="cl-c7f9d601"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">female</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">34</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">3,527</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f9d601"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">Chinstrap</span></p></td><td class="cl-c7f9d601"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">male</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">34</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">3,939</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f9d601"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">Gentoo</span></p></td><td class="cl-c7f9d601"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">female</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">58</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">4,680</span></p></td><td class="cl-c7f9d602"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7f9d60a"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">Gentoo</span></p></td><td class="cl-c7f9d60a"><p class="cl-c7f9c656"><span class="cl-c7f89ab0">male</span></p></td><td class="cl-c7f9d60b"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">61</span></p></td><td class="cl-c7f9d60b"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">5,485</span></p></td><td class="cl-c7f9d60b"><p class="cl-c7f9c660"><span class="cl-c7f89ab0">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-c7fffd50{}.cl-c7fd465a{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7fd465b{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7fd465c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c7fe6094{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7fe609e{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c7fe700c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7fe7016{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7fe7017{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7fe7018{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7fe7019{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c7fe7020{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c7fffd50'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c7fe700c"><p class="cl-c7fe6094"><span class="cl-c7fd465a">species</span></p></th><th class="cl-c7fe700c"><p class="cl-c7fe6094"><span class="cl-c7fd465a">sex</span></p></th><th class="cl-c7fe7016"><p class="cl-c7fe609e"><span class="cl-c7fd465a">No. birds</span></p></th><th class="cl-c7fe7016"><p class="cl-c7fe609e"><span class="cl-c7fd465a">Mean body mass (g)</span></p></th><th class="cl-c7fe7016"><p class="cl-c7fe609e"><span class="cl-c7fd465a">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465b">Adelie</span></p></td><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465c">female</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">73</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">3,369</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465b">Adelie</span></p></td><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465c">male</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">73</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">4,043</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465b">Chinstrap</span></p></td><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465c">female</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">34</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">3,527</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465b">Chinstrap</span></p></td><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465c">male</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">34</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">3,939</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465b">Gentoo</span></p></td><td class="cl-c7fe7017"><p class="cl-c7fe6094"><span class="cl-c7fd465c">female</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">58</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">4,680</span></p></td><td class="cl-c7fe7018"><p class="cl-c7fe609e"><span class="cl-c7fd465c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c7fe7019"><p class="cl-c7fe6094"><span class="cl-c7fd465b">Gentoo</span></p></td><td class="cl-c7fe7019"><p class="cl-c7fe6094"><span class="cl-c7fd465c">male</span></p></td><td class="cl-c7fe7020"><p class="cl-c7fe609e"><span class="cl-c7fd465c">61</span></p></td><td class="cl-c7fe7020"><p class="cl-c7fe609e"><span class="cl-c7fd465c">5,485</span></p></td><td class="cl-c7fe7020"><p class="cl-c7fe609e"><span class="cl-c7fd465c">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-c8048604{}.cl-c801d83c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c802f776{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c802f777{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c80304f0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80304f1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c8048604'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">species</span></p></th><th class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">sex</span></p></th><th class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">No. birds</span></p></th><th class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">Mean body mass (g)</span></p></th><th class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">Adelie</span></p></td><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">female</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">73</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">3,369</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">Adelie</span></p></td><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">male</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">73</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">4,043</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">Chinstrap</span></p></td><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">female</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">34</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">3,527</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">Chinstrap</span></p></td><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">male</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">34</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">3,939</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">Gentoo</span></p></td><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">female</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">58</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">4,680</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">Gentoo</span></p></td><td class="cl-c80304f0"><p class="cl-c802f776"><span class="cl-c801d83c">male</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">61</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">5,485</span></p></td><td class="cl-c80304f1"><p class="cl-c802f777"><span class="cl-c801d83c">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-c8092e20{}.cl-c8068ca6{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c8079be6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c8079be7{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c807a866{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a867{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a868{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a869{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a870{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a871{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a872{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a873{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a87a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a87b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a87c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c807a87d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c8092e20'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c807a866"><p class="cl-c8079be6"><span class="cl-c8068ca6">species</span></p></th><th class="cl-c807a867"><p class="cl-c8079be6"><span class="cl-c8068ca6">sex</span></p></th><th class="cl-c807a868"><p class="cl-c8079be7"><span class="cl-c8068ca6">No. birds</span></p></th><th class="cl-c807a868"><p class="cl-c8079be7"><span class="cl-c8068ca6">Mean body mass (g)</span></p></th><th class="cl-c807a869"><p class="cl-c8079be7"><span class="cl-c8068ca6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c807a870"><p class="cl-c8079be6"><span class="cl-c8068ca6">Adelie</span></p></td><td class="cl-c807a871"><p class="cl-c8079be6"><span class="cl-c8068ca6">female</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">73</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">3,369</span></p></td><td class="cl-c807a873"><p class="cl-c8079be7"><span class="cl-c8068ca6">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c807a870"><p class="cl-c8079be6"><span class="cl-c8068ca6">Adelie</span></p></td><td class="cl-c807a871"><p class="cl-c8079be6"><span class="cl-c8068ca6">male</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">73</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">4,043</span></p></td><td class="cl-c807a873"><p class="cl-c8079be7"><span class="cl-c8068ca6">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c807a870"><p class="cl-c8079be6"><span class="cl-c8068ca6">Chinstrap</span></p></td><td class="cl-c807a871"><p class="cl-c8079be6"><span class="cl-c8068ca6">female</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">34</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">3,527</span></p></td><td class="cl-c807a873"><p class="cl-c8079be7"><span class="cl-c8068ca6">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c807a870"><p class="cl-c8079be6"><span class="cl-c8068ca6">Chinstrap</span></p></td><td class="cl-c807a871"><p class="cl-c8079be6"><span class="cl-c8068ca6">male</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">34</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">3,939</span></p></td><td class="cl-c807a873"><p class="cl-c8079be7"><span class="cl-c8068ca6">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c807a870"><p class="cl-c8079be6"><span class="cl-c8068ca6">Gentoo</span></p></td><td class="cl-c807a871"><p class="cl-c8079be6"><span class="cl-c8068ca6">female</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">58</span></p></td><td class="cl-c807a872"><p class="cl-c8079be7"><span class="cl-c8068ca6">4,680</span></p></td><td class="cl-c807a873"><p class="cl-c8079be7"><span class="cl-c8068ca6">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c807a87a"><p class="cl-c8079be6"><span class="cl-c8068ca6">Gentoo</span></p></td><td class="cl-c807a87b"><p class="cl-c8079be6"><span class="cl-c8068ca6">male</span></p></td><td class="cl-c807a87c"><p class="cl-c8079be7"><span class="cl-c8068ca6">61</span></p></td><td class="cl-c807a87c"><p class="cl-c8079be7"><span class="cl-c8068ca6">5,485</span></p></td><td class="cl-c807a87d"><p class="cl-c8079be7"><span class="cl-c8068ca6">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-c80d5a7c{}.cl-c80a9ea4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c80bb578{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c80bb582{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c80bc3b0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3b1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3b2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3b3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3ba{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3bb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3bc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3bd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3c4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80bc3c5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c80d5a7c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c80bc3b0"><p class="cl-c80bb578"><span class="cl-c80a9ea4">species</span></p></th><th class="cl-c80bc3b0"><p class="cl-c80bb578"><span class="cl-c80a9ea4">sex</span></p></th><th class="cl-c80bc3b1"><p class="cl-c80bb582"><span class="cl-c80a9ea4">No. birds</span></p></th><th class="cl-c80bc3b1"><p class="cl-c80bb582"><span class="cl-c80a9ea4">Mean body mass (g)</span></p></th><th class="cl-c80bc3b1"><p class="cl-c80bb582"><span class="cl-c80a9ea4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c80bc3b2"><p class="cl-c80bb578"><span class="cl-c80a9ea4">Adelie</span></p></td><td class="cl-c80bc3b3"><p class="cl-c80bb578"><span class="cl-c80a9ea4">female</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">73</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">3,369</span></p></td><td class="cl-c80bc3bb"><p class="cl-c80bb582"><span class="cl-c80a9ea4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80bc3b2"><p class="cl-c80bb578"><span class="cl-c80a9ea4">Adelie</span></p></td><td class="cl-c80bc3b3"><p class="cl-c80bb578"><span class="cl-c80a9ea4">male</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">73</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">4,043</span></p></td><td class="cl-c80bc3bb"><p class="cl-c80bb582"><span class="cl-c80a9ea4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80bc3b2"><p class="cl-c80bb578"><span class="cl-c80a9ea4">Chinstrap</span></p></td><td class="cl-c80bc3b3"><p class="cl-c80bb578"><span class="cl-c80a9ea4">female</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">34</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">3,527</span></p></td><td class="cl-c80bc3bb"><p class="cl-c80bb582"><span class="cl-c80a9ea4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80bc3b2"><p class="cl-c80bb578"><span class="cl-c80a9ea4">Chinstrap</span></p></td><td class="cl-c80bc3b3"><p class="cl-c80bb578"><span class="cl-c80a9ea4">male</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">34</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">3,939</span></p></td><td class="cl-c80bc3bb"><p class="cl-c80bb582"><span class="cl-c80a9ea4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80bc3b2"><p class="cl-c80bb578"><span class="cl-c80a9ea4">Gentoo</span></p></td><td class="cl-c80bc3b3"><p class="cl-c80bb578"><span class="cl-c80a9ea4">female</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">58</span></p></td><td class="cl-c80bc3ba"><p class="cl-c80bb582"><span class="cl-c80a9ea4">4,680</span></p></td><td class="cl-c80bc3bb"><p class="cl-c80bb582"><span class="cl-c80a9ea4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80bc3bc"><p class="cl-c80bb578"><span class="cl-c80a9ea4">Gentoo</span></p></td><td class="cl-c80bc3bd"><p class="cl-c80bb578"><span class="cl-c80a9ea4">male</span></p></td><td class="cl-c80bc3c4"><p class="cl-c80bb582"><span class="cl-c80a9ea4">61</span></p></td><td class="cl-c80bc3c4"><p class="cl-c80bb582"><span class="cl-c80a9ea4">5,485</span></p></td><td class="cl-c80bc3c5"><p class="cl-c80bb582"><span class="cl-c80a9ea4">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-c811799a{}.cl-c80ec0ec{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c80fd8ba{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c80fd8bb{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c80fe828{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe829{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe82a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe82b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe832{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe833{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe834{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c80fe835{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c811799a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c80fe828"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">species</span></p></th><th class="cl-c80fe828"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">sex</span></p></th><th class="cl-c80fe829"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">No. birds</span></p></th><th class="cl-c80fe829"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">Mean body mass (g)</span></p></th><th class="cl-c80fe829"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c80fe82a"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">Adelie</span></p></td><td class="cl-c80fe82a"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">female</span></p></td><td class="cl-c80fe82b"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">73</span></p></td><td class="cl-c80fe82b"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">3,369</span></p></td><td class="cl-c80fe82b"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">Adelie</span></p></td><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">male</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">73</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">4,043</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">Chinstrap</span></p></td><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">female</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">34</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">3,527</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">Chinstrap</span></p></td><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">male</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">34</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">3,939</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">Gentoo</span></p></td><td class="cl-c80fe832"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">female</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">58</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">4,680</span></p></td><td class="cl-c80fe833"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c80fe834"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">Gentoo</span></p></td><td class="cl-c80fe834"><p class="cl-c80fd8ba"><span class="cl-c80ec0ec">male</span></p></td><td class="cl-c80fe835"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">61</span></p></td><td class="cl-c80fe835"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">5,485</span></p></td><td class="cl-c80fe835"><p class="cl-c80fd8bb"><span class="cl-c80ec0ec">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-c81805f8{}.cl-c81578ba{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c81578c4{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c8167f6c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c8167f6d{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c8168c96{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168ca0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168ca1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168ca2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168ca3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168caa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168cab{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c8168cac{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c81805f8'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c8168c96"><p class="cl-c8167f6c"><span class="cl-c81578ba">species</span></p></th><th class="cl-c8168c96"><p class="cl-c8167f6c"><span class="cl-c81578ba">island</span></p></th><th class="cl-c8168ca0"><p class="cl-c8167f6d"><span class="cl-c81578ba">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c8168ca1"><p class="cl-c8167f6c"><span class="cl-c81578c4">Adelie</span></p></td><td class="cl-c8168ca1"><p class="cl-c8167f6c"><span class="cl-c81578c4">Torgersen</span></p></td><td class="cl-c8168ca2"><p class="cl-c8167f6d"><span class="cl-c81578c4">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c8168ca3"><p class="cl-c8167f6c"><span class="cl-c81578c4">Adelie</span></p></td><td class="cl-c8168ca3"><p class="cl-c8167f6c"><span class="cl-c81578c4">Dream</span></p></td><td class="cl-c8168caa"><p class="cl-c8167f6d"><span class="cl-c81578c4">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c8168cab"><p class="cl-c8167f6c"><span class="cl-c81578c4">Adelie</span></p></td><td class="cl-c8168cab"><p class="cl-c8167f6c"><span class="cl-c81578c4">Dream</span></p></td><td class="cl-c8168cac"><p class="cl-c8167f6d"><span class="cl-c81578c4">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-c81f2d7e{}.cl-c81c8e70{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c81c8e7a{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c81da594{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c81da59e{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c81db34a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db34b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db34c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db354{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db355{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db356{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db357{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c81db35e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c81f2d7e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c81db34a"><p class="cl-c81da594"><span class="cl-c81c8e70">species</span></p></th><th class="cl-c81db34a"><p class="cl-c81da594"><span class="cl-c81c8e70">sex</span></p></th><th class="cl-c81db34b"><p class="cl-c81da59e"><span class="cl-c81c8e70">No. birds</span></p></th><th class="cl-c81db34b"><p class="cl-c81da59e"><span class="cl-c81c8e70">Mean body mass (g)</span></p></th><th class="cl-c81db34b"><p class="cl-c81da59e"><span class="cl-c81c8e70">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c81db34c"><p class="cl-c81da594"><span class="cl-c81c8e7a">Adelie</span></p></td><td class="cl-c81db34c"><p class="cl-c81da594"><span class="cl-c81c8e7a">female</span></p></td><td class="cl-c81db354"><p class="cl-c81da59e"><span class="cl-c81c8e7a">73</span></p></td><td class="cl-c81db354"><p class="cl-c81da59e"><span class="cl-c81c8e7a">3,368.836</span></p></td><td class="cl-c81db354"><p class="cl-c81da59e"><span class="cl-c81c8e7a">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">Adelie</span></p></td><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">male</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">73</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">4,043.493</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">Chinstrap</span></p></td><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">female</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">34</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">3,527.206</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">Chinstrap</span></p></td><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">male</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">34</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">3,938.971</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">Gentoo</span></p></td><td class="cl-c81db355"><p class="cl-c81da594"><span class="cl-c81c8e7a">female</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">58</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">4,679.741</span></p></td><td class="cl-c81db356"><p class="cl-c81da59e"><span class="cl-c81c8e7a">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c81db357"><p class="cl-c81da594"><span class="cl-c81c8e7a">Gentoo</span></p></td><td class="cl-c81db357"><p class="cl-c81da594"><span class="cl-c81c8e7a">male</span></p></td><td class="cl-c81db35e"><p class="cl-c81da59e"><span class="cl-c81c8e7a">61</span></p></td><td class="cl-c81db35e"><p class="cl-c81da59e"><span class="cl-c81c8e7a">5,484.836</span></p></td><td class="cl-c81db35e"><p class="cl-c81da59e"><span class="cl-c81c8e7a">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-c824295a{}.cl-c820f4ba{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c820f4bb{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c82299c8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c82299c9{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c822a648{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a649{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a64a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a64b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a652{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a653{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a654{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c822a655{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c824295a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c822a648"><p class="cl-c82299c8"><span class="cl-c820f4ba">species</span></p></th><th class="cl-c822a648"><p class="cl-c82299c8"><span class="cl-c820f4ba">sex</span></p></th><th class="cl-c822a649"><p class="cl-c82299c9"><span class="cl-c820f4ba">No. birds</span></p></th><th class="cl-c822a649"><p class="cl-c82299c9"><span class="cl-c820f4ba">Mean body mass (g)</span></p></th><th class="cl-c822a649"><p class="cl-c82299c9"><span class="cl-c820f4ba">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-c822a64a"><p class="cl-c82299c8"><span class="cl-c820f4bb">Adelie</span></p></td><td class="cl-c822a64a"><p class="cl-c82299c8"><span class="cl-c820f4bb">female</span></p></td><td class="cl-c822a64b"><p class="cl-c82299c9"><span class="cl-c820f4bb">73</span></p></td><td class="cl-c822a64b"><p class="cl-c82299c9"><span class="cl-c820f4bb">3369</span></p></td><td class="cl-c822a64b"><p class="cl-c82299c9"><span class="cl-c820f4bb">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">Adelie</span></p></td><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">male</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">73</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">4043</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">Chinstrap</span></p></td><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">female</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">34</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">3527</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">Chinstrap</span></p></td><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">male</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">34</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">3939</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">Gentoo</span></p></td><td class="cl-c822a652"><p class="cl-c82299c8"><span class="cl-c820f4bb">female</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">58</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">4680</span></p></td><td class="cl-c822a653"><p class="cl-c82299c9"><span class="cl-c820f4bb">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c822a654"><p class="cl-c82299c8"><span class="cl-c820f4bb">Gentoo</span></p></td><td class="cl-c822a654"><p class="cl-c82299c8"><span class="cl-c820f4bb">male</span></p></td><td class="cl-c822a655"><p class="cl-c82299c9"><span class="cl-c820f4bb">61</span></p></td><td class="cl-c822a655"><p class="cl-c82299c9"><span class="cl-c820f4bb">5485</span></p></td><td class="cl-c822a655"><p class="cl-c82299c9"><span class="cl-c820f4bb">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-c8292126{}.cl-c8267c3c{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c8267c3d{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-c8279a9a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c8279aa4{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-c827a88c{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a88d{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a896{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a897{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a898{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a8a0{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a8a1{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-c827a8a2{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-c8292126'><thead><tr style="overflow-wrap:break-word;"><th class="cl-c827a88c"><p class="cl-c8279a9a"><span class="cl-c8267c3c">species</span></p></th><th class="cl-c827a88c"><p class="cl-c8279a9a"><span class="cl-c8267c3c">sex</span></p></th><th class="cl-c827a88d"><p class="cl-c8279aa4"><span class="cl-c8267c3c">No. birds</span></p></th><th class="cl-c827a88d"><p class="cl-c8279aa4"><span class="cl-c8267c3c">Mean body mass (g)</span></p></th><th class="cl-c827a88d"><p class="cl-c8279aa4"><span class="cl-c8267c3c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-c827a896"><p class="cl-c8279a9a"><span class="cl-c8267c3d">Adelie</span></p></td><td class="cl-c827a896"><p class="cl-c8279a9a"><span class="cl-c8267c3d">female</span></p></td><td class="cl-c827a897"><p class="cl-c8279aa4"><span class="cl-c8267c3d">73</span></p></td><td class="cl-c827a897"><p class="cl-c8279aa4"><span class="cl-c8267c3d">3,368.836</span></p></td><td class="cl-c827a897"><p class="cl-c8279aa4"><span class="cl-c8267c3d">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c827a898"><p class="cl-c8279a9a"><span class="cl-c8267c3d">male</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">73</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">4,043.493</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-c827a898"><p class="cl-c8279a9a"><span class="cl-c8267c3d">Chinstrap</span></p></td><td class="cl-c827a898"><p class="cl-c8279a9a"><span class="cl-c8267c3d">female</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">34</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">3,527.206</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c827a898"><p class="cl-c8279a9a"><span class="cl-c8267c3d">male</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">34</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">3,938.971</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-c827a8a1"><p class="cl-c8279a9a"><span class="cl-c8267c3d">Gentoo</span></p></td><td class="cl-c827a898"><p class="cl-c8279a9a"><span class="cl-c8267c3d">female</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">58</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">4,679.741</span></p></td><td class="cl-c827a8a0"><p class="cl-c8279aa4"><span class="cl-c8267c3d">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-c827a8a1"><p class="cl-c8279a9a"><span class="cl-c8267c3d">male</span></p></td><td class="cl-c827a8a2"><p class="cl-c8279aa4"><span class="cl-c8267c3d">61</span></p></td><td class="cl-c827a8a2"><p class="cl-c8279aa4"><span class="cl-c8267c3d">5,484.836</span></p></td><td class="cl-c827a8a2"><p class="cl-c8279aa4"><span class="cl-c8267c3d">313.1586</span></p></td></tr></tbody></table></div>
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
