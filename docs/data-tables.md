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
<div id="isnicgqdal" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#isnicgqdal table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#isnicgqdal thead, #isnicgqdal tbody, #isnicgqdal tfoot, #isnicgqdal tr, #isnicgqdal td, #isnicgqdal th {
  border-style: none;
}

#isnicgqdal p {
  margin: 0;
  padding: 0;
}

#isnicgqdal .gt_table {
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

#isnicgqdal .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#isnicgqdal .gt_title {
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

#isnicgqdal .gt_subtitle {
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

#isnicgqdal .gt_heading {
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

#isnicgqdal .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#isnicgqdal .gt_col_headings {
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

#isnicgqdal .gt_col_heading {
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

#isnicgqdal .gt_column_spanner_outer {
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

#isnicgqdal .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#isnicgqdal .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#isnicgqdal .gt_column_spanner {
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

#isnicgqdal .gt_spanner_row {
  border-bottom-style: hidden;
}

#isnicgqdal .gt_group_heading {
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

#isnicgqdal .gt_empty_group_heading {
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

#isnicgqdal .gt_from_md > :first-child {
  margin-top: 0;
}

#isnicgqdal .gt_from_md > :last-child {
  margin-bottom: 0;
}

#isnicgqdal .gt_row {
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

#isnicgqdal .gt_stub {
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

#isnicgqdal .gt_stub_row_group {
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

#isnicgqdal .gt_row_group_first td {
  border-top-width: 2px;
}

#isnicgqdal .gt_row_group_first th {
  border-top-width: 2px;
}

#isnicgqdal .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#isnicgqdal .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#isnicgqdal .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#isnicgqdal .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#isnicgqdal .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#isnicgqdal .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#isnicgqdal .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#isnicgqdal .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#isnicgqdal .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#isnicgqdal .gt_footnotes {
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

#isnicgqdal .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#isnicgqdal .gt_sourcenotes {
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

#isnicgqdal .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#isnicgqdal .gt_left {
  text-align: left;
}

#isnicgqdal .gt_center {
  text-align: center;
}

#isnicgqdal .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#isnicgqdal .gt_font_normal {
  font-weight: normal;
}

#isnicgqdal .gt_font_bold {
  font-weight: bold;
}

#isnicgqdal .gt_font_italic {
  font-style: italic;
}

#isnicgqdal .gt_super {
  font-size: 65%;
}

#isnicgqdal .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#isnicgqdal .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#isnicgqdal .gt_indent_1 {
  text-indent: 5px;
}

#isnicgqdal .gt_indent_2 {
  text-indent: 10px;
}

#isnicgqdal .gt_indent_3 {
  text-indent: 15px;
}

#isnicgqdal .gt_indent_4 {
  text-indent: 20px;
}

#isnicgqdal .gt_indent_5 {
  text-indent: 25px;
}

#isnicgqdal .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#isnicgqdal div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="hrtjkairhx" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#hrtjkairhx table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#hrtjkairhx thead, #hrtjkairhx tbody, #hrtjkairhx tfoot, #hrtjkairhx tr, #hrtjkairhx td, #hrtjkairhx th {
  border-style: none;
}

#hrtjkairhx p {
  margin: 0;
  padding: 0;
}

#hrtjkairhx .gt_table {
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

#hrtjkairhx .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#hrtjkairhx .gt_title {
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

#hrtjkairhx .gt_subtitle {
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

#hrtjkairhx .gt_heading {
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

#hrtjkairhx .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hrtjkairhx .gt_col_headings {
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

#hrtjkairhx .gt_col_heading {
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

#hrtjkairhx .gt_column_spanner_outer {
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

#hrtjkairhx .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#hrtjkairhx .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#hrtjkairhx .gt_column_spanner {
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

#hrtjkairhx .gt_spanner_row {
  border-bottom-style: hidden;
}

#hrtjkairhx .gt_group_heading {
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

#hrtjkairhx .gt_empty_group_heading {
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

#hrtjkairhx .gt_from_md > :first-child {
  margin-top: 0;
}

#hrtjkairhx .gt_from_md > :last-child {
  margin-bottom: 0;
}

#hrtjkairhx .gt_row {
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

#hrtjkairhx .gt_stub {
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

#hrtjkairhx .gt_stub_row_group {
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

