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
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
## ✔ purrr     1.0.2     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(palmerpenguins)

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

<img src="data-tables_files/figure-html/unnamed-chunk-1-1.png" width="672" />

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
  rename_all(str_to_sentence) %>%
  mutate(across(is.numeric, round)) #Let's talk about this one! What is this doing?
```

```
## Warning: There was 1 warning in `mutate()`.
## ℹ In argument: `across(is.numeric, round)`.
## Caused by warning:
## ! Use of bare predicate functions was deprecated in tidyselect 1.1.0.
## ℹ Please use wrap predicates in `where()` instead.
##   # Was:
##   data %>% select(is.numeric)
## 
##   # Now:
##   data %>% select(where(is.numeric))
```

``` r
write_csv(dat_summ_out, "outputs/penguin_summary.csv")
```

*What other formatting details might you want to consider before writing data to a file?*

## An R package for table design: `gt`

The method above is useful if you have a table or two to place in a report. What if you have a lot? The amount of extra time it takes to reformat each (and update every time something changes) might make it worth investing in a more automated solution. The [`gt`](https://gt.rstudio.com/articles/gt.html) package provides just that. It is very flexible, allowing you to create complex, nested tables. Although there is a bit of a learning curve to using the package, the time investment pays off if you produce a lot of tables.

Here's an example of our penguin data in `gt` format:


``` r
library(gt)
gt(dat_summ_out)
```

```{=html}
<div id="toodwbszwp" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#toodwbszwp table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#toodwbszwp thead, #toodwbszwp tbody, #toodwbszwp tfoot, #toodwbszwp tr, #toodwbszwp td, #toodwbszwp th {
  border-style: none;
}

#toodwbszwp p {
  margin: 0;
  padding: 0;
}

#toodwbszwp .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#toodwbszwp .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#toodwbszwp .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#toodwbszwp .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#toodwbszwp .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#toodwbszwp .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#toodwbszwp .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#toodwbszwp .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#toodwbszwp .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#toodwbszwp .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#toodwbszwp .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#toodwbszwp .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#toodwbszwp .gt_spanner_row {
  border-bottom-style: hidden;
}

#toodwbszwp .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#toodwbszwp .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#toodwbszwp .gt_from_md > :first-child {
  margin-top: 0;
}

#toodwbszwp .gt_from_md > :last-child {
  margin-bottom: 0;
}

#toodwbszwp .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#toodwbszwp .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#toodwbszwp .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#toodwbszwp .gt_row_group_first td {
  border-top-width: 2px;
}

#toodwbszwp .gt_row_group_first th {
  border-top-width: 2px;
}

#toodwbszwp .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#toodwbszwp .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#toodwbszwp .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#toodwbszwp .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#toodwbszwp .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#toodwbszwp .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#toodwbszwp .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#toodwbszwp .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#toodwbszwp .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#toodwbszwp .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#toodwbszwp .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#toodwbszwp .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#toodwbszwp .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#toodwbszwp .gt_left {
  text-align: left;
}

#toodwbszwp .gt_center {
  text-align: center;
}

#toodwbszwp .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#toodwbszwp .gt_font_normal {
  font-weight: normal;
}

#toodwbszwp .gt_font_bold {
  font-weight: bold;
}

#toodwbszwp .gt_font_italic {
  font-style: italic;
}

#toodwbszwp .gt_super {
  font-size: 65%;
}

#toodwbszwp .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#toodwbszwp .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#toodwbszwp .gt_indent_1 {
  text-indent: 5px;
}

#toodwbszwp .gt_indent_2 {
  text-indent: 10px;
}

#toodwbszwp .gt_indent_3 {
  text-indent: 15px;
}

#toodwbszwp .gt_indent_4 {
  text-indent: 20px;
}

#toodwbszwp .gt_indent_5 {
  text-indent: 25px;
}

#toodwbszwp .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#toodwbszwp div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <th colspan="4" class="gt_group_heading" scope="colgroup" id="Adelie">Adelie</th>
    </tr>
    <tr class="gt_row_group_first"><td headers="Adelie  Sex" class="gt_row gt_center">female</td>
<td headers="Adelie  No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie  Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Adelie  Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Adelie  Sex" class="gt_row gt_center">male</td>
<td headers="Adelie  No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie  Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Adelie  Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr class="gt_group_heading_row">
      <th colspan="4" class="gt_group_heading" scope="colgroup" id="Chinstrap">Chinstrap</th>
    </tr>
    <tr class="gt_row_group_first"><td headers="Chinstrap  Sex" class="gt_row gt_center">female</td>
<td headers="Chinstrap  No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap  Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Chinstrap  Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Chinstrap  Sex" class="gt_row gt_center">male</td>
<td headers="Chinstrap  No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap  Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Chinstrap  Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr class="gt_group_heading_row">
      <th colspan="4" class="gt_group_heading" scope="colgroup" id="Gentoo">Gentoo</th>
    </tr>
    <tr class="gt_row_group_first"><td headers="Gentoo  Sex" class="gt_row gt_center">female</td>
<td headers="Gentoo  No. birds" class="gt_row gt_right">58</td>
<td headers="Gentoo  Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Gentoo  Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Gentoo  Sex" class="gt_row gt_center">male</td>
<td headers="Gentoo  No. birds" class="gt_row gt_right">61</td>
<td headers="Gentoo  Mean body mass (g)" class="gt_row gt_right">5485</td>
<td headers="Gentoo  Sd body mass" class="gt_row gt_right">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

The formatting might be a little unexpected, because it's different than what we saw before. Now, instead of repeating species across sex, the table is nested by species. `gt` knew to do this because our tibble/data frame was grouped by species:


