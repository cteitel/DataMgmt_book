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
<div id="twuvrjiipd" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#twuvrjiipd table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#twuvrjiipd thead, #twuvrjiipd tbody, #twuvrjiipd tfoot, #twuvrjiipd tr, #twuvrjiipd td, #twuvrjiipd th {
  border-style: none;
}

#twuvrjiipd p {
  margin: 0;
  padding: 0;
}

#twuvrjiipd .gt_table {
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

#twuvrjiipd .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#twuvrjiipd .gt_title {
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

#twuvrjiipd .gt_subtitle {
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

#twuvrjiipd .gt_heading {
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

#twuvrjiipd .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#twuvrjiipd .gt_col_headings {
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

#twuvrjiipd .gt_col_heading {
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

#twuvrjiipd .gt_column_spanner_outer {
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

#twuvrjiipd .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#twuvrjiipd .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#twuvrjiipd .gt_column_spanner {
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

#twuvrjiipd .gt_spanner_row {
  border-bottom-style: hidden;
}

#twuvrjiipd .gt_group_heading {
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

#twuvrjiipd .gt_empty_group_heading {
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

#twuvrjiipd .gt_from_md > :first-child {
  margin-top: 0;
}

#twuvrjiipd .gt_from_md > :last-child {
  margin-bottom: 0;
}

#twuvrjiipd .gt_row {
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

#twuvrjiipd .gt_stub {
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

#twuvrjiipd .gt_stub_row_group {
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

#twuvrjiipd .gt_row_group_first td {
  border-top-width: 2px;
}

#twuvrjiipd .gt_row_group_first th {
  border-top-width: 2px;
}

#twuvrjiipd .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#twuvrjiipd .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#twuvrjiipd .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#twuvrjiipd .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#twuvrjiipd .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#twuvrjiipd .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#twuvrjiipd .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#twuvrjiipd .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#twuvrjiipd .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#twuvrjiipd .gt_footnotes {
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

#twuvrjiipd .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#twuvrjiipd .gt_sourcenotes {
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

#twuvrjiipd .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#twuvrjiipd .gt_left {
  text-align: left;
}

#twuvrjiipd .gt_center {
  text-align: center;
}

#twuvrjiipd .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#twuvrjiipd .gt_font_normal {
  font-weight: normal;
}

#twuvrjiipd .gt_font_bold {
  font-weight: bold;
}

#twuvrjiipd .gt_font_italic {
  font-style: italic;
}

#twuvrjiipd .gt_super {
  font-size: 65%;
}

#twuvrjiipd .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#twuvrjiipd .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#twuvrjiipd .gt_indent_1 {
  text-indent: 5px;
}

#twuvrjiipd .gt_indent_2 {
  text-indent: 10px;
}

#twuvrjiipd .gt_indent_3 {
  text-indent: 15px;
}

#twuvrjiipd .gt_indent_4 {
  text-indent: 20px;
}

#twuvrjiipd .gt_indent_5 {
  text-indent: 25px;
}

#twuvrjiipd .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#twuvrjiipd div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="duoocmgbfe" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#duoocmgbfe table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#duoocmgbfe thead, #duoocmgbfe tbody, #duoocmgbfe tfoot, #duoocmgbfe tr, #duoocmgbfe td, #duoocmgbfe th {
  border-style: none;
}

#duoocmgbfe p {
  margin: 0;
  padding: 0;
}

#duoocmgbfe .gt_table {
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

#duoocmgbfe .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#duoocmgbfe .gt_title {
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

#duoocmgbfe .gt_subtitle {
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

#duoocmgbfe .gt_heading {
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

#duoocmgbfe .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#duoocmgbfe .gt_col_headings {
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

#duoocmgbfe .gt_col_heading {
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

#duoocmgbfe .gt_column_spanner_outer {
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

#duoocmgbfe .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#duoocmgbfe .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#duoocmgbfe .gt_column_spanner {
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

#duoocmgbfe .gt_spanner_row {
  border-bottom-style: hidden;
}

#duoocmgbfe .gt_group_heading {
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

#duoocmgbfe .gt_empty_group_heading {
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

#duoocmgbfe .gt_from_md > :first-child {
  margin-top: 0;
}

#duoocmgbfe .gt_from_md > :last-child {
  margin-bottom: 0;
}

#duoocmgbfe .gt_row {
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