#hrtjkairhx .gt_row_group_first td {
  border-top-width: 2px;
}

#hrtjkairhx .gt_row_group_first th {
  border-top-width: 2px;
}

#hrtjkairhx .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hrtjkairhx .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#hrtjkairhx .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#hrtjkairhx .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hrtjkairhx .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hrtjkairhx .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#hrtjkairhx .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#hrtjkairhx .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#hrtjkairhx .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hrtjkairhx .gt_footnotes {
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

#hrtjkairhx .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hrtjkairhx .gt_sourcenotes {
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

#hrtjkairhx .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hrtjkairhx .gt_left {
  text-align: left;
}

#hrtjkairhx .gt_center {
  text-align: center;
}

#hrtjkairhx .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#hrtjkairhx .gt_font_normal {
  font-weight: normal;
}

#hrtjkairhx .gt_font_bold {
  font-weight: bold;
}

#hrtjkairhx .gt_font_italic {
  font-style: italic;
}

#hrtjkairhx .gt_super {
  font-size: 65%;
}

#hrtjkairhx .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#hrtjkairhx .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#hrtjkairhx .gt_indent_1 {
  text-indent: 5px;
}

#hrtjkairhx .gt_indent_2 {
  text-indent: 10px;
}

#hrtjkairhx .gt_indent_3 {
  text-indent: 15px;
}

#hrtjkairhx .gt_indent_4 {
  text-indent: 20px;
}

#hrtjkairhx .gt_indent_5 {
  text-indent: 25px;
}

#hrtjkairhx .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#hrtjkairhx div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="qkdnhycvrb" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#qkdnhycvrb table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#qkdnhycvrb thead, #qkdnhycvrb tbody, #qkdnhycvrb tfoot, #qkdnhycvrb tr, #qkdnhycvrb td, #qkdnhycvrb th {
  border-style: none;
}

#qkdnhycvrb p {
  margin: 0;
  padding: 0;
}

#qkdnhycvrb .gt_table {
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

#qkdnhycvrb .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#qkdnhycvrb .gt_title {
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

#qkdnhycvrb .gt_subtitle {
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

#qkdnhycvrb .gt_heading {
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

#qkdnhycvrb .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qkdnhycvrb .gt_col_headings {
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

#qkdnhycvrb .gt_col_heading {
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

#qkdnhycvrb .gt_column_spanner_outer {
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

#qkdnhycvrb .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#qkdnhycvrb .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#qkdnhycvrb .gt_column_spanner {
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

#qkdnhycvrb .gt_spanner_row {
  border-bottom-style: hidden;
}

#qkdnhycvrb .gt_group_heading {
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

#qkdnhycvrb .gt_empty_group_heading {
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

#qkdnhycvrb .gt_from_md > :first-child {
  margin-top: 0;
}

#qkdnhycvrb .gt_from_md > :last-child {
  margin-bottom: 0;
}

#qkdnhycvrb .gt_row {
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

#qkdnhycvrb .gt_stub {
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

#qkdnhycvrb .gt_stub_row_group {
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

#qkdnhycvrb .gt_row_group_first td {
  border-top-width: 2px;
}

#qkdnhycvrb .gt_row_group_first th {
  border-top-width: 2px;
}

#qkdnhycvrb .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkdnhycvrb .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#qkdnhycvrb .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#qkdnhycvrb .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qkdnhycvrb .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkdnhycvrb .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#qkdnhycvrb .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#qkdnhycvrb .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#qkdnhycvrb .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qkdnhycvrb .gt_footnotes {
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

#qkdnhycvrb .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkdnhycvrb .gt_sourcenotes {
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

#qkdnhycvrb .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkdnhycvrb .gt_left {
  text-align: left;
}

#qkdnhycvrb .gt_center {
  text-align: center;
}

#qkdnhycvrb .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#qkdnhycvrb .gt_font_normal {
  font-weight: normal;
}

#qkdnhycvrb .gt_font_bold {
  font-weight: bold;
}

#qkdnhycvrb .gt_font_italic {
  font-style: italic;
}

#qkdnhycvrb .gt_super {
  font-size: 65%;
}

#qkdnhycvrb .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#qkdnhycvrb .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#qkdnhycvrb .gt_indent_1 {
  text-indent: 5px;
}

