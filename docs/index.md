--- 
title: "Data Management and Reproducible Science"
author: "Claire S Teitelbaum"
date: "2025-08-05"
site: bookdown::bookdown_site
format:
  html:
    html-table-processing: none
output: bookdown::gitbook
documentclass: book
bibliography: [book.bib, packages.bib]
biblio-style: apalike
link-citations: yes
github-repo: rstudio/bookdown-demo
description: "This book contains lessons and exercises for FANR 5950/8950, Data Management and Reproducible Science, Fall 2025."
---

# Course overview

This book contains lesson and exercises for FANR 5950/8950, Data Management and Reproducible Science, Fall 2025.

Other course materials, including the syllabus, are available on [eLC](https://uga.view.usg.edu/d2l/login).

## How to use this book

Throughout this book, you will find text interspersed with R code. Though not required, it is recommended that you follow along by copying R code from this book into a script on your computer. The code will appear like the one in the box below, and you can copy it by hovering over the box and clicking the clipboard icon in the top-right corner:


``` r
# This is some R code
class_work <- "this"
```




<!-- bookdown::render_book("index.Rmd") -->

## Some general guidance about coding and data management

* There is rarely one right way to write code. One way might be faster, and/or easier to read, and/or simpler, but there are always a *lot* of roads to get to the same destination.
* All programming languages are there to do what you tell them to do. They do that very well. What they do not do is what you want them to do! If your code is throwing errors or producing unexpected results, it's probably because it didn't understand what you wanted it to do. 
* To take the previous two points together: when in doubt, try taking another road and seeing if it leads you to the same place.
* Your system or code may be different than your collaborator's or friend's. That is usually not a problem, as long as they can understand how to find what they need and what to do with it when they do.

## Course schedule

Below is the current schedule for this course. This is subject to change as the semester progresses.

<table class="table table-striped table-hover table-condensed" style="font-size: 12px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;"> Week </th>
   <th style="text-align:left;"> Date </th>
   <th style="text-align:left;"> Theme </th>
   <th style="text-align:left;"> Topic </th>
   <th style="text-align:left;"> Readings </th>
   <th style="text-align:left;"> In-class </th>
   <th style="text-align:left;"> Assignments due (EOD) </th>
   <th style="text-align:left;"> Other notes </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> Thursday, August 14, 2025 </td>
   <td style="text-align:left;"> First class </td>
   <td style="text-align:left;"> Introduction to reproducible science </td>
   <td style="text-align:left;max-width: 5cm; "> Syllabus </td>
   <td style="text-align:left;"> * Installing and setting up R
* Exercise 1 </td>
   <td style="text-align:left;"> | </td>
   <td style="text-align:left;"> | </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:left;"> Tuesday, August 19, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Directories, programming terminology, R projects </td>
   <td style="text-align:left;max-width: 5cm; "> * [Chapter 3](#basics)
* [Alston, J. M., &amp; Rick, J. A. (2021)](https://doi.org/10.1002/bes2.1801). A beginner’s guide to conducting reproducible research. Bulletin o </td>
   <td style="text-align:left;"> the Ecological Society of America, 102(2), 1-14|* D </td>
   <td style="text-align:left;"> scussion: why reproducible science?
* Exercise 1 |Ins </td>
   <td style="text-align:left;"> all software (R and Rstudi </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:left;"> Thursday, August 21, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Importing data, using spreadsheets, data structures, writing data </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 4](#importexport) </td>
   <td style="text-align:left;"> Exercise 2 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:left;"> Tuesday, August 26, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Troubleshooting and error messages, debugging </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 5](#troubleshooting) </td>
   <td style="text-align:left;"> Exercise 3 </td>
   <td style="text-align:left;"> Exercises 1 (directories) &amp; 2 (read/write) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:left;"> Thursday, August 28, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Manipulating data (columns and rows, indexing) </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 6](#filter-select-mutate) </td>
   <td style="text-align:left;"> Exercise 4 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:left;"> Tuesday, September  2, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Characters and numbers in R </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 7](#nums-chrs) </td>
   <td style="text-align:left;"> Exercise 5 </td>
   <td style="text-align:left;"> Exercises 3 (troubleshooting) &amp; 4 (data manipulation) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:left;"> Thursday, September  4, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Dates and times in R </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 8](#lubridate) </td>
   <td style="text-align:left;"> Exercise 6 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:left;"> Tuesday, September  9, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Manipulating data (grouping and summarizing; joins) </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 9](#manipulation) </td>
   <td style="text-align:left;"> Exercise 7 </td>
   <td style="text-align:left;"> Exercises 5 (characters/numbers) &amp; 6 (dates) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:left;"> Thursday, September 11, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Manipulating data (wide and long) </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 9](#manipulation) </td>
   <td style="text-align:left;"> Exercise 7 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:left;"> Tuesday, September 16, 2025 </td>
   <td style="text-align:left;"> Introduction to R </td>
   <td style="text-align:left;"> Programming best practices (commenting, citing, documentation) </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 10](#style) </td>
   <td style="text-align:left;"> Flex, end-of-unit wrap-up </td>
   <td style="text-align:left;"> Exercise 7 (data manipulation) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:left;"> Thursday, September 18, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Principles of data visualization </td>
   <td style="text-align:left;max-width: 5cm; "> [Calling Bullshit Chapter 7](https://research.ebsco.com/plink/92d403ed-42b0-3601-94b4-c84627c98892) </td>
   <td style="text-align:left;"> Discussion: approaches to data visualization </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:left;"> Tuesday, September 23, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Introduction to ggplot2: plotting for data exploration </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 11](#ggplot) </td>
   <td style="text-align:left;"> Exercise 8 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:left;"> Thursday, September 25, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Plotting for data exploration </td>
   <td style="text-align:left;max-width: 5cm; ">  </td>
   <td style="text-align:left;"> Exercise 8 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:left;"> Tuesday, September 30, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Plotting for communication </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 12](#data-presentation) </td>
   <td style="text-align:left;"> Exercise 9 </td>
   <td style="text-align:left;"> Exercise 8 (data exploration) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:left;"> Thursday, October  2, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Plotting for communication </td>
   <td style="text-align:left;max-width: 5cm; ">  </td>
   <td style="text-align:left;"> Exercise 9 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:left;"> Tuesday, October  7, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Plotting for communication - plot critique and discussion </td>
   <td style="text-align:left;max-width: 5cm; "> TBD </td>
   <td style="text-align:left;"> Discussion: good graphics </td>
   <td style="text-align:left;"> Exercise 9 (data visualization) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:left;"> Thursday, October  9, 2025 </td>
   <td style="text-align:left;"> Data visualization </td>
   <td style="text-align:left;"> Tables </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 13](#tables) </td>
   <td style="text-align:left;"> Exercise 10 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:left;"> Tuesday, October 14, 2025 </td>
   <td style="text-align:left;"> Reproducible science </td>
   <td style="text-align:left;"> Principles of reproducibility, data sharing </td>
   <td style="text-align:left;max-width: 5cm; "> external TBD. https://www.nature.com/articles/s41597-021-00981-0 </td>
   <td style="text-align:left;"> Discussion: data sharing </td>
   <td style="text-align:left;"> Exercise 10 (tables) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:left;"> Thursday, October 16, 2025 </td>
   <td style="text-align:left;"> Reproducible science </td>
   <td style="text-align:left;"> Git </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 14](#git) </td>
   <td style="text-align:left;"> Git practice &amp; troubleshooting </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:left;"> Tuesday, October 21, 2025 </td>
   <td style="text-align:left;"> Reproducible science </td>
   <td style="text-align:left;"> GitHub </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 15](#github) </td>
   <td style="text-align:left;"> GitHub practice &amp; troubleshooting </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:left;"> Thursday, October 23, 2025 </td>
   <td style="text-align:left;"> Reproducible science </td>
   <td style="text-align:left;"> Quarto/R Markdown </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 16](#markdown) </td>
   <td style="text-align:left;"> Exercise 11 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 12 </td>
   <td style="text-align:left;"> Tuesday, October 28, 2025 </td>
   <td style="text-align:left;"> More R </td>
   <td style="text-align:left;"> Data cleaning and QA/QC </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 17](#qaqc) </td>
   <td style="text-align:left;"> Exercise 12 </td>
   <td style="text-align:left;"> Exercise 11 (Markdown) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 12 </td>
   <td style="text-align:left;"> Thursday, October 30, 2025 </td>
   <td style="text-align:left;"> Advanced R </td>
   <td style="text-align:left;"> for loops and if statements </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 18](#for-if) </td>
   <td style="text-align:left;"> Exercise 13 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 13 </td>
   <td style="text-align:left;"> Tuesday, November  4, 2025 </td>
   <td style="text-align:left;"> Advanced R </td>
   <td style="text-align:left;"> Functions </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 19](#functions) </td>
   <td style="text-align:left;"> Exercise 14 </td>
   <td style="text-align:left;"> Exercise 12 (data cleaning) &amp; 13 (for/if) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 13 </td>
   <td style="text-align:left;"> Thursday, November  6, 2025 </td>
   <td style="text-align:left;"> Advanced R </td>
   <td style="text-align:left;"> Functions and error handling </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 19](#functions) </td>
   <td style="text-align:left;"> Exercise 14 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 14 </td>
   <td style="text-align:left;"> Tuesday, November 11, 2025 </td>
   <td style="text-align:left;"> Advanced R </td>
   <td style="text-align:left;"> Lists and nested data frames </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 20](#lists) </td>
   <td style="text-align:left;"> Exercise 15 </td>
   <td style="text-align:left;"> Exercise 14 (Functions) </td>
   <td style="text-align:left;"> Withdrawal deadline 11/12 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 14 </td>
   <td style="text-align:left;"> Thursday, November 13, 2025 </td>
   <td style="text-align:left;"> Advanced R </td>
   <td style="text-align:left;"> Basics of spatial data </td>
   <td style="text-align:left;max-width: 5cm; "> [Chapter 21](#spatial) </td>
   <td style="text-align:left;"> Exercise 16 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 15 </td>
   <td style="text-align:left;"> Tuesday, November 18, 2025 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Flex - bonus topics by request </td>
   <td style="text-align:left;max-width: 5cm; ">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Exercise 15 (lists) &amp; 16 (spatial data) </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 15 </td>
   <td style="text-align:left;"> Thursday, November 20, 2025 </td>
   <td style="text-align:left;"> Reproducible science </td>
   <td style="text-align:left;"> Metadata and data/code releases </td>
   <td style="text-align:left;max-width: 5cm; "> https://royalsocietypublishing.org/doi/full/10.1098/rsta.2020.0210 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:left;"> Tuesday, November 25, 2025 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> In-class work day (project); attendance optional </td>
   <td style="text-align:left;max-width: 5cm; ">  </td>
   <td style="text-align:left;"> Project work </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Last day of class </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 17 </td>
   <td style="text-align:left;"> Tuesday, December  2, 2025 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> No class - Friday schedule in effect </td>
   <td style="text-align:left;max-width: 5cm; ">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Final project </td>
   <td style="text-align:left;">  </td>
  </tr>
</tbody>
</table>
