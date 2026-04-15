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
<div class="tabwid"><style>.cl-35f50740{}.cl-35ef8f04{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-35ef8f18{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-35f0e516{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-35f0e520{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-35f0f4e8{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-35f0f4f2{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-35f0f4f3{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-35f0f4f4{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-35f0f4fc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-35f0f4fd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-35f50740'><thead><tr style="overflow-wrap:break-word;"><th class="cl-35f0f4e8"><p class="cl-35f0e516"><span class="cl-35ef8f04">species</span></p></th><th class="cl-35f0f4e8"><p class="cl-35f0e516"><span class="cl-35ef8f04">sex</span></p></th><th class="cl-35f0f4f2"><p class="cl-35f0e520"><span class="cl-35ef8f04">No. birds</span></p></th><th class="cl-35f0f4f2"><p class="cl-35f0e520"><span class="cl-35ef8f04">Mean body mass (g)</span></p></th><th class="cl-35f0f4f2"><p class="cl-35f0e520"><span class="cl-35ef8f04">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-35f0f4f3"><p class="cl-35f0e516"><span class="cl-35ef8f18">Adelie</span></p></td><td class="cl-35f0f4f3"><p class="cl-35f0e516"><span class="cl-35ef8f18">female</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">73</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">3,369</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-35f0f4fc"><p class="cl-35f0e516"><span class="cl-35ef8f18">Adelie</span></p></td><td class="cl-35f0f4fc"><p class="cl-35f0e516"><span class="cl-35ef8f18">male</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">73</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">4,043</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-35f0f4f3"><p class="cl-35f0e516"><span class="cl-35ef8f18">Chinstrap</span></p></td><td class="cl-35f0f4f3"><p class="cl-35f0e516"><span class="cl-35ef8f18">female</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">34</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">3,527</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-35f0f4fc"><p class="cl-35f0e516"><span class="cl-35ef8f18">Chinstrap</span></p></td><td class="cl-35f0f4fc"><p class="cl-35f0e516"><span class="cl-35ef8f18">male</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">34</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">3,939</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-35f0f4f3"><p class="cl-35f0e516"><span class="cl-35ef8f18">Gentoo</span></p></td><td class="cl-35f0f4f3"><p class="cl-35f0e516"><span class="cl-35ef8f18">female</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">58</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">4,680</span></p></td><td class="cl-35f0f4f4"><p class="cl-35f0e520"><span class="cl-35ef8f18">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-35f0f4fc"><p class="cl-35f0e516"><span class="cl-35ef8f18">Gentoo</span></p></td><td class="cl-35f0f4fc"><p class="cl-35f0e516"><span class="cl-35ef8f18">male</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">61</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">5,485</span></p></td><td class="cl-35f0f4fd"><p class="cl-35f0e520"><span class="cl-35ef8f18">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-360a72f6{}.cl-360783a2{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-360783ac{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3608b77c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3608b77d{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3608c672{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3608c67c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3608c67d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3608c67e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3608c686{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3608c687{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-360a72f6'><thead><tr style="overflow-wrap:break-word;"><th class="cl-3608c672"><p class="cl-3608b77c"><span class="cl-360783a2">species</span></p></th><th class="cl-3608c672"><p class="cl-3608b77c"><span class="cl-360783a2">sex</span></p></th><th class="cl-3608c67c"><p class="cl-3608b77d"><span class="cl-360783a2">No. birds</span></p></th><th class="cl-3608c67c"><p class="cl-3608b77d"><span class="cl-360783a2">Mean body mass (g)</span></p></th><th class="cl-3608c67c"><p class="cl-3608b77d"><span class="cl-360783a2">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">Adelie</span></p></td><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">female</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">73</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">3,369</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">Adelie</span></p></td><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">male</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">73</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">4,043</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">Chinstrap</span></p></td><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">female</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">34</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">3,527</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">Chinstrap</span></p></td><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">male</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">34</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">3,939</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">Gentoo</span></p></td><td class="cl-3608c67d"><p class="cl-3608b77c"><span class="cl-360783ac">female</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">58</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">4,680</span></p></td><td class="cl-3608c67e"><p class="cl-3608b77d"><span class="cl-360783ac">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3608c686"><p class="cl-3608b77c"><span class="cl-360783ac">Gentoo</span></p></td><td class="cl-3608c686"><p class="cl-3608b77c"><span class="cl-360783ac">male</span></p></td><td class="cl-3608c687"><p class="cl-3608b77d"><span class="cl-360783ac">61</span></p></td><td class="cl-3608c687"><p class="cl-3608b77d"><span class="cl-360783ac">5,485</span></p></td><td class="cl-3608c687"><p class="cl-3608b77d"><span class="cl-360783ac">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-360f08d4{}.cl-360c1660{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-360d4b3e{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-360d5cc8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-360d5cd2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-360d5cd3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-360f08d4'><thead><tr style="overflow-wrap:break-word;"><th class="cl-360d5cc8"><p class="cl-360d4b3e"><span class="cl-360c1660">species</span></p></th><th class="cl-360d5cc8"><p class="cl-360d4b3e"><span class="cl-360c1660">sex</span></p></th><th class="cl-360d5cc8"><p class="cl-360d4b3e"><span class="cl-360c1660">No. birds</span></p></th><th class="cl-360d5cc8"><p class="cl-360d4b3e"><span class="cl-360c1660">Mean body mass (g)</span></p></th><th class="cl-360d5cc8"><p class="cl-360d4b3e"><span class="cl-360c1660">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">Adelie</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">female</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">73</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">3,369</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">Adelie</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">male</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">73</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">4,043</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">Chinstrap</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">female</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">34</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">3,527</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">Chinstrap</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">male</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">34</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">3,939</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">Gentoo</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">female</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">58</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">4,680</span></p></td><td class="cl-360d5cd2"><p class="cl-360d4b3e"><span class="cl-360c1660">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-360d5cd3"><p class="cl-360d4b3e"><span class="cl-360c1660">Gentoo</span></p></td><td class="cl-360d5cd3"><p class="cl-360d4b3e"><span class="cl-360c1660">male</span></p></td><td class="cl-360d5cd3"><p class="cl-360d4b3e"><span class="cl-360c1660">61</span></p></td><td class="cl-360d5cd3"><p class="cl-360d4b3e"><span class="cl-360c1660">5,485</span></p></td><td class="cl-360d5cd3"><p class="cl-360d4b3e"><span class="cl-360c1660">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-3613c0f4{}.cl-3610a784{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3611f03a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3611f044{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-36120020{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-36120021{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3612002a{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3612002b{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-36120034{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-36120035{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-36120036{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-36120037{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-3613c0f4'><thead><tr style="overflow-wrap:break-word;"><th class="cl-36120020"><p class="cl-3611f03a"><span class="cl-3610a784">species</span></p></th><th class="cl-36120020"><p class="cl-3611f03a"><span class="cl-3610a784">sex</span></p></th><th class="cl-36120021"><p class="cl-3611f044"><span class="cl-3610a784">No. birds</span></p></th><th class="cl-36120021"><p class="cl-3611f044"><span class="cl-3610a784">Mean body mass (g)</span></p></th><th class="cl-36120021"><p class="cl-3611f044"><span class="cl-3610a784">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3612002a"><p class="cl-3611f03a"><span class="cl-3610a784">Adelie</span></p></td><td class="cl-3612002a"><p class="cl-3611f03a"><span class="cl-3610a784">female</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">73</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">3,369</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36120034"><p class="cl-3611f03a"><span class="cl-3610a784">Adelie</span></p></td><td class="cl-36120034"><p class="cl-3611f03a"><span class="cl-3610a784">male</span></p></td><td class="cl-36120035"><p class="cl-3611f044"><span class="cl-3610a784">73</span></p></td><td class="cl-36120035"><p class="cl-3611f044"><span class="cl-3610a784">4,043</span></p></td><td class="cl-36120035"><p class="cl-3611f044"><span class="cl-3610a784">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3612002a"><p class="cl-3611f03a"><span class="cl-3610a784">Chinstrap</span></p></td><td class="cl-3612002a"><p class="cl-3611f03a"><span class="cl-3610a784">female</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">34</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">3,527</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36120034"><p class="cl-3611f03a"><span class="cl-3610a784">Chinstrap</span></p></td><td class="cl-36120034"><p class="cl-3611f03a"><span class="cl-3610a784">male</span></p></td><td class="cl-36120035"><p class="cl-3611f044"><span class="cl-3610a784">34</span></p></td><td class="cl-36120035"><p class="cl-3611f044"><span class="cl-3610a784">3,939</span></p></td><td class="cl-36120035"><p class="cl-3611f044"><span class="cl-3610a784">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3612002a"><p class="cl-3611f03a"><span class="cl-3610a784">Gentoo</span></p></td><td class="cl-3612002a"><p class="cl-3611f03a"><span class="cl-3610a784">female</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">58</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">4,680</span></p></td><td class="cl-3612002b"><p class="cl-3611f044"><span class="cl-3610a784">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36120036"><p class="cl-3611f03a"><span class="cl-3610a784">Gentoo</span></p></td><td class="cl-36120036"><p class="cl-3611f03a"><span class="cl-3610a784">male</span></p></td><td class="cl-36120037"><p class="cl-3611f044"><span class="cl-3610a784">61</span></p></td><td class="cl-36120037"><p class="cl-3611f044"><span class="cl-3610a784">5,485</span></p></td><td class="cl-36120037"><p class="cl-3611f044"><span class="cl-3610a784">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-36196bf8{}.cl-36169fcc{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3617c140{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3617c141{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3617d018{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d022{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d023{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d024{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d025{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d02c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d02d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3617d02e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-36196bf8'><thead><tr style="overflow-wrap:break-word;"><th class="cl-3617d018"><p class="cl-3617c140"><span class="cl-36169fcc">species</span></p></th><th class="cl-3617d018"><p class="cl-3617c140"><span class="cl-36169fcc">sex</span></p></th><th class="cl-3617d022"><p class="cl-3617c141"><span class="cl-36169fcc">No. birds</span></p></th><th class="cl-3617d022"><p class="cl-3617c141"><span class="cl-36169fcc">Mean body mass (g)</span></p></th><th class="cl-3617d022"><p class="cl-3617c141"><span class="cl-36169fcc">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3617d023"><p class="cl-3617c140"><span class="cl-36169fcc">Adelie</span></p></td><td class="cl-3617d023"><p class="cl-3617c140"><span class="cl-36169fcc">female</span></p></td><td class="cl-3617d024"><p class="cl-3617c141"><span class="cl-36169fcc">73</span></p></td><td class="cl-3617d024"><p class="cl-3617c141"><span class="cl-36169fcc">3,369</span></p></td><td class="cl-3617d024"><p class="cl-3617c141"><span class="cl-36169fcc">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3617d023"><p class="cl-3617c140"><span class="cl-36169fcc">Adelie</span></p></td><td class="cl-3617d023"><p class="cl-3617c140"><span class="cl-36169fcc">male</span></p></td><td class="cl-3617d024"><p class="cl-3617c141"><span class="cl-36169fcc">73</span></p></td><td class="cl-3617d024"><p class="cl-3617c141"><span class="cl-36169fcc">4,043</span></p></td><td class="cl-3617d024"><p class="cl-3617c141"><span class="cl-36169fcc">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3617d025"><p class="cl-3617c140"><span class="cl-36169fcc">Chinstrap</span></p></td><td class="cl-3617d025"><p class="cl-3617c140"><span class="cl-36169fcc">female</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">34</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">3,527</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3617d025"><p class="cl-3617c140"><span class="cl-36169fcc">Chinstrap</span></p></td><td class="cl-3617d025"><p class="cl-3617c140"><span class="cl-36169fcc">male</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">34</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">3,939</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3617d025"><p class="cl-3617c140"><span class="cl-36169fcc">Gentoo</span></p></td><td class="cl-3617d025"><p class="cl-3617c140"><span class="cl-36169fcc">female</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">58</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">4,680</span></p></td><td class="cl-3617d02c"><p class="cl-3617c141"><span class="cl-36169fcc">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3617d02d"><p class="cl-3617c140"><span class="cl-36169fcc">Gentoo</span></p></td><td class="cl-3617d02d"><p class="cl-3617c140"><span class="cl-36169fcc">male</span></p></td><td class="cl-3617d02e"><p class="cl-3617c141"><span class="cl-36169fcc">61</span></p></td><td class="cl-3617d02e"><p class="cl-3617c141"><span class="cl-36169fcc">5,485</span></p></td><td class="cl-3617d02e"><p class="cl-3617c141"><span class="cl-36169fcc">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-361e1248{}.cl-361b5fda{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-361b5fe4{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-361b5fe5{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-361c7bb8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-361c7bc2{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-361c8a72{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-361c8a73{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-361c8a7c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-361c8a7d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-361c8a7e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-361c8a86{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-361e1248'><thead><tr style="overflow-wrap:break-word;"><th class="cl-361c8a72"><p class="cl-361c7bb8"><span class="cl-361b5fda">species</span></p></th><th class="cl-361c8a72"><p class="cl-361c7bb8"><span class="cl-361b5fda">sex</span></p></th><th class="cl-361c8a73"><p class="cl-361c7bc2"><span class="cl-361b5fda">No. birds</span></p></th><th class="cl-361c8a73"><p class="cl-361c7bc2"><span class="cl-361b5fda">Mean body mass (g)</span></p></th><th class="cl-361c8a73"><p class="cl-361c7bc2"><span class="cl-361b5fda">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe4">Adelie</span></p></td><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe5">female</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">73</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">3,369</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe4">Adelie</span></p></td><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe5">male</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">73</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">4,043</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe4">Chinstrap</span></p></td><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe5">female</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">34</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">3,527</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe4">Chinstrap</span></p></td><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe5">male</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">34</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">3,939</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe4">Gentoo</span></p></td><td class="cl-361c8a7c"><p class="cl-361c7bb8"><span class="cl-361b5fe5">female</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">58</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">4,680</span></p></td><td class="cl-361c8a7d"><p class="cl-361c7bc2"><span class="cl-361b5fe5">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-361c8a7e"><p class="cl-361c7bb8"><span class="cl-361b5fe4">Gentoo</span></p></td><td class="cl-361c8a7e"><p class="cl-361c7bb8"><span class="cl-361b5fe5">male</span></p></td><td class="cl-361c8a86"><p class="cl-361c7bc2"><span class="cl-361b5fe5">61</span></p></td><td class="cl-361c8a86"><p class="cl-361c7bc2"><span class="cl-361b5fe5">5,485</span></p></td><td class="cl-361c8a86"><p class="cl-361c7bc2"><span class="cl-361b5fe5">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-3622aa92{}.cl-361ffcf2{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-36210e58{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-36210e59{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-36211c04{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-36211c05{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-3622aa92'><thead><tr style="overflow-wrap:break-word;"><th class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">species</span></p></th><th class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">sex</span></p></th><th class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">No. birds</span></p></th><th class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">Mean body mass (g)</span></p></th><th class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">Adelie</span></p></td><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">female</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">73</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">3,369</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">Adelie</span></p></td><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">male</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">73</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">4,043</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">Chinstrap</span></p></td><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">female</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">34</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">3,527</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">Chinstrap</span></p></td><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">male</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">34</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">3,939</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">Gentoo</span></p></td><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">female</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">58</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">4,680</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">Gentoo</span></p></td><td class="cl-36211c04"><p class="cl-36210e58"><span class="cl-361ffcf2">male</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">61</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">5,485</span></p></td><td class="cl-36211c05"><p class="cl-36210e59"><span class="cl-361ffcf2">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-36294596{}.cl-36268fea{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3627a81c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3627a826{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3627b5fa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b5fb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b5fc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b5fd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b604{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b605{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b606{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b60e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b60f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b610{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b611{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3627b618{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-36294596'><thead><tr style="overflow-wrap:break-word;"><th class="cl-3627b5fa"><p class="cl-3627a81c"><span class="cl-36268fea">species</span></p></th><th class="cl-3627b5fb"><p class="cl-3627a81c"><span class="cl-36268fea">sex</span></p></th><th class="cl-3627b5fc"><p class="cl-3627a826"><span class="cl-36268fea">No. birds</span></p></th><th class="cl-3627b5fc"><p class="cl-3627a826"><span class="cl-36268fea">Mean body mass (g)</span></p></th><th class="cl-3627b5fd"><p class="cl-3627a826"><span class="cl-36268fea">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3627b604"><p class="cl-3627a81c"><span class="cl-36268fea">Adelie</span></p></td><td class="cl-3627b605"><p class="cl-3627a81c"><span class="cl-36268fea">female</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">73</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">3,369</span></p></td><td class="cl-3627b60e"><p class="cl-3627a826"><span class="cl-36268fea">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3627b604"><p class="cl-3627a81c"><span class="cl-36268fea">Adelie</span></p></td><td class="cl-3627b605"><p class="cl-3627a81c"><span class="cl-36268fea">male</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">73</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">4,043</span></p></td><td class="cl-3627b60e"><p class="cl-3627a826"><span class="cl-36268fea">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3627b604"><p class="cl-3627a81c"><span class="cl-36268fea">Chinstrap</span></p></td><td class="cl-3627b605"><p class="cl-3627a81c"><span class="cl-36268fea">female</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">34</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">3,527</span></p></td><td class="cl-3627b60e"><p class="cl-3627a826"><span class="cl-36268fea">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3627b604"><p class="cl-3627a81c"><span class="cl-36268fea">Chinstrap</span></p></td><td class="cl-3627b605"><p class="cl-3627a81c"><span class="cl-36268fea">male</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">34</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">3,939</span></p></td><td class="cl-3627b60e"><p class="cl-3627a826"><span class="cl-36268fea">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3627b604"><p class="cl-3627a81c"><span class="cl-36268fea">Gentoo</span></p></td><td class="cl-3627b605"><p class="cl-3627a81c"><span class="cl-36268fea">female</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">58</span></p></td><td class="cl-3627b606"><p class="cl-3627a826"><span class="cl-36268fea">4,680</span></p></td><td class="cl-3627b60e"><p class="cl-3627a826"><span class="cl-36268fea">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3627b60f"><p class="cl-3627a81c"><span class="cl-36268fea">Gentoo</span></p></td><td class="cl-3627b610"><p class="cl-3627a81c"><span class="cl-36268fea">male</span></p></td><td class="cl-3627b611"><p class="cl-3627a826"><span class="cl-36268fea">61</span></p></td><td class="cl-3627b611"><p class="cl-3627a826"><span class="cl-36268fea">5,485</span></p></td><td class="cl-3627b618"><p class="cl-3627a826"><span class="cl-36268fea">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-362d6608{}.cl-362ab728{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-362bcbf4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-362bcbfe{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-362bda04{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda05{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda0e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda0f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda10{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda11{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda12{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda18{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda19{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362bda1a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-362d6608'><thead><tr style="overflow-wrap:break-word;"><th class="cl-362bda04"><p class="cl-362bcbf4"><span class="cl-362ab728">species</span></p></th><th class="cl-362bda04"><p class="cl-362bcbf4"><span class="cl-362ab728">sex</span></p></th><th class="cl-362bda05"><p class="cl-362bcbfe"><span class="cl-362ab728">No. birds</span></p></th><th class="cl-362bda05"><p class="cl-362bcbfe"><span class="cl-362ab728">Mean body mass (g)</span></p></th><th class="cl-362bda05"><p class="cl-362bcbfe"><span class="cl-362ab728">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-362bda0e"><p class="cl-362bcbf4"><span class="cl-362ab728">Adelie</span></p></td><td class="cl-362bda0f"><p class="cl-362bcbf4"><span class="cl-362ab728">female</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">73</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">3,369</span></p></td><td class="cl-362bda11"><p class="cl-362bcbfe"><span class="cl-362ab728">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362bda0e"><p class="cl-362bcbf4"><span class="cl-362ab728">Adelie</span></p></td><td class="cl-362bda0f"><p class="cl-362bcbf4"><span class="cl-362ab728">male</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">73</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">4,043</span></p></td><td class="cl-362bda11"><p class="cl-362bcbfe"><span class="cl-362ab728">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362bda0e"><p class="cl-362bcbf4"><span class="cl-362ab728">Chinstrap</span></p></td><td class="cl-362bda0f"><p class="cl-362bcbf4"><span class="cl-362ab728">female</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">34</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">3,527</span></p></td><td class="cl-362bda11"><p class="cl-362bcbfe"><span class="cl-362ab728">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362bda0e"><p class="cl-362bcbf4"><span class="cl-362ab728">Chinstrap</span></p></td><td class="cl-362bda0f"><p class="cl-362bcbf4"><span class="cl-362ab728">male</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">34</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">3,939</span></p></td><td class="cl-362bda11"><p class="cl-362bcbfe"><span class="cl-362ab728">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362bda0e"><p class="cl-362bcbf4"><span class="cl-362ab728">Gentoo</span></p></td><td class="cl-362bda0f"><p class="cl-362bcbf4"><span class="cl-362ab728">female</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">58</span></p></td><td class="cl-362bda10"><p class="cl-362bcbfe"><span class="cl-362ab728">4,680</span></p></td><td class="cl-362bda11"><p class="cl-362bcbfe"><span class="cl-362ab728">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362bda12"><p class="cl-362bcbf4"><span class="cl-362ab728">Gentoo</span></p></td><td class="cl-362bda18"><p class="cl-362bcbf4"><span class="cl-362ab728">male</span></p></td><td class="cl-362bda19"><p class="cl-362bcbfe"><span class="cl-362ab728">61</span></p></td><td class="cl-362bda19"><p class="cl-362bcbfe"><span class="cl-362ab728">5,485</span></p></td><td class="cl-362bda1a"><p class="cl-362bcbfe"><span class="cl-362ab728">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-36316730{}.cl-362ec1ba{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-362fd942{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-362fd943{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-362fe630{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe63a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe63b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe63c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe63d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe644{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe645{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-362fe646{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-36316730'><thead><tr style="overflow-wrap:break-word;"><th class="cl-362fe630"><p class="cl-362fd942"><span class="cl-362ec1ba">species</span></p></th><th class="cl-362fe630"><p class="cl-362fd942"><span class="cl-362ec1ba">sex</span></p></th><th class="cl-362fe63a"><p class="cl-362fd943"><span class="cl-362ec1ba">No. birds</span></p></th><th class="cl-362fe63a"><p class="cl-362fd943"><span class="cl-362ec1ba">Mean body mass (g)</span></p></th><th class="cl-362fe63a"><p class="cl-362fd943"><span class="cl-362ec1ba">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-362fe63b"><p class="cl-362fd942"><span class="cl-362ec1ba">Adelie</span></p></td><td class="cl-362fe63b"><p class="cl-362fd942"><span class="cl-362ec1ba">female</span></p></td><td class="cl-362fe63c"><p class="cl-362fd943"><span class="cl-362ec1ba">73</span></p></td><td class="cl-362fe63c"><p class="cl-362fd943"><span class="cl-362ec1ba">3,369</span></p></td><td class="cl-362fe63c"><p class="cl-362fd943"><span class="cl-362ec1ba">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">Adelie</span></p></td><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">male</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">73</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">4,043</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">Chinstrap</span></p></td><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">female</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">34</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">3,527</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">Chinstrap</span></p></td><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">male</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">34</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">3,939</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">Gentoo</span></p></td><td class="cl-362fe63d"><p class="cl-362fd942"><span class="cl-362ec1ba">female</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">58</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">4,680</span></p></td><td class="cl-362fe644"><p class="cl-362fd943"><span class="cl-362ec1ba">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-362fe645"><p class="cl-362fd942"><span class="cl-362ec1ba">Gentoo</span></p></td><td class="cl-362fe645"><p class="cl-362fd942"><span class="cl-362ec1ba">male</span></p></td><td class="cl-362fe646"><p class="cl-362fd943"><span class="cl-362ec1ba">61</span></p></td><td class="cl-362fe646"><p class="cl-362fd943"><span class="cl-362ec1ba">5,485</span></p></td><td class="cl-362fe646"><p class="cl-362fd943"><span class="cl-362ec1ba">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-36381c10{}.cl-36358dba{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-36358dc4{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-36369da4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-36369dae{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3636aad8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aad9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aae2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aae3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aae4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aae5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aae6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3636aaec{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-36381c10'><thead><tr style="overflow-wrap:break-word;"><th class="cl-3636aad8"><p class="cl-36369da4"><span class="cl-36358dba">species</span></p></th><th class="cl-3636aad8"><p class="cl-36369da4"><span class="cl-36358dba">island</span></p></th><th class="cl-3636aad9"><p class="cl-36369dae"><span class="cl-36358dba">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3636aae2"><p class="cl-36369da4"><span class="cl-36358dc4">Adelie</span></p></td><td class="cl-3636aae2"><p class="cl-36369da4"><span class="cl-36358dc4">Torgersen</span></p></td><td class="cl-3636aae3"><p class="cl-36369dae"><span class="cl-36358dc4">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3636aae4"><p class="cl-36369da4"><span class="cl-36358dc4">Adelie</span></p></td><td class="cl-3636aae4"><p class="cl-36369da4"><span class="cl-36358dc4">Dream</span></p></td><td class="cl-3636aae5"><p class="cl-36369dae"><span class="cl-36358dc4">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3636aae6"><p class="cl-36369da4"><span class="cl-36358dc4">Adelie</span></p></td><td class="cl-3636aae6"><p class="cl-36369da4"><span class="cl-36358dc4">Dream</span></p></td><td class="cl-3636aaec"><p class="cl-36369dae"><span class="cl-36358dc4">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-363f6056{}.cl-363caa8c{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-363caa96{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-363dc7a0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-363dc7a1{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-363dd560{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd56a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd56b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd56c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd574{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd575{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd576{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-363dd577{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-363f6056'><thead><tr style="overflow-wrap:break-word;"><th class="cl-363dd560"><p class="cl-363dc7a0"><span class="cl-363caa8c">species</span></p></th><th class="cl-363dd560"><p class="cl-363dc7a0"><span class="cl-363caa8c">sex</span></p></th><th class="cl-363dd56a"><p class="cl-363dc7a1"><span class="cl-363caa8c">No. birds</span></p></th><th class="cl-363dd56a"><p class="cl-363dc7a1"><span class="cl-363caa8c">Mean body mass (g)</span></p></th><th class="cl-363dd56a"><p class="cl-363dc7a1"><span class="cl-363caa8c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-363dd56b"><p class="cl-363dc7a0"><span class="cl-363caa96">Adelie</span></p></td><td class="cl-363dd56b"><p class="cl-363dc7a0"><span class="cl-363caa96">female</span></p></td><td class="cl-363dd56c"><p class="cl-363dc7a1"><span class="cl-363caa96">73</span></p></td><td class="cl-363dd56c"><p class="cl-363dc7a1"><span class="cl-363caa96">3,368.836</span></p></td><td class="cl-363dd56c"><p class="cl-363dc7a1"><span class="cl-363caa96">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">Adelie</span></p></td><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">male</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">73</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">4,043.493</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">Chinstrap</span></p></td><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">female</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">34</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">3,527.206</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">Chinstrap</span></p></td><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">male</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">34</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">3,938.971</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">Gentoo</span></p></td><td class="cl-363dd574"><p class="cl-363dc7a0"><span class="cl-363caa96">female</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">58</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">4,679.741</span></p></td><td class="cl-363dd575"><p class="cl-363dc7a1"><span class="cl-363caa96">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-363dd576"><p class="cl-363dc7a0"><span class="cl-363caa96">Gentoo</span></p></td><td class="cl-363dd576"><p class="cl-363dc7a0"><span class="cl-363caa96">male</span></p></td><td class="cl-363dd577"><p class="cl-363dc7a1"><span class="cl-363caa96">61</span></p></td><td class="cl-363dd577"><p class="cl-363dc7a1"><span class="cl-363caa96">5,484.836</span></p></td><td class="cl-363dd577"><p class="cl-363dc7a1"><span class="cl-363caa96">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-3644610a{}.cl-36413674{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-36413675{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3642d876{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3642d877{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3642e5b4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5b5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5b6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5b7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5be{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5bf{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5c0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3642e5c1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-3644610a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-3642e5b4"><p class="cl-3642d876"><span class="cl-36413674">species</span></p></th><th class="cl-3642e5b4"><p class="cl-3642d876"><span class="cl-36413674">sex</span></p></th><th class="cl-3642e5b5"><p class="cl-3642d877"><span class="cl-36413674">No. birds</span></p></th><th class="cl-3642e5b5"><p class="cl-3642d877"><span class="cl-36413674">Mean body mass (g)</span></p></th><th class="cl-3642e5b5"><p class="cl-3642d877"><span class="cl-36413674">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3642e5b6"><p class="cl-3642d876"><span class="cl-36413675">Adelie</span></p></td><td class="cl-3642e5b6"><p class="cl-3642d876"><span class="cl-36413675">female</span></p></td><td class="cl-3642e5b7"><p class="cl-3642d877"><span class="cl-36413675">73</span></p></td><td class="cl-3642e5b7"><p class="cl-3642d877"><span class="cl-36413675">3369</span></p></td><td class="cl-3642e5b7"><p class="cl-3642d877"><span class="cl-36413675">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">Adelie</span></p></td><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">male</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">73</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">4043</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">Chinstrap</span></p></td><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">female</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">34</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">3527</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">Chinstrap</span></p></td><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">male</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">34</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">3939</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">Gentoo</span></p></td><td class="cl-3642e5be"><p class="cl-3642d876"><span class="cl-36413675">female</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">58</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">4680</span></p></td><td class="cl-3642e5bf"><p class="cl-3642d877"><span class="cl-36413675">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3642e5c0"><p class="cl-3642d876"><span class="cl-36413675">Gentoo</span></p></td><td class="cl-3642e5c0"><p class="cl-3642d876"><span class="cl-36413675">male</span></p></td><td class="cl-3642e5c1"><p class="cl-3642d877"><span class="cl-36413675">61</span></p></td><td class="cl-3642e5c1"><p class="cl-3642d877"><span class="cl-36413675">5485</span></p></td><td class="cl-3642e5c1"><p class="cl-3642d877"><span class="cl-36413675">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-3649548a{}.cl-3646b338{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3646b339{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3647c9da{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3647c9db{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3647d7d6{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7d7{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7d8{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7e0{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7e1{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7e2{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7e3{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3647d7e4{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-3649548a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-3647d7d6"><p class="cl-3647c9da"><span class="cl-3646b338">species</span></p></th><th class="cl-3647d7d6"><p class="cl-3647c9da"><span class="cl-3646b338">sex</span></p></th><th class="cl-3647d7d7"><p class="cl-3647c9db"><span class="cl-3646b338">No. birds</span></p></th><th class="cl-3647d7d7"><p class="cl-3647c9db"><span class="cl-3646b338">Mean body mass (g)</span></p></th><th class="cl-3647d7d7"><p class="cl-3647c9db"><span class="cl-3646b338">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-3647d7d8"><p class="cl-3647c9da"><span class="cl-3646b339">Adelie</span></p></td><td class="cl-3647d7d8"><p class="cl-3647c9da"><span class="cl-3646b339">female</span></p></td><td class="cl-3647d7e0"><p class="cl-3647c9db"><span class="cl-3646b339">73</span></p></td><td class="cl-3647d7e0"><p class="cl-3647c9db"><span class="cl-3646b339">3,368.836</span></p></td><td class="cl-3647d7e0"><p class="cl-3647c9db"><span class="cl-3646b339">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3647d7e1"><p class="cl-3647c9da"><span class="cl-3646b339">male</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">73</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">4,043.493</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-3647d7e1"><p class="cl-3647c9da"><span class="cl-3646b339">Chinstrap</span></p></td><td class="cl-3647d7e1"><p class="cl-3647c9da"><span class="cl-3646b339">female</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">34</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">3,527.206</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3647d7e1"><p class="cl-3647c9da"><span class="cl-3646b339">male</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">34</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">3,938.971</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-3647d7e3"><p class="cl-3647c9da"><span class="cl-3646b339">Gentoo</span></p></td><td class="cl-3647d7e1"><p class="cl-3647c9da"><span class="cl-3646b339">female</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">58</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">4,679.741</span></p></td><td class="cl-3647d7e2"><p class="cl-3647c9db"><span class="cl-3646b339">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3647d7e3"><p class="cl-3647c9da"><span class="cl-3646b339">male</span></p></td><td class="cl-3647d7e4"><p class="cl-3647c9db"><span class="cl-3646b339">61</span></p></td><td class="cl-3647d7e4"><p class="cl-3647c9db"><span class="cl-3646b339">5,484.836</span></p></td><td class="cl-3647d7e4"><p class="cl-3647c9db"><span class="cl-3646b339">313.1586</span></p></td></tr></tbody></table></div>
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