``` r
class(dat_summ_out)
```

```
## [1] "grouped_df" "tbl_df"     "tbl"        "data.frame"
```

``` r
groups(dat_summ_out)
```

```
## [[1]]
## Species
```

By default `dplyr` maintains the groups in `group_by()` even after the summary is done. We could ungroup the table, in which case, our output would change:


``` r
dat_summ_out %>%
  ungroup() %>%
  gt()
```

```{=html}
<div id="uyvlptdqsy" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#uyvlptdqsy table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#uyvlptdqsy thead, #uyvlptdqsy tbody, #uyvlptdqsy tfoot, #uyvlptdqsy tr, #uyvlptdqsy td, #uyvlptdqsy th {
  border-style: none;
}

#uyvlptdqsy p {
  margin: 0;
  padding: 0;
}

#uyvlptdqsy .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#uyvlptdqsy .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#uyvlptdqsy .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#uyvlptdqsy .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#uyvlptdqsy .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#uyvlptdqsy .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#uyvlptdqsy .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#uyvlptdqsy .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#uyvlptdqsy .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#uyvlptdqsy .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#uyvlptdqsy .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#uyvlptdqsy .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#uyvlptdqsy .gt_spanner_row {
  border-bottom-style: hidden;
}

#uyvlptdqsy .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#uyvlptdqsy .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#uyvlptdqsy .gt_from_md > :first-child {
  margin-top: 0;
}

#uyvlptdqsy .gt_from_md > :last-child {
  margin-bottom: 0;
}

#uyvlptdqsy .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#uyvlptdqsy .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#uyvlptdqsy .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#uyvlptdqsy .gt_row_group_first td {
  border-top-width: 2px;
}

#uyvlptdqsy .gt_row_group_first th {
  border-top-width: 2px;
}

#uyvlptdqsy .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#uyvlptdqsy .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#uyvlptdqsy .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#uyvlptdqsy .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#uyvlptdqsy .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#uyvlptdqsy .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#uyvlptdqsy .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#uyvlptdqsy .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#uyvlptdqsy .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#uyvlptdqsy .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#uyvlptdqsy .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#uyvlptdqsy .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#uyvlptdqsy .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#uyvlptdqsy .gt_left {
  text-align: left;
}

#uyvlptdqsy .gt_center {
  text-align: center;
}

#uyvlptdqsy .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#uyvlptdqsy .gt_font_normal {
  font-weight: normal;
}

#uyvlptdqsy .gt_font_bold {
  font-weight: bold;
}

#uyvlptdqsy .gt_font_italic {
  font-style: italic;
}

#uyvlptdqsy .gt_super {
  font-size: 65%;
}

#uyvlptdqsy .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#uyvlptdqsy .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#uyvlptdqsy .gt_indent_1 {
  text-indent: 5px;
}

#uyvlptdqsy .gt_indent_2 {
  text-indent: 10px;
}

#uyvlptdqsy .gt_indent_3 {
  text-indent: 15px;
}

#uyvlptdqsy .gt_indent_4 {
  text-indent: 20px;
}

#uyvlptdqsy .gt_indent_5 {
  text-indent: 25px;
}

#uyvlptdqsy .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#uyvlptdqsy div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" scope="col" id="Species">Species</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">58</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">61</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">5485</td>
<td headers="Sd body mass" class="gt_row gt_right">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

This output is an image. You could copy it into a document, but that it wouldn't be embedded in the same way a table is. Instead, we can use the `gtsave()` function with a given file ending to save our output in a format for a word processor.


``` r
dat_summ_out %>%
  gt() %>%
  gtsave("outputs/penguin_bm_summary.docx")
```

<div class="figure" style="text-align: center">
<img src="images/gtable-out.png" alt="The default gtable format in Word" width="80%" />
<p class="caption">(\#fig:unnamed-chunk-8)The default gtable format in Word</p>
</div>

Now this format is readable by Word; if you click in the table, the usual formatting options will appear. Next, let's work on formatting the table to look exactly like the manually created one. To start off, we'll create an object for our table so we can modify it:


``` r
gt_penguins <- gt(ungroup(dat_summ_out))
```

I usually write my documents in Times New Roman, so first I'll change the font using the `opt_table_font()` function, which defines fonts for the entire table:


``` r
gt_penguins <- opt_table_font(gt_penguins, font = "Times")
gt_penguins
```

```{=html}
<div id="psokevglnp" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#psokevglnp table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#psokevglnp thead, #psokevglnp tbody, #psokevglnp tfoot, #psokevglnp tr, #psokevglnp td, #psokevglnp th {
  border-style: none;
}

#psokevglnp p {
  margin: 0;
  padding: 0;
}

#psokevglnp .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#psokevglnp .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#psokevglnp .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#psokevglnp .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#psokevglnp .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#psokevglnp .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#psokevglnp .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#psokevglnp .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#psokevglnp .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#psokevglnp .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#psokevglnp .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#psokevglnp .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#psokevglnp .gt_spanner_row {
  border-bottom-style: hidden;
}

#psokevglnp .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#psokevglnp .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#psokevglnp .gt_from_md > :first-child {
  margin-top: 0;
}

#psokevglnp .gt_from_md > :last-child {
  margin-bottom: 0;
}

#psokevglnp .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#psokevglnp .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#psokevglnp .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#psokevglnp .gt_row_group_first td {
  border-top-width: 2px;
}

#psokevglnp .gt_row_group_first th {
  border-top-width: 2px;
}

#psokevglnp .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#psokevglnp .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#psokevglnp .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#psokevglnp .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#psokevglnp .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#psokevglnp .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#psokevglnp .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#psokevglnp .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#psokevglnp .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#psokevglnp .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#psokevglnp .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#psokevglnp .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#psokevglnp .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#psokevglnp .gt_left {
  text-align: left;
}

