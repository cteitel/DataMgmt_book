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
<div id="vjgsdnwpnf" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#vjgsdnwpnf table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#vjgsdnwpnf thead, #vjgsdnwpnf tbody, #vjgsdnwpnf tfoot, #vjgsdnwpnf tr, #vjgsdnwpnf td, #vjgsdnwpnf th {
  border-style: none;
}

#vjgsdnwpnf p {
  margin: 0;
  padding: 0;
}

#vjgsdnwpnf .gt_table {
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

#vjgsdnwpnf .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#vjgsdnwpnf .gt_title {
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

#vjgsdnwpnf .gt_subtitle {
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

#vjgsdnwpnf .gt_heading {
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

#vjgsdnwpnf .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vjgsdnwpnf .gt_col_headings {
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

#vjgsdnwpnf .gt_col_heading {
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

#vjgsdnwpnf .gt_column_spanner_outer {
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

#vjgsdnwpnf .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#vjgsdnwpnf .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#vjgsdnwpnf .gt_column_spanner {
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

#vjgsdnwpnf .gt_spanner_row {
  border-bottom-style: hidden;
}

#vjgsdnwpnf .gt_group_heading {
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

#vjgsdnwpnf .gt_empty_group_heading {
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

#vjgsdnwpnf .gt_from_md > :first-child {
  margin-top: 0;
}

#vjgsdnwpnf .gt_from_md > :last-child {
  margin-bottom: 0;
}

#vjgsdnwpnf .gt_row {
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

#vjgsdnwpnf .gt_stub {
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

#vjgsdnwpnf .gt_stub_row_group {
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

#vjgsdnwpnf .gt_row_group_first td {
  border-top-width: 2px;
}

#vjgsdnwpnf .gt_row_group_first th {
  border-top-width: 2px;
}

#vjgsdnwpnf .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#vjgsdnwpnf .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#vjgsdnwpnf .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#vjgsdnwpnf .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vjgsdnwpnf .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#vjgsdnwpnf .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#vjgsdnwpnf .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#vjgsdnwpnf .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#vjgsdnwpnf .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#vjgsdnwpnf .gt_footnotes {
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

#vjgsdnwpnf .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#vjgsdnwpnf .gt_sourcenotes {
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

#vjgsdnwpnf .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#vjgsdnwpnf .gt_left {
  text-align: left;
}

#vjgsdnwpnf .gt_center {
  text-align: center;
}

#vjgsdnwpnf .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#vjgsdnwpnf .gt_font_normal {
  font-weight: normal;
}

#vjgsdnwpnf .gt_font_bold {
  font-weight: bold;
}

#vjgsdnwpnf .gt_font_italic {
  font-style: italic;
}

#vjgsdnwpnf .gt_super {
  font-size: 65%;
}

#vjgsdnwpnf .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#vjgsdnwpnf .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#vjgsdnwpnf .gt_indent_1 {
  text-indent: 5px;
}

#vjgsdnwpnf .gt_indent_2 {
  text-indent: 10px;
}

#vjgsdnwpnf .gt_indent_3 {
  text-indent: 15px;
}

#vjgsdnwpnf .gt_indent_4 {
  text-indent: 20px;
}

#vjgsdnwpnf .gt_indent_5 {
  text-indent: 25px;
}

#vjgsdnwpnf .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#vjgsdnwpnf div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="delzevjbrh" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#delzevjbrh table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#delzevjbrh thead, #delzevjbrh tbody, #delzevjbrh tfoot, #delzevjbrh tr, #delzevjbrh td, #delzevjbrh th {
  border-style: none;
}

#delzevjbrh p {
  margin: 0;
  padding: 0;
}

#delzevjbrh .gt_table {
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

#delzevjbrh .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#delzevjbrh .gt_title {
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

#delzevjbrh .gt_subtitle {
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

#delzevjbrh .gt_heading {
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

#delzevjbrh .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#delzevjbrh .gt_col_headings {
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

#delzevjbrh .gt_col_heading {
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

#delzevjbrh .gt_column_spanner_outer {
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

#delzevjbrh .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#delzevjbrh .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#delzevjbrh .gt_column_spanner {
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

#delzevjbrh .gt_spanner_row {
  border-bottom-style: hidden;
}

#delzevjbrh .gt_group_heading {
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