#duoocmgbfe .gt_stub {
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

#duoocmgbfe .gt_stub_row_group {
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

#duoocmgbfe .gt_row_group_first td {
  border-top-width: 2px;
}

#duoocmgbfe .gt_row_group_first th {
  border-top-width: 2px;
}

#duoocmgbfe .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#duoocmgbfe .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#duoocmgbfe .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#duoocmgbfe .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#duoocmgbfe .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#duoocmgbfe .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#duoocmgbfe .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#duoocmgbfe .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#duoocmgbfe .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#duoocmgbfe .gt_footnotes {
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

#duoocmgbfe .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#duoocmgbfe .gt_sourcenotes {
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

#duoocmgbfe .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#duoocmgbfe .gt_left {
  text-align: left;
}

#duoocmgbfe .gt_center {
  text-align: center;
}

#duoocmgbfe .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#duoocmgbfe .gt_font_normal {
  font-weight: normal;
}

#duoocmgbfe .gt_font_bold {
  font-weight: bold;
}

#duoocmgbfe .gt_font_italic {
  font-style: italic;
}

#duoocmgbfe .gt_super {
  font-size: 65%;
}

#duoocmgbfe .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#duoocmgbfe .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#duoocmgbfe .gt_indent_1 {
  text-indent: 5px;
}

#duoocmgbfe .gt_indent_2 {
  text-indent: 10px;
}

#duoocmgbfe .gt_indent_3 {
  text-indent: 15px;
}

#duoocmgbfe .gt_indent_4 {
  text-indent: 20px;
}

#duoocmgbfe .gt_indent_5 {
  text-indent: 25px;
}

#duoocmgbfe .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#duoocmgbfe div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="soqaygdfcw" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#soqaygdfcw table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#soqaygdfcw thead, #soqaygdfcw tbody, #soqaygdfcw tfoot, #soqaygdfcw tr, #soqaygdfcw td, #soqaygdfcw th {
  border-style: none;
}

#soqaygdfcw p {
  margin: 0;
  padding: 0;
}

#soqaygdfcw .gt_table {
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

#soqaygdfcw .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#soqaygdfcw .gt_title {
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

#soqaygdfcw .gt_subtitle {
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

#soqaygdfcw .gt_heading {
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

#soqaygdfcw .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#soqaygdfcw .gt_col_headings {
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

#soqaygdfcw .gt_col_heading {
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

#soqaygdfcw .gt_column_spanner_outer {
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

#soqaygdfcw .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#soqaygdfcw .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#soqaygdfcw .gt_column_spanner {
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

#soqaygdfcw .gt_spanner_row {
  border-bottom-style: hidden;
}

#soqaygdfcw .gt_group_heading {
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

#soqaygdfcw .gt_empty_group_heading {
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

#soqaygdfcw .gt_from_md > :first-child {
  margin-top: 0;
}

#soqaygdfcw .gt_from_md > :last-child {
  margin-bottom: 0;
}

#soqaygdfcw .gt_row {
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

#soqaygdfcw .gt_stub {
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

#soqaygdfcw .gt_stub_row_group {
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

#soqaygdfcw .gt_row_group_first td {
  border-top-width: 2px;
}

#soqaygdfcw .gt_row_group_first th {
  border-top-width: 2px;
}

#soqaygdfcw .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#soqaygdfcw .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#soqaygdfcw .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#soqaygdfcw .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#soqaygdfcw .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#soqaygdfcw .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#soqaygdfcw .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#soqaygdfcw .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#soqaygdfcw .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#soqaygdfcw .gt_footnotes {
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

#soqaygdfcw .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#soqaygdfcw .gt_sourcenotes {
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

#soqaygdfcw .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#soqaygdfcw .gt_left {
  text-align: left;
}

#soqaygdfcw .gt_center {
  text-align: center;
}

#soqaygdfcw .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#soqaygdfcw .gt_font_normal {
  font-weight: normal;
}

#soqaygdfcw .gt_font_bold {
  font-weight: bold;
}

#soqaygdfcw .gt_font_italic {
  font-style: italic;
}

#soqaygdfcw .gt_super {
  font-size: 65%;
}

#soqaygdfcw .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#soqaygdfcw .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#soqaygdfcw .gt_indent_1 {
  text-indent: 5px;
}

#soqaygdfcw .gt_indent_2 {
  text-indent: 10px;
}