#psokevglnp .gt_center {
  text-align: center;
}

#psokevglnp .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#psokevglnp .gt_font_normal {
  font-weight: normal;
}

#psokevglnp .gt_font_bold {
  font-weight: bold;
}

#psokevglnp .gt_font_italic {
  font-style: italic;
}

#psokevglnp .gt_super {
  font-size: 65%;
}

#psokevglnp .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#psokevglnp .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#psokevglnp .gt_indent_1 {
  text-indent: 5px;
}

#psokevglnp .gt_indent_2 {
  text-indent: 10px;
}

#psokevglnp .gt_indent_3 {
  text-indent: 15px;
}

#psokevglnp .gt_indent_4 {
  text-indent: 20px;
}

#psokevglnp .gt_indent_5 {
  text-indent: 25px;
}

#psokevglnp .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#psokevglnp div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" scope="col" id="Species">Species</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">58</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">61</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">5485</td>
<td headers="Sd body mass" class="gt_row gt_right">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

Now I will change the header row to bold font. This time I use the `tab_style()` function, since I only want to change properties for specific cells, not the entire table:


``` r
gt_penguins <- tab_style(gt_penguins, 
                         style = cell_text(weight = "bold"), 
                         location = cells_column_labels())
gt_penguins
```

```{=html}
<div id="mlopavvqtq" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#mlopavvqtq table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#mlopavvqtq thead, #mlopavvqtq tbody, #mlopavvqtq tfoot, #mlopavvqtq tr, #mlopavvqtq td, #mlopavvqtq th {
  border-style: none;
}

#mlopavvqtq p {
  margin: 0;
  padding: 0;
}

#mlopavvqtq .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#mlopavvqtq .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#mlopavvqtq .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#mlopavvqtq .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#mlopavvqtq .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#mlopavvqtq .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#mlopavvqtq .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#mlopavvqtq .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#mlopavvqtq .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#mlopavvqtq .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#mlopavvqtq .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#mlopavvqtq .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#mlopavvqtq .gt_spanner_row {
  border-bottom-style: hidden;
}

#mlopavvqtq .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#mlopavvqtq .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#mlopavvqtq .gt_from_md > :first-child {
  margin-top: 0;
}

#mlopavvqtq .gt_from_md > :last-child {
  margin-bottom: 0;
}

#mlopavvqtq .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#mlopavvqtq .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#mlopavvqtq .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#mlopavvqtq .gt_row_group_first td {
  border-top-width: 2px;
}

#mlopavvqtq .gt_row_group_first th {
  border-top-width: 2px;
}

#mlopavvqtq .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#mlopavvqtq .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#mlopavvqtq .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#mlopavvqtq .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#mlopavvqtq .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#mlopavvqtq .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#mlopavvqtq .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#mlopavvqtq .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#mlopavvqtq .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#mlopavvqtq .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#mlopavvqtq .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#mlopavvqtq .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#mlopavvqtq .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#mlopavvqtq .gt_left {
  text-align: left;
}

#mlopavvqtq .gt_center {
  text-align: center;
}

#mlopavvqtq .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#mlopavvqtq .gt_font_normal {
  font-weight: normal;
}

#mlopavvqtq .gt_font_bold {
  font-weight: bold;
}

#mlopavvqtq .gt_font_italic {
  font-style: italic;
}

#mlopavvqtq .gt_super {
  font-size: 65%;
}

#mlopavvqtq .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#mlopavvqtq .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#mlopavvqtq .gt_indent_1 {
  text-indent: 5px;
}

#mlopavvqtq .gt_indent_2 {
  text-indent: 10px;
}

#mlopavvqtq .gt_indent_3 {
  text-indent: 15px;
}

#mlopavvqtq .gt_indent_4 {
  text-indent: 20px;
}

#mlopavvqtq .gt_indent_5 {
  text-indent: 25px;
}

#mlopavvqtq .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#mlopavvqtq div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Species">Species</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">58</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">61</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">5485</td>
<td headers="Sd body mass" class="gt_row gt_right">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

Looking in the help funciton for `tab_style()`, I learned that to the `style` argument I need to specify a function like `cell_text()` or `cell_fill()`, then the options within that function. The `location` argument indicates which cells to edit. Again, this should be a function starting with `cells_*`, which is essentially a selection argument. The help functions will be useful to you here to find the cells you are looking for.

Now I will work on the borders. This workflow is similar to the one I used above. First, I remove all the borders using an `opt_*` function, then I add borders back where I want them using `tab_style()`:


``` r
gt_penguins <- gt_penguins %>%
  opt_table_lines(extent = "none") %>%
  tab_style(style = cell_borders(sides = c("top", "bottom")),
            location = cells_column_labels()) %>%
  tab_style(style = cell_borders(sides = "bottom"),
            location = cells_body(rows = nrow(dat_summ_out))) #add bottom border to the last row
gt_penguins
```

```{=html}
<div id="dqdoxvofmj" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#dqdoxvofmj table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#dqdoxvofmj thead, #dqdoxvofmj tbody, #dqdoxvofmj tfoot, #dqdoxvofmj tr, #dqdoxvofmj td, #dqdoxvofmj th {
  border-style: none;
}

#dqdoxvofmj p {
  margin: 0;
  padding: 0;
}

#dqdoxvofmj .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#dqdoxvofmj .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#dqdoxvofmj .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#dqdoxvofmj .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#dqdoxvofmj .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#dqdoxvofmj .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#dqdoxvofmj .gt_col_headings {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#dqdoxvofmj .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#dqdoxvofmj .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#dqdoxvofmj .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#dqdoxvofmj .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#dqdoxvofmj .gt_column_spanner {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#dqdoxvofmj .gt_spanner_row {
  border-bottom-style: hidden;
}