#delzevjbrh .gt_empty_group_heading {
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

#delzevjbrh .gt_from_md > :first-child {
  margin-top: 0;
}

#delzevjbrh .gt_from_md > :last-child {
  margin-bottom: 0;
}

#delzevjbrh .gt_row {
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

#delzevjbrh .gt_stub {
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

#delzevjbrh .gt_stub_row_group {
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

#delzevjbrh .gt_row_group_first td {
  border-top-width: 2px;
}

#delzevjbrh .gt_row_group_first th {
  border-top-width: 2px;
}

#delzevjbrh .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#delzevjbrh .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#delzevjbrh .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#delzevjbrh .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#delzevjbrh .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#delzevjbrh .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#delzevjbrh .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#delzevjbrh .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#delzevjbrh .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#delzevjbrh .gt_footnotes {
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

#delzevjbrh .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#delzevjbrh .gt_sourcenotes {
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

#delzevjbrh .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#delzevjbrh .gt_left {
  text-align: left;
}

#delzevjbrh .gt_center {
  text-align: center;
}

#delzevjbrh .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#delzevjbrh .gt_font_normal {
  font-weight: normal;
}

#delzevjbrh .gt_font_bold {
  font-weight: bold;
}

#delzevjbrh .gt_font_italic {
  font-style: italic;
}

#delzevjbrh .gt_super {
  font-size: 65%;
}

#delzevjbrh .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#delzevjbrh .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#delzevjbrh .gt_indent_1 {
  text-indent: 5px;
}

#delzevjbrh .gt_indent_2 {
  text-indent: 10px;
}

#delzevjbrh .gt_indent_3 {
  text-indent: 15px;
}

#delzevjbrh .gt_indent_4 {
  text-indent: 20px;
}

#delzevjbrh .gt_indent_5 {
  text-indent: 25px;
}

#delzevjbrh .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#delzevjbrh div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="ifsejybikb" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#ifsejybikb table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#ifsejybikb thead, #ifsejybikb tbody, #ifsejybikb tfoot, #ifsejybikb tr, #ifsejybikb td, #ifsejybikb th {
  border-style: none;
}

#ifsejybikb p {
  margin: 0;
  padding: 0;
}

#ifsejybikb .gt_table {
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

#ifsejybikb .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#ifsejybikb .gt_title {
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

#ifsejybikb .gt_subtitle {
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

#ifsejybikb .gt_heading {
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

#ifsejybikb .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ifsejybikb .gt_col_headings {
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

#ifsejybikb .gt_col_heading {
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

#ifsejybikb .gt_column_spanner_outer {
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

#ifsejybikb .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ifsejybikb .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ifsejybikb .gt_column_spanner {
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

#ifsejybikb .gt_spanner_row {
  border-bottom-style: hidden;
}

#ifsejybikb .gt_group_heading {
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

#ifsejybikb .gt_empty_group_heading {
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

#ifsejybikb .gt_from_md > :first-child {
  margin-top: 0;
}

#ifsejybikb .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ifsejybikb .gt_row {
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

#ifsejybikb .gt_stub {
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

#ifsejybikb .gt_stub_row_group {
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

#ifsejybikb .gt_row_group_first td {
  border-top-width: 2px;
}

#ifsejybikb .gt_row_group_first th {
  border-top-width: 2px;
}

#ifsejybikb .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ifsejybikb .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#ifsejybikb .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#ifsejybikb .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ifsejybikb .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ifsejybikb .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ifsejybikb .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#ifsejybikb .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ifsejybikb .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ifsejybikb .gt_footnotes {
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

#ifsejybikb .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ifsejybikb .gt_sourcenotes {
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

#ifsejybikb .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ifsejybikb .gt_left {
  text-align: left;
}

#ifsejybikb .gt_center {
  text-align: center;
}

#ifsejybikb .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ifsejybikb .gt_font_normal {
  font-weight: normal;
}

#ifsejybikb .gt_font_bold {
  font-weight: bold;
}

#ifsejybikb .gt_font_italic {
  font-style: italic;
}

#ifsejybikb .gt_super {
  font-size: 65%;
}

#ifsejybikb .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#ifsejybikb .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#ifsejybikb .gt_indent_1 {
  text-indent: 5px;
}

#ifsejybikb .gt_indent_2 {
  text-indent: 10px;
}