#qkdnhycvrb .gt_indent_2 {
  text-indent: 10px;
}

#qkdnhycvrb .gt_indent_3 {
  text-indent: 15px;
}

#qkdnhycvrb .gt_indent_4 {
  text-indent: 20px;
}

#qkdnhycvrb .gt_indent_5 {
  text-indent: 25px;
}

#qkdnhycvrb .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#qkdnhycvrb div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="hfqnxeuspn" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#hfqnxeuspn table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#hfqnxeuspn thead, #hfqnxeuspn tbody, #hfqnxeuspn tfoot, #hfqnxeuspn tr, #hfqnxeuspn td, #hfqnxeuspn th {
  border-style: none;
}

#hfqnxeuspn p {
  margin: 0;
  padding: 0;
}

#hfqnxeuspn .gt_table {
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

#hfqnxeuspn .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#hfqnxeuspn .gt_title {
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

#hfqnxeuspn .gt_subtitle {
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

#hfqnxeuspn .gt_heading {
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

#hfqnxeuspn .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hfqnxeuspn .gt_col_headings {
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

#hfqnxeuspn .gt_col_heading {
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

#hfqnxeuspn .gt_column_spanner_outer {
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

#hfqnxeuspn .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#hfqnxeuspn .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#hfqnxeuspn .gt_column_spanner {
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

#hfqnxeuspn .gt_spanner_row {
  border-bottom-style: hidden;
}

#hfqnxeuspn .gt_group_heading {
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

#hfqnxeuspn .gt_empty_group_heading {
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

#hfqnxeuspn .gt_from_md > :first-child {
  margin-top: 0;
}

#hfqnxeuspn .gt_from_md > :last-child {
  margin-bottom: 0;
}

#hfqnxeuspn .gt_row {
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

#hfqnxeuspn .gt_stub {
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

#hfqnxeuspn .gt_stub_row_group {
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

#hfqnxeuspn .gt_row_group_first td {
  border-top-width: 2px;
}

#hfqnxeuspn .gt_row_group_first th {
  border-top-width: 2px;
}

#hfqnxeuspn .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hfqnxeuspn .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#hfqnxeuspn .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#hfqnxeuspn .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hfqnxeuspn .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hfqnxeuspn .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#hfqnxeuspn .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#hfqnxeuspn .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#hfqnxeuspn .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hfqnxeuspn .gt_footnotes {
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

#hfqnxeuspn .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hfqnxeuspn .gt_sourcenotes {
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

#hfqnxeuspn .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hfqnxeuspn .gt_left {
  text-align: left;
}

#hfqnxeuspn .gt_center {
  text-align: center;
}

#hfqnxeuspn .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#hfqnxeuspn .gt_font_normal {
  font-weight: normal;
}

#hfqnxeuspn .gt_font_bold {
  font-weight: bold;
}

#hfqnxeuspn .gt_font_italic {
  font-style: italic;
}

#hfqnxeuspn .gt_super {
  font-size: 65%;
}

#hfqnxeuspn .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#hfqnxeuspn .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#hfqnxeuspn .gt_indent_1 {
  text-indent: 5px;
}

#hfqnxeuspn .gt_indent_2 {
  text-indent: 10px;
}

#hfqnxeuspn .gt_indent_3 {
  text-indent: 15px;
}

#hfqnxeuspn .gt_indent_4 {
  text-indent: 20px;
}

#hfqnxeuspn .gt_indent_5 {
  text-indent: 25px;
}

#hfqnxeuspn .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#hfqnxeuspn div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="pvsiiaxcpq" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#pvsiiaxcpq table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#pvsiiaxcpq thead, #pvsiiaxcpq tbody, #pvsiiaxcpq tfoot, #pvsiiaxcpq tr, #pvsiiaxcpq td, #pvsiiaxcpq th {
  border-style: none;
}

#pvsiiaxcpq p {
  margin: 0;
  padding: 0;
}

#pvsiiaxcpq .gt_table {
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

#pvsiiaxcpq .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#pvsiiaxcpq .gt_title {
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

#pvsiiaxcpq .gt_subtitle {
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

#pvsiiaxcpq .gt_heading {
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

#pvsiiaxcpq .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#pvsiiaxcpq .gt_col_headings {
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

#pvsiiaxcpq .gt_col_heading {
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