#dqdoxvofmj .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#dqdoxvofmj .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#dqdoxvofmj .gt_from_md > :first-child {
  margin-top: 0;
}

#dqdoxvofmj .gt_from_md > :last-child {
  margin-bottom: 0;
}

#dqdoxvofmj .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: none;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#dqdoxvofmj .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#dqdoxvofmj .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#dqdoxvofmj .gt_row_group_first td {
  border-top-width: 2px;
}

#dqdoxvofmj .gt_row_group_first th {
  border-top-width: 2px;
}

#dqdoxvofmj .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#dqdoxvofmj .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#dqdoxvofmj .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#dqdoxvofmj .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#dqdoxvofmj .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#dqdoxvofmj .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#dqdoxvofmj .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#dqdoxvofmj .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#dqdoxvofmj .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#dqdoxvofmj .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#dqdoxvofmj .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#dqdoxvofmj .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#dqdoxvofmj .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#dqdoxvofmj .gt_left {
  text-align: left;
}

#dqdoxvofmj .gt_center {
  text-align: center;
}

#dqdoxvofmj .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#dqdoxvofmj .gt_font_normal {
  font-weight: normal;
}

#dqdoxvofmj .gt_font_bold {
  font-weight: bold;
}

#dqdoxvofmj .gt_font_italic {
  font-style: italic;
}

#dqdoxvofmj .gt_super {
  font-size: 65%;
}

#dqdoxvofmj .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#dqdoxvofmj .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#dqdoxvofmj .gt_indent_1 {
  text-indent: 5px;
}

#dqdoxvofmj .gt_indent_2 {
  text-indent: 10px;
}

#dqdoxvofmj .gt_indent_3 {
  text-indent: 15px;
}

#dqdoxvofmj .gt_indent_4 {
  text-indent: 20px;
}

#dqdoxvofmj .gt_indent_5 {
  text-indent: 25px;
}

#dqdoxvofmj .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#dqdoxvofmj div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Species">Species</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Adelie</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center">male</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr><td headers="Species" class="gt_row gt_center">Gentoo</td>
<td headers="Sex" class="gt_row gt_center">female</td>
<td headers="No. birds" class="gt_row gt_right">58</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Species" class="gt_row gt_center" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">Gentoo</td>
<td headers="Sex" class="gt_row gt_center" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">male</td>
<td headers="No. birds" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">61</td>
<td headers="Mean body mass (g)" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">5485</td>
<td headers="Sd body mass" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

All that's left is the alignment. I want to left-align my character columns and right-align my numeric columns.


``` r
gt_penguins <- gt_penguins %>%
  tab_style(style = cell_text(align = "left"),
            location = list(cells_column_labels(columns = c("Species","Sex")),
                            cells_body(columns = c("Species","Sex"))))
gt_penguins
```

```{=html}
<div id="mokyeunwsm" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#mokyeunwsm table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#mokyeunwsm thead, #mokyeunwsm tbody, #mokyeunwsm tfoot, #mokyeunwsm tr, #mokyeunwsm td, #mokyeunwsm th {
  border-style: none;
}

#mokyeunwsm p {
  margin: 0;
  padding: 0;
}

#mokyeunwsm .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#mokyeunwsm .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#mokyeunwsm .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#mokyeunwsm .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#mokyeunwsm .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#mokyeunwsm .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#mokyeunwsm .gt_col_headings {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#mokyeunwsm .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#mokyeunwsm .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#mokyeunwsm .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#mokyeunwsm .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#mokyeunwsm .gt_column_spanner {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#mokyeunwsm .gt_spanner_row {
  border-bottom-style: hidden;
}

#mokyeunwsm .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#mokyeunwsm .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#mokyeunwsm .gt_from_md > :first-child {
  margin-top: 0;
}

#mokyeunwsm .gt_from_md > :last-child {
  margin-bottom: 0;
}

#mokyeunwsm .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: none;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#mokyeunwsm .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#mokyeunwsm .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#mokyeunwsm .gt_row_group_first td {
  border-top-width: 2px;
}

#mokyeunwsm .gt_row_group_first th {
  border-top-width: 2px;
}

#mokyeunwsm .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#mokyeunwsm .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#mokyeunwsm .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#mokyeunwsm .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#mokyeunwsm .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#mokyeunwsm .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#mokyeunwsm .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#mokyeunwsm .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#mokyeunwsm .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#mokyeunwsm .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#mokyeunwsm .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#mokyeunwsm .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#mokyeunwsm .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#mokyeunwsm .gt_left {
  text-align: left;
}

#mokyeunwsm .gt_center {
  text-align: center;
}

#mokyeunwsm .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#mokyeunwsm .gt_font_normal {
  font-weight: normal;
}

#mokyeunwsm .gt_font_bold {
  font-weight: bold;
}

#mokyeunwsm .gt_font_italic {
  font-style: italic;
}

#mokyeunwsm .gt_super {
  font-size: 65%;
}

#mokyeunwsm .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#mokyeunwsm .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#mokyeunwsm .gt_indent_1 {
  text-indent: 5px;
}

#mokyeunwsm .gt_indent_2 {
  text-indent: 10px;
}

#mokyeunwsm .gt_indent_3 {
  text-indent: 15px;
}

#mokyeunwsm .gt_indent_4 {
  text-indent: 20px;
}

#mokyeunwsm .gt_indent_5 {
  text-indent: 25px;
}

#mokyeunwsm .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#mokyeunwsm div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold; text-align: left; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Species">Species</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold; text-align: left; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td headers="Species" class="gt_row gt_center" style="text-align: left;">Adelie</td>
<td headers="Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Species" class="gt_row gt_center" style="text-align: left;">Adelie</td>
<td headers="Sex" class="gt_row gt_center" style="text-align: left;">male</td>
<td headers="No. birds" class="gt_row gt_right">73</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr><td headers="Species" class="gt_row gt_center" style="text-align: left;">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Species" class="gt_row gt_center" style="text-align: left;">Chinstrap</td>
<td headers="Sex" class="gt_row gt_center" style="text-align: left;">male</td>
<td headers="No. birds" class="gt_row gt_right">34</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr><td headers="Species" class="gt_row gt_center" style="text-align: left;">Gentoo</td>
<td headers="Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="No. birds" class="gt_row gt_right">58</td>
<td headers="Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Species" class="gt_row gt_center" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000; text-align: left;">Gentoo</td>
<td headers="Sex" class="gt_row gt_center" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000; text-align: left;">male</td>
<td headers="No. birds" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">61</td>
<td headers="Mean body mass (g)" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">5485</td>
<td headers="Sd body mass" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

Notice that because I wanted to do this to both the body cells and the column labels, I had to specify both of those and combine them as a `list`.

Finally, let's do the same thing, but for the grouped data. Now, we need to deal with design choices related to the groups, but first I will change the font:


``` r
gt_penguins <- dat_summ_out %>%
  gt() %>%
  opt_table_font(font = "Times") %>%
  tab_style(style = cell_text(weight = "bold"), 
                         location = cells_column_labels()) 
