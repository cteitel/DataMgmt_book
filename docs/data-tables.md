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
<div class="tabwid"><style>.cl-0fdb2f42{}.cl-0fd5ebfe{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0fd5ec08{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0fd7582c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0fd7582d{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0fd767d6{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fd767e0{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fd767e1{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fd767ea{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fd767eb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fd767ec{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0fdb2f42'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0fd767d6"><p class="cl-0fd7582c"><span class="cl-0fd5ebfe">species</span></p></th><th class="cl-0fd767d6"><p class="cl-0fd7582c"><span class="cl-0fd5ebfe">sex</span></p></th><th class="cl-0fd767e0"><p class="cl-0fd7582d"><span class="cl-0fd5ebfe">No. birds</span></p></th><th class="cl-0fd767e0"><p class="cl-0fd7582d"><span class="cl-0fd5ebfe">Mean body mass (g)</span></p></th><th class="cl-0fd767e0"><p class="cl-0fd7582d"><span class="cl-0fd5ebfe">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-0fd767e1"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">Adelie</span></p></td><td class="cl-0fd767e1"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">female</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">73</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">3,369</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fd767eb"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">Adelie</span></p></td><td class="cl-0fd767eb"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">male</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">73</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">4,043</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fd767e1"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">Chinstrap</span></p></td><td class="cl-0fd767e1"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">female</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">34</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">3,527</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fd767eb"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">Chinstrap</span></p></td><td class="cl-0fd767eb"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">male</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">34</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">3,939</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fd767e1"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">Gentoo</span></p></td><td class="cl-0fd767e1"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">female</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">58</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">4,680</span></p></td><td class="cl-0fd767ea"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fd767eb"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">Gentoo</span></p></td><td class="cl-0fd767eb"><p class="cl-0fd7582c"><span class="cl-0fd5ec08">male</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">61</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">5,485</span></p></td><td class="cl-0fd767ec"><p class="cl-0fd7582d"><span class="cl-0fd5ec08">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-0fee7f52{}.cl-0feb6c04{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0feb6c0e{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0feca5f6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0feca5f7{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0fecb60e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fecb60f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fecb618{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fecb619{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fecb61a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0fecb61b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0fee7f52'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0fecb60e"><p class="cl-0feca5f6"><span class="cl-0feb6c04">species</span></p></th><th class="cl-0fecb60e"><p class="cl-0feca5f6"><span class="cl-0feb6c04">sex</span></p></th><th class="cl-0fecb60f"><p class="cl-0feca5f7"><span class="cl-0feb6c04">No. birds</span></p></th><th class="cl-0fecb60f"><p class="cl-0feca5f7"><span class="cl-0feb6c04">Mean body mass (g)</span></p></th><th class="cl-0fecb60f"><p class="cl-0feca5f7"><span class="cl-0feb6c04">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">Adelie</span></p></td><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">female</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">73</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">3,369</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">Adelie</span></p></td><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">male</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">73</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">4,043</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">Chinstrap</span></p></td><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">female</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">34</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">3,527</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">Chinstrap</span></p></td><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">male</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">34</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">3,939</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">Gentoo</span></p></td><td class="cl-0fecb618"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">female</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">58</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">4,680</span></p></td><td class="cl-0fecb619"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0fecb61a"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">Gentoo</span></p></td><td class="cl-0fecb61a"><p class="cl-0feca5f6"><span class="cl-0feb6c0e">male</span></p></td><td class="cl-0fecb61b"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">61</span></p></td><td class="cl-0fecb61b"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">5,485</span></p></td><td class="cl-0fecb61b"><p class="cl-0feca5f7"><span class="cl-0feb6c0e">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-0ff3442e{}.cl-0ff03018{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0ff176d0{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-0ff1879c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff187a6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff187a7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0ff3442e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0ff1879c"><p class="cl-0ff176d0"><span class="cl-0ff03018">species</span></p></th><th class="cl-0ff1879c"><p class="cl-0ff176d0"><span class="cl-0ff03018">sex</span></p></th><th class="cl-0ff1879c"><p class="cl-0ff176d0"><span class="cl-0ff03018">No. birds</span></p></th><th class="cl-0ff1879c"><p class="cl-0ff176d0"><span class="cl-0ff03018">Mean body mass (g)</span></p></th><th class="cl-0ff1879c"><p class="cl-0ff176d0"><span class="cl-0ff03018">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">Adelie</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">female</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">73</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">3,369</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">Adelie</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">male</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">73</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">4,043</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">Chinstrap</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">female</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">34</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">3,527</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">Chinstrap</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">male</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">34</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">3,939</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">Gentoo</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">female</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">58</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">4,680</span></p></td><td class="cl-0ff187a6"><p class="cl-0ff176d0"><span class="cl-0ff03018">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff187a7"><p class="cl-0ff176d0"><span class="cl-0ff03018">Gentoo</span></p></td><td class="cl-0ff187a7"><p class="cl-0ff176d0"><span class="cl-0ff03018">male</span></p></td><td class="cl-0ff187a7"><p class="cl-0ff176d0"><span class="cl-0ff03018">61</span></p></td><td class="cl-0ff187a7"><p class="cl-0ff176d0"><span class="cl-0ff03018">5,485</span></p></td><td class="cl-0ff187a7"><p class="cl-0ff176d0"><span class="cl-0ff03018">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-0ff81b34{}.cl-0ff4ea86{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0ff634c2{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0ff634c3{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0ff6469c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff6469d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff6469e{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff646a6{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff646a7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff646a8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff646a9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ff646b0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0ff81b34'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0ff6469c"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">species</span></p></th><th class="cl-0ff6469c"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">sex</span></p></th><th class="cl-0ff6469d"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">No. birds</span></p></th><th class="cl-0ff6469d"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">Mean body mass (g)</span></p></th><th class="cl-0ff6469d"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-0ff6469e"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">Adelie</span></p></td><td class="cl-0ff6469e"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">female</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">73</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">3,369</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff646a7"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">Adelie</span></p></td><td class="cl-0ff646a7"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">male</span></p></td><td class="cl-0ff646a8"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">73</span></p></td><td class="cl-0ff646a8"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">4,043</span></p></td><td class="cl-0ff646a8"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff6469e"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">Chinstrap</span></p></td><td class="cl-0ff6469e"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">female</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">34</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">3,527</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff646a7"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">Chinstrap</span></p></td><td class="cl-0ff646a7"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">male</span></p></td><td class="cl-0ff646a8"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">34</span></p></td><td class="cl-0ff646a8"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">3,939</span></p></td><td class="cl-0ff646a8"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff6469e"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">Gentoo</span></p></td><td class="cl-0ff6469e"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">female</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">58</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">4,680</span></p></td><td class="cl-0ff646a6"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ff646a9"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">Gentoo</span></p></td><td class="cl-0ff646a9"><p class="cl-0ff634c2"><span class="cl-0ff4ea86">male</span></p></td><td class="cl-0ff646b0"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">61</span></p></td><td class="cl-0ff646b0"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">5,485</span></p></td><td class="cl-0ff646b0"><p class="cl-0ff634c3"><span class="cl-0ff4ea86">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-0ffdda88{}.cl-0ffaf8e0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0ffc21c0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0ffc21c1{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0ffc30a2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30ac{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30ad{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30b6{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30b7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30b8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30b9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0ffc30c0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0ffdda88'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0ffc30a2"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">species</span></p></th><th class="cl-0ffc30a2"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">sex</span></p></th><th class="cl-0ffc30ac"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">No. birds</span></p></th><th class="cl-0ffc30ac"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">Mean body mass (g)</span></p></th><th class="cl-0ffc30ac"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-0ffc30ad"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">Adelie</span></p></td><td class="cl-0ffc30ad"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">female</span></p></td><td class="cl-0ffc30b6"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">73</span></p></td><td class="cl-0ffc30b6"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">3,369</span></p></td><td class="cl-0ffc30b6"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ffc30ad"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">Adelie</span></p></td><td class="cl-0ffc30ad"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">male</span></p></td><td class="cl-0ffc30b6"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">73</span></p></td><td class="cl-0ffc30b6"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">4,043</span></p></td><td class="cl-0ffc30b6"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ffc30b7"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">Chinstrap</span></p></td><td class="cl-0ffc30b7"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">female</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">34</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">3,527</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ffc30b7"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">Chinstrap</span></p></td><td class="cl-0ffc30b7"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">male</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">34</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">3,939</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ffc30b7"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">Gentoo</span></p></td><td class="cl-0ffc30b7"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">female</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">58</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">4,680</span></p></td><td class="cl-0ffc30b8"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0ffc30b9"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">Gentoo</span></p></td><td class="cl-0ffc30b9"><p class="cl-0ffc21c0"><span class="cl-0ffaf8e0">male</span></p></td><td class="cl-0ffc30c0"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">61</span></p></td><td class="cl-0ffc30c0"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">5,485</span></p></td><td class="cl-0ffc30c0"><p class="cl-0ffc21c1"><span class="cl-0ffaf8e0">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-1002c91c{}.cl-0ffff732{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0ffff73c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0ffff73d{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-100128e6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-100128f0{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-10013732{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10013733{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1001373c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1001373d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1001373e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10013746{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-1002c91c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-10013732"><p class="cl-100128e6"><span class="cl-0ffff732">species</span></p></th><th class="cl-10013732"><p class="cl-100128e6"><span class="cl-0ffff732">sex</span></p></th><th class="cl-10013733"><p class="cl-100128f0"><span class="cl-0ffff732">No. birds</span></p></th><th class="cl-10013733"><p class="cl-100128f0"><span class="cl-0ffff732">Mean body mass (g)</span></p></th><th class="cl-10013733"><p class="cl-100128f0"><span class="cl-0ffff732">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73c">Adelie</span></p></td><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73d">female</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">73</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">3,369</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73c">Adelie</span></p></td><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73d">male</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">73</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">4,043</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73c">Chinstrap</span></p></td><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73d">female</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">34</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">3,527</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73c">Chinstrap</span></p></td><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73d">male</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">34</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">3,939</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73c">Gentoo</span></p></td><td class="cl-1001373c"><p class="cl-100128e6"><span class="cl-0ffff73d">female</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">58</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">4,680</span></p></td><td class="cl-1001373d"><p class="cl-100128f0"><span class="cl-0ffff73d">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1001373e"><p class="cl-100128e6"><span class="cl-0ffff73c">Gentoo</span></p></td><td class="cl-1001373e"><p class="cl-100128e6"><span class="cl-0ffff73d">male</span></p></td><td class="cl-10013746"><p class="cl-100128f0"><span class="cl-0ffff73d">61</span></p></td><td class="cl-10013746"><p class="cl-100128f0"><span class="cl-0ffff73d">5,485</span></p></td><td class="cl-10013746"><p class="cl-100128f0"><span class="cl-0ffff73d">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-1008d2da{}.cl-1004b83a{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-1005dcec{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1005dcf6{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1005ecf0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1005ecfa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-1008d2da'><thead><tr style="overflow-wrap:break-word;"><th class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">species</span></p></th><th class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">sex</span></p></th><th class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">No. birds</span></p></th><th class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">Mean body mass (g)</span></p></th><th class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">Adelie</span></p></td><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">female</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">73</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">3,369</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">Adelie</span></p></td><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">male</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">73</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">4,043</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">Chinstrap</span></p></td><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">female</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">34</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">3,527</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">Chinstrap</span></p></td><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">male</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">34</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">3,939</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">Gentoo</span></p></td><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">female</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">58</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">4,680</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">Gentoo</span></p></td><td class="cl-1005ecf0"><p class="cl-1005dcec"><span class="cl-1004b83a">male</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">61</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">5,485</span></p></td><td class="cl-1005ecfa"><p class="cl-1005dcf6"><span class="cl-1004b83a">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-100e04a8{}.cl-100b4fe2{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-100c660c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-100c6616{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-100c7426{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7427{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7428{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7429{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7430{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7431{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7432{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7433{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7434{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c7435{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c743a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-100c743b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-100e04a8'><thead><tr style="overflow-wrap:break-word;"><th class="cl-100c7426"><p class="cl-100c660c"><span class="cl-100b4fe2">species</span></p></th><th class="cl-100c7427"><p class="cl-100c660c"><span class="cl-100b4fe2">sex</span></p></th><th class="cl-100c7428"><p class="cl-100c6616"><span class="cl-100b4fe2">No. birds</span></p></th><th class="cl-100c7428"><p class="cl-100c6616"><span class="cl-100b4fe2">Mean body mass (g)</span></p></th><th class="cl-100c7429"><p class="cl-100c6616"><span class="cl-100b4fe2">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-100c7430"><p class="cl-100c660c"><span class="cl-100b4fe2">Adelie</span></p></td><td class="cl-100c7431"><p class="cl-100c660c"><span class="cl-100b4fe2">female</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">73</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">3,369</span></p></td><td class="cl-100c7433"><p class="cl-100c6616"><span class="cl-100b4fe2">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-100c7430"><p class="cl-100c660c"><span class="cl-100b4fe2">Adelie</span></p></td><td class="cl-100c7431"><p class="cl-100c660c"><span class="cl-100b4fe2">male</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">73</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">4,043</span></p></td><td class="cl-100c7433"><p class="cl-100c6616"><span class="cl-100b4fe2">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-100c7430"><p class="cl-100c660c"><span class="cl-100b4fe2">Chinstrap</span></p></td><td class="cl-100c7431"><p class="cl-100c660c"><span class="cl-100b4fe2">female</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">34</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">3,527</span></p></td><td class="cl-100c7433"><p class="cl-100c6616"><span class="cl-100b4fe2">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-100c7430"><p class="cl-100c660c"><span class="cl-100b4fe2">Chinstrap</span></p></td><td class="cl-100c7431"><p class="cl-100c660c"><span class="cl-100b4fe2">male</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">34</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">3,939</span></p></td><td class="cl-100c7433"><p class="cl-100c6616"><span class="cl-100b4fe2">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-100c7430"><p class="cl-100c660c"><span class="cl-100b4fe2">Gentoo</span></p></td><td class="cl-100c7431"><p class="cl-100c660c"><span class="cl-100b4fe2">female</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">58</span></p></td><td class="cl-100c7432"><p class="cl-100c6616"><span class="cl-100b4fe2">4,680</span></p></td><td class="cl-100c7433"><p class="cl-100c6616"><span class="cl-100b4fe2">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-100c7434"><p class="cl-100c660c"><span class="cl-100b4fe2">Gentoo</span></p></td><td class="cl-100c7435"><p class="cl-100c660c"><span class="cl-100b4fe2">male</span></p></td><td class="cl-100c743a"><p class="cl-100c6616"><span class="cl-100b4fe2">61</span></p></td><td class="cl-100c743a"><p class="cl-100c6616"><span class="cl-100b4fe2">5,485</span></p></td><td class="cl-100c743b"><p class="cl-100c6616"><span class="cl-100b4fe2">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-10122b14{}.cl-100f789c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-10109218{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-10109219{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1010a06e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a06f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a070{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a071{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a078{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a079{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a07a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a07b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a07c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1010a082{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-10122b14'><thead><tr style="overflow-wrap:break-word;"><th class="cl-1010a06e"><p class="cl-10109218"><span class="cl-100f789c">species</span></p></th><th class="cl-1010a06e"><p class="cl-10109218"><span class="cl-100f789c">sex</span></p></th><th class="cl-1010a06f"><p class="cl-10109219"><span class="cl-100f789c">No. birds</span></p></th><th class="cl-1010a06f"><p class="cl-10109219"><span class="cl-100f789c">Mean body mass (g)</span></p></th><th class="cl-1010a06f"><p class="cl-10109219"><span class="cl-100f789c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-1010a070"><p class="cl-10109218"><span class="cl-100f789c">Adelie</span></p></td><td class="cl-1010a071"><p class="cl-10109218"><span class="cl-100f789c">female</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">73</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">3,369</span></p></td><td class="cl-1010a079"><p class="cl-10109219"><span class="cl-100f789c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1010a070"><p class="cl-10109218"><span class="cl-100f789c">Adelie</span></p></td><td class="cl-1010a071"><p class="cl-10109218"><span class="cl-100f789c">male</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">73</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">4,043</span></p></td><td class="cl-1010a079"><p class="cl-10109219"><span class="cl-100f789c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1010a070"><p class="cl-10109218"><span class="cl-100f789c">Chinstrap</span></p></td><td class="cl-1010a071"><p class="cl-10109218"><span class="cl-100f789c">female</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">34</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">3,527</span></p></td><td class="cl-1010a079"><p class="cl-10109219"><span class="cl-100f789c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1010a070"><p class="cl-10109218"><span class="cl-100f789c">Chinstrap</span></p></td><td class="cl-1010a071"><p class="cl-10109218"><span class="cl-100f789c">male</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">34</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">3,939</span></p></td><td class="cl-1010a079"><p class="cl-10109219"><span class="cl-100f789c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1010a070"><p class="cl-10109218"><span class="cl-100f789c">Gentoo</span></p></td><td class="cl-1010a071"><p class="cl-10109218"><span class="cl-100f789c">female</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">58</span></p></td><td class="cl-1010a078"><p class="cl-10109219"><span class="cl-100f789c">4,680</span></p></td><td class="cl-1010a079"><p class="cl-10109219"><span class="cl-100f789c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1010a07a"><p class="cl-10109218"><span class="cl-100f789c">Gentoo</span></p></td><td class="cl-1010a07b"><p class="cl-10109218"><span class="cl-100f789c">male</span></p></td><td class="cl-1010a07c"><p class="cl-10109219"><span class="cl-100f789c">61</span></p></td><td class="cl-1010a07c"><p class="cl-10109219"><span class="cl-100f789c">5,485</span></p></td><td class="cl-1010a082"><p class="cl-10109219"><span class="cl-100f789c">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-10162caa{}.cl-101380b8{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-10148ca6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-10148cb0{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-10149a3e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a3f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a48{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a49{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a4a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a52{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a53{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-10149a54{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-10162caa'><thead><tr style="overflow-wrap:break-word;"><th class="cl-10149a3e"><p class="cl-10148ca6"><span class="cl-101380b8">species</span></p></th><th class="cl-10149a3e"><p class="cl-10148ca6"><span class="cl-101380b8">sex</span></p></th><th class="cl-10149a3f"><p class="cl-10148cb0"><span class="cl-101380b8">No. birds</span></p></th><th class="cl-10149a3f"><p class="cl-10148cb0"><span class="cl-101380b8">Mean body mass (g)</span></p></th><th class="cl-10149a3f"><p class="cl-10148cb0"><span class="cl-101380b8">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-10149a48"><p class="cl-10148ca6"><span class="cl-101380b8">Adelie</span></p></td><td class="cl-10149a48"><p class="cl-10148ca6"><span class="cl-101380b8">female</span></p></td><td class="cl-10149a49"><p class="cl-10148cb0"><span class="cl-101380b8">73</span></p></td><td class="cl-10149a49"><p class="cl-10148cb0"><span class="cl-101380b8">3,369</span></p></td><td class="cl-10149a49"><p class="cl-10148cb0"><span class="cl-101380b8">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">Adelie</span></p></td><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">male</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">73</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">4,043</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">Chinstrap</span></p></td><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">female</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">34</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">3,527</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">Chinstrap</span></p></td><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">male</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">34</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">3,939</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">Gentoo</span></p></td><td class="cl-10149a4a"><p class="cl-10148ca6"><span class="cl-101380b8">female</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">58</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">4,680</span></p></td><td class="cl-10149a52"><p class="cl-10148cb0"><span class="cl-101380b8">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-10149a53"><p class="cl-10148ca6"><span class="cl-101380b8">Gentoo</span></p></td><td class="cl-10149a53"><p class="cl-10148ca6"><span class="cl-101380b8">male</span></p></td><td class="cl-10149a54"><p class="cl-10148cb0"><span class="cl-101380b8">61</span></p></td><td class="cl-10149a54"><p class="cl-10148cb0"><span class="cl-101380b8">5,485</span></p></td><td class="cl-10149a54"><p class="cl-10148cb0"><span class="cl-101380b8">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-101cdd7a{}.cl-101a52d0{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-101a52d1{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-101b5c98{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-101b5ca2{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-101b6954{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b6955{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b695e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b695f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b6960{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b6968{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b6969{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-101b696a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-101cdd7a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-101b6954"><p class="cl-101b5c98"><span class="cl-101a52d0">species</span></p></th><th class="cl-101b6954"><p class="cl-101b5c98"><span class="cl-101a52d0">island</span></p></th><th class="cl-101b6955"><p class="cl-101b5ca2"><span class="cl-101a52d0">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-101b695e"><p class="cl-101b5c98"><span class="cl-101a52d1">Adelie</span></p></td><td class="cl-101b695e"><p class="cl-101b5c98"><span class="cl-101a52d1">Torgersen</span></p></td><td class="cl-101b695f"><p class="cl-101b5ca2"><span class="cl-101a52d1">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-101b6960"><p class="cl-101b5c98"><span class="cl-101a52d1">Adelie</span></p></td><td class="cl-101b6960"><p class="cl-101b5c98"><span class="cl-101a52d1">Dream</span></p></td><td class="cl-101b6968"><p class="cl-101b5ca2"><span class="cl-101a52d1">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-101b6969"><p class="cl-101b5c98"><span class="cl-101a52d1">Adelie</span></p></td><td class="cl-101b6969"><p class="cl-101b5c98"><span class="cl-101a52d1">Dream</span></p></td><td class="cl-101b696a"><p class="cl-101b5ca2"><span class="cl-101a52d1">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-10245884{}.cl-10218dd4{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-10218dde{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-1022aeda{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1022aedb{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1022bf24{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf2e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf2f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf30{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf31{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf38{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf39{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1022bf3a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-10245884'><thead><tr style="overflow-wrap:break-word;"><th class="cl-1022bf24"><p class="cl-1022aeda"><span class="cl-10218dd4">species</span></p></th><th class="cl-1022bf24"><p class="cl-1022aeda"><span class="cl-10218dd4">sex</span></p></th><th class="cl-1022bf2e"><p class="cl-1022aedb"><span class="cl-10218dd4">No. birds</span></p></th><th class="cl-1022bf2e"><p class="cl-1022aedb"><span class="cl-10218dd4">Mean body mass (g)</span></p></th><th class="cl-1022bf2e"><p class="cl-1022aedb"><span class="cl-10218dd4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-1022bf2f"><p class="cl-1022aeda"><span class="cl-10218dde">Adelie</span></p></td><td class="cl-1022bf2f"><p class="cl-1022aeda"><span class="cl-10218dde">female</span></p></td><td class="cl-1022bf30"><p class="cl-1022aedb"><span class="cl-10218dde">73</span></p></td><td class="cl-1022bf30"><p class="cl-1022aedb"><span class="cl-10218dde">3,368.836</span></p></td><td class="cl-1022bf30"><p class="cl-1022aedb"><span class="cl-10218dde">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">Adelie</span></p></td><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">male</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">73</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">4,043.493</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">Chinstrap</span></p></td><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">female</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">34</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">3,527.206</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">Chinstrap</span></p></td><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">male</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">34</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">3,938.971</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">Gentoo</span></p></td><td class="cl-1022bf31"><p class="cl-1022aeda"><span class="cl-10218dde">female</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">58</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">4,679.741</span></p></td><td class="cl-1022bf38"><p class="cl-1022aedb"><span class="cl-10218dde">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1022bf39"><p class="cl-1022aeda"><span class="cl-10218dde">Gentoo</span></p></td><td class="cl-1022bf39"><p class="cl-1022aeda"><span class="cl-10218dde">male</span></p></td><td class="cl-1022bf3a"><p class="cl-1022aedb"><span class="cl-10218dde">61</span></p></td><td class="cl-1022bf3a"><p class="cl-1022aedb"><span class="cl-10218dde">5,484.836</span></p></td><td class="cl-1022bf3a"><p class="cl-1022aedb"><span class="cl-10218dde">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-10295bd6{}.cl-10261df4{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-10261df5{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-1027c92e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1027c938{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-1027d694{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d69e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d69f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d6a0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d6a1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d6a8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d6a9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-1027d6aa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-10295bd6'><thead><tr style="overflow-wrap:break-word;"><th class="cl-1027d694"><p class="cl-1027c92e"><span class="cl-10261df4">species</span></p></th><th class="cl-1027d694"><p class="cl-1027c92e"><span class="cl-10261df4">sex</span></p></th><th class="cl-1027d69e"><p class="cl-1027c938"><span class="cl-10261df4">No. birds</span></p></th><th class="cl-1027d69e"><p class="cl-1027c938"><span class="cl-10261df4">Mean body mass (g)</span></p></th><th class="cl-1027d69e"><p class="cl-1027c938"><span class="cl-10261df4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-1027d69f"><p class="cl-1027c92e"><span class="cl-10261df5">Adelie</span></p></td><td class="cl-1027d69f"><p class="cl-1027c92e"><span class="cl-10261df5">female</span></p></td><td class="cl-1027d6a0"><p class="cl-1027c938"><span class="cl-10261df5">73</span></p></td><td class="cl-1027d6a0"><p class="cl-1027c938"><span class="cl-10261df5">3369</span></p></td><td class="cl-1027d6a0"><p class="cl-1027c938"><span class="cl-10261df5">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">Adelie</span></p></td><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">male</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">73</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">4043</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">Chinstrap</span></p></td><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">female</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">34</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">3527</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">Chinstrap</span></p></td><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">male</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">34</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">3939</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">Gentoo</span></p></td><td class="cl-1027d6a1"><p class="cl-1027c92e"><span class="cl-10261df5">female</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">58</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">4680</span></p></td><td class="cl-1027d6a8"><p class="cl-1027c938"><span class="cl-10261df5">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-1027d6a9"><p class="cl-1027c92e"><span class="cl-10261df5">Gentoo</span></p></td><td class="cl-1027d6a9"><p class="cl-1027c92e"><span class="cl-10261df5">male</span></p></td><td class="cl-1027d6aa"><p class="cl-1027c938"><span class="cl-10261df5">61</span></p></td><td class="cl-1027d6aa"><p class="cl-1027c938"><span class="cl-10261df5">5485</span></p></td><td class="cl-1027d6aa"><p class="cl-1027c938"><span class="cl-10261df5">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-102e3b4c{}.cl-102ba44a{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-102ba454{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-102cb1d2{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-102cb1d3{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-102cbf56{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf60{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf61{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf62{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf63{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf6a{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf6b{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-102cbf6c{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-102e3b4c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-102cbf56"><p class="cl-102cb1d2"><span class="cl-102ba44a">species</span></p></th><th class="cl-102cbf56"><p class="cl-102cb1d2"><span class="cl-102ba44a">sex</span></p></th><th class="cl-102cbf60"><p class="cl-102cb1d3"><span class="cl-102ba44a">No. birds</span></p></th><th class="cl-102cbf60"><p class="cl-102cb1d3"><span class="cl-102ba44a">Mean body mass (g)</span></p></th><th class="cl-102cbf60"><p class="cl-102cb1d3"><span class="cl-102ba44a">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-102cbf61"><p class="cl-102cb1d2"><span class="cl-102ba454">Adelie</span></p></td><td class="cl-102cbf61"><p class="cl-102cb1d2"><span class="cl-102ba454">female</span></p></td><td class="cl-102cbf62"><p class="cl-102cb1d3"><span class="cl-102ba454">73</span></p></td><td class="cl-102cbf62"><p class="cl-102cb1d3"><span class="cl-102ba454">3,368.836</span></p></td><td class="cl-102cbf62"><p class="cl-102cb1d3"><span class="cl-102ba454">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-102cbf63"><p class="cl-102cb1d2"><span class="cl-102ba454">male</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">73</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">4,043.493</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-102cbf63"><p class="cl-102cb1d2"><span class="cl-102ba454">Chinstrap</span></p></td><td class="cl-102cbf63"><p class="cl-102cb1d2"><span class="cl-102ba454">female</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">34</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">3,527.206</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-102cbf63"><p class="cl-102cb1d2"><span class="cl-102ba454">male</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">34</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">3,938.971</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-102cbf6b"><p class="cl-102cb1d2"><span class="cl-102ba454">Gentoo</span></p></td><td class="cl-102cbf63"><p class="cl-102cb1d2"><span class="cl-102ba454">female</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">58</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">4,679.741</span></p></td><td class="cl-102cbf6a"><p class="cl-102cb1d3"><span class="cl-102ba454">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-102cbf6b"><p class="cl-102cb1d2"><span class="cl-102ba454">male</span></p></td><td class="cl-102cbf6c"><p class="cl-102cb1d3"><span class="cl-102ba454">61</span></p></td><td class="cl-102cbf6c"><p class="cl-102cb1d3"><span class="cl-102ba454">5,484.836</span></p></td><td class="cl-102cbf6c"><p class="cl-102cb1d3"><span class="cl-102ba454">313.1586</span></p></td></tr></tbody></table></div>
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
