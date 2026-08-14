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
<div class="tabwid"><style>.cl-80f84420{}.cl-80f41274{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-80f41275{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-80f6389c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-80f638a6{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-80f64a12{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-80f64a1c{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-80f64a1d{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-80f64a1e{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-80f64a26{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-80f64a27{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-80f84420'><thead><tr style="overflow-wrap:break-word;"><th class="cl-80f64a12"><p class="cl-80f6389c"><span class="cl-80f41274">species</span></p></th><th class="cl-80f64a12"><p class="cl-80f6389c"><span class="cl-80f41274">sex</span></p></th><th class="cl-80f64a1c"><p class="cl-80f638a6"><span class="cl-80f41274">No. birds</span></p></th><th class="cl-80f64a1c"><p class="cl-80f638a6"><span class="cl-80f41274">Mean body mass (g)</span></p></th><th class="cl-80f64a1c"><p class="cl-80f638a6"><span class="cl-80f41274">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-80f64a1d"><p class="cl-80f6389c"><span class="cl-80f41275">Adelie</span></p></td><td class="cl-80f64a1d"><p class="cl-80f6389c"><span class="cl-80f41275">female</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">73</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">3,369</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-80f64a26"><p class="cl-80f6389c"><span class="cl-80f41275">Adelie</span></p></td><td class="cl-80f64a26"><p class="cl-80f6389c"><span class="cl-80f41275">male</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">73</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">4,043</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-80f64a1d"><p class="cl-80f6389c"><span class="cl-80f41275">Chinstrap</span></p></td><td class="cl-80f64a1d"><p class="cl-80f6389c"><span class="cl-80f41275">female</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">34</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">3,527</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-80f64a26"><p class="cl-80f6389c"><span class="cl-80f41275">Chinstrap</span></p></td><td class="cl-80f64a26"><p class="cl-80f6389c"><span class="cl-80f41275">male</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">34</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">3,939</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-80f64a1d"><p class="cl-80f6389c"><span class="cl-80f41275">Gentoo</span></p></td><td class="cl-80f64a1d"><p class="cl-80f6389c"><span class="cl-80f41275">female</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">58</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">4,680</span></p></td><td class="cl-80f64a1e"><p class="cl-80f638a6"><span class="cl-80f41275">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-80f64a26"><p class="cl-80f6389c"><span class="cl-80f41275">Gentoo</span></p></td><td class="cl-80f64a26"><p class="cl-80f6389c"><span class="cl-80f41275">male</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">61</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">5,485</span></p></td><td class="cl-80f64a27"><p class="cl-80f638a6"><span class="cl-80f41275">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-810cc26a{}.cl-8109cc0e{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8109cc18{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-810b0178{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-810b0179{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-810b1118{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810b1119{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810b111a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810b1122{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810b1123{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810b1124{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-810cc26a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-810b1118"><p class="cl-810b0178"><span class="cl-8109cc0e">species</span></p></th><th class="cl-810b1118"><p class="cl-810b0178"><span class="cl-8109cc0e">sex</span></p></th><th class="cl-810b1119"><p class="cl-810b0179"><span class="cl-8109cc0e">No. birds</span></p></th><th class="cl-810b1119"><p class="cl-810b0179"><span class="cl-8109cc0e">Mean body mass (g)</span></p></th><th class="cl-810b1119"><p class="cl-810b0179"><span class="cl-8109cc0e">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">Adelie</span></p></td><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">female</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">73</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">3,369</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">Adelie</span></p></td><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">male</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">73</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">4,043</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">Chinstrap</span></p></td><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">female</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">34</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">3,527</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">Chinstrap</span></p></td><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">male</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">34</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">3,939</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">Gentoo</span></p></td><td class="cl-810b111a"><p class="cl-810b0178"><span class="cl-8109cc18">female</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">58</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">4,680</span></p></td><td class="cl-810b1122"><p class="cl-810b0179"><span class="cl-8109cc18">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810b1123"><p class="cl-810b0178"><span class="cl-8109cc18">Gentoo</span></p></td><td class="cl-810b1123"><p class="cl-810b0178"><span class="cl-8109cc18">male</span></p></td><td class="cl-810b1124"><p class="cl-810b0179"><span class="cl-8109cc18">61</span></p></td><td class="cl-810b1124"><p class="cl-810b0179"><span class="cl-8109cc18">5,485</span></p></td><td class="cl-810b1124"><p class="cl-810b0179"><span class="cl-8109cc18">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-811195e2{}.cl-810e765a{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-810fc12c{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-810fd1ee{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810fd1ef{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-810fd1f0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-811195e2'><thead><tr style="overflow-wrap:break-word;"><th class="cl-810fd1ee"><p class="cl-810fc12c"><span class="cl-810e765a">species</span></p></th><th class="cl-810fd1ee"><p class="cl-810fc12c"><span class="cl-810e765a">sex</span></p></th><th class="cl-810fd1ee"><p class="cl-810fc12c"><span class="cl-810e765a">No. birds</span></p></th><th class="cl-810fd1ee"><p class="cl-810fc12c"><span class="cl-810e765a">Mean body mass (g)</span></p></th><th class="cl-810fd1ee"><p class="cl-810fc12c"><span class="cl-810e765a">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">Adelie</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">female</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">73</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">3,369</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">Adelie</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">male</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">73</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">4,043</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">Chinstrap</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">female</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">34</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">3,527</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">Chinstrap</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">male</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">34</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">3,939</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">Gentoo</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">female</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">58</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">4,680</span></p></td><td class="cl-810fd1ef"><p class="cl-810fc12c"><span class="cl-810e765a">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-810fd1f0"><p class="cl-810fc12c"><span class="cl-810e765a">Gentoo</span></p></td><td class="cl-810fd1f0"><p class="cl-810fc12c"><span class="cl-810e765a">male</span></p></td><td class="cl-810fd1f0"><p class="cl-810fc12c"><span class="cl-810e765a">61</span></p></td><td class="cl-810fd1f0"><p class="cl-810fc12c"><span class="cl-810e765a">5,485</span></p></td><td class="cl-810fd1f0"><p class="cl-810fc12c"><span class="cl-810e765a">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-81165bcc{}.cl-811339d8{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-81147dd4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-81147dd5{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-81148ebe{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148ec8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148ec9{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148eca{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148ed2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148ed3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148ed4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81148ed5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-81165bcc'><thead><tr style="overflow-wrap:break-word;"><th class="cl-81148ebe"><p class="cl-81147dd4"><span class="cl-811339d8">species</span></p></th><th class="cl-81148ebe"><p class="cl-81147dd4"><span class="cl-811339d8">sex</span></p></th><th class="cl-81148ec8"><p class="cl-81147dd5"><span class="cl-811339d8">No. birds</span></p></th><th class="cl-81148ec8"><p class="cl-81147dd5"><span class="cl-811339d8">Mean body mass (g)</span></p></th><th class="cl-81148ec8"><p class="cl-81147dd5"><span class="cl-811339d8">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-81148ec9"><p class="cl-81147dd4"><span class="cl-811339d8">Adelie</span></p></td><td class="cl-81148ec9"><p class="cl-81147dd4"><span class="cl-811339d8">female</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">73</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">3,369</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81148ed2"><p class="cl-81147dd4"><span class="cl-811339d8">Adelie</span></p></td><td class="cl-81148ed2"><p class="cl-81147dd4"><span class="cl-811339d8">male</span></p></td><td class="cl-81148ed3"><p class="cl-81147dd5"><span class="cl-811339d8">73</span></p></td><td class="cl-81148ed3"><p class="cl-81147dd5"><span class="cl-811339d8">4,043</span></p></td><td class="cl-81148ed3"><p class="cl-81147dd5"><span class="cl-811339d8">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81148ec9"><p class="cl-81147dd4"><span class="cl-811339d8">Chinstrap</span></p></td><td class="cl-81148ec9"><p class="cl-81147dd4"><span class="cl-811339d8">female</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">34</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">3,527</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81148ed2"><p class="cl-81147dd4"><span class="cl-811339d8">Chinstrap</span></p></td><td class="cl-81148ed2"><p class="cl-81147dd4"><span class="cl-811339d8">male</span></p></td><td class="cl-81148ed3"><p class="cl-81147dd5"><span class="cl-811339d8">34</span></p></td><td class="cl-81148ed3"><p class="cl-81147dd5"><span class="cl-811339d8">3,939</span></p></td><td class="cl-81148ed3"><p class="cl-81147dd5"><span class="cl-811339d8">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81148ec9"><p class="cl-81147dd4"><span class="cl-811339d8">Gentoo</span></p></td><td class="cl-81148ec9"><p class="cl-81147dd4"><span class="cl-811339d8">female</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">58</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">4,680</span></p></td><td class="cl-81148eca"><p class="cl-81147dd5"><span class="cl-811339d8">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81148ed4"><p class="cl-81147dd4"><span class="cl-811339d8">Gentoo</span></p></td><td class="cl-81148ed4"><p class="cl-81147dd4"><span class="cl-811339d8">male</span></p></td><td class="cl-81148ed5"><p class="cl-81147dd5"><span class="cl-811339d8">61</span></p></td><td class="cl-81148ed5"><p class="cl-81147dd5"><span class="cl-811339d8">5,485</span></p></td><td class="cl-81148ed5"><p class="cl-81147dd5"><span class="cl-811339d8">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-811c55ae{}.cl-81198482{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-811aaf10{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-811aaf1a{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-811abe74{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe7e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe7f{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe80{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe81{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe88{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe89{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811abe8a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-811c55ae'><thead><tr style="overflow-wrap:break-word;"><th class="cl-811abe74"><p class="cl-811aaf10"><span class="cl-81198482">species</span></p></th><th class="cl-811abe74"><p class="cl-811aaf10"><span class="cl-81198482">sex</span></p></th><th class="cl-811abe7e"><p class="cl-811aaf1a"><span class="cl-81198482">No. birds</span></p></th><th class="cl-811abe7e"><p class="cl-811aaf1a"><span class="cl-81198482">Mean body mass (g)</span></p></th><th class="cl-811abe7e"><p class="cl-811aaf1a"><span class="cl-81198482">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-811abe7f"><p class="cl-811aaf10"><span class="cl-81198482">Adelie</span></p></td><td class="cl-811abe7f"><p class="cl-811aaf10"><span class="cl-81198482">female</span></p></td><td class="cl-811abe80"><p class="cl-811aaf1a"><span class="cl-81198482">73</span></p></td><td class="cl-811abe80"><p class="cl-811aaf1a"><span class="cl-81198482">3,369</span></p></td><td class="cl-811abe80"><p class="cl-811aaf1a"><span class="cl-81198482">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811abe7f"><p class="cl-811aaf10"><span class="cl-81198482">Adelie</span></p></td><td class="cl-811abe7f"><p class="cl-811aaf10"><span class="cl-81198482">male</span></p></td><td class="cl-811abe80"><p class="cl-811aaf1a"><span class="cl-81198482">73</span></p></td><td class="cl-811abe80"><p class="cl-811aaf1a"><span class="cl-81198482">4,043</span></p></td><td class="cl-811abe80"><p class="cl-811aaf1a"><span class="cl-81198482">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811abe81"><p class="cl-811aaf10"><span class="cl-81198482">Chinstrap</span></p></td><td class="cl-811abe81"><p class="cl-811aaf10"><span class="cl-81198482">female</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">34</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">3,527</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811abe81"><p class="cl-811aaf10"><span class="cl-81198482">Chinstrap</span></p></td><td class="cl-811abe81"><p class="cl-811aaf10"><span class="cl-81198482">male</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">34</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">3,939</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811abe81"><p class="cl-811aaf10"><span class="cl-81198482">Gentoo</span></p></td><td class="cl-811abe81"><p class="cl-811aaf10"><span class="cl-81198482">female</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">58</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">4,680</span></p></td><td class="cl-811abe88"><p class="cl-811aaf1a"><span class="cl-81198482">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811abe89"><p class="cl-811aaf10"><span class="cl-81198482">Gentoo</span></p></td><td class="cl-811abe89"><p class="cl-811aaf10"><span class="cl-81198482">male</span></p></td><td class="cl-811abe8a"><p class="cl-811aaf1a"><span class="cl-81198482">61</span></p></td><td class="cl-811abe8a"><p class="cl-811aaf1a"><span class="cl-81198482">5,485</span></p></td><td class="cl-811abe8a"><p class="cl-811aaf1a"><span class="cl-81198482">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-81214776{}.cl-811e4bb6{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-811e4bb7{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-811e4bc0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-811f79b4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-811f79b5{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-811f8936{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811f8937{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811f8938{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811f8939{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811f8940{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-811f8941{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-81214776'><thead><tr style="overflow-wrap:break-word;"><th class="cl-811f8936"><p class="cl-811f79b4"><span class="cl-811e4bb6">species</span></p></th><th class="cl-811f8936"><p class="cl-811f79b4"><span class="cl-811e4bb6">sex</span></p></th><th class="cl-811f8937"><p class="cl-811f79b5"><span class="cl-811e4bb6">No. birds</span></p></th><th class="cl-811f8937"><p class="cl-811f79b5"><span class="cl-811e4bb6">Mean body mass (g)</span></p></th><th class="cl-811f8937"><p class="cl-811f79b5"><span class="cl-811e4bb6">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bb7">Adelie</span></p></td><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bc0">female</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">73</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">3,369</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bb7">Adelie</span></p></td><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bc0">male</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">73</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">4,043</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bb7">Chinstrap</span></p></td><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bc0">female</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">34</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">3,527</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bb7">Chinstrap</span></p></td><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bc0">male</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">34</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">3,939</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bb7">Gentoo</span></p></td><td class="cl-811f8938"><p class="cl-811f79b4"><span class="cl-811e4bc0">female</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">58</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">4,680</span></p></td><td class="cl-811f8939"><p class="cl-811f79b5"><span class="cl-811e4bc0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-811f8940"><p class="cl-811f79b4"><span class="cl-811e4bb7">Gentoo</span></p></td><td class="cl-811f8940"><p class="cl-811f79b4"><span class="cl-811e4bc0">male</span></p></td><td class="cl-811f8941"><p class="cl-811f79b5"><span class="cl-811e4bc0">61</span></p></td><td class="cl-811f8941"><p class="cl-811f79b5"><span class="cl-811e4bc0">5,485</span></p></td><td class="cl-811f8941"><p class="cl-811f79b5"><span class="cl-811e4bc0">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-812643f2{}.cl-81235462{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-812489c2{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-812489c3{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-81249a02{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-81249a0c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-812643f2'><thead><tr style="overflow-wrap:break-word;"><th class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">species</span></p></th><th class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">sex</span></p></th><th class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">No. birds</span></p></th><th class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">Mean body mass (g)</span></p></th><th class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">Adelie</span></p></td><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">female</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">73</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">3,369</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">Adelie</span></p></td><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">male</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">73</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">4,043</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">Chinstrap</span></p></td><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">female</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">34</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">3,527</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">Chinstrap</span></p></td><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">male</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">34</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">3,939</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">Gentoo</span></p></td><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">female</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">58</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">4,680</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">Gentoo</span></p></td><td class="cl-81249a02"><p class="cl-812489c2"><span class="cl-81235462">male</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">61</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">5,485</span></p></td><td class="cl-81249a0c"><p class="cl-812489c3"><span class="cl-81235462">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-812b9c30{}.cl-8127da8c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8129d512{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8129d513{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8129e53e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e548{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e549{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e54a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e552{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e553{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e554{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e555{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e55c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e55d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e55e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8129e566{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-812b9c30'><thead><tr style="overflow-wrap:break-word;"><th class="cl-8129e53e"><p class="cl-8129d512"><span class="cl-8127da8c">species</span></p></th><th class="cl-8129e548"><p class="cl-8129d512"><span class="cl-8127da8c">sex</span></p></th><th class="cl-8129e549"><p class="cl-8129d513"><span class="cl-8127da8c">No. birds</span></p></th><th class="cl-8129e549"><p class="cl-8129d513"><span class="cl-8127da8c">Mean body mass (g)</span></p></th><th class="cl-8129e54a"><p class="cl-8129d513"><span class="cl-8127da8c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-8129e552"><p class="cl-8129d512"><span class="cl-8127da8c">Adelie</span></p></td><td class="cl-8129e553"><p class="cl-8129d512"><span class="cl-8127da8c">female</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">73</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">3,369</span></p></td><td class="cl-8129e555"><p class="cl-8129d513"><span class="cl-8127da8c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8129e552"><p class="cl-8129d512"><span class="cl-8127da8c">Adelie</span></p></td><td class="cl-8129e553"><p class="cl-8129d512"><span class="cl-8127da8c">male</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">73</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">4,043</span></p></td><td class="cl-8129e555"><p class="cl-8129d513"><span class="cl-8127da8c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8129e552"><p class="cl-8129d512"><span class="cl-8127da8c">Chinstrap</span></p></td><td class="cl-8129e553"><p class="cl-8129d512"><span class="cl-8127da8c">female</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">34</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">3,527</span></p></td><td class="cl-8129e555"><p class="cl-8129d513"><span class="cl-8127da8c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8129e552"><p class="cl-8129d512"><span class="cl-8127da8c">Chinstrap</span></p></td><td class="cl-8129e553"><p class="cl-8129d512"><span class="cl-8127da8c">male</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">34</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">3,939</span></p></td><td class="cl-8129e555"><p class="cl-8129d513"><span class="cl-8127da8c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8129e552"><p class="cl-8129d512"><span class="cl-8127da8c">Gentoo</span></p></td><td class="cl-8129e553"><p class="cl-8129d512"><span class="cl-8127da8c">female</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">58</span></p></td><td class="cl-8129e554"><p class="cl-8129d513"><span class="cl-8127da8c">4,680</span></p></td><td class="cl-8129e555"><p class="cl-8129d513"><span class="cl-8127da8c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8129e55c"><p class="cl-8129d512"><span class="cl-8127da8c">Gentoo</span></p></td><td class="cl-8129e55d"><p class="cl-8129d512"><span class="cl-8127da8c">male</span></p></td><td class="cl-8129e55e"><p class="cl-8129d513"><span class="cl-8127da8c">61</span></p></td><td class="cl-8129e55e"><p class="cl-8129d513"><span class="cl-8127da8c">5,485</span></p></td><td class="cl-8129e566"><p class="cl-8129d513"><span class="cl-8127da8c">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-81300e1e{}.cl-812d21e0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-812e56f0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-812e56fa{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-812e69ce{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69cf{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69d0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69d1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69d8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69d9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69da{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69db{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69e2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-812e69e3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-81300e1e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-812e69ce"><p class="cl-812e56f0"><span class="cl-812d21e0">species</span></p></th><th class="cl-812e69ce"><p class="cl-812e56f0"><span class="cl-812d21e0">sex</span></p></th><th class="cl-812e69cf"><p class="cl-812e56fa"><span class="cl-812d21e0">No. birds</span></p></th><th class="cl-812e69cf"><p class="cl-812e56fa"><span class="cl-812d21e0">Mean body mass (g)</span></p></th><th class="cl-812e69cf"><p class="cl-812e56fa"><span class="cl-812d21e0">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-812e69d0"><p class="cl-812e56f0"><span class="cl-812d21e0">Adelie</span></p></td><td class="cl-812e69d1"><p class="cl-812e56f0"><span class="cl-812d21e0">female</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">73</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">3,369</span></p></td><td class="cl-812e69d9"><p class="cl-812e56fa"><span class="cl-812d21e0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-812e69d0"><p class="cl-812e56f0"><span class="cl-812d21e0">Adelie</span></p></td><td class="cl-812e69d1"><p class="cl-812e56f0"><span class="cl-812d21e0">male</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">73</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">4,043</span></p></td><td class="cl-812e69d9"><p class="cl-812e56fa"><span class="cl-812d21e0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-812e69d0"><p class="cl-812e56f0"><span class="cl-812d21e0">Chinstrap</span></p></td><td class="cl-812e69d1"><p class="cl-812e56f0"><span class="cl-812d21e0">female</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">34</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">3,527</span></p></td><td class="cl-812e69d9"><p class="cl-812e56fa"><span class="cl-812d21e0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-812e69d0"><p class="cl-812e56f0"><span class="cl-812d21e0">Chinstrap</span></p></td><td class="cl-812e69d1"><p class="cl-812e56f0"><span class="cl-812d21e0">male</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">34</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">3,939</span></p></td><td class="cl-812e69d9"><p class="cl-812e56fa"><span class="cl-812d21e0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-812e69d0"><p class="cl-812e56f0"><span class="cl-812d21e0">Gentoo</span></p></td><td class="cl-812e69d1"><p class="cl-812e56f0"><span class="cl-812d21e0">female</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">58</span></p></td><td class="cl-812e69d8"><p class="cl-812e56fa"><span class="cl-812d21e0">4,680</span></p></td><td class="cl-812e69d9"><p class="cl-812e56fa"><span class="cl-812d21e0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-812e69da"><p class="cl-812e56f0"><span class="cl-812d21e0">Gentoo</span></p></td><td class="cl-812e69db"><p class="cl-812e56f0"><span class="cl-812d21e0">male</span></p></td><td class="cl-812e69e2"><p class="cl-812e56fa"><span class="cl-812d21e0">61</span></p></td><td class="cl-812e69e2"><p class="cl-812e56fa"><span class="cl-812d21e0">5,485</span></p></td><td class="cl-812e69e3"><p class="cl-812e56fa"><span class="cl-812d21e0">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-8134753a{}.cl-81318708{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8132b7e0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8132b7e1{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8132c8fc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c8fd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c8fe{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c8ff{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c906{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c907{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c908{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8132c909{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-8134753a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-8132c8fc"><p class="cl-8132b7e0"><span class="cl-81318708">species</span></p></th><th class="cl-8132c8fc"><p class="cl-8132b7e0"><span class="cl-81318708">sex</span></p></th><th class="cl-8132c8fd"><p class="cl-8132b7e1"><span class="cl-81318708">No. birds</span></p></th><th class="cl-8132c8fd"><p class="cl-8132b7e1"><span class="cl-81318708">Mean body mass (g)</span></p></th><th class="cl-8132c8fd"><p class="cl-8132b7e1"><span class="cl-81318708">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-8132c8fe"><p class="cl-8132b7e0"><span class="cl-81318708">Adelie</span></p></td><td class="cl-8132c8fe"><p class="cl-8132b7e0"><span class="cl-81318708">female</span></p></td><td class="cl-8132c8ff"><p class="cl-8132b7e1"><span class="cl-81318708">73</span></p></td><td class="cl-8132c8ff"><p class="cl-8132b7e1"><span class="cl-81318708">3,369</span></p></td><td class="cl-8132c8ff"><p class="cl-8132b7e1"><span class="cl-81318708">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">Adelie</span></p></td><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">male</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">73</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">4,043</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">Chinstrap</span></p></td><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">female</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">34</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">3,527</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">Chinstrap</span></p></td><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">male</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">34</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">3,939</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">Gentoo</span></p></td><td class="cl-8132c906"><p class="cl-8132b7e0"><span class="cl-81318708">female</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">58</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">4,680</span></p></td><td class="cl-8132c907"><p class="cl-8132b7e1"><span class="cl-81318708">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8132c908"><p class="cl-8132b7e0"><span class="cl-81318708">Gentoo</span></p></td><td class="cl-8132c908"><p class="cl-8132b7e0"><span class="cl-81318708">male</span></p></td><td class="cl-8132c909"><p class="cl-8132b7e1"><span class="cl-81318708">61</span></p></td><td class="cl-8132c909"><p class="cl-8132b7e1"><span class="cl-81318708">5,485</span></p></td><td class="cl-8132c909"><p class="cl-8132b7e1"><span class="cl-81318708">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-813bb688{}.cl-8138222a{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-81382234{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-813a19e0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-813a19ea{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-813a291c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a2926{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a2927{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a2928{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a2929{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a292a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a2930{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-813a2931{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-813bb688'><thead><tr style="overflow-wrap:break-word;"><th class="cl-813a291c"><p class="cl-813a19e0"><span class="cl-8138222a">species</span></p></th><th class="cl-813a291c"><p class="cl-813a19e0"><span class="cl-8138222a">island</span></p></th><th class="cl-813a2926"><p class="cl-813a19ea"><span class="cl-8138222a">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-813a2927"><p class="cl-813a19e0"><span class="cl-81382234">Adelie</span></p></td><td class="cl-813a2927"><p class="cl-813a19e0"><span class="cl-81382234">Torgersen</span></p></td><td class="cl-813a2928"><p class="cl-813a19ea"><span class="cl-81382234">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-813a2929"><p class="cl-813a19e0"><span class="cl-81382234">Adelie</span></p></td><td class="cl-813a2929"><p class="cl-813a19e0"><span class="cl-81382234">Dream</span></p></td><td class="cl-813a292a"><p class="cl-813a19ea"><span class="cl-81382234">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-813a2930"><p class="cl-813a19e0"><span class="cl-81382234">Adelie</span></p></td><td class="cl-813a2930"><p class="cl-813a19e0"><span class="cl-81382234">Dream</span></p></td><td class="cl-813a2931"><p class="cl-813a19ea"><span class="cl-81382234">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-814380a2{}.cl-81409220{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-81409221{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8141caaa{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8141caab{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8141da72{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da73{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da7c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da7d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da7e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da7f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da80{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8141da86{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-814380a2'><thead><tr style="overflow-wrap:break-word;"><th class="cl-8141da72"><p class="cl-8141caaa"><span class="cl-81409220">species</span></p></th><th class="cl-8141da72"><p class="cl-8141caaa"><span class="cl-81409220">sex</span></p></th><th class="cl-8141da73"><p class="cl-8141caab"><span class="cl-81409220">No. birds</span></p></th><th class="cl-8141da73"><p class="cl-8141caab"><span class="cl-81409220">Mean body mass (g)</span></p></th><th class="cl-8141da73"><p class="cl-8141caab"><span class="cl-81409220">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-8141da7c"><p class="cl-8141caaa"><span class="cl-81409221">Adelie</span></p></td><td class="cl-8141da7c"><p class="cl-8141caaa"><span class="cl-81409221">female</span></p></td><td class="cl-8141da7d"><p class="cl-8141caab"><span class="cl-81409221">73</span></p></td><td class="cl-8141da7d"><p class="cl-8141caab"><span class="cl-81409221">3,368.836</span></p></td><td class="cl-8141da7d"><p class="cl-8141caab"><span class="cl-81409221">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">Adelie</span></p></td><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">male</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">73</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">4,043.493</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">Chinstrap</span></p></td><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">female</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">34</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">3,527.206</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">Chinstrap</span></p></td><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">male</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">34</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">3,938.971</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">Gentoo</span></p></td><td class="cl-8141da7e"><p class="cl-8141caaa"><span class="cl-81409221">female</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">58</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">4,679.741</span></p></td><td class="cl-8141da7f"><p class="cl-8141caab"><span class="cl-81409221">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8141da80"><p class="cl-8141caaa"><span class="cl-81409221">Gentoo</span></p></td><td class="cl-8141da80"><p class="cl-8141caaa"><span class="cl-81409221">male</span></p></td><td class="cl-8141da86"><p class="cl-8141caab"><span class="cl-81409221">61</span></p></td><td class="cl-8141da86"><p class="cl-8141caab"><span class="cl-81409221">5,484.836</span></p></td><td class="cl-8141da86"><p class="cl-8141caab"><span class="cl-81409221">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-814915d0{}.cl-8145847e{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-81458488{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8146b682{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8146b68c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8146c654{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c655{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c65e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c65f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c660{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c661{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c668{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8146c669{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-814915d0'><thead><tr style="overflow-wrap:break-word;"><th class="cl-8146c654"><p class="cl-8146b682"><span class="cl-8145847e">species</span></p></th><th class="cl-8146c654"><p class="cl-8146b682"><span class="cl-8145847e">sex</span></p></th><th class="cl-8146c655"><p class="cl-8146b68c"><span class="cl-8145847e">No. birds</span></p></th><th class="cl-8146c655"><p class="cl-8146b68c"><span class="cl-8145847e">Mean body mass (g)</span></p></th><th class="cl-8146c655"><p class="cl-8146b68c"><span class="cl-8145847e">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-8146c65e"><p class="cl-8146b682"><span class="cl-81458488">Adelie</span></p></td><td class="cl-8146c65e"><p class="cl-8146b682"><span class="cl-81458488">female</span></p></td><td class="cl-8146c65f"><p class="cl-8146b68c"><span class="cl-81458488">73</span></p></td><td class="cl-8146c65f"><p class="cl-8146b68c"><span class="cl-81458488">3369</span></p></td><td class="cl-8146c65f"><p class="cl-8146b68c"><span class="cl-81458488">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">Adelie</span></p></td><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">male</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">73</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">4043</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">Chinstrap</span></p></td><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">female</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">34</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">3527</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">Chinstrap</span></p></td><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">male</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">34</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">3939</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">Gentoo</span></p></td><td class="cl-8146c660"><p class="cl-8146b682"><span class="cl-81458488">female</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">58</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">4680</span></p></td><td class="cl-8146c661"><p class="cl-8146b68c"><span class="cl-81458488">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8146c668"><p class="cl-8146b682"><span class="cl-81458488">Gentoo</span></p></td><td class="cl-8146c668"><p class="cl-8146b682"><span class="cl-81458488">male</span></p></td><td class="cl-8146c669"><p class="cl-8146b68c"><span class="cl-81458488">61</span></p></td><td class="cl-8146c669"><p class="cl-8146b68c"><span class="cl-81458488">5485</span></p></td><td class="cl-8146c669"><p class="cl-8146b68c"><span class="cl-81458488">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-814e78cc{}.cl-814b8a18{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-814b8a19{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-814cc2ca{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-814cc2cb{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-814cd260{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd261{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd26a{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd26b{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd26c{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd274{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd275{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-814cd276{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-814e78cc'><thead><tr style="overflow-wrap:break-word;"><th class="cl-814cd260"><p class="cl-814cc2ca"><span class="cl-814b8a18">species</span></p></th><th class="cl-814cd260"><p class="cl-814cc2ca"><span class="cl-814b8a18">sex</span></p></th><th class="cl-814cd261"><p class="cl-814cc2cb"><span class="cl-814b8a18">No. birds</span></p></th><th class="cl-814cd261"><p class="cl-814cc2cb"><span class="cl-814b8a18">Mean body mass (g)</span></p></th><th class="cl-814cd261"><p class="cl-814cc2cb"><span class="cl-814b8a18">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-814cd26a"><p class="cl-814cc2ca"><span class="cl-814b8a19">Adelie</span></p></td><td class="cl-814cd26a"><p class="cl-814cc2ca"><span class="cl-814b8a19">female</span></p></td><td class="cl-814cd26b"><p class="cl-814cc2cb"><span class="cl-814b8a19">73</span></p></td><td class="cl-814cd26b"><p class="cl-814cc2cb"><span class="cl-814b8a19">3,368.836</span></p></td><td class="cl-814cd26b"><p class="cl-814cc2cb"><span class="cl-814b8a19">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-814cd26c"><p class="cl-814cc2ca"><span class="cl-814b8a19">male</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">73</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">4,043.493</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-814cd26c"><p class="cl-814cc2ca"><span class="cl-814b8a19">Chinstrap</span></p></td><td class="cl-814cd26c"><p class="cl-814cc2ca"><span class="cl-814b8a19">female</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">34</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">3,527.206</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-814cd26c"><p class="cl-814cc2ca"><span class="cl-814b8a19">male</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">34</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">3,938.971</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-814cd275"><p class="cl-814cc2ca"><span class="cl-814b8a19">Gentoo</span></p></td><td class="cl-814cd26c"><p class="cl-814cc2ca"><span class="cl-814b8a19">female</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">58</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">4,679.741</span></p></td><td class="cl-814cd274"><p class="cl-814cc2cb"><span class="cl-814b8a19">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-814cd275"><p class="cl-814cc2ca"><span class="cl-814b8a19">male</span></p></td><td class="cl-814cd276"><p class="cl-814cc2cb"><span class="cl-814b8a19">61</span></p></td><td class="cl-814cd276"><p class="cl-814cc2cb"><span class="cl-814b8a19">5,484.836</span></p></td><td class="cl-814cd276"><p class="cl-814cc2cb"><span class="cl-814b8a19">313.1586</span></p></td></tr></tbody></table></div>
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