#ifsejybikb .gt_indent_3 {
  text-indent: 15px;
}

#ifsejybikb .gt_indent_4 {
  text-indent: 20px;
}

#ifsejybikb .gt_indent_5 {
  text-indent: 25px;
}

#ifsejybikb .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#ifsejybikb div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="dyjqzmcdvc" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#dyjqzmcdvc table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#dyjqzmcdvc thead, #dyjqzmcdvc tbody, #dyjqzmcdvc tfoot, #dyjqzmcdvc tr, #dyjqzmcdvc td, #dyjqzmcdvc th {
  border-style: none;
}

#dyjqzmcdvc p {
  margin: 0;
  padding: 0;
}

#dyjqzmcdvc .gt_table {
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

#dyjqzmcdvc .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#dyjqzmcdvc .gt_title {
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

#dyjqzmcdvc .gt_subtitle {
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

#dyjqzmcdvc .gt_heading {
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

#dyjqzmcdvc .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#dyjqzmcdvc .gt_col_headings {
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

#dyjqzmcdvc .gt_col_heading {
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

#dyjqzmcdvc .gt_column_spanner_outer {
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

#dyjqzmcdvc .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#dyjqzmcdvc .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#dyjqzmcdvc .gt_column_spanner {
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

#dyjqzmcdvc .gt_spanner_row {
  border-bottom-style: hidden;
}

#dyjqzmcdvc .gt_group_heading {
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

#dyjqzmcdvc .gt_empty_group_heading {
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

#dyjqzmcdvc .gt_from_md > :first-child {
  margin-top: 0;
}

#dyjqzmcdvc .gt_from_md > :last-child {
  margin-bottom: 0;
}

#dyjqzmcdvc .gt_row {
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

#dyjqzmcdvc .gt_stub {
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

#dyjqzmcdvc .gt_stub_row_group {
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

#dyjqzmcdvc .gt_row_group_first td {
  border-top-width: 2px;
}

#dyjqzmcdvc .gt_row_group_first th {
  border-top-width: 2px;
}

#dyjqzmcdvc .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#dyjqzmcdvc .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#dyjqzmcdvc .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#dyjqzmcdvc .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#dyjqzmcdvc .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#dyjqzmcdvc .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#dyjqzmcdvc .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#dyjqzmcdvc .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#dyjqzmcdvc .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#dyjqzmcdvc .gt_footnotes {
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

#dyjqzmcdvc .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#dyjqzmcdvc .gt_sourcenotes {
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

#dyjqzmcdvc .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#dyjqzmcdvc .gt_left {
  text-align: left;
}

#dyjqzmcdvc .gt_center {
  text-align: center;
}

#dyjqzmcdvc .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#dyjqzmcdvc .gt_font_normal {
  font-weight: normal;
}

#dyjqzmcdvc .gt_font_bold {
  font-weight: bold;
}

#dyjqzmcdvc .gt_font_italic {
  font-style: italic;
}

#dyjqzmcdvc .gt_super {
  font-size: 65%;
}

#dyjqzmcdvc .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#dyjqzmcdvc .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#dyjqzmcdvc .gt_indent_1 {
  text-indent: 5px;
}

#dyjqzmcdvc .gt_indent_2 {
  text-indent: 10px;
}

#dyjqzmcdvc .gt_indent_3 {
  text-indent: 15px;
}

#dyjqzmcdvc .gt_indent_4 {
  text-indent: 20px;
}

#dyjqzmcdvc .gt_indent_5 {
  text-indent: 25px;
}

#dyjqzmcdvc .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#dyjqzmcdvc div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="lmvjqdhzki" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#lmvjqdhzki table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#lmvjqdhzki thead, #lmvjqdhzki tbody, #lmvjqdhzki tfoot, #lmvjqdhzki tr, #lmvjqdhzki td, #lmvjqdhzki th {
  border-style: none;
}

#lmvjqdhzki p {
  margin: 0;
  padding: 0;
}

#lmvjqdhzki .gt_table {
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

#lmvjqdhzki .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#lmvjqdhzki .gt_title {
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

#lmvjqdhzki .gt_subtitle {
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

#lmvjqdhzki .gt_heading {
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

#lmvjqdhzki .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lmvjqdhzki .gt_col_headings {
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

