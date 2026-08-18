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
<div class="tabwid"><style>.cl-825cc6ec{}.cl-8255f98e{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8255f998{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82583942{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8258394c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82584b30{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82584b31{width:0.75in;background-color:rgba(207, 207, 207, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82584b3a{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82584b3b{width:0.75in;background-color:rgba(239, 239, 239, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82584b3c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82584b44{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-825cc6ec'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82584b30"><p class="cl-82583942"><span class="cl-8255f98e">species</span></p></th><th class="cl-82584b30"><p class="cl-82583942"><span class="cl-8255f98e">sex</span></p></th><th class="cl-82584b31"><p class="cl-8258394c"><span class="cl-8255f98e">No. birds</span></p></th><th class="cl-82584b31"><p class="cl-8258394c"><span class="cl-8255f98e">Mean body mass (g)</span></p></th><th class="cl-82584b31"><p class="cl-8258394c"><span class="cl-8255f98e">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-82584b3a"><p class="cl-82583942"><span class="cl-8255f998">Adelie</span></p></td><td class="cl-82584b3a"><p class="cl-82583942"><span class="cl-8255f998">female</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">73</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">3,369</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82584b3c"><p class="cl-82583942"><span class="cl-8255f998">Adelie</span></p></td><td class="cl-82584b3c"><p class="cl-82583942"><span class="cl-8255f998">male</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">73</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">4,043</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82584b3a"><p class="cl-82583942"><span class="cl-8255f998">Chinstrap</span></p></td><td class="cl-82584b3a"><p class="cl-82583942"><span class="cl-8255f998">female</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">34</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">3,527</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82584b3c"><p class="cl-82583942"><span class="cl-8255f998">Chinstrap</span></p></td><td class="cl-82584b3c"><p class="cl-82583942"><span class="cl-8255f998">male</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">34</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">3,939</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82584b3a"><p class="cl-82583942"><span class="cl-8255f998">Gentoo</span></p></td><td class="cl-82584b3a"><p class="cl-82583942"><span class="cl-8255f998">female</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">58</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">4,680</span></p></td><td class="cl-82584b3b"><p class="cl-8258394c"><span class="cl-8255f998">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82584b3c"><p class="cl-82583942"><span class="cl-8255f998">Gentoo</span></p></td><td class="cl-82584b3c"><p class="cl-82583942"><span class="cl-8255f998">male</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">61</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">5,485</span></p></td><td class="cl-82584b44"><p class="cl-8258394c"><span class="cl-8255f998">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-8277f764{}.cl-8274dbce{font-family:'Arial';font-size:10pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8274dbcf{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82761f34{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82761f35{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82763154{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8276315e{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8276315f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82763168{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82763169{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8276316a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-8277f764'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82763154"><p class="cl-82761f34"><span class="cl-8274dbce">species</span></p></th><th class="cl-82763154"><p class="cl-82761f34"><span class="cl-8274dbce">sex</span></p></th><th class="cl-8276315e"><p class="cl-82761f35"><span class="cl-8274dbce">No. birds</span></p></th><th class="cl-8276315e"><p class="cl-82761f35"><span class="cl-8274dbce">Mean body mass (g)</span></p></th><th class="cl-8276315e"><p class="cl-82761f35"><span class="cl-8274dbce">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">Adelie</span></p></td><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">female</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">73</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">3,369</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">Adelie</span></p></td><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">male</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">73</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">4,043</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">Chinstrap</span></p></td><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">female</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">34</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">3,527</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">Chinstrap</span></p></td><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">male</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">34</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">3,939</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">Gentoo</span></p></td><td class="cl-8276315f"><p class="cl-82761f34"><span class="cl-8274dbcf">female</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">58</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">4,680</span></p></td><td class="cl-82763168"><p class="cl-82761f35"><span class="cl-8274dbcf">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82763169"><p class="cl-82761f34"><span class="cl-8274dbcf">Gentoo</span></p></td><td class="cl-82763169"><p class="cl-82761f34"><span class="cl-8274dbcf">male</span></p></td><td class="cl-8276316a"><p class="cl-82761f35"><span class="cl-8274dbcf">61</span></p></td><td class="cl-8276316a"><p class="cl-82761f35"><span class="cl-8274dbcf">5,485</span></p></td><td class="cl-8276316a"><p class="cl-82761f35"><span class="cl-8274dbcf">313</span></p></td></tr></tbody></table></div>
```

``` r
# Center all text
ft %>%
  style(part = "all", pr_p = fp_par(text.align = "center"))
```

```{=html}
<div class="tabwid"><style>.cl-827cbe0c{}.cl-8279ab90{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-827af7c0{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:0;padding-top:0;padding-left:0;padding-right:0;line-height: 1;background-color:transparent;}.cl-827b076a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827b076b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827b0774{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-827cbe0c'><thead><tr style="overflow-wrap:break-word;"><th class="cl-827b076a"><p class="cl-827af7c0"><span class="cl-8279ab90">species</span></p></th><th class="cl-827b076a"><p class="cl-827af7c0"><span class="cl-8279ab90">sex</span></p></th><th class="cl-827b076a"><p class="cl-827af7c0"><span class="cl-8279ab90">No. birds</span></p></th><th class="cl-827b076a"><p class="cl-827af7c0"><span class="cl-8279ab90">Mean body mass (g)</span></p></th><th class="cl-827b076a"><p class="cl-827af7c0"><span class="cl-8279ab90">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">Adelie</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">female</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">73</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">3,369</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">Adelie</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">male</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">73</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">4,043</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">Chinstrap</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">female</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">34</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">3,527</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">Chinstrap</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">male</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">34</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">3,939</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">Gentoo</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">female</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">58</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">4,680</span></p></td><td class="cl-827b076b"><p class="cl-827af7c0"><span class="cl-8279ab90">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827b0774"><p class="cl-827af7c0"><span class="cl-8279ab90">Gentoo</span></p></td><td class="cl-827b0774"><p class="cl-827af7c0"><span class="cl-8279ab90">male</span></p></td><td class="cl-827b0774"><p class="cl-827af7c0"><span class="cl-8279ab90">61</span></p></td><td class="cl-827b0774"><p class="cl-827af7c0"><span class="cl-8279ab90">5,485</span></p></td><td class="cl-827b0774"><p class="cl-827af7c0"><span class="cl-8279ab90">313</span></p></td></tr></tbody></table></div>
```

``` r
# Shade alternating cells in orange
ft %>%
  style(i = seq(1,6,2), part = "body",
        pr_c = fp_cell(background.color = "orange"))
```

```{=html}
<div class="tabwid"><style>.cl-82819792{}.cl-827e64dc{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-827fb7ce{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-827fb7d8{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-827fc674{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc675{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc676{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc67e{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc67f{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc680{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc681{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-827fc682{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-82819792'><thead><tr style="overflow-wrap:break-word;"><th class="cl-827fc674"><p class="cl-827fb7ce"><span class="cl-827e64dc">species</span></p></th><th class="cl-827fc674"><p class="cl-827fb7ce"><span class="cl-827e64dc">sex</span></p></th><th class="cl-827fc675"><p class="cl-827fb7d8"><span class="cl-827e64dc">No. birds</span></p></th><th class="cl-827fc675"><p class="cl-827fb7d8"><span class="cl-827e64dc">Mean body mass (g)</span></p></th><th class="cl-827fc675"><p class="cl-827fb7d8"><span class="cl-827e64dc">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-827fc676"><p class="cl-827fb7ce"><span class="cl-827e64dc">Adelie</span></p></td><td class="cl-827fc676"><p class="cl-827fb7ce"><span class="cl-827e64dc">female</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">73</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">3,369</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827fc67f"><p class="cl-827fb7ce"><span class="cl-827e64dc">Adelie</span></p></td><td class="cl-827fc67f"><p class="cl-827fb7ce"><span class="cl-827e64dc">male</span></p></td><td class="cl-827fc680"><p class="cl-827fb7d8"><span class="cl-827e64dc">73</span></p></td><td class="cl-827fc680"><p class="cl-827fb7d8"><span class="cl-827e64dc">4,043</span></p></td><td class="cl-827fc680"><p class="cl-827fb7d8"><span class="cl-827e64dc">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827fc676"><p class="cl-827fb7ce"><span class="cl-827e64dc">Chinstrap</span></p></td><td class="cl-827fc676"><p class="cl-827fb7ce"><span class="cl-827e64dc">female</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">34</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">3,527</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827fc67f"><p class="cl-827fb7ce"><span class="cl-827e64dc">Chinstrap</span></p></td><td class="cl-827fc67f"><p class="cl-827fb7ce"><span class="cl-827e64dc">male</span></p></td><td class="cl-827fc680"><p class="cl-827fb7d8"><span class="cl-827e64dc">34</span></p></td><td class="cl-827fc680"><p class="cl-827fb7d8"><span class="cl-827e64dc">3,939</span></p></td><td class="cl-827fc680"><p class="cl-827fb7d8"><span class="cl-827e64dc">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827fc676"><p class="cl-827fb7ce"><span class="cl-827e64dc">Gentoo</span></p></td><td class="cl-827fc676"><p class="cl-827fb7ce"><span class="cl-827e64dc">female</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">58</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">4,680</span></p></td><td class="cl-827fc67e"><p class="cl-827fb7d8"><span class="cl-827e64dc">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-827fc681"><p class="cl-827fb7ce"><span class="cl-827e64dc">Gentoo</span></p></td><td class="cl-827fc681"><p class="cl-827fb7ce"><span class="cl-827e64dc">male</span></p></td><td class="cl-827fc682"><p class="cl-827fb7d8"><span class="cl-827e64dc">61</span></p></td><td class="cl-827fc682"><p class="cl-827fb7d8"><span class="cl-827e64dc">5,485</span></p></td><td class="cl-827fc682"><p class="cl-827fb7d8"><span class="cl-827e64dc">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-828787ce{}.cl-8284a310{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8285ced4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8285ced5{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8285dde8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddf2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddf3{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddf4{width:0.75in;background-color:rgba(255, 165, 0, 1.00);vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddf5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddfc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddfd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-8285ddfe{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-828787ce'><thead><tr style="overflow-wrap:break-word;"><th class="cl-8285dde8"><p class="cl-8285ced4"><span class="cl-8284a310">species</span></p></th><th class="cl-8285dde8"><p class="cl-8285ced4"><span class="cl-8284a310">sex</span></p></th><th class="cl-8285ddf2"><p class="cl-8285ced5"><span class="cl-8284a310">No. birds</span></p></th><th class="cl-8285ddf2"><p class="cl-8285ced5"><span class="cl-8284a310">Mean body mass (g)</span></p></th><th class="cl-8285ddf2"><p class="cl-8285ced5"><span class="cl-8284a310">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-8285ddf3"><p class="cl-8285ced4"><span class="cl-8284a310">Adelie</span></p></td><td class="cl-8285ddf3"><p class="cl-8285ced4"><span class="cl-8284a310">female</span></p></td><td class="cl-8285ddf4"><p class="cl-8285ced5"><span class="cl-8284a310">73</span></p></td><td class="cl-8285ddf4"><p class="cl-8285ced5"><span class="cl-8284a310">3,369</span></p></td><td class="cl-8285ddf4"><p class="cl-8285ced5"><span class="cl-8284a310">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8285ddf3"><p class="cl-8285ced4"><span class="cl-8284a310">Adelie</span></p></td><td class="cl-8285ddf3"><p class="cl-8285ced4"><span class="cl-8284a310">male</span></p></td><td class="cl-8285ddf4"><p class="cl-8285ced5"><span class="cl-8284a310">73</span></p></td><td class="cl-8285ddf4"><p class="cl-8285ced5"><span class="cl-8284a310">4,043</span></p></td><td class="cl-8285ddf4"><p class="cl-8285ced5"><span class="cl-8284a310">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8285ddf5"><p class="cl-8285ced4"><span class="cl-8284a310">Chinstrap</span></p></td><td class="cl-8285ddf5"><p class="cl-8285ced4"><span class="cl-8284a310">female</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">34</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">3,527</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8285ddf5"><p class="cl-8285ced4"><span class="cl-8284a310">Chinstrap</span></p></td><td class="cl-8285ddf5"><p class="cl-8285ced4"><span class="cl-8284a310">male</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">34</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">3,939</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8285ddf5"><p class="cl-8285ced4"><span class="cl-8284a310">Gentoo</span></p></td><td class="cl-8285ddf5"><p class="cl-8285ced4"><span class="cl-8284a310">female</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">58</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">4,680</span></p></td><td class="cl-8285ddfc"><p class="cl-8285ced5"><span class="cl-8284a310">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-8285ddfd"><p class="cl-8285ced4"><span class="cl-8284a310">Gentoo</span></p></td><td class="cl-8285ddfd"><p class="cl-8285ced4"><span class="cl-8284a310">male</span></p></td><td class="cl-8285ddfe"><p class="cl-8285ced5"><span class="cl-8284a310">61</span></p></td><td class="cl-8285ddfe"><p class="cl-8285ced5"><span class="cl-8284a310">5,485</span></p></td><td class="cl-8285ddfe"><p class="cl-8285ced5"><span class="cl-8284a310">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-828c67b2{}.cl-82898664{font-family:'Helvetica';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8289866e{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:italic;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8289866f{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-828ab4c6{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-828ab4d0{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-828ac3f8{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-828ac402{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-828ac403{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-828ac404{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-828ac40c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-828ac40d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-828c67b2'><thead><tr style="overflow-wrap:break-word;"><th class="cl-828ac3f8"><p class="cl-828ab4c6"><span class="cl-82898664">species</span></p></th><th class="cl-828ac3f8"><p class="cl-828ab4c6"><span class="cl-82898664">sex</span></p></th><th class="cl-828ac402"><p class="cl-828ab4d0"><span class="cl-82898664">No. birds</span></p></th><th class="cl-828ac402"><p class="cl-828ab4d0"><span class="cl-82898664">Mean body mass (g)</span></p></th><th class="cl-828ac402"><p class="cl-828ab4d0"><span class="cl-82898664">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866e">Adelie</span></p></td><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866f">female</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">73</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">3,369</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866e">Adelie</span></p></td><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866f">male</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">73</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">4,043</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866e">Chinstrap</span></p></td><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866f">female</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">34</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">3,527</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866e">Chinstrap</span></p></td><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866f">male</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">34</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">3,939</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866e">Gentoo</span></p></td><td class="cl-828ac403"><p class="cl-828ab4c6"><span class="cl-8289866f">female</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">58</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">4,680</span></p></td><td class="cl-828ac404"><p class="cl-828ab4d0"><span class="cl-8289866f">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828ac40c"><p class="cl-828ab4c6"><span class="cl-8289866e">Gentoo</span></p></td><td class="cl-828ac40c"><p class="cl-828ab4c6"><span class="cl-8289866f">male</span></p></td><td class="cl-828ac40d"><p class="cl-828ab4d0"><span class="cl-8289866f">61</span></p></td><td class="cl-828ac40d"><p class="cl-828ab4d0"><span class="cl-8289866f">5,485</span></p></td><td class="cl-828ac40d"><p class="cl-828ab4d0"><span class="cl-8289866f">313</span></p></td></tr></tbody></table></div>
```

Borders are specified in a similar way as in Word (inner and outer, horizontal and vertical) and can also be specified for each cell, row, or column.


``` r
# Remove all borders
ft %>%
  border_remove()
```

```{=html}
<div class="tabwid"><style>.cl-82914ed0{}.cl-828e5b30{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-828f907c{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-828f907d{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-828fa058{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-828fa059{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-82914ed0'><thead><tr style="overflow-wrap:break-word;"><th class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">species</span></p></th><th class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">sex</span></p></th><th class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">No. birds</span></p></th><th class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">Mean body mass (g)</span></p></th><th class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">Adelie</span></p></td><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">female</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">73</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">3,369</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">Adelie</span></p></td><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">male</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">73</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">4,043</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">Chinstrap</span></p></td><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">female</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">34</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">3,527</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">Chinstrap</span></p></td><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">male</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">34</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">3,939</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">Gentoo</span></p></td><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">female</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">58</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">4,680</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">Gentoo</span></p></td><td class="cl-828fa058"><p class="cl-828f907c"><span class="cl-828e5b30">male</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">61</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">5,485</span></p></td><td class="cl-828fa059"><p class="cl-828f907d"><span class="cl-828e5b30">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the whole table
ft %>%
  border_outer(part = "all")
```

```{=html}
<div class="tabwid"><style>.cl-8296c608{}.cl-8292f032{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-8294fb3e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8294fb48{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82950ac0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950aca{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950acb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950acc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950acd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950ace{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950ad4{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950ad5{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950ad6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950ade{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950adf{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82950ae0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-8296c608'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82950ac0"><p class="cl-8294fb3e"><span class="cl-8292f032">species</span></p></th><th class="cl-82950aca"><p class="cl-8294fb3e"><span class="cl-8292f032">sex</span></p></th><th class="cl-82950acb"><p class="cl-8294fb48"><span class="cl-8292f032">No. birds</span></p></th><th class="cl-82950acb"><p class="cl-8294fb48"><span class="cl-8292f032">Mean body mass (g)</span></p></th><th class="cl-82950acc"><p class="cl-8294fb48"><span class="cl-8292f032">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-82950acd"><p class="cl-8294fb3e"><span class="cl-8292f032">Adelie</span></p></td><td class="cl-82950ace"><p class="cl-8294fb3e"><span class="cl-8292f032">female</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">73</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">3,369</span></p></td><td class="cl-82950ad5"><p class="cl-8294fb48"><span class="cl-8292f032">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82950acd"><p class="cl-8294fb3e"><span class="cl-8292f032">Adelie</span></p></td><td class="cl-82950ace"><p class="cl-8294fb3e"><span class="cl-8292f032">male</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">73</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">4,043</span></p></td><td class="cl-82950ad5"><p class="cl-8294fb48"><span class="cl-8292f032">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82950acd"><p class="cl-8294fb3e"><span class="cl-8292f032">Chinstrap</span></p></td><td class="cl-82950ace"><p class="cl-8294fb3e"><span class="cl-8292f032">female</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">34</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">3,527</span></p></td><td class="cl-82950ad5"><p class="cl-8294fb48"><span class="cl-8292f032">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82950acd"><p class="cl-8294fb3e"><span class="cl-8292f032">Chinstrap</span></p></td><td class="cl-82950ace"><p class="cl-8294fb3e"><span class="cl-8292f032">male</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">34</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">3,939</span></p></td><td class="cl-82950ad5"><p class="cl-8294fb48"><span class="cl-8292f032">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82950acd"><p class="cl-8294fb3e"><span class="cl-8292f032">Gentoo</span></p></td><td class="cl-82950ace"><p class="cl-8294fb3e"><span class="cl-8292f032">female</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">58</span></p></td><td class="cl-82950ad4"><p class="cl-8294fb48"><span class="cl-8292f032">4,680</span></p></td><td class="cl-82950ad5"><p class="cl-8294fb48"><span class="cl-8292f032">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82950ad6"><p class="cl-8294fb3e"><span class="cl-8292f032">Gentoo</span></p></td><td class="cl-82950ade"><p class="cl-8294fb3e"><span class="cl-8292f032">male</span></p></td><td class="cl-82950adf"><p class="cl-8294fb48"><span class="cl-8292f032">61</span></p></td><td class="cl-82950adf"><p class="cl-8294fb48"><span class="cl-8292f032">5,485</span></p></td><td class="cl-82950ae0"><p class="cl-8294fb48"><span class="cl-8292f032">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add a box around the body
ft %>%
  border_outer(part = "body")
```

```{=html}
<div class="tabwid"><style>.cl-829b2540{}.cl-829845f0{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82997722{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-8299772c{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-829985e6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985f0{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985f1{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985f2{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985f3{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985fa{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985fb{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 1pt solid rgba(102, 102, 102, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985fc{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985fd{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829985fe{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 1pt solid rgba(102, 102, 102, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-829b2540'><thead><tr style="overflow-wrap:break-word;"><th class="cl-829985e6"><p class="cl-82997722"><span class="cl-829845f0">species</span></p></th><th class="cl-829985e6"><p class="cl-82997722"><span class="cl-829845f0">sex</span></p></th><th class="cl-829985f0"><p class="cl-8299772c"><span class="cl-829845f0">No. birds</span></p></th><th class="cl-829985f0"><p class="cl-8299772c"><span class="cl-829845f0">Mean body mass (g)</span></p></th><th class="cl-829985f0"><p class="cl-8299772c"><span class="cl-829845f0">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-829985f1"><p class="cl-82997722"><span class="cl-829845f0">Adelie</span></p></td><td class="cl-829985f2"><p class="cl-82997722"><span class="cl-829845f0">female</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">73</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">3,369</span></p></td><td class="cl-829985fa"><p class="cl-8299772c"><span class="cl-829845f0">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829985f1"><p class="cl-82997722"><span class="cl-829845f0">Adelie</span></p></td><td class="cl-829985f2"><p class="cl-82997722"><span class="cl-829845f0">male</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">73</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">4,043</span></p></td><td class="cl-829985fa"><p class="cl-8299772c"><span class="cl-829845f0">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829985f1"><p class="cl-82997722"><span class="cl-829845f0">Chinstrap</span></p></td><td class="cl-829985f2"><p class="cl-82997722"><span class="cl-829845f0">female</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">34</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">3,527</span></p></td><td class="cl-829985fa"><p class="cl-8299772c"><span class="cl-829845f0">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829985f1"><p class="cl-82997722"><span class="cl-829845f0">Chinstrap</span></p></td><td class="cl-829985f2"><p class="cl-82997722"><span class="cl-829845f0">male</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">34</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">3,939</span></p></td><td class="cl-829985fa"><p class="cl-8299772c"><span class="cl-829845f0">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829985f1"><p class="cl-82997722"><span class="cl-829845f0">Gentoo</span></p></td><td class="cl-829985f2"><p class="cl-82997722"><span class="cl-829845f0">female</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">58</span></p></td><td class="cl-829985f3"><p class="cl-8299772c"><span class="cl-829845f0">4,680</span></p></td><td class="cl-829985fa"><p class="cl-8299772c"><span class="cl-829845f0">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829985fb"><p class="cl-82997722"><span class="cl-829845f0">Gentoo</span></p></td><td class="cl-829985fc"><p class="cl-82997722"><span class="cl-829845f0">male</span></p></td><td class="cl-829985fd"><p class="cl-8299772c"><span class="cl-829845f0">61</span></p></td><td class="cl-829985fd"><p class="cl-8299772c"><span class="cl-829845f0">5,485</span></p></td><td class="cl-829985fe"><p class="cl-8299772c"><span class="cl-829845f0">313</span></p></td></tr></tbody></table></div>
```

``` r
# Add all horizontal borders
ft %>%
  border_inner_h()
```

```{=html}
<div class="tabwid"><style>.cl-829f995e{}.cl-829ca334{font-family:'Helvetica';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-829dcfca{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-829dcfd4{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-829ddf10{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf11{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf12{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf1a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf1b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf1c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf1d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-829ddf24{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-829f995e'><thead><tr style="overflow-wrap:break-word;"><th class="cl-829ddf10"><p class="cl-829dcfca"><span class="cl-829ca334">species</span></p></th><th class="cl-829ddf10"><p class="cl-829dcfca"><span class="cl-829ca334">sex</span></p></th><th class="cl-829ddf11"><p class="cl-829dcfd4"><span class="cl-829ca334">No. birds</span></p></th><th class="cl-829ddf11"><p class="cl-829dcfd4"><span class="cl-829ca334">Mean body mass (g)</span></p></th><th class="cl-829ddf11"><p class="cl-829dcfd4"><span class="cl-829ca334">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-829ddf12"><p class="cl-829dcfca"><span class="cl-829ca334">Adelie</span></p></td><td class="cl-829ddf12"><p class="cl-829dcfca"><span class="cl-829ca334">female</span></p></td><td class="cl-829ddf1a"><p class="cl-829dcfd4"><span class="cl-829ca334">73</span></p></td><td class="cl-829ddf1a"><p class="cl-829dcfd4"><span class="cl-829ca334">3,369</span></p></td><td class="cl-829ddf1a"><p class="cl-829dcfd4"><span class="cl-829ca334">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">Adelie</span></p></td><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">male</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">73</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">4,043</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">Chinstrap</span></p></td><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">female</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">34</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">3,527</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">Chinstrap</span></p></td><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">male</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">34</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">3,939</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">Gentoo</span></p></td><td class="cl-829ddf1b"><p class="cl-829dcfca"><span class="cl-829ca334">female</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">58</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">4,680</span></p></td><td class="cl-829ddf1c"><p class="cl-829dcfd4"><span class="cl-829ca334">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-829ddf1d"><p class="cl-829dcfca"><span class="cl-829ca334">Gentoo</span></p></td><td class="cl-829ddf1d"><p class="cl-829dcfca"><span class="cl-829ca334">male</span></p></td><td class="cl-829ddf24"><p class="cl-829dcfd4"><span class="cl-829ca334">61</span></p></td><td class="cl-829ddf24"><p class="cl-829dcfd4"><span class="cl-829ca334">5,485</span></p></td><td class="cl-829ddf24"><p class="cl-829dcfd4"><span class="cl-829ca334">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-82a6e74a{}.cl-82a35f12{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82a35f13{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82a5444e{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82a54458{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82a55286{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a55287{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a55288{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a55289{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a5528a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a5528b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a55290{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82a55291{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-82a6e74a'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82a55286"><p class="cl-82a5444e"><span class="cl-82a35f12">species</span></p></th><th class="cl-82a55286"><p class="cl-82a5444e"><span class="cl-82a35f12">island</span></p></th><th class="cl-82a55287"><p class="cl-82a54458"><span class="cl-82a35f12">bill_length_mm</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-82a55288"><p class="cl-82a5444e"><span class="cl-82a35f13">Adelie</span></p></td><td class="cl-82a55288"><p class="cl-82a5444e"><span class="cl-82a35f13">Torgersen</span></p></td><td class="cl-82a55289"><p class="cl-82a54458"><span class="cl-82a35f13">39.1</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82a5528a"><p class="cl-82a5444e"><span class="cl-82a35f13">Adelie</span></p></td><td class="cl-82a5528a"><p class="cl-82a5444e"><span class="cl-82a35f13">Dream</span></p></td><td class="cl-82a5528b"><p class="cl-82a54458"><span class="cl-82a35f13">42.3</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82a55290"><p class="cl-82a5444e"><span class="cl-82a35f13">Adelie</span></p></td><td class="cl-82a55290"><p class="cl-82a5444e"><span class="cl-82a35f13">Dream</span></p></td><td class="cl-82a55291"><p class="cl-82a54458"><span class="cl-82a35f13">43.2</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-82af1e92{}.cl-82ac1fe4{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82ac1fee{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82ad5b20{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82ad5b21{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82ad6b56{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b60{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b61{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b62{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b63{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b6a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b6b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82ad6b6c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-82af1e92'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82ad6b56"><p class="cl-82ad5b20"><span class="cl-82ac1fe4">species</span></p></th><th class="cl-82ad6b56"><p class="cl-82ad5b20"><span class="cl-82ac1fe4">sex</span></p></th><th class="cl-82ad6b60"><p class="cl-82ad5b21"><span class="cl-82ac1fe4">No. birds</span></p></th><th class="cl-82ad6b60"><p class="cl-82ad5b21"><span class="cl-82ac1fe4">Mean body mass (g)</span></p></th><th class="cl-82ad6b60"><p class="cl-82ad5b21"><span class="cl-82ac1fe4">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-82ad6b61"><p class="cl-82ad5b20"><span class="cl-82ac1fee">Adelie</span></p></td><td class="cl-82ad6b61"><p class="cl-82ad5b20"><span class="cl-82ac1fee">female</span></p></td><td class="cl-82ad6b62"><p class="cl-82ad5b21"><span class="cl-82ac1fee">73</span></p></td><td class="cl-82ad6b62"><p class="cl-82ad5b21"><span class="cl-82ac1fee">3,368.836</span></p></td><td class="cl-82ad6b62"><p class="cl-82ad5b21"><span class="cl-82ac1fee">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">Adelie</span></p></td><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">male</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">73</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">4,043.493</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">Chinstrap</span></p></td><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">female</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">34</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">3,527.206</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">Chinstrap</span></p></td><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">male</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">34</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">3,938.971</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">Gentoo</span></p></td><td class="cl-82ad6b63"><p class="cl-82ad5b20"><span class="cl-82ac1fee">female</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">58</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">4,679.741</span></p></td><td class="cl-82ad6b6a"><p class="cl-82ad5b21"><span class="cl-82ac1fee">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82ad6b6b"><p class="cl-82ad5b20"><span class="cl-82ac1fee">Gentoo</span></p></td><td class="cl-82ad6b6b"><p class="cl-82ad5b20"><span class="cl-82ac1fee">male</span></p></td><td class="cl-82ad6b6c"><p class="cl-82ad5b21"><span class="cl-82ac1fee">61</span></p></td><td class="cl-82ad6b6c"><p class="cl-82ad5b21"><span class="cl-82ac1fee">5,484.836</span></p></td><td class="cl-82ad6b6c"><p class="cl-82ad5b21"><span class="cl-82ac1fee">313.1586</span></p></td></tr></tbody></table></div>
```

``` r
# Round all numeric columns to zero decimal places
# And remove commas
ft %>%
  colformat_double(digits = 0,
                   big.mark = "")
```

```{=html}
<div class="tabwid"><style>.cl-82b4ad08{}.cl-82b125fc{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82b12606{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82b2576a{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82b2576b{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82b266f6{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b26700{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b26701{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b26702{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b2670a{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b2670b{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b2670c{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b2670d{width:0.75in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-82b4ad08'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82b266f6"><p class="cl-82b2576a"><span class="cl-82b125fc">species</span></p></th><th class="cl-82b266f6"><p class="cl-82b2576a"><span class="cl-82b125fc">sex</span></p></th><th class="cl-82b26700"><p class="cl-82b2576b"><span class="cl-82b125fc">No. birds</span></p></th><th class="cl-82b26700"><p class="cl-82b2576b"><span class="cl-82b125fc">Mean body mass (g)</span></p></th><th class="cl-82b26700"><p class="cl-82b2576b"><span class="cl-82b125fc">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-82b26701"><p class="cl-82b2576a"><span class="cl-82b12606">Adelie</span></p></td><td class="cl-82b26701"><p class="cl-82b2576a"><span class="cl-82b12606">female</span></p></td><td class="cl-82b26702"><p class="cl-82b2576b"><span class="cl-82b12606">73</span></p></td><td class="cl-82b26702"><p class="cl-82b2576b"><span class="cl-82b12606">3369</span></p></td><td class="cl-82b26702"><p class="cl-82b2576b"><span class="cl-82b12606">269</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">Adelie</span></p></td><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">male</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">73</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">4043</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">347</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">Chinstrap</span></p></td><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">female</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">34</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">3527</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">285</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">Chinstrap</span></p></td><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">male</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">34</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">3939</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">362</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">Gentoo</span></p></td><td class="cl-82b2670a"><p class="cl-82b2576a"><span class="cl-82b12606">female</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">58</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">4680</span></p></td><td class="cl-82b2670b"><p class="cl-82b2576b"><span class="cl-82b12606">282</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b2670c"><p class="cl-82b2576a"><span class="cl-82b12606">Gentoo</span></p></td><td class="cl-82b2670c"><p class="cl-82b2576a"><span class="cl-82b12606">male</span></p></td><td class="cl-82b2670d"><p class="cl-82b2576b"><span class="cl-82b12606">61</span></p></td><td class="cl-82b2670d"><p class="cl-82b2576b"><span class="cl-82b12606">5485</span></p></td><td class="cl-82b2670d"><p class="cl-82b2576b"><span class="cl-82b12606">313</span></p></td></tr></tbody></table></div>
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
<div class="tabwid"><style>.cl-82ba2b84{}.cl-82b7252e{font-family:'Times';font-size:11pt;font-weight:bold;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82b72538{font-family:'Times';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-82b867a4{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82b867ae{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-82b87758{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b87759{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 1.5pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b87762{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b87763{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b87764{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b87765{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 0.75pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b8776c{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-82b8776d{width:1in;background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(169, 169, 169, 1.00);border-top: 0.75pt solid rgba(169, 169, 169, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-82ba2b84'><thead><tr style="overflow-wrap:break-word;"><th class="cl-82b87758"><p class="cl-82b867a4"><span class="cl-82b7252e">species</span></p></th><th class="cl-82b87758"><p class="cl-82b867a4"><span class="cl-82b7252e">sex</span></p></th><th class="cl-82b87759"><p class="cl-82b867ae"><span class="cl-82b7252e">No. birds</span></p></th><th class="cl-82b87759"><p class="cl-82b867ae"><span class="cl-82b7252e">Mean body mass (g)</span></p></th><th class="cl-82b87759"><p class="cl-82b867ae"><span class="cl-82b7252e">SD body mass</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-82b87762"><p class="cl-82b867a4"><span class="cl-82b72538">Adelie</span></p></td><td class="cl-82b87762"><p class="cl-82b867a4"><span class="cl-82b72538">female</span></p></td><td class="cl-82b87763"><p class="cl-82b867ae"><span class="cl-82b72538">73</span></p></td><td class="cl-82b87763"><p class="cl-82b867ae"><span class="cl-82b72538">3,368.836</span></p></td><td class="cl-82b87763"><p class="cl-82b867ae"><span class="cl-82b72538">269.3801</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b87764"><p class="cl-82b867a4"><span class="cl-82b72538">male</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">73</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">4,043.493</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">346.8116</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-82b87764"><p class="cl-82b867a4"><span class="cl-82b72538">Chinstrap</span></p></td><td class="cl-82b87764"><p class="cl-82b867a4"><span class="cl-82b72538">female</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">34</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">3,527.206</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">285.3339</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b87764"><p class="cl-82b867a4"><span class="cl-82b72538">male</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">34</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">3,938.971</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">362.1376</span></p></td></tr><tr style="overflow-wrap:break-word;"><td  rowspan="2"class="cl-82b8776c"><p class="cl-82b867a4"><span class="cl-82b72538">Gentoo</span></p></td><td class="cl-82b87764"><p class="cl-82b867a4"><span class="cl-82b72538">female</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">58</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">4,679.741</span></p></td><td class="cl-82b87765"><p class="cl-82b867ae"><span class="cl-82b72538">281.5783</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-82b8776c"><p class="cl-82b867a4"><span class="cl-82b72538">male</span></p></td><td class="cl-82b8776d"><p class="cl-82b867ae"><span class="cl-82b72538">61</span></p></td><td class="cl-82b8776d"><p class="cl-82b867ae"><span class="cl-82b72538">5,484.836</span></p></td><td class="cl-82b8776d"><p class="cl-82b867ae"><span class="cl-82b72538">313.1586</span></p></td></tr></tbody></table></div>
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
