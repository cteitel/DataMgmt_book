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
<div id="elwuvozzbe" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#elwuvozzbe table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#elwuvozzbe thead, #elwuvozzbe tbody, #elwuvozzbe tfoot, #elwuvozzbe tr, #elwuvozzbe td, #elwuvozzbe th {
  border-style: none;
}

#elwuvozzbe p {
  margin: 0;
  padding: 0;
}

#elwuvozzbe .gt_table {
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

#elwuvozzbe .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#elwuvozzbe .gt_title {
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

#elwuvozzbe .gt_subtitle {
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

#elwuvozzbe .gt_heading {
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

#elwuvozzbe .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#elwuvozzbe .gt_col_headings {
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

#elwuvozzbe .gt_col_heading {
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

#elwuvozzbe .gt_column_spanner_outer {
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

#elwuvozzbe .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#elwuvozzbe .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#elwuvozzbe .gt_column_spanner {
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

#elwuvozzbe .gt_spanner_row {
  border-bottom-style: hidden;
}

#elwuvozzbe .gt_group_heading {
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

#elwuvozzbe .gt_empty_group_heading {
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

#elwuvozzbe .gt_from_md > :first-child {
  margin-top: 0;
}

#elwuvozzbe .gt_from_md > :last-child {
  margin-bottom: 0;
}

#elwuvozzbe .gt_row {
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

#elwuvozzbe .gt_stub {
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

#elwuvozzbe .gt_stub_row_group {
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

#elwuvozzbe .gt_row_group_first td {
  border-top-width: 2px;
}

#elwuvozzbe .gt_row_group_first th {
  border-top-width: 2px;
}

#elwuvozzbe .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#elwuvozzbe .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#elwuvozzbe .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#elwuvozzbe .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#elwuvozzbe .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#elwuvozzbe .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#elwuvozzbe .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#elwuvozzbe .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#elwuvozzbe .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#elwuvozzbe .gt_footnotes {
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

#elwuvozzbe .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#elwuvozzbe .gt_sourcenotes {
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

#elwuvozzbe .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#elwuvozzbe .gt_left {
  text-align: left;
}

#elwuvozzbe .gt_center {
  text-align: center;
}

#elwuvozzbe .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#elwuvozzbe .gt_font_normal {
  font-weight: normal;
}

#elwuvozzbe .gt_font_bold {
  font-weight: bold;
}

#elwuvozzbe .gt_font_italic {
  font-style: italic;
}

#elwuvozzbe .gt_super {
  font-size: 65%;
}

#elwuvozzbe .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#elwuvozzbe .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#elwuvozzbe .gt_indent_1 {
  text-indent: 5px;
}

#elwuvozzbe .gt_indent_2 {
  text-indent: 10px;
}

#elwuvozzbe .gt_indent_3 {
  text-indent: 15px;
}

#elwuvozzbe .gt_indent_4 {
  text-indent: 20px;
}

#elwuvozzbe .gt_indent_5 {
  text-indent: 25px;
}

#elwuvozzbe .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#elwuvozzbe div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="qorrhmlheo" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#qorrhmlheo table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#qorrhmlheo thead, #qorrhmlheo tbody, #qorrhmlheo tfoot, #qorrhmlheo tr, #qorrhmlheo td, #qorrhmlheo th {
  border-style: none;
}

#qorrhmlheo p {
  margin: 0;
  padding: 0;
}

#qorrhmlheo .gt_table {
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

#qorrhmlheo .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#qorrhmlheo .gt_title {
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

#qorrhmlheo .gt_subtitle {
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

#qorrhmlheo .gt_heading {
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

#qorrhmlheo .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qorrhmlheo .gt_col_headings {
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

#qorrhmlheo .gt_col_heading {
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

#qorrhmlheo .gt_column_spanner_outer {
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

#qorrhmlheo .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#qorrhmlheo .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#qorrhmlheo .gt_column_spanner {
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

#qorrhmlheo .gt_spanner_row {
  border-bottom-style: hidden;
}

#qorrhmlheo .gt_group_heading {
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

#qorrhmlheo .gt_empty_group_heading {
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

#qorrhmlheo .gt_from_md > :first-child {
  margin-top: 0;
}

#qorrhmlheo .gt_from_md > :last-child {
  margin-bottom: 0;
}

#qorrhmlheo .gt_row {
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