#soqaygdfcw .gt_indent_3 {
  text-indent: 15px;
}

#soqaygdfcw .gt_indent_4 {
  text-indent: 20px;
}

#soqaygdfcw .gt_indent_5 {
  text-indent: 25px;
}

#soqaygdfcw .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#soqaygdfcw div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="zhgawbjghr" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#zhgawbjghr table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#zhgawbjghr thead, #zhgawbjghr tbody, #zhgawbjghr tfoot, #zhgawbjghr tr, #zhgawbjghr td, #zhgawbjghr th {
  border-style: none;
}

#zhgawbjghr p {
  margin: 0;
  padding: 0;
}

#zhgawbjghr .gt_table {
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

#zhgawbjghr .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#zhgawbjghr .gt_title {
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

#zhgawbjghr .gt_subtitle {
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

#zhgawbjghr .gt_heading {
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

#zhgawbjghr .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zhgawbjghr .gt_col_headings {
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

#zhgawbjghr .gt_col_heading {
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

#zhgawbjghr .gt_column_spanner_outer {
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

#zhgawbjghr .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#zhgawbjghr .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#zhgawbjghr .gt_column_spanner {
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

#zhgawbjghr .gt_spanner_row {
  border-bottom-style: hidden;
}

#zhgawbjghr .gt_group_heading {
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

#zhgawbjghr .gt_empty_group_heading {
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

#zhgawbjghr .gt_from_md > :first-child {
  margin-top: 0;
}

#zhgawbjghr .gt_from_md > :last-child {
  margin-bottom: 0;
}

#zhgawbjghr .gt_row {
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

#zhgawbjghr .gt_stub {
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

#zhgawbjghr .gt_stub_row_group {
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

#zhgawbjghr .gt_row_group_first td {
  border-top-width: 2px;
}

#zhgawbjghr .gt_row_group_first th {
  border-top-width: 2px;
}

#zhgawbjghr .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#zhgawbjghr .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#zhgawbjghr .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#zhgawbjghr .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zhgawbjghr .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#zhgawbjghr .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#zhgawbjghr .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#zhgawbjghr .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#zhgawbjghr .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zhgawbjghr .gt_footnotes {
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

#zhgawbjghr .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#zhgawbjghr .gt_sourcenotes {
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

#zhgawbjghr .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#zhgawbjghr .gt_left {
  text-align: left;
}

#zhgawbjghr .gt_center {
  text-align: center;
}

#zhgawbjghr .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#zhgawbjghr .gt_font_normal {
  font-weight: normal;
}

#zhgawbjghr .gt_font_bold {
  font-weight: bold;
}

#zhgawbjghr .gt_font_italic {
  font-style: italic;
}

#zhgawbjghr .gt_super {
  font-size: 65%;
}

#zhgawbjghr .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#zhgawbjghr .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#zhgawbjghr .gt_indent_1 {
  text-indent: 5px;
}

#zhgawbjghr .gt_indent_2 {
  text-indent: 10px;
}

#zhgawbjghr .gt_indent_3 {
  text-indent: 15px;
}

#zhgawbjghr .gt_indent_4 {
  text-indent: 20px;
}

#zhgawbjghr .gt_indent_5 {
  text-indent: 25px;
}

#zhgawbjghr .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#zhgawbjghr div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="rjtnqtztsg" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#rjtnqtztsg table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#rjtnqtztsg thead, #rjtnqtztsg tbody, #rjtnqtztsg tfoot, #rjtnqtztsg tr, #rjtnqtztsg td, #rjtnqtztsg th {
  border-style: none;
}

#rjtnqtztsg p {
  margin: 0;
  padding: 0;
}

#rjtnqtztsg .gt_table {
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

#rjtnqtztsg .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#rjtnqtztsg .gt_title {
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

#rjtnqtztsg .gt_subtitle {
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

#rjtnqtztsg .gt_heading {
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

#rjtnqtztsg .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rjtnqtztsg .gt_col_headings {
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

#rjtnqtztsg .gt_col_heading {
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

#rjtnqtztsg .gt_column_spanner_outer {
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

#rjtnqtztsg .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#rjtnqtztsg .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#rjtnqtztsg .gt_column_spanner {
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

#rjtnqtztsg .gt_spanner_row {
  border-bottom-style: hidden;
}