#lmvjqdhzki .gt_col_heading {
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

#lmvjqdhzki .gt_column_spanner_outer {
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

#lmvjqdhzki .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#lmvjqdhzki .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#lmvjqdhzki .gt_column_spanner {
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

#lmvjqdhzki .gt_spanner_row {
  border-bottom-style: hidden;
}

#lmvjqdhzki .gt_group_heading {
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

#lmvjqdhzki .gt_empty_group_heading {
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

#lmvjqdhzki .gt_from_md > :first-child {
  margin-top: 0;
}

#lmvjqdhzki .gt_from_md > :last-child {
  margin-bottom: 0;
}

#lmvjqdhzki .gt_row {
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

#lmvjqdhzki .gt_stub {
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

#lmvjqdhzki .gt_stub_row_group {
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

#lmvjqdhzki .gt_row_group_first td {
  border-top-width: 2px;
}

#lmvjqdhzki .gt_row_group_first th {
  border-top-width: 2px;
}

#lmvjqdhzki .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#lmvjqdhzki .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#lmvjqdhzki .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#lmvjqdhzki .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lmvjqdhzki .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#lmvjqdhzki .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#lmvjqdhzki .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#lmvjqdhzki .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#lmvjqdhzki .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lmvjqdhzki .gt_footnotes {
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

#lmvjqdhzki .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#lmvjqdhzki .gt_sourcenotes {
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

#lmvjqdhzki .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#lmvjqdhzki .gt_left {
  text-align: left;
}

#lmvjqdhzki .gt_center {
  text-align: center;
}

#lmvjqdhzki .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#lmvjqdhzki .gt_font_normal {
  font-weight: normal;
}

#lmvjqdhzki .gt_font_bold {
  font-weight: bold;
}

#lmvjqdhzki .gt_font_italic {
  font-style: italic;
}

#lmvjqdhzki .gt_super {
  font-size: 65%;
}

#lmvjqdhzki .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#lmvjqdhzki .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#lmvjqdhzki .gt_indent_1 {
  text-indent: 5px;
}

#lmvjqdhzki .gt_indent_2 {
  text-indent: 10px;
}

#lmvjqdhzki .gt_indent_3 {
  text-indent: 15px;
}

#lmvjqdhzki .gt_indent_4 {
  text-indent: 20px;
}

#lmvjqdhzki .gt_indent_5 {
  text-indent: 25px;
}

#lmvjqdhzki .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#lmvjqdhzki div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="hppsgvhyyh" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#hppsgvhyyh table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#hppsgvhyyh thead, #hppsgvhyyh tbody, #hppsgvhyyh tfoot, #hppsgvhyyh tr, #hppsgvhyyh td, #hppsgvhyyh th {
  border-style: none;
}

#hppsgvhyyh p {
  margin: 0;
  padding: 0;
}

#hppsgvhyyh .gt_table {
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

#hppsgvhyyh .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#hppsgvhyyh .gt_title {
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

#hppsgvhyyh .gt_subtitle {
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

#hppsgvhyyh .gt_heading {
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

#hppsgvhyyh .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hppsgvhyyh .gt_col_headings {
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

#hppsgvhyyh .gt_col_heading {
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

#hppsgvhyyh .gt_column_spanner_outer {
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

#hppsgvhyyh .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#hppsgvhyyh .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#hppsgvhyyh .gt_column_spanner {
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

#hppsgvhyyh .gt_spanner_row {
  border-bottom-style: hidden;
}

#hppsgvhyyh .gt_group_heading {
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

#hppsgvhyyh .gt_empty_group_heading {
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

#hppsgvhyyh .gt_from_md > :first-child {
  margin-top: 0;
}

#hppsgvhyyh .gt_from_md > :last-child {
  margin-bottom: 0;
}

#hppsgvhyyh .gt_row {
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

#hppsgvhyyh .gt_stub {
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

#hppsgvhyyh .gt_stub_row_group {
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

#hppsgvhyyh .gt_row_group_first td {
  border-top-width: 2px;
}

#hppsgvhyyh .gt_row_group_first th {
  border-top-width: 2px;
}

#hppsgvhyyh .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hppsgvhyyh .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#hppsgvhyyh .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#hppsgvhyyh .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hppsgvhyyh .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hppsgvhyyh .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#hppsgvhyyh .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#hppsgvhyyh .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#hppsgvhyyh .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hppsgvhyyh .gt_footnotes {
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