#qorrhmlheo .gt_stub {
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

#qorrhmlheo .gt_stub_row_group {
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

#qorrhmlheo .gt_row_group_first td {
  border-top-width: 2px;
}

#qorrhmlheo .gt_row_group_first th {
  border-top-width: 2px;
}

#qorrhmlheo .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qorrhmlheo .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#qorrhmlheo .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#qorrhmlheo .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qorrhmlheo .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qorrhmlheo .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#qorrhmlheo .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#qorrhmlheo .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#qorrhmlheo .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qorrhmlheo .gt_footnotes {
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

#qorrhmlheo .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qorrhmlheo .gt_sourcenotes {
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

#qorrhmlheo .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qorrhmlheo .gt_left {
  text-align: left;
}

#qorrhmlheo .gt_center {
  text-align: center;
}

#qorrhmlheo .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#qorrhmlheo .gt_font_normal {
  font-weight: normal;
}

#qorrhmlheo .gt_font_bold {
  font-weight: bold;
}

#qorrhmlheo .gt_font_italic {
  font-style: italic;
}

#qorrhmlheo .gt_super {
  font-size: 65%;
}

#qorrhmlheo .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#qorrhmlheo .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#qorrhmlheo .gt_indent_1 {
  text-indent: 5px;
}

#qorrhmlheo .gt_indent_2 {
  text-indent: 10px;
}

#qorrhmlheo .gt_indent_3 {
  text-indent: 15px;
}

#qorrhmlheo .gt_indent_4 {
  text-indent: 20px;
}

#qorrhmlheo .gt_indent_5 {
  text-indent: 25px;
}

#qorrhmlheo .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#qorrhmlheo div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="shfhyaywcl" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#shfhyaywcl table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#shfhyaywcl thead, #shfhyaywcl tbody, #shfhyaywcl tfoot, #shfhyaywcl tr, #shfhyaywcl td, #shfhyaywcl th {
  border-style: none;
}

#shfhyaywcl p {
  margin: 0;
  padding: 0;
}

#shfhyaywcl .gt_table {
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

#shfhyaywcl .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#shfhyaywcl .gt_title {
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

#shfhyaywcl .gt_subtitle {
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

#shfhyaywcl .gt_heading {
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

#shfhyaywcl .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#shfhyaywcl .gt_col_headings {
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

#shfhyaywcl .gt_col_heading {
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

#shfhyaywcl .gt_column_spanner_outer {
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

#shfhyaywcl .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#shfhyaywcl .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#shfhyaywcl .gt_column_spanner {
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

#shfhyaywcl .gt_spanner_row {
  border-bottom-style: hidden;
}

#shfhyaywcl .gt_group_heading {
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

#shfhyaywcl .gt_empty_group_heading {
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

#shfhyaywcl .gt_from_md > :first-child {
  margin-top: 0;
}

#shfhyaywcl .gt_from_md > :last-child {
  margin-bottom: 0;
}

#shfhyaywcl .gt_row {
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

#shfhyaywcl .gt_stub {
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

#shfhyaywcl .gt_stub_row_group {
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

#shfhyaywcl .gt_row_group_first td {
  border-top-width: 2px;
}

#shfhyaywcl .gt_row_group_first th {
  border-top-width: 2px;
}

#shfhyaywcl .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#shfhyaywcl .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#shfhyaywcl .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#shfhyaywcl .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#shfhyaywcl .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#shfhyaywcl .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#shfhyaywcl .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#shfhyaywcl .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#shfhyaywcl .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#shfhyaywcl .gt_footnotes {
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

#shfhyaywcl .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#shfhyaywcl .gt_sourcenotes {
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

#shfhyaywcl .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#shfhyaywcl .gt_left {
  text-align: left;
}

#shfhyaywcl .gt_center {
  text-align: center;
}

#shfhyaywcl .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#shfhyaywcl .gt_font_normal {
  font-weight: normal;
}

#shfhyaywcl .gt_font_bold {
  font-weight: bold;
}

#shfhyaywcl .gt_font_italic {
  font-style: italic;
}

#shfhyaywcl .gt_super {
  font-size: 65%;
}

#shfhyaywcl .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#shfhyaywcl .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#shfhyaywcl .gt_indent_1 {
  text-indent: 5px;
}