#rjtnqtztsg .gt_group_heading {
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

#rjtnqtztsg .gt_empty_group_heading {
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

#rjtnqtztsg .gt_from_md > :first-child {
  margin-top: 0;
}

#rjtnqtztsg .gt_from_md > :last-child {
  margin-bottom: 0;
}

#rjtnqtztsg .gt_row {
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

#rjtnqtztsg .gt_stub {
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

#rjtnqtztsg .gt_stub_row_group {
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

#rjtnqtztsg .gt_row_group_first td {
  border-top-width: 2px;
}

#rjtnqtztsg .gt_row_group_first th {
  border-top-width: 2px;
}

#rjtnqtztsg .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rjtnqtztsg .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#rjtnqtztsg .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#rjtnqtztsg .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rjtnqtztsg .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rjtnqtztsg .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#rjtnqtztsg .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#rjtnqtztsg .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#rjtnqtztsg .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rjtnqtztsg .gt_footnotes {
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

#rjtnqtztsg .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#rjtnqtztsg .gt_sourcenotes {
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

#rjtnqtztsg .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#rjtnqtztsg .gt_left {
  text-align: left;
}

#rjtnqtztsg .gt_center {
  text-align: center;
}

#rjtnqtztsg .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#rjtnqtztsg .gt_font_normal {
  font-weight: normal;
}

#rjtnqtztsg .gt_font_bold {
  font-weight: bold;
}

#rjtnqtztsg .gt_font_italic {
  font-style: italic;
}

#rjtnqtztsg .gt_super {
  font-size: 65%;
}

#rjtnqtztsg .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#rjtnqtztsg .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#rjtnqtztsg .gt_indent_1 {
  text-indent: 5px;
}

#rjtnqtztsg .gt_indent_2 {
  text-indent: 10px;
}

#rjtnqtztsg .gt_indent_3 {
  text-indent: 15px;
}

#rjtnqtztsg .gt_indent_4 {
  text-indent: 20px;
}

#rjtnqtztsg .gt_indent_5 {
  text-indent: 25px;
}

#rjtnqtztsg .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#rjtnqtztsg div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="rglhqhkzpi" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#rglhqhkzpi table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#rglhqhkzpi thead, #rglhqhkzpi tbody, #rglhqhkzpi tfoot, #rglhqhkzpi tr, #rglhqhkzpi td, #rglhqhkzpi th {
  border-style: none;
}

#rglhqhkzpi p {
  margin: 0;
  padding: 0;
}

#rglhqhkzpi .gt_table {
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

#rglhqhkzpi .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#rglhqhkzpi .gt_title {
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

#rglhqhkzpi .gt_subtitle {
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

#rglhqhkzpi .gt_heading {
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

#rglhqhkzpi .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rglhqhkzpi .gt_col_headings {
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

#rglhqhkzpi .gt_col_heading {
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

#rglhqhkzpi .gt_column_spanner_outer {
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

#rglhqhkzpi .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#rglhqhkzpi .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#rglhqhkzpi .gt_column_spanner {
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

#rglhqhkzpi .gt_spanner_row {
  border-bottom-style: hidden;
}

#rglhqhkzpi .gt_group_heading {
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

#rglhqhkzpi .gt_empty_group_heading {
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

#rglhqhkzpi .gt_from_md > :first-child {
  margin-top: 0;
}

#rglhqhkzpi .gt_from_md > :last-child {
  margin-bottom: 0;
}

#rglhqhkzpi .gt_row {
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

#rglhqhkzpi .gt_stub {
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

#rglhqhkzpi .gt_stub_row_group {
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

#rglhqhkzpi .gt_row_group_first td {
  border-top-width: 2px;
}

#rglhqhkzpi .gt_row_group_first th {
  border-top-width: 2px;
}

#rglhqhkzpi .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rglhqhkzpi .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#rglhqhkzpi .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#rglhqhkzpi .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rglhqhkzpi .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rglhqhkzpi .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#rglhqhkzpi .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#rglhqhkzpi .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#rglhqhkzpi .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rglhqhkzpi .gt_footnotes {
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

#rglhqhkzpi .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#rglhqhkzpi .gt_sourcenotes {
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

#rglhqhkzpi .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#rglhqhkzpi .gt_left {
  text-align: left;
}

#rglhqhkzpi .gt_center {
  text-align: center;
}