gt_penguins 
```

```{=html}
<div id="rstdqnscrd" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#rstdqnscrd table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#rstdqnscrd thead, #rstdqnscrd tbody, #rstdqnscrd tfoot, #rstdqnscrd tr, #rstdqnscrd td, #rstdqnscrd th {
  border-style: none;
}

#rstdqnscrd p {
  margin: 0;
  padding: 0;
}

#rstdqnscrd .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#rstdqnscrd .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#rstdqnscrd .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#rstdqnscrd .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#rstdqnscrd .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#rstdqnscrd .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rstdqnscrd .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#rstdqnscrd .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#rstdqnscrd .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#rstdqnscrd .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#rstdqnscrd .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#rstdqnscrd .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#rstdqnscrd .gt_spanner_row {
  border-bottom-style: hidden;
}

#rstdqnscrd .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#rstdqnscrd .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#rstdqnscrd .gt_from_md > :first-child {
  margin-top: 0;
}

#rstdqnscrd .gt_from_md > :last-child {
  margin-bottom: 0;
}

#rstdqnscrd .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#rstdqnscrd .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#rstdqnscrd .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#rstdqnscrd .gt_row_group_first td {
  border-top-width: 2px;
}

#rstdqnscrd .gt_row_group_first th {
  border-top-width: 2px;
}

#rstdqnscrd .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rstdqnscrd .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#rstdqnscrd .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#rstdqnscrd .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rstdqnscrd .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rstdqnscrd .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#rstdqnscrd .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#rstdqnscrd .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#rstdqnscrd .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rstdqnscrd .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#rstdqnscrd .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#rstdqnscrd .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#rstdqnscrd .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#rstdqnscrd .gt_left {
  text-align: left;
}

#rstdqnscrd .gt_center {
  text-align: center;
}

#rstdqnscrd .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#rstdqnscrd .gt_font_normal {
  font-weight: normal;
}

#rstdqnscrd .gt_font_bold {
  font-weight: bold;
}

#rstdqnscrd .gt_font_italic {
  font-style: italic;
}

#rstdqnscrd .gt_super {
  font-size: 65%;
}

#rstdqnscrd .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#rstdqnscrd .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#rstdqnscrd .gt_indent_1 {
  text-indent: 5px;
}

#rstdqnscrd .gt_indent_2 {
  text-indent: 10px;
}

#rstdqnscrd .gt_indent_3 {
  text-indent: 15px;
}

#rstdqnscrd .gt_indent_4 {
  text-indent: 20px;
}

#rstdqnscrd .gt_indent_5 {
  text-indent: 25px;
}

#rstdqnscrd .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#rstdqnscrd div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_group_heading_row">
      <th colspan="4" class="gt_group_heading" scope="colgroup" id="Adelie">Adelie</th>
    </tr>
    <tr class="gt_row_group_first"><td headers="Adelie  Sex" class="gt_row gt_center">female</td>
<td headers="Adelie  No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie  Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Adelie  Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Adelie  Sex" class="gt_row gt_center">male</td>
<td headers="Adelie  No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie  Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Adelie  Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr class="gt_group_heading_row">
      <th colspan="4" class="gt_group_heading" scope="colgroup" id="Chinstrap">Chinstrap</th>
    </tr>
    <tr class="gt_row_group_first"><td headers="Chinstrap  Sex" class="gt_row gt_center">female</td>
<td headers="Chinstrap  No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap  Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Chinstrap  Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Chinstrap  Sex" class="gt_row gt_center">male</td>
<td headers="Chinstrap  No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap  Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Chinstrap  Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr class="gt_group_heading_row">
      <th colspan="4" class="gt_group_heading" scope="colgroup" id="Gentoo">Gentoo</th>
    </tr>
    <tr class="gt_row_group_first"><td headers="Gentoo  Sex" class="gt_row gt_center">female</td>