#hppsgvhyyh .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hppsgvhyyh .gt_sourcenotes {
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

#hppsgvhyyh .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hppsgvhyyh .gt_left {
  text-align: left;
}

#hppsgvhyyh .gt_center {
  text-align: center;
}

#hppsgvhyyh .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#hppsgvhyyh .gt_font_normal {
  font-weight: normal;
}

#hppsgvhyyh .gt_font_bold {
  font-weight: bold;
}

#hppsgvhyyh .gt_font_italic {
  font-style: italic;
}

#hppsgvhyyh .gt_super {
  font-size: 65%;
}

#hppsgvhyyh .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#hppsgvhyyh .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#hppsgvhyyh .gt_indent_1 {
  text-indent: 5px;
}

#hppsgvhyyh .gt_indent_2 {
  text-indent: 10px;
}

#hppsgvhyyh .gt_indent_3 {
  text-indent: 15px;
}

#hppsgvhyyh .gt_indent_4 {
  text-indent: 20px;
}

#hppsgvhyyh .gt_indent_5 {
  text-indent: 25px;
}

#hppsgvhyyh .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#hppsgvhyyh div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="uswnxbmrge" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#uswnxbmrge table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#uswnxbmrge thead, #uswnxbmrge tbody, #uswnxbmrge tfoot, #uswnxbmrge tr, #uswnxbmrge td, #uswnxbmrge th {
  border-style: none;
}

#uswnxbmrge p {
  margin: 0;
  padding: 0;
}

#uswnxbmrge .gt_table {
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

#uswnxbmrge .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#uswnxbmrge .gt_title {
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

#uswnxbmrge .gt_subtitle {
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

#uswnxbmrge .gt_heading {
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

#uswnxbmrge .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#uswnxbmrge .gt_col_headings {
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

#uswnxbmrge .gt_col_heading {
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

#uswnxbmrge .gt_column_spanner_outer {
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

#uswnxbmrge .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#uswnxbmrge .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#uswnxbmrge .gt_column_spanner {
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

#uswnxbmrge .gt_spanner_row {
  border-bottom-style: hidden;
}

#uswnxbmrge .gt_group_heading {
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

#uswnxbmrge .gt_empty_group_heading {
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

#uswnxbmrge .gt_from_md > :first-child {
  margin-top: 0;
}

#uswnxbmrge .gt_from_md > :last-child {
  margin-bottom: 0;
}

#uswnxbmrge .gt_row {
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

#uswnxbmrge .gt_stub {
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

#uswnxbmrge .gt_stub_row_group {
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

#uswnxbmrge .gt_row_group_first td {
  border-top-width: 2px;
}

#uswnxbmrge .gt_row_group_first th {
  border-top-width: 2px;
}

#uswnxbmrge .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#uswnxbmrge .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#uswnxbmrge .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#uswnxbmrge .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#uswnxbmrge .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#uswnxbmrge .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#uswnxbmrge .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#uswnxbmrge .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#uswnxbmrge .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#uswnxbmrge .gt_footnotes {
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

#uswnxbmrge .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#uswnxbmrge .gt_sourcenotes {
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

#uswnxbmrge .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#uswnxbmrge .gt_left {
  text-align: left;
}

#uswnxbmrge .gt_center {
  text-align: center;
}

#uswnxbmrge .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#uswnxbmrge .gt_font_normal {
  font-weight: normal;
}

#uswnxbmrge .gt_font_bold {
  font-weight: bold;
}

#uswnxbmrge .gt_font_italic {
  font-style: italic;
}

#uswnxbmrge .gt_super {
  font-size: 65%;
}

#uswnxbmrge .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#uswnxbmrge .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#uswnxbmrge .gt_indent_1 {
  text-indent: 5px;
}

#uswnxbmrge .gt_indent_2 {
  text-indent: 10px;
}

#uswnxbmrge .gt_indent_3 {
  text-indent: 15px;
}

#uswnxbmrge .gt_indent_4 {
  text-indent: 20px;
}

#uswnxbmrge .gt_indent_5 {
  text-indent: 25px;
}