#rglhqhkzpi .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#rglhqhkzpi .gt_font_normal {
  font-weight: normal;
}

#rglhqhkzpi .gt_font_bold {
  font-weight: bold;
}

#rglhqhkzpi .gt_font_italic {
  font-style: italic;
}

#rglhqhkzpi .gt_super {
  font-size: 65%;
}

#rglhqhkzpi .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#rglhqhkzpi .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#rglhqhkzpi .gt_indent_1 {
  text-indent: 5px;
}

#rglhqhkzpi .gt_indent_2 {
  text-indent: 10px;
}

#rglhqhkzpi .gt_indent_3 {
  text-indent: 15px;
}

#rglhqhkzpi .gt_indent_4 {
  text-indent: 20px;
}

#rglhqhkzpi .gt_indent_5 {
  text-indent: 25px;
}

#rglhqhkzpi .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#rglhqhkzpi div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="vrxzfcaqoj" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#vrxzfcaqoj table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#vrxzfcaqoj thead, #vrxzfcaqoj tbody, #vrxzfcaqoj tfoot, #vrxzfcaqoj tr, #vrxzfcaqoj td, #vrxzfcaqoj th {
  border-style: none;
}

#vrxzfcaqoj p {
  margin: 0;
  padding: 0;
}

#vrxzfcaqoj .gt_table {
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

#vrxzfcaqoj .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#vrxzfcaqoj .gt_title {
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

#vrxzfcaqoj .gt_subtitle {
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

#vrxzfcaqoj .gt_heading {
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

#vrxzfcaqoj .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vrxzfcaqoj .gt_col_headings {
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

#vrxzfcaqoj .gt_col_heading {
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

#vrxzfcaqoj .gt_column_spanner_outer {
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

#vrxzfcaqoj .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#vrxzfcaqoj .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#vrxzfcaqoj .gt_column_spanner {
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

#vrxzfcaqoj .gt_spanner_row {
  border-bottom-style: hidden;
}

#vrxzfcaqoj .gt_group_heading {
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

#vrxzfcaqoj .gt_empty_group_heading {
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

#vrxzfcaqoj .gt_from_md > :first-child {
  margin-top: 0;
}

#vrxzfcaqoj .gt_from_md > :last-child {
  margin-bottom: 0;
}

#vrxzfcaqoj .gt_row {
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

#vrxzfcaqoj .gt_stub {
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

#vrxzfcaqoj .gt_stub_row_group {
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

#vrxzfcaqoj .gt_row_group_first td {
  border-top-width: 2px;
}

#vrxzfcaqoj .gt_row_group_first th {
  border-top-width: 2px;
}

#vrxzfcaqoj .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxzfcaqoj .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#vrxzfcaqoj .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#vrxzfcaqoj .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vrxzfcaqoj .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxzfcaqoj .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#vrxzfcaqoj .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#vrxzfcaqoj .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#vrxzfcaqoj .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vrxzfcaqoj .gt_footnotes {
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

#vrxzfcaqoj .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxzfcaqoj .gt_sourcenotes {
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

#vrxzfcaqoj .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#vrxzfcaqoj .gt_left {
  text-align: left;
}

#vrxzfcaqoj .gt_center {
  text-align: center;
}

#vrxzfcaqoj .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#vrxzfcaqoj .gt_font_normal {
  font-weight: normal;
}

#vrxzfcaqoj .gt_font_bold {
  font-weight: bold;
}

#vrxzfcaqoj .gt_font_italic {
  font-style: italic;
}

#vrxzfcaqoj .gt_super {
  font-size: 65%;
}

#vrxzfcaqoj .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#vrxzfcaqoj .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#vrxzfcaqoj .gt_indent_1 {
  text-indent: 5px;
}

#vrxzfcaqoj .gt_indent_2 {
  text-indent: 10px;
}

#vrxzfcaqoj .gt_indent_3 {
  text-indent: 15px;
}

#vrxzfcaqoj .gt_indent_4 {
  text-indent: 20px;
}

#vrxzfcaqoj .gt_indent_5 {
  text-indent: 25px;
}

#vrxzfcaqoj .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#vrxzfcaqoj div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="nazgewqnir" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#nazgewqnir table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#nazgewqnir thead, #nazgewqnir tbody, #nazgewqnir tfoot, #nazgewqnir tr, #nazgewqnir td, #nazgewqnir th {
  border-style: none;
}