#shfhyaywcl .gt_indent_2 {
  text-indent: 10px;
}

#shfhyaywcl .gt_indent_3 {
  text-indent: 15px;
}

#shfhyaywcl .gt_indent_4 {
  text-indent: 20px;
}

#shfhyaywcl .gt_indent_5 {
  text-indent: 25px;
}

#shfhyaywcl .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#shfhyaywcl div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="ehjqrqdlwk" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#ehjqrqdlwk table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#ehjqrqdlwk thead, #ehjqrqdlwk tbody, #ehjqrqdlwk tfoot, #ehjqrqdlwk tr, #ehjqrqdlwk td, #ehjqrqdlwk th {
  border-style: none;
}

#ehjqrqdlwk p {
  margin: 0;
  padding: 0;
}

#ehjqrqdlwk .gt_table {
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

#ehjqrqdlwk .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#ehjqrqdlwk .gt_title {
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

#ehjqrqdlwk .gt_subtitle {
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

#ehjqrqdlwk .gt_heading {
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

#ehjqrqdlwk .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ehjqrqdlwk .gt_col_headings {
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

#ehjqrqdlwk .gt_col_heading {
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

#ehjqrqdlwk .gt_column_spanner_outer {
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

#ehjqrqdlwk .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ehjqrqdlwk .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ehjqrqdlwk .gt_column_spanner {
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

#ehjqrqdlwk .gt_spanner_row {
  border-bottom-style: hidden;
}

#ehjqrqdlwk .gt_group_heading {
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

#ehjqrqdlwk .gt_empty_group_heading {
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

#ehjqrqdlwk .gt_from_md > :first-child {
  margin-top: 0;
}

#ehjqrqdlwk .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ehjqrqdlwk .gt_row {
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

#ehjqrqdlwk .gt_stub {
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

#ehjqrqdlwk .gt_stub_row_group {
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

#ehjqrqdlwk .gt_row_group_first td {
  border-top-width: 2px;
}

#ehjqrqdlwk .gt_row_group_first th {
  border-top-width: 2px;
}

#ehjqrqdlwk .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ehjqrqdlwk .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#ehjqrqdlwk .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#ehjqrqdlwk .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ehjqrqdlwk .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ehjqrqdlwk .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ehjqrqdlwk .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#ehjqrqdlwk .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ehjqrqdlwk .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ehjqrqdlwk .gt_footnotes {
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

#ehjqrqdlwk .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ehjqrqdlwk .gt_sourcenotes {
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

#ehjqrqdlwk .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ehjqrqdlwk .gt_left {
  text-align: left;
}

#ehjqrqdlwk .gt_center {
  text-align: center;
}

#ehjqrqdlwk .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ehjqrqdlwk .gt_font_normal {
  font-weight: normal;
}

#ehjqrqdlwk .gt_font_bold {
  font-weight: bold;
}

#ehjqrqdlwk .gt_font_italic {
  font-style: italic;
}

#ehjqrqdlwk .gt_super {
  font-size: 65%;
}

#ehjqrqdlwk .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#ehjqrqdlwk .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#ehjqrqdlwk .gt_indent_1 {
  text-indent: 5px;
}

#ehjqrqdlwk .gt_indent_2 {
  text-indent: 10px;
}

#ehjqrqdlwk .gt_indent_3 {
  text-indent: 15px;
}

#ehjqrqdlwk .gt_indent_4 {
  text-indent: 20px;
}

#ehjqrqdlwk .gt_indent_5 {
  text-indent: 25px;
}

#ehjqrqdlwk .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#ehjqrqdlwk div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="leewqsgcsk" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#leewqsgcsk table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#leewqsgcsk thead, #leewqsgcsk tbody, #leewqsgcsk tfoot, #leewqsgcsk tr, #leewqsgcsk td, #leewqsgcsk th {
  border-style: none;
}

#leewqsgcsk p {
  margin: 0;
  padding: 0;
}

#leewqsgcsk .gt_table {
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

#leewqsgcsk .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#leewqsgcsk .gt_title {
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

#leewqsgcsk .gt_subtitle {
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

#leewqsgcsk .gt_heading {
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

#leewqsgcsk .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#leewqsgcsk .gt_col_headings {
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

#leewqsgcsk .gt_col_heading {
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

#leewqsgcsk .gt_column_spanner_outer {
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