<td headers="Gentoo  No. birds" class="gt_row gt_right">58</td>
<td headers="Gentoo  Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Gentoo  Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Gentoo  Sex" class="gt_row gt_center">male</td>
<td headers="Gentoo  No. birds" class="gt_row gt_right">61</td>
<td headers="Gentoo  Mean body mass (g)" class="gt_row gt_right">5485</td>
<td headers="Gentoo  Sd body mass" class="gt_row gt_right">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

Now, notice that the species are on their own line. I typically would include them as another column, so I'll do that using the strategy I found on [this Stack Overflow page](https://stackoverflow.com/questions/76260847/how-can-i-put-the-groups-in-an-extra-column-in-a-gt-table):


``` r
gt_penguins <- gt_penguins %>%
  tab_options(row_group.as_column = TRUE)
gt_penguins 
```

```{=html}
<div id="tunliotabo" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#tunliotabo table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#tunliotabo thead, #tunliotabo tbody, #tunliotabo tfoot, #tunliotabo tr, #tunliotabo td, #tunliotabo th {
  border-style: none;
}

#tunliotabo p {
  margin: 0;
  padding: 0;
}

#tunliotabo .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#tunliotabo .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#tunliotabo .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#tunliotabo .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#tunliotabo .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#tunliotabo .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tunliotabo .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#tunliotabo .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#tunliotabo .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#tunliotabo .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#tunliotabo .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#tunliotabo .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#tunliotabo .gt_spanner_row {
  border-bottom-style: hidden;
}

#tunliotabo .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#tunliotabo .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#tunliotabo .gt_from_md > :first-child {
  margin-top: 0;
}

#tunliotabo .gt_from_md > :last-child {
  margin-bottom: 0;
}

#tunliotabo .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#tunliotabo .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#tunliotabo .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#tunliotabo .gt_row_group_first td {
  border-top-width: 2px;
}

#tunliotabo .gt_row_group_first th {
  border-top-width: 2px;
}

#tunliotabo .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tunliotabo .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#tunliotabo .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#tunliotabo .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tunliotabo .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tunliotabo .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#tunliotabo .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#tunliotabo .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#tunliotabo .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tunliotabo .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#tunliotabo .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#tunliotabo .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#tunliotabo .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#tunliotabo .gt_left {
  text-align: left;
}

#tunliotabo .gt_center {
  text-align: center;
}

#tunliotabo .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#tunliotabo .gt_font_normal {
  font-weight: normal;
}

#tunliotabo .gt_font_bold {
  font-weight: bold;
}

#tunliotabo .gt_font_italic {
  font-style: italic;
}

#tunliotabo .gt_super {
  font-size: 65%;
}

#tunliotabo .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#tunliotabo .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#tunliotabo .gt_indent_1 {
  text-indent: 5px;
}

#tunliotabo .gt_indent_2 {
  text-indent: 10px;
}

#tunliotabo .gt_indent_3 {
  text-indent: 15px;
}

#tunliotabo .gt_indent_4 {
  text-indent: 20px;
}

#tunliotabo .gt_indent_5 {
  text-indent: 25px;
}

#tunliotabo .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#tunliotabo div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col" id="a::stub"></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_row_group_first"><td headers="Adelie stub_1_1 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Adelie</td>
<td headers="Adelie stub_1_1 Sex" class="gt_row gt_center">female</td>
<td headers="Adelie stub_1_1 No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie stub_1_1 Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Adelie stub_1_1 Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Adelie Sex_2 Sex" class="gt_row gt_center">male</td>
<td headers="Adelie Sex_2 No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie Sex_2 Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Adelie Sex_2 Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr class="gt_row_group_first"><td headers="Chinstrap stub_1_3 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Chinstrap</td>
<td headers="Chinstrap stub_1_3 Sex" class="gt_row gt_center">female</td>
<td headers="Chinstrap stub_1_3 No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap stub_1_3 Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Chinstrap stub_1_3 Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Chinstrap Sex_4 Sex" class="gt_row gt_center">male</td>
<td headers="Chinstrap Sex_4 No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap Sex_4 Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Chinstrap Sex_4 Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr class="gt_row_group_first"><td headers="Gentoo stub_1_5 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Gentoo</td>
<td headers="Gentoo stub_1_5 Sex" class="gt_row gt_center">female</td>
<td headers="Gentoo stub_1_5 No. birds" class="gt_row gt_right">58</td>
<td headers="Gentoo stub_1_5 Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Gentoo stub_1_5 Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Gentoo Sex_6 Sex" class="gt_row gt_center">male</td>
<td headers="Gentoo Sex_6 No. birds" class="gt_row gt_right">61</td>
<td headers="Gentoo Sex_6 Mean body mass (g)" class="gt_row gt_right">5485</td>
<td headers="Gentoo Sex_6 Sd body mass" class="gt_row gt_right">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

Now I can use the same lines and alignment as I did before:


``` r
gt_penguins %>%
  tab_style(style = cell_text(align = "left"),
            location = list(cells_column_labels(columns = "Sex"),
                            cells_body(columns = "Sex"))) %>%
  opt_table_lines(extent = "none") %>%
  tab_style(style = cell_borders(sides = c("top", "bottom")),
            location = cells_column_labels()) %>%
  tab_style(style = cell_borders(sides = "bottom"),
            location = cells_body(rows = nrow(dat_summ_out))) 
```

```{=html}
<div id="hkbggkiaph" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#hkbggkiaph table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#hkbggkiaph thead, #hkbggkiaph tbody, #hkbggkiaph tfoot, #hkbggkiaph tr, #hkbggkiaph td, #hkbggkiaph th {
  border-style: none;
}

#hkbggkiaph p {
  margin: 0;
  padding: 0;
}