#uswnxbmrge .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#uswnxbmrge div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="hwbhopkrte" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#hwbhopkrte table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#hwbhopkrte thead, #hwbhopkrte tbody, #hwbhopkrte tfoot, #hwbhopkrte tr, #hwbhopkrte td, #hwbhopkrte th {
  border-style: none;
}

#hwbhopkrte p {
  margin: 0;
  padding: 0;
}

#hwbhopkrte .gt_table {
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

#hwbhopkrte .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#hwbhopkrte .gt_title {
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

#hwbhopkrte .gt_subtitle {
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

#hwbhopkrte .gt_heading {
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

#hwbhopkrte .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hwbhopkrte .gt_col_headings {
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

#hwbhopkrte .gt_col_heading {
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

#hwbhopkrte .gt_column_spanner_outer {
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

#hwbhopkrte .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#hwbhopkrte .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#hwbhopkrte .gt_column_spanner {
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

#hwbhopkrte .gt_spanner_row {
  border-bottom-style: hidden;
}

#hwbhopkrte .gt_group_heading {
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

#hwbhopkrte .gt_empty_group_heading {
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

#hwbhopkrte .gt_from_md > :first-child {
  margin-top: 0;
}

#hwbhopkrte .gt_from_md > :last-child {
  margin-bottom: 0;
}

#hwbhopkrte .gt_row {
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

#hwbhopkrte .gt_stub {
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

#hwbhopkrte .gt_stub_row_group {
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

#hwbhopkrte .gt_row_group_first td {
  border-top-width: 2px;
}

#hwbhopkrte .gt_row_group_first th {
  border-top-width: 2px;
}

#hwbhopkrte .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hwbhopkrte .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#hwbhopkrte .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#hwbhopkrte .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hwbhopkrte .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hwbhopkrte .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#hwbhopkrte .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#hwbhopkrte .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#hwbhopkrte .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hwbhopkrte .gt_footnotes {
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

#hwbhopkrte .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hwbhopkrte .gt_sourcenotes {
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

#hwbhopkrte .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hwbhopkrte .gt_left {
  text-align: left;
}

#hwbhopkrte .gt_center {
  text-align: center;
}

#hwbhopkrte .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#hwbhopkrte .gt_font_normal {
  font-weight: normal;
}

#hwbhopkrte .gt_font_bold {
  font-weight: bold;
}

#hwbhopkrte .gt_font_italic {
  font-style: italic;
}

#hwbhopkrte .gt_super {
  font-size: 65%;
}

#hwbhopkrte .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#hwbhopkrte .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#hwbhopkrte .gt_indent_1 {
  text-indent: 5px;
}

#hwbhopkrte .gt_indent_2 {
  text-indent: 10px;
}

#hwbhopkrte .gt_indent_3 {
  text-indent: 15px;
}

#hwbhopkrte .gt_indent_4 {
  text-indent: 20px;
}

#hwbhopkrte .gt_indent_5 {
  text-indent: 25px;
}

#hwbhopkrte .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#hwbhopkrte div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="palikasmem" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#palikasmem table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#palikasmem thead, #palikasmem tbody, #palikasmem tfoot, #palikasmem tr, #palikasmem td, #palikasmem th {
  border-style: none;
}

#palikasmem p {
  margin: 0;
  padding: 0;
}

#palikasmem .gt_table {
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

#palikasmem .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#palikasmem .gt_title {
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

#palikasmem .gt_subtitle {
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

#palikasmem .gt_heading {
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

#palikasmem .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#palikasmem .gt_col_headings {
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

#palikasmem .gt_col_heading {
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

#palikasmem .gt_column_spanner_outer {
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

#palikasmem .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#palikasmem .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#palikasmem .gt_column_spanner {
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

#palikasmem .gt_spanner_row {
  border-bottom-style: hidden;
}

#palikasmem .gt_group_heading {
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

#palikasmem .gt_empty_group_heading {
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

#palikasmem .gt_from_md > :first-child {
  margin-top: 0;
}

#palikasmem .gt_from_md > :last-child {
  margin-bottom: 0;
}

#palikasmem .gt_row {
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

#palikasmem .gt_stub {
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

#palikasmem .gt_stub_row_group {
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

#palikasmem .gt_row_group_first td {
  border-top-width: 2px;
}

#palikasmem .gt_row_group_first th {
  border-top-width: 2px;
}