#nazgewqnir p {
  margin: 0;
  padding: 0;
}

#nazgewqnir .gt_table {
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

#nazgewqnir .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#nazgewqnir .gt_title {
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

#nazgewqnir .gt_subtitle {
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

#nazgewqnir .gt_heading {
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

#nazgewqnir .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nazgewqnir .gt_col_headings {
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

#nazgewqnir .gt_col_heading {
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

#nazgewqnir .gt_column_spanner_outer {
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

#nazgewqnir .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#nazgewqnir .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#nazgewqnir .gt_column_spanner {
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

#nazgewqnir .gt_spanner_row {
  border-bottom-style: hidden;
}

#nazgewqnir .gt_group_heading {
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

#nazgewqnir .gt_empty_group_heading {
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

#nazgewqnir .gt_from_md > :first-child {
  margin-top: 0;
}

#nazgewqnir .gt_from_md > :last-child {
  margin-bottom: 0;
}

#nazgewqnir .gt_row {
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

#nazgewqnir .gt_stub {
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

#nazgewqnir .gt_stub_row_group {
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

#nazgewqnir .gt_row_group_first td {
  border-top-width: 2px;
}

#nazgewqnir .gt_row_group_first th {
  border-top-width: 2px;
}

#nazgewqnir .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nazgewqnir .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#nazgewqnir .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#nazgewqnir .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nazgewqnir .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nazgewqnir .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#nazgewqnir .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#nazgewqnir .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#nazgewqnir .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nazgewqnir .gt_footnotes {
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

#nazgewqnir .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#nazgewqnir .gt_sourcenotes {
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

#nazgewqnir .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#nazgewqnir .gt_left {
  text-align: left;
}

#nazgewqnir .gt_center {
  text-align: center;
}

#nazgewqnir .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#nazgewqnir .gt_font_normal {
  font-weight: normal;
}

#nazgewqnir .gt_font_bold {
  font-weight: bold;
}

#nazgewqnir .gt_font_italic {
  font-style: italic;
}

#nazgewqnir .gt_super {
  font-size: 65%;
}

#nazgewqnir .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#nazgewqnir .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#nazgewqnir .gt_indent_1 {
  text-indent: 5px;
}

#nazgewqnir .gt_indent_2 {
  text-indent: 10px;
}

#nazgewqnir .gt_indent_3 {
  text-indent: 15px;
}

#nazgewqnir .gt_indent_4 {
  text-indent: 20px;
}

#nazgewqnir .gt_indent_5 {
  text-indent: 25px;
}

#nazgewqnir .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#nazgewqnir div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="sqxzpqqitq" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#sqxzpqqitq table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#sqxzpqqitq thead, #sqxzpqqitq tbody, #sqxzpqqitq tfoot, #sqxzpqqitq tr, #sqxzpqqitq td, #sqxzpqqitq th {
  border-style: none;
}

#sqxzpqqitq p {
  margin: 0;
  padding: 0;
}

#sqxzpqqitq .gt_table {
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

#sqxzpqqitq .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#sqxzpqqitq .gt_title {
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

#sqxzpqqitq .gt_subtitle {
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

#sqxzpqqitq .gt_heading {
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

#sqxzpqqitq .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#sqxzpqqitq .gt_col_headings {
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

#sqxzpqqitq .gt_col_heading {
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

#sqxzpqqitq .gt_column_spanner_outer {
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

#sqxzpqqitq .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#sqxzpqqitq .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#sqxzpqqitq .gt_column_spanner {
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

#sqxzpqqitq .gt_spanner_row {
  border-bottom-style: hidden;
}

#sqxzpqqitq .gt_group_heading {
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

#sqxzpqqitq .gt_empty_group_heading {
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

#sqxzpqqitq .gt_from_md > :first-child {
  margin-top: 0;
}

#sqxzpqqitq .gt_from_md > :last-child {
  margin-bottom: 0;
}

#sqxzpqqitq .gt_row {
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

#sqxzpqqitq .gt_stub {
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

#sqxzpqqitq .gt_stub_row_group {
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

#sqxzpqqitq .gt_row_group_first td {
  border-top-width: 2px;
}

#sqxzpqqitq .gt_row_group_first th {
  border-top-width: 2px;
}