#hkbggkiaph .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#hkbggkiaph .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#hkbggkiaph .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#hkbggkiaph .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#hkbggkiaph .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#hkbggkiaph .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hkbggkiaph .gt_col_headings {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#hkbggkiaph .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#hkbggkiaph .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#hkbggkiaph .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#hkbggkiaph .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#hkbggkiaph .gt_column_spanner {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#hkbggkiaph .gt_spanner_row {
  border-bottom-style: hidden;
}

#hkbggkiaph .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#hkbggkiaph .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#hkbggkiaph .gt_from_md > :first-child {
  margin-top: 0;
}

#hkbggkiaph .gt_from_md > :last-child {
  margin-bottom: 0;
}

#hkbggkiaph .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: none;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#hkbggkiaph .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#hkbggkiaph .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#hkbggkiaph .gt_row_group_first td {
  border-top-width: 2px;
}

#hkbggkiaph .gt_row_group_first th {
  border-top-width: 2px;
}

#hkbggkiaph .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hkbggkiaph .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#hkbggkiaph .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#hkbggkiaph .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hkbggkiaph .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hkbggkiaph .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#hkbggkiaph .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#hkbggkiaph .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#hkbggkiaph .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hkbggkiaph .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#hkbggkiaph .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hkbggkiaph .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#hkbggkiaph .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hkbggkiaph .gt_left {
  text-align: left;
}

#hkbggkiaph .gt_center {
  text-align: center;
}

#hkbggkiaph .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#hkbggkiaph .gt_font_normal {
  font-weight: normal;
}

#hkbggkiaph .gt_font_bold {
  font-weight: bold;
}

#hkbggkiaph .gt_font_italic {
  font-style: italic;
}

#hkbggkiaph .gt_super {
  font-size: 65%;
}

#hkbggkiaph .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#hkbggkiaph .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#hkbggkiaph .gt_indent_1 {
  text-indent: 5px;
}

#hkbggkiaph .gt_indent_2 {
  text-indent: 10px;
}

#hkbggkiaph .gt_indent_3 {
  text-indent: 15px;
}

#hkbggkiaph .gt_indent_4 {
  text-indent: 20px;
}

#hkbggkiaph .gt_indent_5 {
  text-indent: 25px;
}

#hkbggkiaph .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#hkbggkiaph div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col" id="a::stub"></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold; text-align: left; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_row_group_first"><td headers="Adelie stub_1_1 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Adelie</td>
<td headers="Adelie stub_1_1 Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="Adelie stub_1_1 No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie stub_1_1 Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Adelie stub_1_1 Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Adelie Sex_2 Sex" class="gt_row gt_center" style="text-align: left;">male</td>
<td headers="Adelie Sex_2 No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie Sex_2 Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Adelie Sex_2 Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr class="gt_row_group_first"><td headers="Chinstrap stub_1_3 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Chinstrap</td>
<td headers="Chinstrap stub_1_3 Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="Chinstrap stub_1_3 No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap stub_1_3 Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Chinstrap stub_1_3 Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Chinstrap Sex_4 Sex" class="gt_row gt_center" style="text-align: left;">male</td>
<td headers="Chinstrap Sex_4 No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap Sex_4 Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Chinstrap Sex_4 Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr class="gt_row_group_first"><td headers="Gentoo stub_1_5 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Gentoo</td>
<td headers="Gentoo stub_1_5 Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="Gentoo stub_1_5 No. birds" class="gt_row gt_right">58</td>
<td headers="Gentoo stub_1_5 Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Gentoo stub_1_5 Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Gentoo Sex_6 Sex" class="gt_row gt_center" style="text-align: left; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">male</td>
<td headers="Gentoo Sex_6 No. birds" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">61</td>
<td headers="Gentoo Sex_6 Mean body mass (g)" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">5485</td>
<td headers="Gentoo Sex_6 Sd body mass" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

Oops, that's not what I wanted. I guess `cells_column_labels()` only gives me named columns, whereas the species column no longer has a name. Trying again:


``` r
gt_penguins <- gt_penguins %>%
  tab_style(style = cell_text(align = "left"),
            location = list(cells_column_labels(columns = "Sex"),
                            cells_body(columns = "Sex"))) %>%
  opt_table_lines(extent = "none") %>%
  tab_style(style = cell_borders(sides = c("top", "bottom")),
            location = list(cells_column_labels(), 
                            cells_stubhead())) %>%
  tab_style(style = cell_borders(sides = "bottom"),
            location = list(cells_body(rows = nrow(dat_summ_out)),
                            cells_row_groups(groups = n_distinct(dat_summ_out$Species)))) 
gt_penguins
```