#palikasmem .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#palikasmem .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#palikasmem .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#palikasmem .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#palikasmem .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#palikasmem .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#palikasmem .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#palikasmem .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#palikasmem .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#palikasmem .gt_footnotes {
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

#palikasmem .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#palikasmem .gt_sourcenotes {
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

#palikasmem .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#palikasmem .gt_left {
  text-align: left;
}

#palikasmem .gt_center {
  text-align: center;
}

#palikasmem .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#palikasmem .gt_font_normal {
  font-weight: normal;
}

#palikasmem .gt_font_bold {
  font-weight: bold;
}

#palikasmem .gt_font_italic {
  font-style: italic;
}

#palikasmem .gt_super {
  font-size: 65%;
}

#palikasmem .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#palikasmem .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#palikasmem .gt_indent_1 {
  text-indent: 5px;
}

#palikasmem .gt_indent_2 {
  text-indent: 10px;
}

#palikasmem .gt_indent_3 {
  text-indent: 15px;
}

#palikasmem .gt_indent_4 {
  text-indent: 20px;
}

#palikasmem .gt_indent_5 {
  text-indent: 25px;
}

#palikasmem .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#palikasmem div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
<div id="icoytgricu" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#icoytgricu table {
  font-family: Times, system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#icoytgricu thead, #icoytgricu tbody, #icoytgricu tfoot, #icoytgricu tr, #icoytgricu td, #icoytgricu th {
  border-style: none;
}

#icoytgricu p {
  margin: 0;
  padding: 0;
}

#icoytgricu .gt_table {
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

#icoytgricu .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#icoytgricu .gt_title {
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

#icoytgricu .gt_subtitle {
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

#icoytgricu .gt_heading {
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

#icoytgricu .gt_bottom_border {
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#icoytgricu .gt_col_headings {
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

#icoytgricu .gt_col_heading {
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

#icoytgricu .gt_column_spanner_outer {
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

#icoytgricu .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#icoytgricu .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#icoytgricu .gt_column_spanner {
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

#icoytgricu .gt_spanner_row {
  border-bottom-style: hidden;
}

#icoytgricu .gt_group_heading {
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

#icoytgricu .gt_empty_group_heading {
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

#icoytgricu .gt_from_md > :first-child {
  margin-top: 0;
}

#icoytgricu .gt_from_md > :last-child {
  margin-bottom: 0;
}

#icoytgricu .gt_row {
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

#icoytgricu .gt_stub {
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

#icoytgricu .gt_stub_row_group {
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

#icoytgricu .gt_row_group_first td {
  border-top-width: 2px;
}

#icoytgricu .gt_row_group_first th {
  border-top-width: 2px;
}

#icoytgricu .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#icoytgricu .gt_first_summary_row {
  border-top-style: none;
  border-top-color: #D3D3D3;
}

#icoytgricu .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#icoytgricu .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#icoytgricu .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#icoytgricu .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: none;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#icoytgricu .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: none;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#icoytgricu .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#icoytgricu .gt_table_body {
  border-top-style: none;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#icoytgricu .gt_footnotes {
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

#icoytgricu .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#icoytgricu .gt_sourcenotes {
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

#icoytgricu .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#icoytgricu .gt_left {
  text-align: left;
}

#icoytgricu .gt_center {
  text-align: center;
}

#icoytgricu .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#icoytgricu .gt_font_normal {
  font-weight: normal;
}

#icoytgricu .gt_font_bold {
  font-weight: bold;
}

#icoytgricu .gt_font_italic {
  font-style: italic;
}

#icoytgricu .gt_super {
  font-size: 65%;
}

#icoytgricu .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#icoytgricu .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#icoytgricu .gt_indent_1 {
  text-indent: 5px;
}

#icoytgricu .gt_indent_2 {
  text-indent: 10px;
}

#icoytgricu .gt_indent_3 {
  text-indent: 15px;
}

#icoytgricu .gt_indent_4 {
  text-indent: 20px;
}

#icoytgricu .gt_indent_5 {
  text-indent: 25px;
}

#icoytgricu .katex-display {
  display: inline-flex !important;
  margin-bottom: 0.75em !important;
}

#icoytgricu div.Reactable > div.rt-table > div.rt-thead > div.rt-tr.rt-tr-group-header > div.rt-th-group:after {
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