#pvsiiaxcpq .gt_column_spanner_outer {
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

#pvsiiaxcpq .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#pvsiiaxcpq .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#pvsiiaxcpq .gt_column_spanner {
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

#pvsiiaxcpq .gt_spanner_row {
  border-bottom-style: hidden;
}

#pvsiiaxcpq .gt_group_heading {
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

#pvsiiaxcpq .gt_empty_group_heading {
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

#pvsiiaxcpq .gt_from_md > :first-child {
  margin-top: 0;
}

#pvsiiaxcpq .gt_from_md > :last-child {
  margin-bottom: 0;
}

#pvsiiaxcpq .gt_row {
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

#pvsiiaxcpq .gt_stub {
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

#pvsiiaxcpq .gt_stub_row_group {
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

#pvsiiaxcpq .gt_row_group_first td {
  border-top-width: 2px;
}

#pvsiiaxcpq .gt_row_group_first th {
  border-top-width: 2px;
}

#pvsiiaxcpq .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#pvsiiaxcpq .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#pvsiiaxcpq .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#pvsiiaxcpq .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#pvsiiaxcpq .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#pvsiiaxcpq .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#pvsiiaxcpq .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#pvsiiaxcpq .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#pvsiiaxcpq .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#pvsiiaxcpq .gt_footnotes {
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

#pvsiiaxcpq .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#pvsiiaxcpq .gt_sourcenotes {
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

#pvsiiaxcpq .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#pvsiiaxcpq .gt_left {
  text-align: left;
}

#pvsiiaxcpq .gt_center {
  text-align: center;
}

#pvsiiaxcpq .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#pvsiiaxcpq .gt_font_normal {
  font-weight: normal;
}

#pvsiiaxcpq .gt_font_bold {
  font-weight: bold;
}

#pvsiiaxcpq .gt_font_italic {
  font-style: italic;
}

#pvsiiaxcpq .gt_super {
  font-size: 65%;
}

#pvsiiaxcpq .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#pvsiiaxcpq .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#pvsiiaxcpq .gt_indent_1 {
  text-indent: 5px;
}

#pvsiiaxcpq .gt_indent_2 {
  text-indent: 10px;
}

#pvsiiaxcpq .gt_indent_3 {
  text-indent: 15px;
}

#pvsiiaxcpq .gt_indent_4 {
  text-indent: 20px;
}

#pvsiiaxcpq .gt_indent_5 {
  text-indent: 25px;
}

#pvsiiaxcpq .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#pvsiiaxcpq div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="gojulodisk" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#gojulodisk table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#gojulodisk thead, #gojulodisk tbody, #gojulodisk tfoot, #gojulodisk tr, #gojulodisk td, #gojulodisk th {
  border-style: none;
}

#gojulodisk p {
  margin: 0;
  padding: 0;
}

#gojulodisk .gt_table {
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

#gojulodisk .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#gojulodisk .gt_title {
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

#gojulodisk .gt_subtitle {
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

#gojulodisk .gt_heading {
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

#gojulodisk .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#gojulodisk .gt_col_headings {
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

#gojulodisk .gt_col_heading {
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

#gojulodisk .gt_column_spanner_outer {
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

#gojulodisk .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#gojulodisk .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#gojulodisk .gt_column_spanner {
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

#gojulodisk .gt_spanner_row {
  border-bottom-style: hidden;
}

#gojulodisk .gt_group_heading {
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

#gojulodisk .gt_empty_group_heading {
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

#gojulodisk .gt_from_md > :first-child {
  margin-top: 0;
}

#gojulodisk .gt_from_md > :last-child {
  margin-bottom: 0;
}

#gojulodisk .gt_row {
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

#gojulodisk .gt_stub {
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

#gojulodisk .gt_stub_row_group {
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

#gojulodisk .gt_row_group_first td {
  border-top-width: 2px;
}

#gojulodisk .gt_row_group_first th {
  border-top-width: 2px;
}

#gojulodisk .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#gojulodisk .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#gojulodisk .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#gojulodisk .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#gojulodisk .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#gojulodisk .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#gojulodisk .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#gojulodisk .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#gojulodisk .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#gojulodisk .gt_footnotes {
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

#gojulodisk .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#gojulodisk .gt_sourcenotes {
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

