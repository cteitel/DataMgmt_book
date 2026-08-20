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
<div class="tabwid"><style>.cl-9d856d16{}.cl-9d813624{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9d81362e{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9d834f72{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9d834f7c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9d83612e{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d836138{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d836139{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d836142{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d836143{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d836144{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9d856d16'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9d83612e"><p class="cl-9d834f72"><span class="cl-9d813624">species</span></p></th><th class="cl-9d83612e"><p class="cl-9d834f72"><span class="cl-9d813624">sex</span></p></th><th class="cl-9d836138"><p class="cl-9d834f7c"><span class="cl-9d813624">No. birds</span></p></th><th class="cl-9d836138"><p class="cl-9d834f7c"><span class="cl-9d813624">Mean body mass (g)</span></p></th><th class="cl-9d836138"><p class="cl-9d834f7c"><span class="cl-9d813624">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9d836139"><p class="cl-9d834f72"><span class="cl-9d81362e">Adelie</span></p></td><td class="cl-9d836139"><p class="cl-9d834f72"><span class="cl-9d81362e">female</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">73</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">3,369</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d836143"><p class="cl-9d834f72"><span class="cl-9d81362e">Adelie</span></p></td><td class="cl-9d836143"><p class="cl-9d834f72"><span class="cl-9d81362e">male</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">73</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">4,043</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d836139"><p class="cl-9d834f72"><span class="cl-9d81362e">Chinstrap</span></p></td><td class="cl-9d836139"><p class="cl-9d834f72"><span class="cl-9d81362e">female</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">34</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">3,527</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d836143"><p class="cl-9d834f72"><span class="cl-9d81362e">Chinstrap</span></p></td><td class="cl-9d836143"><p class="cl-9d834f72"><span class="cl-9d81362e">male</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">34</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">3,939</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d836139"><p class="cl-9d834f72"><span class="cl-9d81362e">Gentoo</span></p></td><td class="cl-9d836139"><p class="cl-9d834f72"><span class="cl-9d81362e">female</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">58</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">4,680</span></p></td><td class="cl-9d836142"><p class="cl-9d834f7c"><span class="cl-9d81362e">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d836143"><p class="cl-9d834f72"><span class="cl-9d81362e">Gentoo</span></p></td><td class="cl-9d836143"><p class="cl-9d834f72"><span class="cl-9d81362e">male</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">61</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">5,485</span></p></td><td class="cl-9d836144"><p class="cl-9d834f7c"><span class="cl-9d81362e">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-9d9ba72a{}.cl-9d986a60{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9d986a6a{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9d99c1e4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9d99c1e5{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9d99d30a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d99d30b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d99d30c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d99d30d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d99d314{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d99d315{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9d9ba72a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9d99d30a"><p class="cl-9d99c1e4"><span class="cl-9d986a60">species</span></p></th><th class="cl-9d99d30a"><p class="cl-9d99c1e4"><span class="cl-9d986a60">sex</span></p></th><th class="cl-9d99d30b"><p class="cl-9d99c1e5"><span class="cl-9d986a60">No. birds</span></p></th><th class="cl-9d99d30b"><p class="cl-9d99c1e5"><span class="cl-9d986a60">Mean body mass (g)</span></p></th><th class="cl-9d99d30b"><p class="cl-9d99c1e5"><span class="cl-9d986a60">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">Adelie</span></p></td><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">female</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">73</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">3,369</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">Adelie</span></p></td><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">male</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">73</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">4,043</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">Chinstrap</span></p></td><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">female</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">34</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">3,527</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">Chinstrap</span></p></td><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">male</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">34</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">3,939</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">Gentoo</span></p></td><td class="cl-9d99d30c"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">female</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">58</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">4,680</span></p></td><td class="cl-9d99d30d"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d99d314"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">Gentoo</span></p></td><td class="cl-9d99d314"><p class="cl-9d99c1e4"><span class="cl-9d986a6a">male</span></p></td><td class="cl-9d99d315"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">61</span></p></td><td class="cl-9d99d315"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">5,485</span></p></td><td class="cl-9d99d315"><p class="cl-9d99c1e5"><span class="cl-9d986a6a">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-9da0e226{}.cl-9d9d8518{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9d9ef0e2{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-9d9f01b8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d9f01b9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9d9f01ba{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9da0e226'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9d9f01b8"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">species</span></p></th><th class="cl-9d9f01b8"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">sex</span></p></th><th class="cl-9d9f01b8"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">No. birds</span></p></th><th class="cl-9d9f01b8"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Mean body mass (g)</span></p></th><th class="cl-9d9f01b8"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Adelie</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">female</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">73</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">3,369</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Adelie</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">male</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">73</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">4,043</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Chinstrap</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">female</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">34</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">3,527</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Chinstrap</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">male</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">34</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">3,939</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Gentoo</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">female</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">58</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">4,680</span></p></td><td class="cl-9d9f01b9"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9d9f01ba"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">Gentoo</span></p></td><td class="cl-9d9f01ba"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">male</span></p></td><td class="cl-9d9f01ba"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">61</span></p></td><td class="cl-9d9f01ba"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">5,485</span></p></td><td class="cl-9d9f01ba"><p class="cl-9d9ef0e2"><span class="cl-9d9d8518">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-9da61412{}.cl-9da29d8c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9da40136{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9da40137{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9da4143c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da4143d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da41446{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da41447{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da41448{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da41449{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da41450{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9da41451{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9da61412'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9da4143c"><p class="cl-9da40136"><span class="cl-9da29d8c">species</span></p></th><th class="cl-9da4143c"><p class="cl-9da40136"><span class="cl-9da29d8c">sex</span></p></th><th class="cl-9da4143d"><p class="cl-9da40137"><span class="cl-9da29d8c">No. birds</span></p></th><th class="cl-9da4143d"><p class="cl-9da40137"><span class="cl-9da29d8c">Mean body mass (g)</span></p></th><th class="cl-9da4143d"><p class="cl-9da40137"><span class="cl-9da29d8c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9da41446"><p class="cl-9da40136"><span class="cl-9da29d8c">Adelie</span></p></td><td class="cl-9da41446"><p class="cl-9da40136"><span class="cl-9da29d8c">female</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">73</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">3,369</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9da41448"><p class="cl-9da40136"><span class="cl-9da29d8c">Adelie</span></p></td><td class="cl-9da41448"><p class="cl-9da40136"><span class="cl-9da29d8c">male</span></p></td><td class="cl-9da41449"><p class="cl-9da40137"><span class="cl-9da29d8c">73</span></p></td><td class="cl-9da41449"><p class="cl-9da40137"><span class="cl-9da29d8c">4,043</span></p></td><td class="cl-9da41449"><p class="cl-9da40137"><span class="cl-9da29d8c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9da41446"><p class="cl-9da40136"><span class="cl-9da29d8c">Chinstrap</span></p></td><td class="cl-9da41446"><p class="cl-9da40136"><span class="cl-9da29d8c">female</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">34</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">3,527</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9da41448"><p class="cl-9da40136"><span class="cl-9da29d8c">Chinstrap</span></p></td><td class="cl-9da41448"><p class="cl-9da40136"><span class="cl-9da29d8c">male</span></p></td><td class="cl-9da41449"><p class="cl-9da40137"><span class="cl-9da29d8c">34</span></p></td><td class="cl-9da41449"><p class="cl-9da40137"><span class="cl-9da29d8c">3,939</span></p></td><td class="cl-9da41449"><p class="cl-9da40137"><span class="cl-9da29d8c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9da41446"><p class="cl-9da40136"><span class="cl-9da29d8c">Gentoo</span></p></td><td class="cl-9da41446"><p class="cl-9da40136"><span class="cl-9da29d8c">female</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">58</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">4,680</span></p></td><td class="cl-9da41447"><p class="cl-9da40137"><span class="cl-9da29d8c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9da41450"><p class="cl-9da40136"><span class="cl-9da29d8c">Gentoo</span></p></td><td class="cl-9da41450"><p class="cl-9da40136"><span class="cl-9da29d8c">male</span></p></td><td class="cl-9da41451"><p class="cl-9da40137"><span class="cl-9da29d8c">61</span></p></td><td class="cl-9da41451"><p class="cl-9da40137"><span class="cl-9da29d8c">5,485</span></p></td><td class="cl-9da41451"><p class="cl-9da40137"><span class="cl-9da29d8c">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-9dac5ec6{}.cl-9da93a3e{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9daa83f8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9daa83f9{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9daa9488{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa9492{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa9493{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa9494{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa949c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa949d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa949e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9daa949f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9dac5ec6'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9daa9488"><p class="cl-9daa83f8"><span class="cl-9da93a3e">species</span></p></th><th class="cl-9daa9488"><p class="cl-9daa83f8"><span class="cl-9da93a3e">sex</span></p></th><th class="cl-9daa9492"><p class="cl-9daa83f9"><span class="cl-9da93a3e">No. birds</span></p></th><th class="cl-9daa9492"><p class="cl-9daa83f9"><span class="cl-9da93a3e">Mean body mass (g)</span></p></th><th class="cl-9daa9492"><p class="cl-9daa83f9"><span class="cl-9da93a3e">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9daa9493"><p class="cl-9daa83f8"><span class="cl-9da93a3e">Adelie</span></p></td><td class="cl-9daa9493"><p class="cl-9daa83f8"><span class="cl-9da93a3e">female</span></p></td><td class="cl-9daa9494"><p class="cl-9daa83f9"><span class="cl-9da93a3e">73</span></p></td><td class="cl-9daa9494"><p class="cl-9daa83f9"><span class="cl-9da93a3e">3,369</span></p></td><td class="cl-9daa9494"><p class="cl-9daa83f9"><span class="cl-9da93a3e">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9daa9493"><p class="cl-9daa83f8"><span class="cl-9da93a3e">Adelie</span></p></td><td class="cl-9daa9493"><p class="cl-9daa83f8"><span class="cl-9da93a3e">male</span></p></td><td class="cl-9daa9494"><p class="cl-9daa83f9"><span class="cl-9da93a3e">73</span></p></td><td class="cl-9daa9494"><p class="cl-9daa83f9"><span class="cl-9da93a3e">4,043</span></p></td><td class="cl-9daa9494"><p class="cl-9daa83f9"><span class="cl-9da93a3e">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9daa949c"><p class="cl-9daa83f8"><span class="cl-9da93a3e">Chinstrap</span></p></td><td class="cl-9daa949c"><p class="cl-9daa83f8"><span class="cl-9da93a3e">female</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">34</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">3,527</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9daa949c"><p class="cl-9daa83f8"><span class="cl-9da93a3e">Chinstrap</span></p></td><td class="cl-9daa949c"><p class="cl-9daa83f8"><span class="cl-9da93a3e">male</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">34</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">3,939</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9daa949c"><p class="cl-9daa83f8"><span class="cl-9da93a3e">Gentoo</span></p></td><td class="cl-9daa949c"><p class="cl-9daa83f8"><span class="cl-9da93a3e">female</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">58</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">4,680</span></p></td><td class="cl-9daa949d"><p class="cl-9daa83f9"><span class="cl-9da93a3e">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9daa949e"><p class="cl-9daa83f8"><span class="cl-9da93a3e">Gentoo</span></p></td><td class="cl-9daa949e"><p class="cl-9daa83f8"><span class="cl-9da93a3e">male</span></p></td><td class="cl-9daa949f"><p class="cl-9daa83f9"><span class="cl-9da93a3e">61</span></p></td><td class="cl-9daa949f"><p class="cl-9daa83f9"><span class="cl-9da93a3e">5,485</span></p></td><td class="cl-9daa949f"><p class="cl-9daa83f9"><span class="cl-9da93a3e">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-9db1d1d0{}.cl-9daea230{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9daea23a{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9daea23b{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9daff220{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9daff22a{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9db00328{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9db00332{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9db00333{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9db0033c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9db0033d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9db0033e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9db1d1d0'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9db00328"><p class="cl-9daff220"><span class="cl-9daea230">species</span></p></th><th class="cl-9db00328"><p class="cl-9daff220"><span class="cl-9daea230">sex</span></p></th><th class="cl-9db00332"><p class="cl-9daff22a"><span class="cl-9daea230">No. birds</span></p></th><th class="cl-9db00332"><p class="cl-9daff22a"><span class="cl-9daea230">Mean body mass (g)</span></p></th><th class="cl-9db00332"><p class="cl-9daff22a"><span class="cl-9daea230">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23a">Adelie</span></p></td><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23b">female</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">73</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">3,369</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23a">Adelie</span></p></td><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23b">male</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">73</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">4,043</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23a">Chinstrap</span></p></td><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23b">female</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">34</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">3,527</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23a">Chinstrap</span></p></td><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23b">male</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">34</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">3,939</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23a">Gentoo</span></p></td><td class="cl-9db00333"><p class="cl-9daff220"><span class="cl-9daea23b">female</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">58</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">4,680</span></p></td><td class="cl-9db0033c"><p class="cl-9daff22a"><span class="cl-9daea23b">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db0033d"><p class="cl-9daff220"><span class="cl-9daea23a">Gentoo</span></p></td><td class="cl-9db0033d"><p class="cl-9daff220"><span class="cl-9daea23b">male</span></p></td><td class="cl-9db0033e"><p class="cl-9daff22a"><span class="cl-9daea23b">61</span></p></td><td class="cl-9db0033e"><p class="cl-9daff22a"><span class="cl-9daea23b">5,485</span></p></td><td class="cl-9db0033e"><p class="cl-9daff22a"><span class="cl-9daea23b">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-9db7214e{}.cl-9db40784{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9db5612e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9db56138{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9db570ec{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9db570f6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9db7214e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">species</span></p></th><th class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">sex</span></p></th><th class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">No. birds</span></p></th><th class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">Mean body mass (g)</span></p></th><th class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">Adelie</span></p></td><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">female</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">73</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">3,369</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">Adelie</span></p></td><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">male</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">73</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">4,043</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">Chinstrap</span></p></td><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">female</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">34</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">3,527</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">Chinstrap</span></p></td><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">male</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">34</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">3,939</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">Gentoo</span></p></td><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">female</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">58</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">4,680</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">Gentoo</span></p></td><td class="cl-9db570ec"><p class="cl-9db5612e"><span class="cl-9db40784">male</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">61</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">5,485</span></p></td><td class="cl-9db570f6"><p class="cl-9db56138"><span class="cl-9db40784">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-9dbc5308{}.cl-9db8c152{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9dbaa026{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dbaa030{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dbaaefe{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaeff{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf08{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf09{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf0a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf0b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf0c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf12{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf13{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf14{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf15{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbaaf16{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9dbc5308'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9dbaaefe"><p class="cl-9dbaa026"><span class="cl-9db8c152">species</span></p></th><th class="cl-9dbaaeff"><p class="cl-9dbaa026"><span class="cl-9db8c152">sex</span></p></th><th class="cl-9dbaaf08"><p class="cl-9dbaa030"><span class="cl-9db8c152">No. birds</span></p></th><th class="cl-9dbaaf08"><p class="cl-9dbaa030"><span class="cl-9db8c152">Mean body mass (g)</span></p></th><th class="cl-9dbaaf09"><p class="cl-9dbaa030"><span class="cl-9db8c152">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9dbaaf0a"><p class="cl-9dbaa026"><span class="cl-9db8c152">Adelie</span></p></td><td class="cl-9dbaaf0b"><p class="cl-9dbaa026"><span class="cl-9db8c152">female</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">73</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">3,369</span></p></td><td class="cl-9dbaaf12"><p class="cl-9dbaa030"><span class="cl-9db8c152">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbaaf0a"><p class="cl-9dbaa026"><span class="cl-9db8c152">Adelie</span></p></td><td class="cl-9dbaaf0b"><p class="cl-9dbaa026"><span class="cl-9db8c152">male</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">73</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">4,043</span></p></td><td class="cl-9dbaaf12"><p class="cl-9dbaa030"><span class="cl-9db8c152">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbaaf0a"><p class="cl-9dbaa026"><span class="cl-9db8c152">Chinstrap</span></p></td><td class="cl-9dbaaf0b"><p class="cl-9dbaa026"><span class="cl-9db8c152">female</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">34</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">3,527</span></p></td><td class="cl-9dbaaf12"><p class="cl-9dbaa030"><span class="cl-9db8c152">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbaaf0a"><p class="cl-9dbaa026"><span class="cl-9db8c152">Chinstrap</span></p></td><td class="cl-9dbaaf0b"><p class="cl-9dbaa026"><span class="cl-9db8c152">male</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">34</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">3,939</span></p></td><td class="cl-9dbaaf12"><p class="cl-9dbaa030"><span class="cl-9db8c152">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbaaf0a"><p class="cl-9dbaa026"><span class="cl-9db8c152">Gentoo</span></p></td><td class="cl-9dbaaf0b"><p class="cl-9dbaa026"><span class="cl-9db8c152">female</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">58</span></p></td><td class="cl-9dbaaf0c"><p class="cl-9dbaa030"><span class="cl-9db8c152">4,680</span></p></td><td class="cl-9dbaaf12"><p class="cl-9dbaa030"><span class="cl-9db8c152">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbaaf13"><p class="cl-9dbaa026"><span class="cl-9db8c152">Gentoo</span></p></td><td class="cl-9dbaaf14"><p class="cl-9dbaa026"><span class="cl-9db8c152">male</span></p></td><td class="cl-9dbaaf15"><p class="cl-9dbaa030"><span class="cl-9db8c152">61</span></p></td><td class="cl-9dbaaf15"><p class="cl-9dbaa030"><span class="cl-9db8c152">5,485</span></p></td><td class="cl-9dbaaf16"><p class="cl-9dbaa030"><span class="cl-9db8c152">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-9dc0ce42{}.cl-9dbde574{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9dbf1a02{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dbf1a03{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dbf2a06{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a10{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a11{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a12{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a13{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a1a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a1b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a1c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a1d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dbf2a24{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9dc0ce42'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9dbf2a06"><p class="cl-9dbf1a02"><span class="cl-9dbde574">species</span></p></th><th class="cl-9dbf2a06"><p class="cl-9dbf1a02"><span class="cl-9dbde574">sex</span></p></th><th class="cl-9dbf2a10"><p class="cl-9dbf1a03"><span class="cl-9dbde574">No. birds</span></p></th><th class="cl-9dbf2a10"><p class="cl-9dbf1a03"><span class="cl-9dbde574">Mean body mass (g)</span></p></th><th class="cl-9dbf2a10"><p class="cl-9dbf1a03"><span class="cl-9dbde574">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9dbf2a11"><p class="cl-9dbf1a02"><span class="cl-9dbde574">Adelie</span></p></td><td class="cl-9dbf2a12"><p class="cl-9dbf1a02"><span class="cl-9dbde574">female</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">73</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">3,369</span></p></td><td class="cl-9dbf2a1a"><p class="cl-9dbf1a03"><span class="cl-9dbde574">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbf2a11"><p class="cl-9dbf1a02"><span class="cl-9dbde574">Adelie</span></p></td><td class="cl-9dbf2a12"><p class="cl-9dbf1a02"><span class="cl-9dbde574">male</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">73</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">4,043</span></p></td><td class="cl-9dbf2a1a"><p class="cl-9dbf1a03"><span class="cl-9dbde574">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbf2a11"><p class="cl-9dbf1a02"><span class="cl-9dbde574">Chinstrap</span></p></td><td class="cl-9dbf2a12"><p class="cl-9dbf1a02"><span class="cl-9dbde574">female</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">34</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">3,527</span></p></td><td class="cl-9dbf2a1a"><p class="cl-9dbf1a03"><span class="cl-9dbde574">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbf2a11"><p class="cl-9dbf1a02"><span class="cl-9dbde574">Chinstrap</span></p></td><td class="cl-9dbf2a12"><p class="cl-9dbf1a02"><span class="cl-9dbde574">male</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">34</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">3,939</span></p></td><td class="cl-9dbf2a1a"><p class="cl-9dbf1a03"><span class="cl-9dbde574">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbf2a11"><p class="cl-9dbf1a02"><span class="cl-9dbde574">Gentoo</span></p></td><td class="cl-9dbf2a12"><p class="cl-9dbf1a02"><span class="cl-9dbde574">female</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">58</span></p></td><td class="cl-9dbf2a13"><p class="cl-9dbf1a03"><span class="cl-9dbde574">4,680</span></p></td><td class="cl-9dbf2a1a"><p class="cl-9dbf1a03"><span class="cl-9dbde574">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dbf2a1b"><p class="cl-9dbf1a02"><span class="cl-9dbde574">Gentoo</span></p></td><td class="cl-9dbf2a1c"><p class="cl-9dbf1a02"><span class="cl-9dbde574">male</span></p></td><td class="cl-9dbf2a1d"><p class="cl-9dbf1a03"><span class="cl-9dbde574">61</span></p></td><td class="cl-9dbf2a1d"><p class="cl-9dbf1a03"><span class="cl-9dbde574">5,485</span></p></td><td class="cl-9dbf2a24"><p class="cl-9dbf1a03"><span class="cl-9dbde574">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-9dc538ba{}.cl-9dc24b78{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9dc378b8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dc378b9{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dc389e8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389e9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389ea{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389eb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389f2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389f3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389f4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dc389f5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9dc538ba'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9dc389e8"><p class="cl-9dc378b8"><span class="cl-9dc24b78">species</span></p></th><th class="cl-9dc389e8"><p class="cl-9dc378b8"><span class="cl-9dc24b78">sex</span></p></th><th class="cl-9dc389e9"><p class="cl-9dc378b9"><span class="cl-9dc24b78">No. birds</span></p></th><th class="cl-9dc389e9"><p class="cl-9dc378b9"><span class="cl-9dc24b78">Mean body mass (g)</span></p></th><th class="cl-9dc389e9"><p class="cl-9dc378b9"><span class="cl-9dc24b78">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9dc389ea"><p class="cl-9dc378b8"><span class="cl-9dc24b78">Adelie</span></p></td><td class="cl-9dc389ea"><p class="cl-9dc378b8"><span class="cl-9dc24b78">female</span></p></td><td class="cl-9dc389eb"><p class="cl-9dc378b9"><span class="cl-9dc24b78">73</span></p></td><td class="cl-9dc389eb"><p class="cl-9dc378b9"><span class="cl-9dc24b78">3,369</span></p></td><td class="cl-9dc389eb"><p class="cl-9dc378b9"><span class="cl-9dc24b78">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">Adelie</span></p></td><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">male</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">73</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">4,043</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">Chinstrap</span></p></td><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">female</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">34</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">3,527</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">Chinstrap</span></p></td><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">male</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">34</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">3,939</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">Gentoo</span></p></td><td class="cl-9dc389f2"><p class="cl-9dc378b8"><span class="cl-9dc24b78">female</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">58</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">4,680</span></p></td><td class="cl-9dc389f3"><p class="cl-9dc378b9"><span class="cl-9dc24b78">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dc389f4"><p class="cl-9dc378b8"><span class="cl-9dc24b78">Gentoo</span></p></td><td class="cl-9dc389f4"><p class="cl-9dc378b8"><span class="cl-9dc24b78">male</span></p></td><td class="cl-9dc389f5"><p class="cl-9dc378b9"><span class="cl-9dc24b78">61</span></p></td><td class="cl-9dc389f5"><p class="cl-9dc378b9"><span class="cl-9dc24b78">5,485</span></p></td><td class="cl-9dc389f5"><p class="cl-9dc378b9"><span class="cl-9dc24b78">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-9dd28916{}.cl-9dc8cad4{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9dc8cad5{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9dcfe4e0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dcfe4ea{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9dd0338c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd03396{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd033a0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd033a1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd033aa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd033ab{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd033ac{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9dd033ad{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9dd28916'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9dd0338c"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad4">species</span></p></th><th class="cl-9dd0338c"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad4">island</span></p></th><th class="cl-9dd03396"><p class="cl-9dcfe4ea"><span class="cl-9dc8cad4">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9dd033a0"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad5">Adelie</span></p></td><td class="cl-9dd033a0"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad5">Torgersen</span></p></td><td class="cl-9dd033a1"><p class="cl-9dcfe4ea"><span class="cl-9dc8cad5">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dd033aa"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad5">Adelie</span></p></td><td class="cl-9dd033aa"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad5">Dream</span></p></td><td class="cl-9dd033ab"><p class="cl-9dcfe4ea"><span class="cl-9dc8cad5">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9dd033ac"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad5">Adelie</span></p></td><td class="cl-9dd033ac"><p class="cl-9dcfe4e0"><span class="cl-9dc8cad5">Dream</span></p></td><td class="cl-9dd033ad"><p class="cl-9dcfe4ea"><span class="cl-9dc8cad5">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-9ddd29d4{}.cl-9dd99daa{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9dd99db4{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9ddb5f64{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9ddb5f65{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9ddb6ffe{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb6fff{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb7008{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb7009{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb700a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb7012{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb7013{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9ddb7014{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9ddd29d4'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9ddb6ffe"><p class="cl-9ddb5f64"><span class="cl-9dd99daa">species</span></p></th><th class="cl-9ddb6ffe"><p class="cl-9ddb5f64"><span class="cl-9dd99daa">sex</span></p></th><th class="cl-9ddb6fff"><p class="cl-9ddb5f65"><span class="cl-9dd99daa">No. birds</span></p></th><th class="cl-9ddb6fff"><p class="cl-9ddb5f65"><span class="cl-9dd99daa">Mean body mass (g)</span></p></th><th class="cl-9ddb6fff"><p class="cl-9ddb5f65"><span class="cl-9dd99daa">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9ddb7008"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">Adelie</span></p></td><td class="cl-9ddb7008"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">female</span></p></td><td class="cl-9ddb7009"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">73</span></p></td><td class="cl-9ddb7009"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">3,368.836</span></p></td><td class="cl-9ddb7009"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">Adelie</span></p></td><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">male</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">73</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">4,043.493</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">Chinstrap</span></p></td><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">female</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">34</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">3,527.206</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">Chinstrap</span></p></td><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">male</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">34</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">3,938.971</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">Gentoo</span></p></td><td class="cl-9ddb700a"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">female</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">58</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">4,679.741</span></p></td><td class="cl-9ddb7012"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9ddb7013"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">Gentoo</span></p></td><td class="cl-9ddb7013"><p class="cl-9ddb5f64"><span class="cl-9dd99db4">male</span></p></td><td class="cl-9ddb7014"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">61</span></p></td><td class="cl-9ddb7014"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">5,484.836</span></p></td><td class="cl-9ddb7014"><p class="cl-9ddb5f65"><span class="cl-9dd99db4">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-9de2cdee{}.cl-9ddf4430{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9ddf443a{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9de07328{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9de07329{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9de0826e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de08278{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de08279{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de08282{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de08283{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de08284{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de0828c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de0828d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9de2cdee'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9de0826e"><p class="cl-9de07328"><span class="cl-9ddf4430">species</span></p></th><th class="cl-9de0826e"><p class="cl-9de07328"><span class="cl-9ddf4430">sex</span></p></th><th class="cl-9de08278"><p class="cl-9de07329"><span class="cl-9ddf4430">No. birds</span></p></th><th class="cl-9de08278"><p class="cl-9de07329"><span class="cl-9ddf4430">Mean body mass (g)</span></p></th><th class="cl-9de08278"><p class="cl-9de07329"><span class="cl-9ddf4430">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-9de08279"><p class="cl-9de07328"><span class="cl-9ddf443a">Adelie</span></p></td><td class="cl-9de08279"><p class="cl-9de07328"><span class="cl-9ddf443a">female</span></p></td><td class="cl-9de08282"><p class="cl-9de07329"><span class="cl-9ddf443a">73</span></p></td><td class="cl-9de08282"><p class="cl-9de07329"><span class="cl-9ddf443a">3369</span></p></td><td class="cl-9de08282"><p class="cl-9de07329"><span class="cl-9ddf443a">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">Adelie</span></p></td><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">male</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">73</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">4043</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">Chinstrap</span></p></td><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">female</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">34</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">3527</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">Chinstrap</span></p></td><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">male</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">34</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">3939</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">Gentoo</span></p></td><td class="cl-9de08283"><p class="cl-9de07328"><span class="cl-9ddf443a">female</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">58</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">4680</span></p></td><td class="cl-9de08284"><p class="cl-9de07329"><span class="cl-9ddf443a">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de0828c"><p class="cl-9de07328"><span class="cl-9ddf443a">Gentoo</span></p></td><td class="cl-9de0828c"><p class="cl-9de07328"><span class="cl-9ddf443a">male</span></p></td><td class="cl-9de0828d"><p class="cl-9de07329"><span class="cl-9ddf443a">61</span></p></td><td class="cl-9de0828d"><p class="cl-9de07329"><span class="cl-9ddf443a">5485</span></p></td><td class="cl-9de0828d"><p class="cl-9de07329"><span class="cl-9ddf443a">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-9de88590{}.cl-9de57580{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9de57581{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-9de6aa0e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9de6aa0f{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-9de6b92c{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b92d{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b92e{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b92f{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b930{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b936{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b937{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-9de6b938{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-9de88590'><thead><tr style="overflow-wrap:break-word;"><th class="cl-9de6b92c"><p class="cl-9de6aa0e"><span class="cl-9de57580">species</span></p></th><th class="cl-9de6b92c"><p class="cl-9de6aa0e"><span class="cl-9de57580">sex</span></p></th><th class="cl-9de6b92d"><p class="cl-9de6aa0f"><span class="cl-9de57580">No. birds</span></p></th><th class="cl-9de6b92d"><p class="cl-9de6aa0f"><span class="cl-9de57580">Mean body mass (g)</span></p></th><th class="cl-9de6b92d"><p class="cl-9de6aa0f"><span class="cl-9de57580">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-9de6b92e"><p class="cl-9de6aa0e"><span class="cl-9de57581">Adelie</span></p></td><td class="cl-9de6b92e"><p class="cl-9de6aa0e"><span class="cl-9de57581">female</span></p></td><td class="cl-9de6b92f"><p class="cl-9de6aa0f"><span class="cl-9de57581">73</span></p></td><td class="cl-9de6b92f"><p class="cl-9de6aa0f"><span class="cl-9de57581">3,368.836</span></p></td><td class="cl-9de6b92f"><p class="cl-9de6aa0f"><span class="cl-9de57581">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de6b930"><p class="cl-9de6aa0e"><span class="cl-9de57581">male</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">73</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">4,043.493</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-9de6b930"><p class="cl-9de6aa0e"><span class="cl-9de57581">Chinstrap</span></p></td><td class="cl-9de6b930"><p class="cl-9de6aa0e"><span class="cl-9de57581">female</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">34</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">3,527.206</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de6b930"><p class="cl-9de6aa0e"><span class="cl-9de57581">male</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">34</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">3,938.971</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-9de6b937"><p class="cl-9de6aa0e"><span class="cl-9de57581">Gentoo</span></p></td><td class="cl-9de6b930"><p class="cl-9de6aa0e"><span class="cl-9de57581">female</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">58</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">4,679.741</span></p></td><td class="cl-9de6b936"><p class="cl-9de6aa0f"><span class="cl-9de57581">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-9de6b937"><p class="cl-9de6aa0e"><span class="cl-9de57581">male</span></p></td><td class="cl-9de6b938"><p class="cl-9de6aa0f"><span class="cl-9de57581">61</span></p></td><td class="cl-9de6b938"><p class="cl-9de6aa0f"><span class="cl-9de57581">5,484.836</span></p></td><td class="cl-9de6b938"><p class="cl-9de6aa0f"><span class="cl-9de57581">313.1586</span></p></td></tr></tbody></table></div>
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
