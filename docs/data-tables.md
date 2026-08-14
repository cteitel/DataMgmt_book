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
## ✔ dplyr     1.2.1     ✔ readr     2.2.0
## ✔ forcats   1.0.1     ✔ stringr   1.6.0
## ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
## ✔ purrr     1.2.2     
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
## `summarise()` has regrouped the output.
## ℹ Summaries were computed grouped by species and sex.
## ℹ Output is grouped by species.
## ℹ Use `summarise(.groups = "drop_last")` to silence this message.
## ℹ Use `summarise(.by = c(species, sex))` for per-operation grouping
##   (`?dplyr::dplyr_by`) instead.
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
<div class="tabwid"><style>.cl-03c2d390{}.cl-03be8542{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03be854c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03c0adf4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03c0adf5{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03c0c12c{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03c0c12d{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03c0c140{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03c0c141{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03c0c14a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03c0c14b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03c2d390'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03c0c12c"><p class="cl-03c0adf4"><span class="cl-03be8542">species</span></p></th><th class="cl-03c0c12c"><p class="cl-03c0adf4"><span class="cl-03be8542">sex</span></p></th><th class="cl-03c0c12d"><p class="cl-03c0adf5"><span class="cl-03be8542">No. birds</span></p></th><th class="cl-03c0c12d"><p class="cl-03c0adf5"><span class="cl-03be8542">Mean body mass (g)</span></p></th><th class="cl-03c0c12d"><p class="cl-03c0adf5"><span class="cl-03be8542">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03c0c140"><p class="cl-03c0adf4"><span class="cl-03be854c">Adelie</span></p></td><td class="cl-03c0c140"><p class="cl-03c0adf4"><span class="cl-03be854c">female</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">73</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">3,369</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03c0c14a"><p class="cl-03c0adf4"><span class="cl-03be854c">Adelie</span></p></td><td class="cl-03c0c14a"><p class="cl-03c0adf4"><span class="cl-03be854c">male</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">73</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">4,043</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03c0c140"><p class="cl-03c0adf4"><span class="cl-03be854c">Chinstrap</span></p></td><td class="cl-03c0c140"><p class="cl-03c0adf4"><span class="cl-03be854c">female</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">34</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">3,527</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03c0c14a"><p class="cl-03c0adf4"><span class="cl-03be854c">Chinstrap</span></p></td><td class="cl-03c0c14a"><p class="cl-03c0adf4"><span class="cl-03be854c">male</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">34</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">3,939</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03c0c140"><p class="cl-03c0adf4"><span class="cl-03be854c">Gentoo</span></p></td><td class="cl-03c0c140"><p class="cl-03c0adf4"><span class="cl-03be854c">female</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">58</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">4,680</span></p></td><td class="cl-03c0c141"><p class="cl-03c0adf5"><span class="cl-03be854c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03c0c14a"><p class="cl-03c0adf4"><span class="cl-03be854c">Gentoo</span></p></td><td class="cl-03c0c14a"><p class="cl-03c0adf4"><span class="cl-03be854c">male</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">61</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">5,485</span></p></td><td class="cl-03c0c14b"><p class="cl-03c0adf5"><span class="cl-03be854c">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-03dfff92{}.cl-03dc6a58{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03dc6a62{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03ddf4e0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03ddf4ea{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03de0552{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03de0553{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03de055c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03de055d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03de055e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03de0566{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03dfff92'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03de0552"><p class="cl-03ddf4e0"><span class="cl-03dc6a58">species</span></p></th><th class="cl-03de0552"><p class="cl-03ddf4e0"><span class="cl-03dc6a58">sex</span></p></th><th class="cl-03de0553"><p class="cl-03ddf4ea"><span class="cl-03dc6a58">No. birds</span></p></th><th class="cl-03de0553"><p class="cl-03ddf4ea"><span class="cl-03dc6a58">Mean body mass (g)</span></p></th><th class="cl-03de0553"><p class="cl-03ddf4ea"><span class="cl-03dc6a58">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">Adelie</span></p></td><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">female</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">73</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">3,369</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">Adelie</span></p></td><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">male</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">73</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">4,043</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">Chinstrap</span></p></td><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">female</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">34</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">3,527</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">Chinstrap</span></p></td><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">male</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">34</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">3,939</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">Gentoo</span></p></td><td class="cl-03de055c"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">female</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">58</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">4,680</span></p></td><td class="cl-03de055d"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03de055e"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">Gentoo</span></p></td><td class="cl-03de055e"><p class="cl-03ddf4e0"><span class="cl-03dc6a62">male</span></p></td><td class="cl-03de0566"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">61</span></p></td><td class="cl-03de0566"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">5,485</span></p></td><td class="cl-03de0566"><p class="cl-03ddf4ea"><span class="cl-03dc6a62">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-03e52fa8{}.cl-03e1d862{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03e34cc4{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-03e3610a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e3610b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e36114{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03e52fa8'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03e3610a"><p class="cl-03e34cc4"><span class="cl-03e1d862">species</span></p></th><th class="cl-03e3610a"><p class="cl-03e34cc4"><span class="cl-03e1d862">sex</span></p></th><th class="cl-03e3610a"><p class="cl-03e34cc4"><span class="cl-03e1d862">No. birds</span></p></th><th class="cl-03e3610a"><p class="cl-03e34cc4"><span class="cl-03e1d862">Mean body mass (g)</span></p></th><th class="cl-03e3610a"><p class="cl-03e34cc4"><span class="cl-03e1d862">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">Adelie</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">female</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">73</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">3,369</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">Adelie</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">male</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">73</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">4,043</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">Chinstrap</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">female</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">34</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">3,527</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">Chinstrap</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">male</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">34</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">3,939</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">Gentoo</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">female</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">58</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">4,680</span></p></td><td class="cl-03e3610b"><p class="cl-03e34cc4"><span class="cl-03e1d862">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e36114"><p class="cl-03e34cc4"><span class="cl-03e1d862">Gentoo</span></p></td><td class="cl-03e36114"><p class="cl-03e34cc4"><span class="cl-03e1d862">male</span></p></td><td class="cl-03e36114"><p class="cl-03e34cc4"><span class="cl-03e1d862">61</span></p></td><td class="cl-03e36114"><p class="cl-03e34cc4"><span class="cl-03e1d862">5,485</span></p></td><td class="cl-03e36114"><p class="cl-03e34cc4"><span class="cl-03e1d862">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-03ea8804{}.cl-03e6e9c4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03e86dda{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03e86de4{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03e8863a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e8863b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e88644{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e88645{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e88646{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e88647{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e8864e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03e8864f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03ea8804'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03e8863a"><p class="cl-03e86dda"><span class="cl-03e6e9c4">species</span></p></th><th class="cl-03e8863a"><p class="cl-03e86dda"><span class="cl-03e6e9c4">sex</span></p></th><th class="cl-03e8863b"><p class="cl-03e86de4"><span class="cl-03e6e9c4">No. birds</span></p></th><th class="cl-03e8863b"><p class="cl-03e86de4"><span class="cl-03e6e9c4">Mean body mass (g)</span></p></th><th class="cl-03e8863b"><p class="cl-03e86de4"><span class="cl-03e6e9c4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03e88644"><p class="cl-03e86dda"><span class="cl-03e6e9c4">Adelie</span></p></td><td class="cl-03e88644"><p class="cl-03e86dda"><span class="cl-03e6e9c4">female</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">73</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">3,369</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e88646"><p class="cl-03e86dda"><span class="cl-03e6e9c4">Adelie</span></p></td><td class="cl-03e88646"><p class="cl-03e86dda"><span class="cl-03e6e9c4">male</span></p></td><td class="cl-03e88647"><p class="cl-03e86de4"><span class="cl-03e6e9c4">73</span></p></td><td class="cl-03e88647"><p class="cl-03e86de4"><span class="cl-03e6e9c4">4,043</span></p></td><td class="cl-03e88647"><p class="cl-03e86de4"><span class="cl-03e6e9c4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e88644"><p class="cl-03e86dda"><span class="cl-03e6e9c4">Chinstrap</span></p></td><td class="cl-03e88644"><p class="cl-03e86dda"><span class="cl-03e6e9c4">female</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">34</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">3,527</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e88646"><p class="cl-03e86dda"><span class="cl-03e6e9c4">Chinstrap</span></p></td><td class="cl-03e88646"><p class="cl-03e86dda"><span class="cl-03e6e9c4">male</span></p></td><td class="cl-03e88647"><p class="cl-03e86de4"><span class="cl-03e6e9c4">34</span></p></td><td class="cl-03e88647"><p class="cl-03e86de4"><span class="cl-03e6e9c4">3,939</span></p></td><td class="cl-03e88647"><p class="cl-03e86de4"><span class="cl-03e6e9c4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e88644"><p class="cl-03e86dda"><span class="cl-03e6e9c4">Gentoo</span></p></td><td class="cl-03e88644"><p class="cl-03e86dda"><span class="cl-03e6e9c4">female</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">58</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">4,680</span></p></td><td class="cl-03e88645"><p class="cl-03e86de4"><span class="cl-03e6e9c4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03e8864e"><p class="cl-03e86dda"><span class="cl-03e6e9c4">Gentoo</span></p></td><td class="cl-03e8864e"><p class="cl-03e86dda"><span class="cl-03e6e9c4">male</span></p></td><td class="cl-03e8864f"><p class="cl-03e86de4"><span class="cl-03e6e9c4">61</span></p></td><td class="cl-03e8864f"><p class="cl-03e86de4"><span class="cl-03e6e9c4">5,485</span></p></td><td class="cl-03e8864f"><p class="cl-03e86de4"><span class="cl-03e6e9c4">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-03f1c376{}.cl-03eea006{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03efd430{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03efd431{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03efe416{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe417{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe418{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe420{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe421{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe422{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe42a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03efe42b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03f1c376'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03efe416"><p class="cl-03efd430"><span class="cl-03eea006">species</span></p></th><th class="cl-03efe416"><p class="cl-03efd430"><span class="cl-03eea006">sex</span></p></th><th class="cl-03efe417"><p class="cl-03efd431"><span class="cl-03eea006">No. birds</span></p></th><th class="cl-03efe417"><p class="cl-03efd431"><span class="cl-03eea006">Mean body mass (g)</span></p></th><th class="cl-03efe417"><p class="cl-03efd431"><span class="cl-03eea006">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03efe418"><p class="cl-03efd430"><span class="cl-03eea006">Adelie</span></p></td><td class="cl-03efe418"><p class="cl-03efd430"><span class="cl-03eea006">female</span></p></td><td class="cl-03efe420"><p class="cl-03efd431"><span class="cl-03eea006">73</span></p></td><td class="cl-03efe420"><p class="cl-03efd431"><span class="cl-03eea006">3,369</span></p></td><td class="cl-03efe420"><p class="cl-03efd431"><span class="cl-03eea006">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03efe418"><p class="cl-03efd430"><span class="cl-03eea006">Adelie</span></p></td><td class="cl-03efe418"><p class="cl-03efd430"><span class="cl-03eea006">male</span></p></td><td class="cl-03efe420"><p class="cl-03efd431"><span class="cl-03eea006">73</span></p></td><td class="cl-03efe420"><p class="cl-03efd431"><span class="cl-03eea006">4,043</span></p></td><td class="cl-03efe420"><p class="cl-03efd431"><span class="cl-03eea006">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03efe421"><p class="cl-03efd430"><span class="cl-03eea006">Chinstrap</span></p></td><td class="cl-03efe421"><p class="cl-03efd430"><span class="cl-03eea006">female</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">34</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">3,527</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03efe421"><p class="cl-03efd430"><span class="cl-03eea006">Chinstrap</span></p></td><td class="cl-03efe421"><p class="cl-03efd430"><span class="cl-03eea006">male</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">34</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">3,939</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03efe421"><p class="cl-03efd430"><span class="cl-03eea006">Gentoo</span></p></td><td class="cl-03efe421"><p class="cl-03efd430"><span class="cl-03eea006">female</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">58</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">4,680</span></p></td><td class="cl-03efe422"><p class="cl-03efd431"><span class="cl-03eea006">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03efe42a"><p class="cl-03efd430"><span class="cl-03eea006">Gentoo</span></p></td><td class="cl-03efe42a"><p class="cl-03efd430"><span class="cl-03eea006">male</span></p></td><td class="cl-03efe42b"><p class="cl-03efd431"><span class="cl-03eea006">61</span></p></td><td class="cl-03efe42b"><p class="cl-03efd431"><span class="cl-03eea006">5,485</span></p></td><td class="cl-03efe42b"><p class="cl-03efd431"><span class="cl-03eea006">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-03f779d8{}.cl-03f4389a{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03f438a4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03f438a5{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03f59d52{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03f59d53{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03f5af9a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03f5afa4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03f5afa5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03f5afa6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03f5afa7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03f5afae{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03f779d8'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03f5af9a"><p class="cl-03f59d52"><span class="cl-03f4389a">species</span></p></th><th class="cl-03f5af9a"><p class="cl-03f59d52"><span class="cl-03f4389a">sex</span></p></th><th class="cl-03f5afa4"><p class="cl-03f59d53"><span class="cl-03f4389a">No. birds</span></p></th><th class="cl-03f5afa4"><p class="cl-03f59d53"><span class="cl-03f4389a">Mean body mass (g)</span></p></th><th class="cl-03f5afa4"><p class="cl-03f59d53"><span class="cl-03f4389a">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a4">Adelie</span></p></td><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a5">female</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">73</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">3,369</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a4">Adelie</span></p></td><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a5">male</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">73</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">4,043</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a4">Chinstrap</span></p></td><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a5">female</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">34</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">3,527</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a4">Chinstrap</span></p></td><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a5">male</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">34</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">3,939</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a4">Gentoo</span></p></td><td class="cl-03f5afa5"><p class="cl-03f59d52"><span class="cl-03f438a5">female</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">58</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">4,680</span></p></td><td class="cl-03f5afa6"><p class="cl-03f59d53"><span class="cl-03f438a5">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03f5afa7"><p class="cl-03f59d52"><span class="cl-03f438a4">Gentoo</span></p></td><td class="cl-03f5afa7"><p class="cl-03f59d52"><span class="cl-03f438a5">male</span></p></td><td class="cl-03f5afae"><p class="cl-03f59d53"><span class="cl-03f438a5">61</span></p></td><td class="cl-03f5afae"><p class="cl-03f59d53"><span class="cl-03f438a5">5,485</span></p></td><td class="cl-03f5afae"><p class="cl-03f59d53"><span class="cl-03f438a5">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-03ffb74c{}.cl-03fb6bd8{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-03fd61e0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03fd61ea{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-03fd734c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-03fd734d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-03ffb74c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">species</span></p></th><th class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">sex</span></p></th><th class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">No. birds</span></p></th><th class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">Mean body mass (g)</span></p></th><th class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">Adelie</span></p></td><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">female</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">73</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">3,369</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">Adelie</span></p></td><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">male</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">73</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">4,043</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">Chinstrap</span></p></td><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">female</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">34</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">3,527</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">Chinstrap</span></p></td><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">male</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">34</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">3,939</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">Gentoo</span></p></td><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">female</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">58</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">4,680</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">Gentoo</span></p></td><td class="cl-03fd734c"><p class="cl-03fd61e0"><span class="cl-03fb6bd8">male</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">61</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">5,485</span></p></td><td class="cl-03fd734d"><p class="cl-03fd61ea"><span class="cl-03fb6bd8">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-0406377a{}.cl-0401c1d6{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-04042d4a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-04042d5e{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-04043f42{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f4c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f4d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f4e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f4f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f56{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f57{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f58{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f60{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f61{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f6a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04043f6b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0406377a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-04043f42"><p class="cl-04042d4a"><span class="cl-0401c1d6">species</span></p></th><th class="cl-04043f4c"><p class="cl-04042d4a"><span class="cl-0401c1d6">sex</span></p></th><th class="cl-04043f4d"><p class="cl-04042d5e"><span class="cl-0401c1d6">No. birds</span></p></th><th class="cl-04043f4d"><p class="cl-04042d5e"><span class="cl-0401c1d6">Mean body mass (g)</span></p></th><th class="cl-04043f4e"><p class="cl-04042d5e"><span class="cl-0401c1d6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-04043f4f"><p class="cl-04042d4a"><span class="cl-0401c1d6">Adelie</span></p></td><td class="cl-04043f56"><p class="cl-04042d4a"><span class="cl-0401c1d6">female</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">73</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">3,369</span></p></td><td class="cl-04043f58"><p class="cl-04042d5e"><span class="cl-0401c1d6">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04043f4f"><p class="cl-04042d4a"><span class="cl-0401c1d6">Adelie</span></p></td><td class="cl-04043f56"><p class="cl-04042d4a"><span class="cl-0401c1d6">male</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">73</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">4,043</span></p></td><td class="cl-04043f58"><p class="cl-04042d5e"><span class="cl-0401c1d6">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04043f4f"><p class="cl-04042d4a"><span class="cl-0401c1d6">Chinstrap</span></p></td><td class="cl-04043f56"><p class="cl-04042d4a"><span class="cl-0401c1d6">female</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">34</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">3,527</span></p></td><td class="cl-04043f58"><p class="cl-04042d5e"><span class="cl-0401c1d6">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04043f4f"><p class="cl-04042d4a"><span class="cl-0401c1d6">Chinstrap</span></p></td><td class="cl-04043f56"><p class="cl-04042d4a"><span class="cl-0401c1d6">male</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">34</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">3,939</span></p></td><td class="cl-04043f58"><p class="cl-04042d5e"><span class="cl-0401c1d6">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04043f4f"><p class="cl-04042d4a"><span class="cl-0401c1d6">Gentoo</span></p></td><td class="cl-04043f56"><p class="cl-04042d4a"><span class="cl-0401c1d6">female</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">58</span></p></td><td class="cl-04043f57"><p class="cl-04042d5e"><span class="cl-0401c1d6">4,680</span></p></td><td class="cl-04043f58"><p class="cl-04042d5e"><span class="cl-0401c1d6">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04043f60"><p class="cl-04042d4a"><span class="cl-0401c1d6">Gentoo</span></p></td><td class="cl-04043f61"><p class="cl-04042d4a"><span class="cl-0401c1d6">male</span></p></td><td class="cl-04043f6a"><p class="cl-04042d5e"><span class="cl-0401c1d6">61</span></p></td><td class="cl-04043f6a"><p class="cl-04042d5e"><span class="cl-0401c1d6">5,485</span></p></td><td class="cl-04043f6b"><p class="cl-04042d5e"><span class="cl-0401c1d6">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-040b76fe{}.cl-0407e1f6{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-040950cc{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-040950cd{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-040963a0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963a1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963aa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963ab{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963ac{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963b4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963b5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963b6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963b7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040963b8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-040b76fe'><thead><tr style="overflow-wrap:break-word;"><th class="cl-040963a0"><p class="cl-040950cc"><span class="cl-0407e1f6">species</span></p></th><th class="cl-040963a0"><p class="cl-040950cc"><span class="cl-0407e1f6">sex</span></p></th><th class="cl-040963a1"><p class="cl-040950cd"><span class="cl-0407e1f6">No. birds</span></p></th><th class="cl-040963a1"><p class="cl-040950cd"><span class="cl-0407e1f6">Mean body mass (g)</span></p></th><th class="cl-040963a1"><p class="cl-040950cd"><span class="cl-0407e1f6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-040963aa"><p class="cl-040950cc"><span class="cl-0407e1f6">Adelie</span></p></td><td class="cl-040963ab"><p class="cl-040950cc"><span class="cl-0407e1f6">female</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">73</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">3,369</span></p></td><td class="cl-040963b4"><p class="cl-040950cd"><span class="cl-0407e1f6">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040963aa"><p class="cl-040950cc"><span class="cl-0407e1f6">Adelie</span></p></td><td class="cl-040963ab"><p class="cl-040950cc"><span class="cl-0407e1f6">male</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">73</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">4,043</span></p></td><td class="cl-040963b4"><p class="cl-040950cd"><span class="cl-0407e1f6">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040963aa"><p class="cl-040950cc"><span class="cl-0407e1f6">Chinstrap</span></p></td><td class="cl-040963ab"><p class="cl-040950cc"><span class="cl-0407e1f6">female</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">34</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">3,527</span></p></td><td class="cl-040963b4"><p class="cl-040950cd"><span class="cl-0407e1f6">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040963aa"><p class="cl-040950cc"><span class="cl-0407e1f6">Chinstrap</span></p></td><td class="cl-040963ab"><p class="cl-040950cc"><span class="cl-0407e1f6">male</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">34</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">3,939</span></p></td><td class="cl-040963b4"><p class="cl-040950cd"><span class="cl-0407e1f6">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040963aa"><p class="cl-040950cc"><span class="cl-0407e1f6">Gentoo</span></p></td><td class="cl-040963ab"><p class="cl-040950cc"><span class="cl-0407e1f6">female</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">58</span></p></td><td class="cl-040963ac"><p class="cl-040950cd"><span class="cl-0407e1f6">4,680</span></p></td><td class="cl-040963b4"><p class="cl-040950cd"><span class="cl-0407e1f6">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040963b5"><p class="cl-040950cc"><span class="cl-0407e1f6">Gentoo</span></p></td><td class="cl-040963b6"><p class="cl-040950cc"><span class="cl-0407e1f6">male</span></p></td><td class="cl-040963b7"><p class="cl-040950cd"><span class="cl-0407e1f6">61</span></p></td><td class="cl-040963b7"><p class="cl-040950cd"><span class="cl-0407e1f6">5,485</span></p></td><td class="cl-040963b8"><p class="cl-040950cd"><span class="cl-0407e1f6">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-04103450{}.cl-040d1fa4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-040e663e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-040e6648{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-040e77aa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77ab{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77b4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77b5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77b6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77b7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77be{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-040e77bf{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-04103450'><thead><tr style="overflow-wrap:break-word;"><th class="cl-040e77aa"><p class="cl-040e663e"><span class="cl-040d1fa4">species</span></p></th><th class="cl-040e77aa"><p class="cl-040e663e"><span class="cl-040d1fa4">sex</span></p></th><th class="cl-040e77ab"><p class="cl-040e6648"><span class="cl-040d1fa4">No. birds</span></p></th><th class="cl-040e77ab"><p class="cl-040e6648"><span class="cl-040d1fa4">Mean body mass (g)</span></p></th><th class="cl-040e77ab"><p class="cl-040e6648"><span class="cl-040d1fa4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-040e77b4"><p class="cl-040e663e"><span class="cl-040d1fa4">Adelie</span></p></td><td class="cl-040e77b4"><p class="cl-040e663e"><span class="cl-040d1fa4">female</span></p></td><td class="cl-040e77b5"><p class="cl-040e6648"><span class="cl-040d1fa4">73</span></p></td><td class="cl-040e77b5"><p class="cl-040e6648"><span class="cl-040d1fa4">3,369</span></p></td><td class="cl-040e77b5"><p class="cl-040e6648"><span class="cl-040d1fa4">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">Adelie</span></p></td><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">male</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">73</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">4,043</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">Chinstrap</span></p></td><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">female</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">34</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">3,527</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">Chinstrap</span></p></td><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">male</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">34</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">3,939</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">Gentoo</span></p></td><td class="cl-040e77b6"><p class="cl-040e663e"><span class="cl-040d1fa4">female</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">58</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">4,680</span></p></td><td class="cl-040e77b7"><p class="cl-040e6648"><span class="cl-040d1fa4">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-040e77be"><p class="cl-040e663e"><span class="cl-040d1fa4">Gentoo</span></p></td><td class="cl-040e77be"><p class="cl-040e663e"><span class="cl-040d1fa4">male</span></p></td><td class="cl-040e77bf"><p class="cl-040e6648"><span class="cl-040d1fa4">61</span></p></td><td class="cl-040e77bf"><p class="cl-040e6648"><span class="cl-040d1fa4">5,485</span></p></td><td class="cl-040e77bf"><p class="cl-040e6648"><span class="cl-040d1fa4">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-04180130{}.cl-041421aa{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-041421b4{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-041648d6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-041648d7{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-041658e4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658e5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658e6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658ee{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658ef{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658f0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658f8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041658f9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-04180130'><thead><tr style="overflow-wrap:break-word;"><th class="cl-041658e4"><p class="cl-041648d6"><span class="cl-041421aa">species</span></p></th><th class="cl-041658e4"><p class="cl-041648d6"><span class="cl-041421aa">island</span></p></th><th class="cl-041658e5"><p class="cl-041648d7"><span class="cl-041421aa">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-041658e6"><p class="cl-041648d6"><span class="cl-041421b4">Adelie</span></p></td><td class="cl-041658e6"><p class="cl-041648d6"><span class="cl-041421b4">Torgersen</span></p></td><td class="cl-041658ee"><p class="cl-041648d7"><span class="cl-041421b4">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041658ef"><p class="cl-041648d6"><span class="cl-041421b4">Adelie</span></p></td><td class="cl-041658ef"><p class="cl-041648d6"><span class="cl-041421b4">Dream</span></p></td><td class="cl-041658f0"><p class="cl-041648d7"><span class="cl-041421b4">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041658f8"><p class="cl-041648d6"><span class="cl-041421b4">Adelie</span></p></td><td class="cl-041658f8"><p class="cl-041648d6"><span class="cl-041421b4">Dream</span></p></td><td class="cl-041658f9"><p class="cl-041648d7"><span class="cl-041421b4">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-04201b7c{}.cl-041d29a8{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-041d29a9{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-041e68b8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-041e68c2{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-041e78bc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78bd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78be{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78c6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78c7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78c8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78c9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-041e78ca{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-04201b7c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-041e78bc"><p class="cl-041e68b8"><span class="cl-041d29a8">species</span></p></th><th class="cl-041e78bc"><p class="cl-041e68b8"><span class="cl-041d29a8">sex</span></p></th><th class="cl-041e78bd"><p class="cl-041e68c2"><span class="cl-041d29a8">No. birds</span></p></th><th class="cl-041e78bd"><p class="cl-041e68c2"><span class="cl-041d29a8">Mean body mass (g)</span></p></th><th class="cl-041e78bd"><p class="cl-041e68c2"><span class="cl-041d29a8">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-041e78be"><p class="cl-041e68b8"><span class="cl-041d29a9">Adelie</span></p></td><td class="cl-041e78be"><p class="cl-041e68b8"><span class="cl-041d29a9">female</span></p></td><td class="cl-041e78c6"><p class="cl-041e68c2"><span class="cl-041d29a9">73</span></p></td><td class="cl-041e78c6"><p class="cl-041e68c2"><span class="cl-041d29a9">3,368.836</span></p></td><td class="cl-041e78c6"><p class="cl-041e68c2"><span class="cl-041d29a9">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">Adelie</span></p></td><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">male</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">73</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">4,043.493</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">Chinstrap</span></p></td><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">female</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">34</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">3,527.206</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">Chinstrap</span></p></td><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">male</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">34</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">3,938.971</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">Gentoo</span></p></td><td class="cl-041e78c7"><p class="cl-041e68b8"><span class="cl-041d29a9">female</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">58</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">4,679.741</span></p></td><td class="cl-041e78c8"><p class="cl-041e68c2"><span class="cl-041d29a9">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-041e78c9"><p class="cl-041e68b8"><span class="cl-041d29a9">Gentoo</span></p></td><td class="cl-041e78c9"><p class="cl-041e68b8"><span class="cl-041d29a9">male</span></p></td><td class="cl-041e78ca"><p class="cl-041e68c2"><span class="cl-041d29a9">61</span></p></td><td class="cl-041e78ca"><p class="cl-041e68c2"><span class="cl-041d29a9">5,484.836</span></p></td><td class="cl-041e78ca"><p class="cl-041e68c2"><span class="cl-041d29a9">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-0425b410{}.cl-04221f26{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-04221f27{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-042353d2{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-042353d3{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0423634a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04236354{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04236355{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04236356{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04236357{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0423635e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0423635f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-04236360{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-0425b410'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0423634a"><p class="cl-042353d2"><span class="cl-04221f26">species</span></p></th><th class="cl-0423634a"><p class="cl-042353d2"><span class="cl-04221f26">sex</span></p></th><th class="cl-04236354"><p class="cl-042353d3"><span class="cl-04221f26">No. birds</span></p></th><th class="cl-04236354"><p class="cl-042353d3"><span class="cl-04221f26">Mean body mass (g)</span></p></th><th class="cl-04236354"><p class="cl-042353d3"><span class="cl-04221f26">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-04236355"><p class="cl-042353d2"><span class="cl-04221f27">Adelie</span></p></td><td class="cl-04236355"><p class="cl-042353d2"><span class="cl-04221f27">female</span></p></td><td class="cl-04236356"><p class="cl-042353d3"><span class="cl-04221f27">73</span></p></td><td class="cl-04236356"><p class="cl-042353d3"><span class="cl-04221f27">3369</span></p></td><td class="cl-04236356"><p class="cl-042353d3"><span class="cl-04221f27">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">Adelie</span></p></td><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">male</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">73</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">4043</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">Chinstrap</span></p></td><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">female</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">34</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">3527</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">Chinstrap</span></p></td><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">male</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">34</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">3939</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">Gentoo</span></p></td><td class="cl-04236357"><p class="cl-042353d2"><span class="cl-04221f27">female</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">58</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">4680</span></p></td><td class="cl-0423635e"><p class="cl-042353d3"><span class="cl-04221f27">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0423635f"><p class="cl-042353d2"><span class="cl-04221f27">Gentoo</span></p></td><td class="cl-0423635f"><p class="cl-042353d2"><span class="cl-04221f27">male</span></p></td><td class="cl-04236360"><p class="cl-042353d3"><span class="cl-04221f27">61</span></p></td><td class="cl-04236360"><p class="cl-042353d3"><span class="cl-04221f27">5485</span></p></td><td class="cl-04236360"><p class="cl-042353d3"><span class="cl-04221f27">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-042b6d7e{}.cl-042856c0{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-042856c1{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-0429a80e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0429a818{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-0429b844{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b845{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b84e{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b84f{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b858{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b859{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b85a{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-0429b862{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-042b6d7e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-0429b844"><p class="cl-0429a80e"><span class="cl-042856c0">species</span></p></th><th class="cl-0429b844"><p class="cl-0429a80e"><span class="cl-042856c0">sex</span></p></th><th class="cl-0429b845"><p class="cl-0429a818"><span class="cl-042856c0">No. birds</span></p></th><th class="cl-0429b845"><p class="cl-0429a818"><span class="cl-042856c0">Mean body mass (g)</span></p></th><th class="cl-0429b845"><p class="cl-0429a818"><span class="cl-042856c0">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-0429b84e"><p class="cl-0429a80e"><span class="cl-042856c1">Adelie</span></p></td><td class="cl-0429b84e"><p class="cl-0429a80e"><span class="cl-042856c1">female</span></p></td><td class="cl-0429b84f"><p class="cl-0429a818"><span class="cl-042856c1">73</span></p></td><td class="cl-0429b84f"><p class="cl-0429a818"><span class="cl-042856c1">3,368.836</span></p></td><td class="cl-0429b84f"><p class="cl-0429a818"><span class="cl-042856c1">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0429b858"><p class="cl-0429a80e"><span class="cl-042856c1">male</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">73</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">4,043.493</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-0429b858"><p class="cl-0429a80e"><span class="cl-042856c1">Chinstrap</span></p></td><td class="cl-0429b858"><p class="cl-0429a80e"><span class="cl-042856c1">female</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">34</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">3,527.206</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0429b858"><p class="cl-0429a80e"><span class="cl-042856c1">male</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">34</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">3,938.971</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-0429b85a"><p class="cl-0429a80e"><span class="cl-042856c1">Gentoo</span></p></td><td class="cl-0429b858"><p class="cl-0429a80e"><span class="cl-042856c1">female</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">58</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">4,679.741</span></p></td><td class="cl-0429b859"><p class="cl-0429a818"><span class="cl-042856c1">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-0429b85a"><p class="cl-0429a80e"><span class="cl-042856c1">male</span></p></td><td class="cl-0429b862"><p class="cl-0429a818"><span class="cl-042856c1">61</span></p></td><td class="cl-0429b862"><p class="cl-0429a818"><span class="cl-042856c1">5,484.836</span></p></td><td class="cl-0429b862"><p class="cl-0429a818"><span class="cl-042856c1">313.1586</span></p></td></tr></tbody></table></div>
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