#leewqsgcsk .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#leewqsgcsk .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#leewqsgcsk .gt_column_spanner {
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

#leewqsgcsk .gt_spanner_row {
  border-bottom-style: hidden;
}

#leewqsgcsk .gt_group_heading {
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

#leewqsgcsk .gt_empty_group_heading {
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

#leewqsgcsk .gt_from_md > :first-child {
  margin-top: 0;
}

#leewqsgcsk .gt_from_md > :last-child {
  margin-bottom: 0;
}

#leewqsgcsk .gt_row {
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

#leewqsgcsk .gt_stub {
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

#leewqsgcsk .gt_stub_row_group {
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

#leewqsgcsk .gt_row_group_first td {
  border-top-width: 2px;
}

#leewqsgcsk .gt_row_group_first th {
  border-top-width: 2px;
}

#leewqsgcsk .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#leewqsgcsk .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#leewqsgcsk .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#leewqsgcsk .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#leewqsgcsk .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#leewqsgcsk .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#leewqsgcsk .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#leewqsgcsk .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#leewqsgcsk .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#leewqsgcsk .gt_footnotes {
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

#leewqsgcsk .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#leewqsgcsk .gt_sourcenotes {
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

#leewqsgcsk .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#leewqsgcsk .gt_left {
  text-align: left;
}

#leewqsgcsk .gt_center {
  text-align: center;
}

#leewqsgcsk .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#leewqsgcsk .gt_font_normal {
  font-weight: normal;
}

#leewqsgcsk .gt_font_bold {
  font-weight: bold;
}

#leewqsgcsk .gt_font_italic {
  font-style: italic;
}

#leewqsgcsk .gt_super {
  font-size: 65%;
}

#leewqsgcsk .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#leewqsgcsk .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#leewqsgcsk .gt_indent_1 {
  text-indent: 5px;
}

#leewqsgcsk .gt_indent_2 {
  text-indent: 10px;
}

#leewqsgcsk .gt_indent_3 {
  text-indent: 15px;
}

#leewqsgcsk .gt_indent_4 {
  text-indent: 20px;
}

#leewqsgcsk .gt_indent_5 {
  text-indent: 25px;
}

#leewqsgcsk .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#leewqsgcsk div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="obbinwbxup" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#obbinwbxup table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#obbinwbxup thead, #obbinwbxup tbody, #obbinwbxup tfoot, #obbinwbxup tr, #obbinwbxup td, #obbinwbxup th {
  border-style: none;
}

#obbinwbxup p {
  margin: 0;
  padding: 0;
}

#obbinwbxup .gt_table {
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

#obbinwbxup .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#obbinwbxup .gt_title {
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

#obbinwbxup .gt_subtitle {
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

#obbinwbxup .gt_heading {
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

#obbinwbxup .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#obbinwbxup .gt_col_headings {
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

#obbinwbxup .gt_col_heading {
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

#obbinwbxup .gt_column_spanner_outer {
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

#obbinwbxup .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#obbinwbxup .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#obbinwbxup .gt_column_spanner {
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

#obbinwbxup .gt_spanner_row {
  border-bottom-style: hidden;
}

#obbinwbxup .gt_group_heading {
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

#obbinwbxup .gt_empty_group_heading {
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

#obbinwbxup .gt_from_md > :first-child {
  margin-top: 0;
}

#obbinwbxup .gt_from_md > :last-child {
  margin-bottom: 0;
}

#obbinwbxup .gt_row {
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

#obbinwbxup .gt_stub {
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

#obbinwbxup .gt_stub_row_group {
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

#obbinwbxup .gt_row_group_first td {
  border-top-width: 2px;
}

#obbinwbxup .gt_row_group_first th {
  border-top-width: 2px;
}

#obbinwbxup .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#obbinwbxup .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#obbinwbxup .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#obbinwbxup .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#obbinwbxup .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#obbinwbxup .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#obbinwbxup .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#obbinwbxup .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#obbinwbxup .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#obbinwbxup .gt_footnotes {
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

#obbinwbxup .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#obbinwbxup .gt_sourcenotes {
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

#obbinwbxup .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#obbinwbxup .gt_left {
  text-align: left;
}

#obbinwbxup .gt_center {
  text-align: center;
}

#obbinwbxup .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#obbinwbxup .gt_font_normal {
  font-weight: normal;
}