#gojulodisk .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#gojulodisk .gt_left {
  text-align: left;
}

#gojulodisk .gt_center {
  text-align: center;
}

#gojulodisk .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#gojulodisk .gt_font_normal {
  font-weight: normal;
}

#gojulodisk .gt_font_bold {
  font-weight: bold;
}

#gojulodisk .gt_font_italic {
  font-style: italic;
}

#gojulodisk .gt_super {
  font-size: 65%;
}

#gojulodisk .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#gojulodisk .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#gojulodisk .gt_indent_1 {
  text-indent: 5px;
}

#gojulodisk .gt_indent_2 {
  text-indent: 10px;
}

#gojulodisk .gt_indent_3 {
  text-indent: 15px;
}

#gojulodisk .gt_indent_4 {
  text-indent: 20px;
}

#gojulodisk .gt_indent_5 {
  text-indent: 25px;
}

#gojulodisk .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#gojulodisk div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="fhvfmcnpoa" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#fhvfmcnpoa table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#fhvfmcnpoa thead, #fhvfmcnpoa tbody, #fhvfmcnpoa tfoot, #fhvfmcnpoa tr, #fhvfmcnpoa td, #fhvfmcnpoa th {
  border-style: none;
}

#fhvfmcnpoa p {
  margin: 0;
  padding: 0;
}

#fhvfmcnpoa .gt_table {
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

#fhvfmcnpoa .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#fhvfmcnpoa .gt_title {
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

#fhvfmcnpoa .gt_subtitle {
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

#fhvfmcnpoa .gt_heading {
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

#fhvfmcnpoa .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#fhvfmcnpoa .gt_col_headings {
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

#fhvfmcnpoa .gt_col_heading {
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

#fhvfmcnpoa .gt_column_spanner_outer {
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

#fhvfmcnpoa .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#fhvfmcnpoa .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#fhvfmcnpoa .gt_column_spanner {
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

#fhvfmcnpoa .gt_spanner_row {
  border-bottom-style: hidden;
}

#fhvfmcnpoa .gt_group_heading {
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

#fhvfmcnpoa .gt_empty_group_heading {
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

#fhvfmcnpoa .gt_from_md > :first-child {
  margin-top: 0;
}

#fhvfmcnpoa .gt_from_md > :last-child {
  margin-bottom: 0;
}

#fhvfmcnpoa .gt_row {
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

#fhvfmcnpoa .gt_stub {
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

#fhvfmcnpoa .gt_stub_row_group {
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

#fhvfmcnpoa .gt_row_group_first td {
  border-top-width: 2px;
}

#fhvfmcnpoa .gt_row_group_first th {
  border-top-width: 2px;
}

#fhvfmcnpoa .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#fhvfmcnpoa .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#fhvfmcnpoa .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#fhvfmcnpoa .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#fhvfmcnpoa .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#fhvfmcnpoa .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#fhvfmcnpoa .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#fhvfmcnpoa .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#fhvfmcnpoa .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#fhvfmcnpoa .gt_footnotes {
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

#fhvfmcnpoa .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#fhvfmcnpoa .gt_sourcenotes {
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

#fhvfmcnpoa .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#fhvfmcnpoa .gt_left {
  text-align: left;
}

#fhvfmcnpoa .gt_center {
  text-align: center;
}

#fhvfmcnpoa .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#fhvfmcnpoa .gt_font_normal {
  font-weight: normal;
}

#fhvfmcnpoa .gt_font_bold {
  font-weight: bold;
}

#fhvfmcnpoa .gt_font_italic {
  font-style: italic;
}

#fhvfmcnpoa .gt_super {
  font-size: 65%;
}

#fhvfmcnpoa .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#fhvfmcnpoa .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#fhvfmcnpoa .gt_indent_1 {
  text-indent: 5px;
}

#fhvfmcnpoa .gt_indent_2 {
  text-indent: 10px;
}

#fhvfmcnpoa .gt_indent_3 {
  text-indent: 15px;
}

#fhvfmcnpoa .gt_indent_4 {
  text-indent: 20px;
}

#fhvfmcnpoa .gt_indent_5 {
  text-indent: 25px;
}

