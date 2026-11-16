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
<div class="tabwid"><style>.cl-b4aa7510{}.cl-b4a65c28{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4a65c32{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4a876c0{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4a876ca{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4a888a4{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4a888a5{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4a888a6{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4a888a7{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4a888ae{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4a888b8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4aa7510'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4a888a4"><p class="cl-b4a876c0"><span class="cl-b4a65c28">species</span></p></th><th class="cl-b4a888a4"><p class="cl-b4a876c0"><span class="cl-b4a65c28">sex</span></p></th><th class="cl-b4a888a5"><p class="cl-b4a876ca"><span class="cl-b4a65c28">No. birds</span></p></th><th class="cl-b4a888a5"><p class="cl-b4a876ca"><span class="cl-b4a65c28">Mean body mass (g)</span></p></th><th class="cl-b4a888a5"><p class="cl-b4a876ca"><span class="cl-b4a65c28">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4a888a6"><p class="cl-b4a876c0"><span class="cl-b4a65c32">Adelie</span></p></td><td class="cl-b4a888a6"><p class="cl-b4a876c0"><span class="cl-b4a65c32">female</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">73</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">3,369</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4a888ae"><p class="cl-b4a876c0"><span class="cl-b4a65c32">Adelie</span></p></td><td class="cl-b4a888ae"><p class="cl-b4a876c0"><span class="cl-b4a65c32">male</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">73</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">4,043</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4a888a6"><p class="cl-b4a876c0"><span class="cl-b4a65c32">Chinstrap</span></p></td><td class="cl-b4a888a6"><p class="cl-b4a876c0"><span class="cl-b4a65c32">female</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">34</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">3,527</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4a888ae"><p class="cl-b4a876c0"><span class="cl-b4a65c32">Chinstrap</span></p></td><td class="cl-b4a888ae"><p class="cl-b4a876c0"><span class="cl-b4a65c32">male</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">34</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">3,939</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4a888a6"><p class="cl-b4a876c0"><span class="cl-b4a65c32">Gentoo</span></p></td><td class="cl-b4a888a6"><p class="cl-b4a876c0"><span class="cl-b4a65c32">female</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">58</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">4,680</span></p></td><td class="cl-b4a888a7"><p class="cl-b4a876ca"><span class="cl-b4a65c32">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4a888ae"><p class="cl-b4a876c0"><span class="cl-b4a65c32">Gentoo</span></p></td><td class="cl-b4a888ae"><p class="cl-b4a876c0"><span class="cl-b4a65c32">male</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">61</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">5,485</span></p></td><td class="cl-b4a888b8"><p class="cl-b4a876ca"><span class="cl-b4a65c32">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-b4c62c38{}.cl-b4c2fa90{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4c2fa91{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4c44b02{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4c44b0c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4c45de0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c45de1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c45de2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c45dea{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c45df4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c45df5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4c62c38'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4c45de0"><p class="cl-b4c44b02"><span class="cl-b4c2fa90">species</span></p></th><th class="cl-b4c45de0"><p class="cl-b4c44b02"><span class="cl-b4c2fa90">sex</span></p></th><th class="cl-b4c45de1"><p class="cl-b4c44b0c"><span class="cl-b4c2fa90">No. birds</span></p></th><th class="cl-b4c45de1"><p class="cl-b4c44b0c"><span class="cl-b4c2fa90">Mean body mass (g)</span></p></th><th class="cl-b4c45de1"><p class="cl-b4c44b0c"><span class="cl-b4c2fa90">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">Adelie</span></p></td><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">female</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">73</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">3,369</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">Adelie</span></p></td><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">male</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">73</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">4,043</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">Chinstrap</span></p></td><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">female</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">34</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">3,527</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">Chinstrap</span></p></td><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">male</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">34</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">3,939</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">Gentoo</span></p></td><td class="cl-b4c45de2"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">female</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">58</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">4,680</span></p></td><td class="cl-b4c45dea"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c45df4"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">Gentoo</span></p></td><td class="cl-b4c45df4"><p class="cl-b4c44b02"><span class="cl-b4c2fa91">male</span></p></td><td class="cl-b4c45df5"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">61</span></p></td><td class="cl-b4c45df5"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">5,485</span></p></td><td class="cl-b4c45df5"><p class="cl-b4c44b0c"><span class="cl-b4c2fa91">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-b4cb285a{}.cl-b4c7f05e{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4c941ca{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-b4c953a4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c953ae{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4c953af{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4cb285a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4c953a4"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">species</span></p></th><th class="cl-b4c953a4"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">sex</span></p></th><th class="cl-b4c953a4"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">No. birds</span></p></th><th class="cl-b4c953a4"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Mean body mass (g)</span></p></th><th class="cl-b4c953a4"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Adelie</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">female</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">73</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">3,369</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Adelie</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">male</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">73</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">4,043</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Chinstrap</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">female</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">34</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">3,527</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Chinstrap</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">male</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">34</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">3,939</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Gentoo</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">female</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">58</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">4,680</span></p></td><td class="cl-b4c953ae"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4c953af"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">Gentoo</span></p></td><td class="cl-b4c953af"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">male</span></p></td><td class="cl-b4c953af"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">61</span></p></td><td class="cl-b4c953af"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">5,485</span></p></td><td class="cl-b4c953af"><p class="cl-b4c941ca"><span class="cl-b4c7f05e">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-b4d000f0{}.cl-b4ccd3a8{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4ce217c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4ce2186{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4ce33d8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33e2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33e3{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33ec{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33ed{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33f6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33f7{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ce33f8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4d000f0'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4ce33d8"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">species</span></p></th><th class="cl-b4ce33d8"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">sex</span></p></th><th class="cl-b4ce33e2"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">No. birds</span></p></th><th class="cl-b4ce33e2"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">Mean body mass (g)</span></p></th><th class="cl-b4ce33e2"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4ce33e3"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">Adelie</span></p></td><td class="cl-b4ce33e3"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">female</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">73</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">3,369</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ce33ed"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">Adelie</span></p></td><td class="cl-b4ce33ed"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">male</span></p></td><td class="cl-b4ce33f6"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">73</span></p></td><td class="cl-b4ce33f6"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">4,043</span></p></td><td class="cl-b4ce33f6"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ce33e3"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">Chinstrap</span></p></td><td class="cl-b4ce33e3"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">female</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">34</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">3,527</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ce33ed"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">Chinstrap</span></p></td><td class="cl-b4ce33ed"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">male</span></p></td><td class="cl-b4ce33f6"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">34</span></p></td><td class="cl-b4ce33f6"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">3,939</span></p></td><td class="cl-b4ce33f6"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ce33e3"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">Gentoo</span></p></td><td class="cl-b4ce33e3"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">female</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">58</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">4,680</span></p></td><td class="cl-b4ce33ec"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ce33f7"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">Gentoo</span></p></td><td class="cl-b4ce33f7"><p class="cl-b4ce217c"><span class="cl-b4ccd3a8">male</span></p></td><td class="cl-b4ce33f8"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">61</span></p></td><td class="cl-b4ce33f8"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">5,485</span></p></td><td class="cl-b4ce33f8"><p class="cl-b4ce2186"><span class="cl-b4ccd3a8">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-b4d625c0{}.cl-b4d338e2{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4d46154{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4d4615e{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4d46f78{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f79{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f7a{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f82{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f83{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f84{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f85{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d46f86{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4d625c0'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4d46f78"><p class="cl-b4d46154"><span class="cl-b4d338e2">species</span></p></th><th class="cl-b4d46f78"><p class="cl-b4d46154"><span class="cl-b4d338e2">sex</span></p></th><th class="cl-b4d46f79"><p class="cl-b4d4615e"><span class="cl-b4d338e2">No. birds</span></p></th><th class="cl-b4d46f79"><p class="cl-b4d4615e"><span class="cl-b4d338e2">Mean body mass (g)</span></p></th><th class="cl-b4d46f79"><p class="cl-b4d4615e"><span class="cl-b4d338e2">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4d46f7a"><p class="cl-b4d46154"><span class="cl-b4d338e2">Adelie</span></p></td><td class="cl-b4d46f7a"><p class="cl-b4d46154"><span class="cl-b4d338e2">female</span></p></td><td class="cl-b4d46f82"><p class="cl-b4d4615e"><span class="cl-b4d338e2">73</span></p></td><td class="cl-b4d46f82"><p class="cl-b4d4615e"><span class="cl-b4d338e2">3,369</span></p></td><td class="cl-b4d46f82"><p class="cl-b4d4615e"><span class="cl-b4d338e2">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d46f7a"><p class="cl-b4d46154"><span class="cl-b4d338e2">Adelie</span></p></td><td class="cl-b4d46f7a"><p class="cl-b4d46154"><span class="cl-b4d338e2">male</span></p></td><td class="cl-b4d46f82"><p class="cl-b4d4615e"><span class="cl-b4d338e2">73</span></p></td><td class="cl-b4d46f82"><p class="cl-b4d4615e"><span class="cl-b4d338e2">4,043</span></p></td><td class="cl-b4d46f82"><p class="cl-b4d4615e"><span class="cl-b4d338e2">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d46f83"><p class="cl-b4d46154"><span class="cl-b4d338e2">Chinstrap</span></p></td><td class="cl-b4d46f83"><p class="cl-b4d46154"><span class="cl-b4d338e2">female</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">34</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">3,527</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d46f83"><p class="cl-b4d46154"><span class="cl-b4d338e2">Chinstrap</span></p></td><td class="cl-b4d46f83"><p class="cl-b4d46154"><span class="cl-b4d338e2">male</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">34</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">3,939</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d46f83"><p class="cl-b4d46154"><span class="cl-b4d338e2">Gentoo</span></p></td><td class="cl-b4d46f83"><p class="cl-b4d46154"><span class="cl-b4d338e2">female</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">58</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">4,680</span></p></td><td class="cl-b4d46f84"><p class="cl-b4d4615e"><span class="cl-b4d338e2">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d46f85"><p class="cl-b4d46154"><span class="cl-b4d338e2">Gentoo</span></p></td><td class="cl-b4d46f85"><p class="cl-b4d46154"><span class="cl-b4d338e2">male</span></p></td><td class="cl-b4d46f86"><p class="cl-b4d4615e"><span class="cl-b4d338e2">61</span></p></td><td class="cl-b4d46f86"><p class="cl-b4d4615e"><span class="cl-b4d338e2">5,485</span></p></td><td class="cl-b4d46f86"><p class="cl-b4d4615e"><span class="cl-b4d338e2">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-b4db0766{}.cl-b4d820fa{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4d820fb{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4d82104{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4d94d90{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4d94d9a{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4d95e34{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d95e35{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d95e36{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d95e3e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d95e3f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4d95e40{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4db0766'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4d95e34"><p class="cl-b4d94d90"><span class="cl-b4d820fa">species</span></p></th><th class="cl-b4d95e34"><p class="cl-b4d94d90"><span class="cl-b4d820fa">sex</span></p></th><th class="cl-b4d95e35"><p class="cl-b4d94d9a"><span class="cl-b4d820fa">No. birds</span></p></th><th class="cl-b4d95e35"><p class="cl-b4d94d9a"><span class="cl-b4d820fa">Mean body mass (g)</span></p></th><th class="cl-b4d95e35"><p class="cl-b4d94d9a"><span class="cl-b4d820fa">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d820fb">Adelie</span></p></td><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d82104">female</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">73</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">3,369</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d820fb">Adelie</span></p></td><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d82104">male</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">73</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">4,043</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d820fb">Chinstrap</span></p></td><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d82104">female</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">34</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">3,527</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d820fb">Chinstrap</span></p></td><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d82104">male</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">34</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">3,939</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d820fb">Gentoo</span></p></td><td class="cl-b4d95e36"><p class="cl-b4d94d90"><span class="cl-b4d82104">female</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">58</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">4,680</span></p></td><td class="cl-b4d95e3e"><p class="cl-b4d94d9a"><span class="cl-b4d82104">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4d95e3f"><p class="cl-b4d94d90"><span class="cl-b4d820fb">Gentoo</span></p></td><td class="cl-b4d95e3f"><p class="cl-b4d94d90"><span class="cl-b4d82104">male</span></p></td><td class="cl-b4d95e40"><p class="cl-b4d94d9a"><span class="cl-b4d82104">61</span></p></td><td class="cl-b4d95e40"><p class="cl-b4d94d9a"><span class="cl-b4d82104">5,485</span></p></td><td class="cl-b4d95e40"><p class="cl-b4d94d9a"><span class="cl-b4d82104">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-b4e04046{}.cl-b4dd5548{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4de9214{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4de9215{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4dea2ae{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4dea2af{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4e04046'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">species</span></p></th><th class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">sex</span></p></th><th class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">No. birds</span></p></th><th class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">Mean body mass (g)</span></p></th><th class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">Adelie</span></p></td><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">female</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">73</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">3,369</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">Adelie</span></p></td><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">male</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">73</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">4,043</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">Chinstrap</span></p></td><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">female</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">34</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">3,527</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">Chinstrap</span></p></td><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">male</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">34</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">3,939</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">Gentoo</span></p></td><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">female</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">58</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">4,680</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">Gentoo</span></p></td><td class="cl-b4dea2ae"><p class="cl-b4de9214"><span class="cl-b4dd5548">male</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">61</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">5,485</span></p></td><td class="cl-b4dea2af"><p class="cl-b4de9215"><span class="cl-b4dd5548">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-b4e56224{}.cl-b4e1d190{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4e3b82a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4e3b82b{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4e3c702{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c70c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c70d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c70e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c70f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c710{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c716{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c717{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c718{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c719{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c71a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e3c720{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4e56224'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4e3c702"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">species</span></p></th><th class="cl-b4e3c70c"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">sex</span></p></th><th class="cl-b4e3c70d"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">No. birds</span></p></th><th class="cl-b4e3c70d"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">Mean body mass (g)</span></p></th><th class="cl-b4e3c70e"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4e3c70f"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">Adelie</span></p></td><td class="cl-b4e3c710"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">female</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">73</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">3,369</span></p></td><td class="cl-b4e3c717"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e3c70f"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">Adelie</span></p></td><td class="cl-b4e3c710"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">male</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">73</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">4,043</span></p></td><td class="cl-b4e3c717"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e3c70f"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">Chinstrap</span></p></td><td class="cl-b4e3c710"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">female</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">34</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">3,527</span></p></td><td class="cl-b4e3c717"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e3c70f"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">Chinstrap</span></p></td><td class="cl-b4e3c710"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">male</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">34</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">3,939</span></p></td><td class="cl-b4e3c717"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e3c70f"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">Gentoo</span></p></td><td class="cl-b4e3c710"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">female</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">58</span></p></td><td class="cl-b4e3c716"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">4,680</span></p></td><td class="cl-b4e3c717"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e3c718"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">Gentoo</span></p></td><td class="cl-b4e3c719"><p class="cl-b4e3b82a"><span class="cl-b4e1d190">male</span></p></td><td class="cl-b4e3c71a"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">61</span></p></td><td class="cl-b4e3c71a"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">5,485</span></p></td><td class="cl-b4e3c720"><p class="cl-b4e3b82b"><span class="cl-b4e1d190">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-b4e9c6fc{}.cl-b4e6e52c{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4e81208{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4e81209{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4e821ee{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821ef{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821f0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821f1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821f8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821f9{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821fa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821fb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e821fc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4e82202{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4e9c6fc'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4e821ee"><p class="cl-b4e81208"><span class="cl-b4e6e52c">species</span></p></th><th class="cl-b4e821ee"><p class="cl-b4e81208"><span class="cl-b4e6e52c">sex</span></p></th><th class="cl-b4e821ef"><p class="cl-b4e81209"><span class="cl-b4e6e52c">No. birds</span></p></th><th class="cl-b4e821ef"><p class="cl-b4e81209"><span class="cl-b4e6e52c">Mean body mass (g)</span></p></th><th class="cl-b4e821ef"><p class="cl-b4e81209"><span class="cl-b4e6e52c">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4e821f0"><p class="cl-b4e81208"><span class="cl-b4e6e52c">Adelie</span></p></td><td class="cl-b4e821f1"><p class="cl-b4e81208"><span class="cl-b4e6e52c">female</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">73</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">3,369</span></p></td><td class="cl-b4e821f9"><p class="cl-b4e81209"><span class="cl-b4e6e52c">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e821f0"><p class="cl-b4e81208"><span class="cl-b4e6e52c">Adelie</span></p></td><td class="cl-b4e821f1"><p class="cl-b4e81208"><span class="cl-b4e6e52c">male</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">73</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">4,043</span></p></td><td class="cl-b4e821f9"><p class="cl-b4e81209"><span class="cl-b4e6e52c">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e821f0"><p class="cl-b4e81208"><span class="cl-b4e6e52c">Chinstrap</span></p></td><td class="cl-b4e821f1"><p class="cl-b4e81208"><span class="cl-b4e6e52c">female</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">34</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">3,527</span></p></td><td class="cl-b4e821f9"><p class="cl-b4e81209"><span class="cl-b4e6e52c">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e821f0"><p class="cl-b4e81208"><span class="cl-b4e6e52c">Chinstrap</span></p></td><td class="cl-b4e821f1"><p class="cl-b4e81208"><span class="cl-b4e6e52c">male</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">34</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">3,939</span></p></td><td class="cl-b4e821f9"><p class="cl-b4e81209"><span class="cl-b4e6e52c">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e821f0"><p class="cl-b4e81208"><span class="cl-b4e6e52c">Gentoo</span></p></td><td class="cl-b4e821f1"><p class="cl-b4e81208"><span class="cl-b4e6e52c">female</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">58</span></p></td><td class="cl-b4e821f8"><p class="cl-b4e81209"><span class="cl-b4e6e52c">4,680</span></p></td><td class="cl-b4e821f9"><p class="cl-b4e81209"><span class="cl-b4e6e52c">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4e821fa"><p class="cl-b4e81208"><span class="cl-b4e6e52c">Gentoo</span></p></td><td class="cl-b4e821fb"><p class="cl-b4e81208"><span class="cl-b4e6e52c">male</span></p></td><td class="cl-b4e821fc"><p class="cl-b4e81209"><span class="cl-b4e6e52c">61</span></p></td><td class="cl-b4e821fc"><p class="cl-b4e81209"><span class="cl-b4e6e52c">5,485</span></p></td><td class="cl-b4e82202"><p class="cl-b4e81209"><span class="cl-b4e6e52c">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-b4ee25bc{}.cl-b4eb4720{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4ec7424{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4ec7425{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4ec832e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec8338{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec8339{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec8342{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec8343{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec8344{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec8345{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4ec834c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4ee25bc'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4ec832e"><p class="cl-b4ec7424"><span class="cl-b4eb4720">species</span></p></th><th class="cl-b4ec832e"><p class="cl-b4ec7424"><span class="cl-b4eb4720">sex</span></p></th><th class="cl-b4ec8338"><p class="cl-b4ec7425"><span class="cl-b4eb4720">No. birds</span></p></th><th class="cl-b4ec8338"><p class="cl-b4ec7425"><span class="cl-b4eb4720">Mean body mass (g)</span></p></th><th class="cl-b4ec8338"><p class="cl-b4ec7425"><span class="cl-b4eb4720">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4ec8339"><p class="cl-b4ec7424"><span class="cl-b4eb4720">Adelie</span></p></td><td class="cl-b4ec8339"><p class="cl-b4ec7424"><span class="cl-b4eb4720">female</span></p></td><td class="cl-b4ec8342"><p class="cl-b4ec7425"><span class="cl-b4eb4720">73</span></p></td><td class="cl-b4ec8342"><p class="cl-b4ec7425"><span class="cl-b4eb4720">3,369</span></p></td><td class="cl-b4ec8342"><p class="cl-b4ec7425"><span class="cl-b4eb4720">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">Adelie</span></p></td><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">male</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">73</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">4,043</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">Chinstrap</span></p></td><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">female</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">34</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">3,527</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">Chinstrap</span></p></td><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">male</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">34</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">3,939</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">Gentoo</span></p></td><td class="cl-b4ec8343"><p class="cl-b4ec7424"><span class="cl-b4eb4720">female</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">58</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">4,680</span></p></td><td class="cl-b4ec8344"><p class="cl-b4ec7425"><span class="cl-b4eb4720">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4ec8345"><p class="cl-b4ec7424"><span class="cl-b4eb4720">Gentoo</span></p></td><td class="cl-b4ec8345"><p class="cl-b4ec7424"><span class="cl-b4eb4720">male</span></p></td><td class="cl-b4ec834c"><p class="cl-b4ec7425"><span class="cl-b4eb4720">61</span></p></td><td class="cl-b4ec834c"><p class="cl-b4ec7425"><span class="cl-b4eb4720">5,485</span></p></td><td class="cl-b4ec834c"><p class="cl-b4ec7425"><span class="cl-b4eb4720">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-b4f5586e{}.cl-b4f1d040{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4f1d041{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4f3bb62{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4f3bb6c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4f3cad0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cad1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cad2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cada{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cadb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cadc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cadd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4f3cae4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4f5586e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4f3cad0"><p class="cl-b4f3bb62"><span class="cl-b4f1d040">species</span></p></th><th class="cl-b4f3cad0"><p class="cl-b4f3bb62"><span class="cl-b4f1d040">island</span></p></th><th class="cl-b4f3cad1"><p class="cl-b4f3bb6c"><span class="cl-b4f1d040">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4f3cad2"><p class="cl-b4f3bb62"><span class="cl-b4f1d041">Adelie</span></p></td><td class="cl-b4f3cad2"><p class="cl-b4f3bb62"><span class="cl-b4f1d041">Torgersen</span></p></td><td class="cl-b4f3cada"><p class="cl-b4f3bb6c"><span class="cl-b4f1d041">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4f3cadb"><p class="cl-b4f3bb62"><span class="cl-b4f1d041">Adelie</span></p></td><td class="cl-b4f3cadb"><p class="cl-b4f3bb62"><span class="cl-b4f1d041">Dream</span></p></td><td class="cl-b4f3cadc"><p class="cl-b4f3bb6c"><span class="cl-b4f1d041">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4f3cadd"><p class="cl-b4f3bb62"><span class="cl-b4f1d041">Adelie</span></p></td><td class="cl-b4f3cadd"><p class="cl-b4f3bb62"><span class="cl-b4f1d041">Dream</span></p></td><td class="cl-b4f3cae4"><p class="cl-b4f3bb6c"><span class="cl-b4f1d041">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-b4fd553c{}.cl-b4fa6c28{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4fa6c29{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4fb9f3a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4fb9f3b{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b4fbaf2a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf2b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf2c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf2d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf34{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf35{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf36{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b4fbaf37{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b4fd553c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b4fbaf2a"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c28">species</span></p></th><th class="cl-b4fbaf2a"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c28">sex</span></p></th><th class="cl-b4fbaf2b"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c28">No. birds</span></p></th><th class="cl-b4fbaf2b"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c28">Mean body mass (g)</span></p></th><th class="cl-b4fbaf2b"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c28">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b4fbaf2c"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">Adelie</span></p></td><td class="cl-b4fbaf2c"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">female</span></p></td><td class="cl-b4fbaf2d"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">73</span></p></td><td class="cl-b4fbaf2d"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">3,368.836</span></p></td><td class="cl-b4fbaf2d"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">Adelie</span></p></td><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">male</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">73</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">4,043.493</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">Chinstrap</span></p></td><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">female</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">34</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">3,527.206</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">Chinstrap</span></p></td><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">male</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">34</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">3,938.971</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">Gentoo</span></p></td><td class="cl-b4fbaf34"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">female</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">58</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">4,679.741</span></p></td><td class="cl-b4fbaf35"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b4fbaf36"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">Gentoo</span></p></td><td class="cl-b4fbaf36"><p class="cl-b4fb9f3a"><span class="cl-b4fa6c29">male</span></p></td><td class="cl-b4fbaf37"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">61</span></p></td><td class="cl-b4fbaf37"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">5,484.836</span></p></td><td class="cl-b4fbaf37"><p class="cl-b4fb9f3b"><span class="cl-b4fa6c29">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-b502e39e{}.cl-b4ff67b4{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b4ff67b5{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b5009a94{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b5009a95{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b500ab1a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab1b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab1c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab24{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab25{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab26{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab2e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b500ab2f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b502e39e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b500ab1a"><p class="cl-b5009a94"><span class="cl-b4ff67b4">species</span></p></th><th class="cl-b500ab1a"><p class="cl-b5009a94"><span class="cl-b4ff67b4">sex</span></p></th><th class="cl-b500ab1b"><p class="cl-b5009a95"><span class="cl-b4ff67b4">No. birds</span></p></th><th class="cl-b500ab1b"><p class="cl-b5009a95"><span class="cl-b4ff67b4">Mean body mass (g)</span></p></th><th class="cl-b500ab1b"><p class="cl-b5009a95"><span class="cl-b4ff67b4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-b500ab1c"><p class="cl-b5009a94"><span class="cl-b4ff67b5">Adelie</span></p></td><td class="cl-b500ab1c"><p class="cl-b5009a94"><span class="cl-b4ff67b5">female</span></p></td><td class="cl-b500ab24"><p class="cl-b5009a95"><span class="cl-b4ff67b5">73</span></p></td><td class="cl-b500ab24"><p class="cl-b5009a95"><span class="cl-b4ff67b5">3369</span></p></td><td class="cl-b500ab24"><p class="cl-b5009a95"><span class="cl-b4ff67b5">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">Adelie</span></p></td><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">male</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">73</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">4043</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">Chinstrap</span></p></td><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">female</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">34</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">3527</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">Chinstrap</span></p></td><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">male</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">34</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">3939</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">Gentoo</span></p></td><td class="cl-b500ab25"><p class="cl-b5009a94"><span class="cl-b4ff67b5">female</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">58</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">4680</span></p></td><td class="cl-b500ab26"><p class="cl-b5009a95"><span class="cl-b4ff67b5">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b500ab2e"><p class="cl-b5009a94"><span class="cl-b4ff67b5">Gentoo</span></p></td><td class="cl-b500ab2e"><p class="cl-b5009a94"><span class="cl-b4ff67b5">male</span></p></td><td class="cl-b500ab2f"><p class="cl-b5009a95"><span class="cl-b4ff67b5">61</span></p></td><td class="cl-b500ab2f"><p class="cl-b5009a95"><span class="cl-b4ff67b5">5485</span></p></td><td class="cl-b500ab2f"><p class="cl-b5009a95"><span class="cl-b4ff67b5">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-b5086f30{}.cl-b5056e16{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b5056e17{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-b506b94c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b506b956{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-b506c9e6{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9f0{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9f1{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9f2{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9f3{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9f4{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9fa{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-b506c9fb{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-b5086f30'><thead><tr style="overflow-wrap:break-word;"><th class="cl-b506c9e6"><p class="cl-b506b94c"><span class="cl-b5056e16">species</span></p></th><th class="cl-b506c9e6"><p class="cl-b506b94c"><span class="cl-b5056e16">sex</span></p></th><th class="cl-b506c9f0"><p class="cl-b506b956"><span class="cl-b5056e16">No. birds</span></p></th><th class="cl-b506c9f0"><p class="cl-b506b956"><span class="cl-b5056e16">Mean body mass (g)</span></p></th><th class="cl-b506c9f0"><p class="cl-b506b956"><span class="cl-b5056e16">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-b506c9f1"><p class="cl-b506b94c"><span class="cl-b5056e17">Adelie</span></p></td><td class="cl-b506c9f1"><p class="cl-b506b94c"><span class="cl-b5056e17">female</span></p></td><td class="cl-b506c9f2"><p class="cl-b506b956"><span class="cl-b5056e17">73</span></p></td><td class="cl-b506c9f2"><p class="cl-b506b956"><span class="cl-b5056e17">3,368.836</span></p></td><td class="cl-b506c9f2"><p class="cl-b506b956"><span class="cl-b5056e17">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b506c9f3"><p class="cl-b506b94c"><span class="cl-b5056e17">male</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">73</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">4,043.493</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-b506c9f3"><p class="cl-b506b94c"><span class="cl-b5056e17">Chinstrap</span></p></td><td class="cl-b506c9f3"><p class="cl-b506b94c"><span class="cl-b5056e17">female</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">34</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">3,527.206</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b506c9f3"><p class="cl-b506b94c"><span class="cl-b5056e17">male</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">34</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">3,938.971</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-b506c9fa"><p class="cl-b506b94c"><span class="cl-b5056e17">Gentoo</span></p></td><td class="cl-b506c9f3"><p class="cl-b506b94c"><span class="cl-b5056e17">female</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">58</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">4,679.741</span></p></td><td class="cl-b506c9f4"><p class="cl-b506b956"><span class="cl-b5056e17">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-b506c9fa"><p class="cl-b506b94c"><span class="cl-b5056e17">male</span></p></td><td class="cl-b506c9fb"><p class="cl-b506b956"><span class="cl-b5056e17">61</span></p></td><td class="cl-b506c9fb"><p class="cl-b506b956"><span class="cl-b5056e17">5,484.836</span></p></td><td class="cl-b506c9fb"><p class="cl-b506b956"><span class="cl-b5056e17">313.1586</span></p></td></tr></tbody></table></div>
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