#obbinwbxup .gt_font_bold {
  font-weight: bold;
}

#obbinwbxup .gt_font_italic {
  font-style: italic;
}

#obbinwbxup .gt_super {
  font-size: 65%;
}

#obbinwbxup .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#obbinwbxup .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#obbinwbxup .gt_indent_1 {
  text-indent: 5px;
}

#obbinwbxup .gt_indent_2 {
  text-indent: 10px;
}

#obbinwbxup .gt_indent_3 {
  text-indent: 15px;
}

#obbinwbxup .gt_indent_4 {
  text-indent: 20px;
}

#obbinwbxup .gt_indent_5 {
  text-indent: 25px;
}

#obbinwbxup .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#obbinwbxup div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="yfvvwzejbs" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#yfvvwzejbs table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#yfvvwzejbs thead, #yfvvwzejbs tbody, #yfvvwzejbs tfoot, #yfvvwzejbs tr, #yfvvwzejbs td, #yfvvwzejbs th {
  border-style: none;
}

#yfvvwzejbs p {
  margin: 0;
  padding: 0;
}

#yfvvwzejbs .gt_table {
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

#yfvvwzejbs .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#yfvvwzejbs .gt_title {
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

#yfvvwzejbs .gt_subtitle {
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

#yfvvwzejbs .gt_heading {
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

#yfvvwzejbs .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#yfvvwzejbs .gt_col_headings {
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

#yfvvwzejbs .gt_col_heading {
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

#yfvvwzejbs .gt_column_spanner_outer {
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

#yfvvwzejbs .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#yfvvwzejbs .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#yfvvwzejbs .gt_column_spanner {
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

#yfvvwzejbs .gt_spanner_row {
  border-bottom-style: hidden;
}

#yfvvwzejbs .gt_group_heading {
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

#yfvvwzejbs .gt_empty_group_heading {
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

#yfvvwzejbs .gt_from_md > :first-child {
  margin-top: 0;
}

#yfvvwzejbs .gt_from_md > :last-child {
  margin-bottom: 0;
}

#yfvvwzejbs .gt_row {
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

#yfvvwzejbs .gt_stub {
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

#yfvvwzejbs .gt_stub_row_group {
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

#yfvvwzejbs .gt_row_group_first td {
  border-top-width: 2px;
}

#yfvvwzejbs .gt_row_group_first th {
  border-top-width: 2px;
}

#yfvvwzejbs .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#yfvvwzejbs .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#yfvvwzejbs .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#yfvvwzejbs .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#yfvvwzejbs .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#yfvvwzejbs .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#yfvvwzejbs .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#yfvvwzejbs .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#yfvvwzejbs .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#yfvvwzejbs .gt_footnotes {
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

#yfvvwzejbs .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#yfvvwzejbs .gt_sourcenotes {
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

#yfvvwzejbs .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#yfvvwzejbs .gt_left {
  text-align: left;
}

#yfvvwzejbs .gt_center {
  text-align: center;
}

#yfvvwzejbs .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#yfvvwzejbs .gt_font_normal {
  font-weight: normal;
}

#yfvvwzejbs .gt_font_bold {
  font-weight: bold;
}

#yfvvwzejbs .gt_font_italic {
  font-style: italic;
}

#yfvvwzejbs .gt_super {
  font-size: 65%;
}

#yfvvwzejbs .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#yfvvwzejbs .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#yfvvwzejbs .gt_indent_1 {
  text-indent: 5px;
}

#yfvvwzejbs .gt_indent_2 {
  text-indent: 10px;
}

#yfvvwzejbs .gt_indent_3 {
  text-indent: 15px;
}

#yfvvwzejbs .gt_indent_4 {
  text-indent: 20px;
}

#yfvvwzejbs .gt_indent_5 {
  text-indent: 25px;
}

#yfvvwzejbs .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#yfvvwzejbs div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="pqbxylrkwc" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#pqbxylrkwc table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#pqbxylrkwc thead, #pqbxylrkwc tbody, #pqbxylrkwc tfoot, #pqbxylrkwc tr, #pqbxylrkwc td, #pqbxylrkwc th {
  border-style: none;
}

#pqbxylrkwc p {
  margin: 0;
  padding: 0;
}

#pqbxylrkwc .gt_table {
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