#fhvfmcnpoa .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#fhvfmcnpoa div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="zuayxfgaer" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#zuayxfgaer table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#zuayxfgaer thead, #zuayxfgaer tbody, #zuayxfgaer tfoot, #zuayxfgaer tr, #zuayxfgaer td, #zuayxfgaer th {
  border-style: none;
}

#zuayxfgaer p {
  margin: 0;
  padding: 0;
}

#zuayxfgaer .gt_table {
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

#zuayxfgaer .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#zuayxfgaer .gt_title {
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

#zuayxfgaer .gt_subtitle {
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

#zuayxfgaer .gt_heading {
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

#zuayxfgaer .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zuayxfgaer .gt_col_headings {
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

#zuayxfgaer .gt_col_heading {
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

#zuayxfgaer .gt_column_spanner_outer {
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

#zuayxfgaer .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#zuayxfgaer .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#zuayxfgaer .gt_column_spanner {
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

#zuayxfgaer .gt_spanner_row {
  border-bottom-style: hidden;
}

#zuayxfgaer .gt_group_heading {
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

#zuayxfgaer .gt_empty_group_heading {
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

#zuayxfgaer .gt_from_md > :first-child {
  margin-top: 0;
}

#zuayxfgaer .gt_from_md > :last-child {
  margin-bottom: 0;
}

#zuayxfgaer .gt_row {
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

#zuayxfgaer .gt_stub {
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

#zuayxfgaer .gt_stub_row_group {
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

#zuayxfgaer .gt_row_group_first td {
  border-top-width: 2px;
}

#zuayxfgaer .gt_row_group_first th {
  border-top-width: 2px;
}

#zuayxfgaer .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#zuayxfgaer .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#zuayxfgaer .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#zuayxfgaer .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zuayxfgaer .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#zuayxfgaer .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#zuayxfgaer .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#zuayxfgaer .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#zuayxfgaer .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#zuayxfgaer .gt_footnotes {
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

#zuayxfgaer .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#zuayxfgaer .gt_sourcenotes {
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

#zuayxfgaer .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#zuayxfgaer .gt_left {
  text-align: left;
}

#zuayxfgaer .gt_center {
  text-align: center;
}

#zuayxfgaer .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#zuayxfgaer .gt_font_normal {
  font-weight: normal;
}

#zuayxfgaer .gt_font_bold {
  font-weight: bold;
}

#zuayxfgaer .gt_font_italic {
  font-style: italic;
}

#zuayxfgaer .gt_super {
  font-size: 65%;
}

#zuayxfgaer .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#zuayxfgaer .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#zuayxfgaer .gt_indent_1 {
  text-indent: 5px;
}

#zuayxfgaer .gt_indent_2 {
  text-indent: 10px;
}

#zuayxfgaer .gt_indent_3 {
  text-indent: 15px;
}

#zuayxfgaer .gt_indent_4 {
  text-indent: 20px;
}

#zuayxfgaer .gt_indent_5 {
  text-indent: 25px;
}

#zuayxfgaer .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#zuayxfgaer div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="madbefmxvp" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#madbefmxvp table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#madbefmxvp thead, #madbefmxvp tbody, #madbefmxvp tfoot, #madbefmxvp tr, #madbefmxvp td, #madbefmxvp th {
  border-style: none;
}

#madbefmxvp p {
  margin: 0;
  padding: 0;
}

#madbefmxvp .gt_table {
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

#madbefmxvp .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#madbefmxvp .gt_title {
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

#madbefmxvp .gt_subtitle {
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

#madbefmxvp .gt_heading {
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

#madbefmxvp .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#madbefmxvp .gt_col_headings {
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

#madbefmxvp .gt_col_heading {
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

#madbefmxvp .gt_column_spanner_outer {
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

#madbefmxvp .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#madbefmxvp .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#madbefmxvp .gt_column_spanner {
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

#madbefmxvp .gt_spanner_row {
  border-bottom-style: hidden;
}

#madbefmxvp .gt_group_heading {
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

#madbefmxvp .gt_empty_group_heading {
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

#madbefmxvp .gt_from_md > :first-child {
  margin-top: 0;
}

#madbefmxvp .gt_from_md > :last-child {
  margin-bottom: 0;
}

#madbefmxvp .gt_row {
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

#madbefmxvp .gt_stub {
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

#madbefmxvp .gt_stub_row_group {
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

#madbefmxvp .gt_row_group_first td {
  border-top-width: 2px;
}