```{=html}
<div id="qggqtzhjls" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#qggqtzhjls table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#qggqtzhjls thead, #qggqtzhjls tbody, #qggqtzhjls tfoot, #qggqtzhjls tr, #qggqtzhjls td, #qggqtzhjls th {
  border-style: none;
}

#qggqtzhjls p {
  margin: 0;
  padding: 0;
}

#qggqtzhjls .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#qggqtzhjls .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#qggqtzhjls .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#qggqtzhjls .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#qggqtzhjls .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#qggqtzhjls .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qggqtzhjls .gt_col_headings {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#qggqtzhjls .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#qggqtzhjls .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#qggqtzhjls .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#qggqtzhjls .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#qggqtzhjls .gt_column_spanner {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#qggqtzhjls .gt_spanner_row {
  border-bottom-style: hidden;
}

#qggqtzhjls .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#qggqtzhjls .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#qggqtzhjls .gt_from_md > :first-child {
  margin-top: 0;
}

#qggqtzhjls .gt_from_md > :last-child {
  margin-bottom: 0;
}

#qggqtzhjls .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: none;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#qggqtzhjls .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#qggqtzhjls .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#qggqtzhjls .gt_row_group_first td {
  border-top-width: 2px;
}

#qggqtzhjls .gt_row_group_first th {
  border-top-width: 2px;
}

#qggqtzhjls .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qggqtzhjls .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#qggqtzhjls .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#qggqtzhjls .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qggqtzhjls .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qggqtzhjls .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#qggqtzhjls .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#qggqtzhjls .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#qggqtzhjls .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qggqtzhjls .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#qggqtzhjls .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qggqtzhjls .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#qggqtzhjls .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qggqtzhjls .gt_left {
  text-align: left;
}

#qggqtzhjls .gt_center {
  text-align: center;
}

#qggqtzhjls .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#qggqtzhjls .gt_font_normal {
  font-weight: normal;
}

#qggqtzhjls .gt_font_bold {
  font-weight: bold;
}

#qggqtzhjls .gt_font_italic {
  font-style: italic;
}

#qggqtzhjls .gt_super {
  font-size: 65%;
}

#qggqtzhjls .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#qggqtzhjls .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#qggqtzhjls .gt_indent_1 {
  text-indent: 5px;
}

#qggqtzhjls .gt_indent_2 {
  text-indent: 10px;
}

#qggqtzhjls .gt_indent_3 {
  text-indent: 15px;
}

#qggqtzhjls .gt_indent_4 {
  text-indent: 20px;
}

#qggqtzhjls .gt_indent_5 {
  text-indent: 25px;
}

#qggqtzhjls .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#qggqtzhjls div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
  height: 0px !important;
}
</style>
<table class="gt_table" data-quarto-disable-processing="false" data-quarto-bootstrap="false">
  <thead>
    <tr class="gt_col_headings">
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" style="border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="a::stub"></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1" style="font-weight: bold; text-align: left; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sex">Sex</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="No.-birds">No. birds</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Mean-body-mass-(g)">Mean body mass (g)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" style="font-weight: bold; border-top-width: 1px; border-top-style: solid; border-top-color: #000000; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;" scope="col" id="Sd-body-mass">Sd body mass</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr class="gt_row_group_first"><td headers="Adelie stub_1_1 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Adelie</td>
<td headers="Adelie stub_1_1 Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="Adelie stub_1_1 No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie stub_1_1 Mean body mass (g)" class="gt_row gt_right">3369</td>
<td headers="Adelie stub_1_1 Sd body mass" class="gt_row gt_right">269</td></tr>
    <tr><td headers="Adelie Sex_2 Sex" class="gt_row gt_center" style="text-align: left;">male</td>
<td headers="Adelie Sex_2 No. birds" class="gt_row gt_right">73</td>
<td headers="Adelie Sex_2 Mean body mass (g)" class="gt_row gt_right">4043</td>
<td headers="Adelie Sex_2 Sd body mass" class="gt_row gt_right">347</td></tr>
    <tr class="gt_row_group_first"><td headers="Chinstrap stub_1_3 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group">Chinstrap</td>
<td headers="Chinstrap stub_1_3 Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="Chinstrap stub_1_3 No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap stub_1_3 Mean body mass (g)" class="gt_row gt_right">3527</td>
<td headers="Chinstrap stub_1_3 Sd body mass" class="gt_row gt_right">285</td></tr>
    <tr><td headers="Chinstrap Sex_4 Sex" class="gt_row gt_center" style="text-align: left;">male</td>
<td headers="Chinstrap Sex_4 No. birds" class="gt_row gt_right">34</td>
<td headers="Chinstrap Sex_4 Mean body mass (g)" class="gt_row gt_right">3939</td>
<td headers="Chinstrap Sex_4 Sd body mass" class="gt_row gt_right">362</td></tr>
    <tr class="gt_row_group_first"><td headers="Gentoo stub_1_5 stub_1" rowspan="2" class="gt_row gt_left gt_stub_row_group" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">Gentoo</td>
<td headers="Gentoo stub_1_5 Sex" class="gt_row gt_center" style="text-align: left;">female</td>
<td headers="Gentoo stub_1_5 No. birds" class="gt_row gt_right">58</td>
<td headers="Gentoo stub_1_5 Mean body mass (g)" class="gt_row gt_right">4680</td>
<td headers="Gentoo stub_1_5 Sd body mass" class="gt_row gt_right">282</td></tr>
    <tr><td headers="Gentoo Sex_6 Sex" class="gt_row gt_center" style="text-align: left; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">male</td>
<td headers="Gentoo Sex_6 No. birds" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">61</td>
<td headers="Gentoo Sex_6 Mean body mass (g)" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">5485</td>
<td headers="Gentoo Sex_6 Sd body mass" class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">313</td></tr>
  </tbody>
  
  
</table>
</div>
```

The **stub** is the area to the left in a table that contains row labels, row group labels, and/or summary labels; in this case it is the species group. The `cells_stubhead()` function finds the header of these cells, so in this case the top-leftmost cell of the table. Similarly, `cells_row_groups()` finds all the grouping rows; I just want the last one, so I use the number of species groups present (`n_distinct(dat_summ_out$Species`)).

This last example highlights one of the challenges of `gt`: there are so many options that they can be hard to find. The [documentation](https://gt.rstudio.com/reference/index.html) is very helpful. Also, once you understand the basic structure of the functions, you can try using autofill to suggest other options. For example, if you know you want to select a cell type but don't know what it's called, try typing in `?cell_` and checking out your options.

Additional features of `gt` include:

* Footnotes
* Embedded title and subtitle
* Grouped columns
* ...

A note: at the time of writing, formatting is not preserved perfectly across output formats in `gt` (e.g., what you see in the Viewer is not exactly what Word shows). [This fix is in progress](https://github.com/rstudio/gt/issues/1098).