#pqbxylrkwc .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#pqbxylrkwc .gt_title {
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

#pqbxylrkwc .gt_subtitle {
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

#pqbxylrkwc .gt_heading {
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

#pqbxylrkwc .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#pqbxylrkwc .gt_col_headings {
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

#pqbxylrkwc .gt_col_heading {
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

#pqbxylrkwc .gt_column_spanner_outer {
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

#pqbxylrkwc .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#pqbxylrkwc .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#pqbxylrkwc .gt_column_spanner {
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

#pqbxylrkwc .gt_spanner_row {
  border-bottom-style: hidden;
}

#pqbxylrkwc .gt_group_heading {
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

#pqbxylrkwc .gt_empty_group_heading {
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

#pqbxylrkwc .gt_from_md > :first-child {
  margin-top: 0;
}

#pqbxylrkwc .gt_from_md > :last-child {
  margin-bottom: 0;
}

#pqbxylrkwc .gt_row {
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

#pqbxylrkwc .gt_stub {
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

#pqbxylrkwc .gt_stub_row_group {
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

#pqbxylrkwc .gt_row_group_first td {
  border-top-width: 2px;
}

#pqbxylrkwc .gt_row_group_first th {
  border-top-width: 2px;
}

#pqbxylrkwc .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#pqbxylrkwc .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#pqbxylrkwc .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#pqbxylrkwc .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#pqbxylrkwc .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#pqbxylrkwc .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#pqbxylrkwc .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#pqbxylrkwc .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#pqbxylrkwc .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#pqbxylrkwc .gt_footnotes {
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

#pqbxylrkwc .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#pqbxylrkwc .gt_sourcenotes {
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

#pqbxylrkwc .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#pqbxylrkwc .gt_left {
  text-align: left;
}

#pqbxylrkwc .gt_center {
  text-align: center;
}

#pqbxylrkwc .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#pqbxylrkwc .gt_font_normal {
  font-weight: normal;
}

#pqbxylrkwc .gt_font_bold {
  font-weight: bold;
}

#pqbxylrkwc .gt_font_italic {
  font-style: italic;
}

#pqbxylrkwc .gt_super {
  font-size: 65%;
}

#pqbxylrkwc .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#pqbxylrkwc .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#pqbxylrkwc .gt_indent_1 {
  text-indent: 5px;
}

#pqbxylrkwc .gt_indent_2 {
  text-indent: 10px;
}

#pqbxylrkwc .gt_indent_3 {
  text-indent: 15px;
}

#pqbxylrkwc .gt_indent_4 {
  text-indent: 20px;
}

#pqbxylrkwc .gt_indent_5 {
  text-indent: 25px;
}

#pqbxylrkwc .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#pqbxylrkwc div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="bluiktxijf" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#bluiktxijf table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#bluiktxijf thead, #bluiktxijf tbody, #bluiktxijf tfoot, #bluiktxijf tr, #bluiktxijf td, #bluiktxijf th {
  border-style: none;
}

#bluiktxijf p {
  margin: 0;
  padding: 0;
}

#bluiktxijf .gt_table {
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

#bluiktxijf .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#bluiktxijf .gt_title {
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

#bluiktxijf .gt_subtitle {
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

#bluiktxijf .gt_heading {
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

#bluiktxijf .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#bluiktxijf .gt_col_headings {
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

#bluiktxijf .gt_col_heading {
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

#bluiktxijf .gt_column_spanner_outer {
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

#bluiktxijf .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#bluiktxijf .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#bluiktxijf .gt_column_spanner {
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

#bluiktxijf .gt_spanner_row {
  border-bottom-style: hidden;
}

#bluiktxijf .gt_group_heading {
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

#bluiktxijf .gt_empty_group_heading {
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

#bluiktxijf .gt_from_md > :first-child {
  margin-top: 0;
}

#bluiktxijf .gt_from_md > :last-child {
  margin-bottom: 0;
}

#bluiktxijf .gt_row {
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

#bluiktxijf .gt_stub {
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

#bluiktxijf .gt_stub_row_group {
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

#bluiktxijf .gt_row_group_first td {
  border-top-width: 2px;
}

#bluiktxijf .gt_row_group_first th {
  border-top-width: 2px;
}