#madbefmxvp .gt_row_group_first th {
  border-top-width: 2px;
}

#madbefmxvp .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#madbefmxvp .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#madbefmxvp .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#madbefmxvp .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#madbefmxvp .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#madbefmxvp .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#madbefmxvp .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#madbefmxvp .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#madbefmxvp .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#madbefmxvp .gt_footnotes {
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

#madbefmxvp .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#madbefmxvp .gt_sourcenotes {
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

#madbefmxvp .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#madbefmxvp .gt_left {
  text-align: left;
}

#madbefmxvp .gt_center {
  text-align: center;
}

#madbefmxvp .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#madbefmxvp .gt_font_normal {
  font-weight: normal;
}

#madbefmxvp .gt_font_bold {
  font-weight: bold;
}

#madbefmxvp .gt_font_italic {
  font-style: italic;
}

#madbefmxvp .gt_super {
  font-size: 65%;
}

#madbefmxvp .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#madbefmxvp .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#madbefmxvp .gt_indent_1 {
  text-indent: 5px;
}

#madbefmxvp .gt_indent_2 {
  text-indent: 10px;
}

#madbefmxvp .gt_indent_3 {
  text-indent: 15px;
}

#madbefmxvp .gt_indent_4 {
  text-indent: 20px;
}

#madbefmxvp .gt_indent_5 {
  text-indent: 25px;
}

#madbefmxvp .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#madbefmxvp div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="tovriyjqci" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#tovriyjqci table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#tovriyjqci thead, #tovriyjqci tbody, #tovriyjqci tfoot, #tovriyjqci tr, #tovriyjqci td, #tovriyjqci th {
  border-style: none;
}

#tovriyjqci p {
  margin: 0;
  padding: 0;
}

#tovriyjqci .gt_table {
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

#tovriyjqci .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#tovriyjqci .gt_title {
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

#tovriyjqci .gt_subtitle {
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

#tovriyjqci .gt_heading {
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

#tovriyjqci .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tovriyjqci .gt_col_headings {
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

#tovriyjqci .gt_col_heading {
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

#tovriyjqci .gt_column_spanner_outer {
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

#tovriyjqci .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#tovriyjqci .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#tovriyjqci .gt_column_spanner {
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

#tovriyjqci .gt_spanner_row {
  border-bottom-style: hidden;
}

#tovriyjqci .gt_group_heading {
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

#tovriyjqci .gt_empty_group_heading {
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

#tovriyjqci .gt_from_md > :first-child {
  margin-top: 0;
}

#tovriyjqci .gt_from_md > :last-child {
  margin-bottom: 0;
}

#tovriyjqci .gt_row {
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

#tovriyjqci .gt_stub {
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

#tovriyjqci .gt_stub_row_group {
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

#tovriyjqci .gt_row_group_first td {
  border-top-width: 2px;
}

#tovriyjqci .gt_row_group_first th {
  border-top-width: 2px;
}

#tovriyjqci .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tovriyjqci .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#tovriyjqci .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#tovriyjqci .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tovriyjqci .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#tovriyjqci .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#tovriyjqci .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#tovriyjqci .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#tovriyjqci .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#tovriyjqci .gt_footnotes {
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

#tovriyjqci .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#tovriyjqci .gt_sourcenotes {
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

#tovriyjqci .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#tovriyjqci .gt_left {
  text-align: left;
}

#tovriyjqci .gt_center {
  text-align: center;
}

#tovriyjqci .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#tovriyjqci .gt_font_normal {
  font-weight: normal;
}

#tovriyjqci .gt_font_bold {
  font-weight: bold;
}

#tovriyjqci .gt_font_italic {
  font-style: italic;
}

#tovriyjqci .gt_super {
  font-size: 65%;
}

#tovriyjqci .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#tovriyjqci .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#tovriyjqci .gt_indent_1 {
  text-indent: 5px;
}

#tovriyjqci .gt_indent_2 {
  text-indent: 10px;
}

#tovriyjqci .gt_indent_3 {
  text-indent: 15px;
}

#tovriyjqci .gt_indent_4 {
  text-indent: 20px;
}

#tovriyjqci .gt_indent_5 {
  text-indent: 25px;
}

#tovriyjqci .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#tovriyjqci div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