#sqxzpqqitq .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#sqxzpqqitq .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#sqxzpqqitq .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#sqxzpqqitq .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#sqxzpqqitq .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#sqxzpqqitq .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#sqxzpqqitq .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#sqxzpqqitq .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#sqxzpqqitq .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#sqxzpqqitq .gt_footnotes {
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

#sqxzpqqitq .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#sqxzpqqitq .gt_sourcenotes {
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

#sqxzpqqitq .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#sqxzpqqitq .gt_left {
  text-align: left;
}

#sqxzpqqitq .gt_center {
  text-align: center;
}

#sqxzpqqitq .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#sqxzpqqitq .gt_font_normal {
  font-weight: normal;
}

#sqxzpqqitq .gt_font_bold {
  font-weight: bold;
}

#sqxzpqqitq .gt_font_italic {
  font-style: italic;
}

#sqxzpqqitq .gt_super {
  font-size: 65%;
}

#sqxzpqqitq .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#sqxzpqqitq .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#sqxzpqqitq .gt_indent_1 {
  text-indent: 5px;
}

#sqxzpqqitq .gt_indent_2 {
  text-indent: 10px;
}

#sqxzpqqitq .gt_indent_3 {
  text-indent: 15px;
}

#sqxzpqqitq .gt_indent_4 {
  text-indent: 20px;
}

#sqxzpqqitq .gt_indent_5 {
  text-indent: 25px;
}

#sqxzpqqitq .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#sqxzpqqitq div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="ztheuurgbx" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#ztheuurgbx table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#ztheuurgbx thead, #ztheuurgbx tbody, #ztheuurgbx tfoot, #ztheuurgbx tr, #ztheuurgbx td, #ztheuurgbx th {
  border-style: none;
}

#ztheuurgbx p {
  margin: 0;
  padding: 0;
}

#ztheuurgbx .gt_table {
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

#ztheuurgbx .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#ztheuurgbx .gt_title {
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

#ztheuurgbx .gt_subtitle {
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

#ztheuurgbx .gt_heading {
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

#ztheuurgbx .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ztheuurgbx .gt_col_headings {
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

#ztheuurgbx .gt_col_heading {
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

#ztheuurgbx .gt_column_spanner_outer {
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

#ztheuurgbx .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ztheuurgbx .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ztheuurgbx .gt_column_spanner {
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

#ztheuurgbx .gt_spanner_row {
  border-bottom-style: hidden;
}

#ztheuurgbx .gt_group_heading {
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

#ztheuurgbx .gt_empty_group_heading {
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

#ztheuurgbx .gt_from_md > :first-child {
  margin-top: 0;
}

#ztheuurgbx .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ztheuurgbx .gt_row {
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

#ztheuurgbx .gt_stub {
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

#ztheuurgbx .gt_stub_row_group {
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

#ztheuurgbx .gt_row_group_first td {
  border-top-width: 2px;
}

#ztheuurgbx .gt_row_group_first th {
  border-top-width: 2px;
}

#ztheuurgbx .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ztheuurgbx .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#ztheuurgbx .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#ztheuurgbx .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ztheuurgbx .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ztheuurgbx .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ztheuurgbx .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#ztheuurgbx .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ztheuurgbx .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ztheuurgbx .gt_footnotes {
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

#ztheuurgbx .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ztheuurgbx .gt_sourcenotes {
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

#ztheuurgbx .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ztheuurgbx .gt_left {
  text-align: left;
}

#ztheuurgbx .gt_center {
  text-align: center;
}

#ztheuurgbx .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ztheuurgbx .gt_font_normal {
  font-weight: normal;
}

#ztheuurgbx .gt_font_bold {
  font-weight: bold;
}

#ztheuurgbx .gt_font_italic {
  font-style: italic;
}

#ztheuurgbx .gt_super {
  font-size: 65%;
}

#ztheuurgbx .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#ztheuurgbx .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#ztheuurgbx .gt_indent_1 {
  text-indent: 5px;
}

#ztheuurgbx .gt_indent_2 {
  text-indent: 10px;
}

#ztheuurgbx .gt_indent_3 {
  text-indent: 15px;
}

#ztheuurgbx .gt_indent_4 {
  text-indent: 20px;
}

#ztheuurgbx .gt_indent_5 {
  text-indent: 25px;
}

#ztheuurgbx .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#ztheuurgbx div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