#bluiktxijf .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#bluiktxijf .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#bluiktxijf .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#bluiktxijf .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#bluiktxijf .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#bluiktxijf .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#bluiktxijf .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#bluiktxijf .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#bluiktxijf .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#bluiktxijf .gt_footnotes {
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

#bluiktxijf .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#bluiktxijf .gt_sourcenotes {
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

#bluiktxijf .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#bluiktxijf .gt_left {
  text-align: left;
}

#bluiktxijf .gt_center {
  text-align: center;
}

#bluiktxijf .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#bluiktxijf .gt_font_normal {
  font-weight: normal;
}

#bluiktxijf .gt_font_bold {
  font-weight: bold;
}

#bluiktxijf .gt_font_italic {
  font-style: italic;
}

#bluiktxijf .gt_super {
  font-size: 65%;
}

#bluiktxijf .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#bluiktxijf .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#bluiktxijf .gt_indent_1 {
  text-indent: 5px;
}

#bluiktxijf .gt_indent_2 {
  text-indent: 10px;
}

#bluiktxijf .gt_indent_3 {
  text-indent: 15px;
}

#bluiktxijf .gt_indent_4 {
  text-indent: 20px;
}

#bluiktxijf .gt_indent_5 {
  text-indent: 25px;
}

#bluiktxijf .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#bluiktxijf div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="qnlafkwvuy" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#qnlafkwvuy table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#qnlafkwvuy thead, #qnlafkwvuy tbody, #qnlafkwvuy tfoot, #qnlafkwvuy tr, #qnlafkwvuy td, #qnlafkwvuy th {
  border-style: none;
}

#qnlafkwvuy p {
  margin: 0;
  padding: 0;
}

#qnlafkwvuy .gt_table {
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

#qnlafkwvuy .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#qnlafkwvuy .gt_title {
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

#qnlafkwvuy .gt_subtitle {
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

#qnlafkwvuy .gt_heading {
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

#qnlafkwvuy .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qnlafkwvuy .gt_col_headings {
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

#qnlafkwvuy .gt_col_heading {
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

#qnlafkwvuy .gt_column_spanner_outer {
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

#qnlafkwvuy .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#qnlafkwvuy .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#qnlafkwvuy .gt_column_spanner {
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

#qnlafkwvuy .gt_spanner_row {
  border-bottom-style: hidden;
}

#qnlafkwvuy .gt_group_heading {
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

#qnlafkwvuy .gt_empty_group_heading {
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

#qnlafkwvuy .gt_from_md > :first-child {
  margin-top: 0;
}

#qnlafkwvuy .gt_from_md > :last-child {
  margin-bottom: 0;
}

#qnlafkwvuy .gt_row {
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

#qnlafkwvuy .gt_stub {
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

#qnlafkwvuy .gt_stub_row_group {
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

#qnlafkwvuy .gt_row_group_first td {
  border-top-width: 2px;
}

#qnlafkwvuy .gt_row_group_first th {
  border-top-width: 2px;
}

#qnlafkwvuy .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnlafkwvuy .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#qnlafkwvuy .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#qnlafkwvuy .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qnlafkwvuy .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnlafkwvuy .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#qnlafkwvuy .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#qnlafkwvuy .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#qnlafkwvuy .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qnlafkwvuy .gt_footnotes {
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

#qnlafkwvuy .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnlafkwvuy .gt_sourcenotes {
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

#qnlafkwvuy .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnlafkwvuy .gt_left {
  text-align: left;
}

#qnlafkwvuy .gt_center {
  text-align: center;
}

#qnlafkwvuy .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#qnlafkwvuy .gt_font_normal {
  font-weight: normal;
}

#qnlafkwvuy .gt_font_bold {
  font-weight: bold;
}

#qnlafkwvuy .gt_font_italic {
  font-style: italic;
}

#qnlafkwvuy .gt_super {
  font-size: 65%;
}

#qnlafkwvuy .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#qnlafkwvuy .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#qnlafkwvuy .gt_indent_1 {
  text-indent: 5px;
}

#qnlafkwvuy .gt_indent_2 {
  text-indent: 10px;
}

#qnlafkwvuy .gt_indent_3 {
  text-indent: 15px;
}

#qnlafkwvuy .gt_indent_4 {
  text-indent: 20px;
}

#qnlafkwvuy .gt_indent_5 {
  text-indent: 25px;
}

#qnlafkwvuy .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#qnlafkwvuy div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
