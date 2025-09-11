--- 
title: "Data Management and Reproducible Science"
author: "Claire S Teitelbaum"
date: "2025-09-11"
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

## Course schedule

Below is the current schedule for this course. This is subject to change as the semester progresses and will be updated here and on eLC.


| Week|Date                         |Theme                |Topic                                                             |Readings                                                 |In-class                                                                                          |Assignments due (EOD)                                 |Other notes                         |
|----:|:----------------------------|:--------------------|:-----------------------------------------------------------------|:--------------------------------------------------------|:-------------------------------------------------------------------------------------------------|:-----------------------------------------------------|:-----------------------------------|
|    1|Thursday, August 14, 2025    |First class          |Introduction to reproducible science                              |Syllabus                                                 |Course overview and pre-survey; installing and setting up R; Discussion: why reproducible science |                                                      |                                    |
|    2|Tuesday, August 19, 2025     |Introduction to R    |Directories, programming terminology, R projects                  |[Chapter 3](#basics)                                     |Exercise 1                                                                                        |Install software (R and Rstudio)                      |Add/drop ends                       |
|    2|Thursday, August 21, 2025    |Introduction to R    |Importing data, using spreadsheets, data structures, writing data |[Chapter 4](#importexport)                               |Exercise 2                                                                                        |                                                      |                                    |
|    3|Tuesday, August 26, 2025     |Introduction to R    |Troubleshooting and error messages, debugging                     |[Chapter 5](#troubleshooting)                            |Exercise 3                                                                                        |Exercises 1 (directories) & 2 (read/write)            |                                    |
|    3|Thursday, August 28, 2025    |Introduction to R    |Manipulating data (columns and rows, indexing)                    |[Chapter 6](#filter-select-mutate)                       |Exercise 4                                                                                        |                                                      |                                    |
|    4|Tuesday, September  2, 2025  |Introduction to R    |Characters and numbers in R                                       |[Chapter 7](#nums-chrs)                                  |Exercise 5                                                                                        |Exercises 3 (troubleshooting) & 4 (data manipulation) |                                    |
|    4|Thursday, September  4, 2025 |Introduction to R    |Dates and times in R                                              |[Chapter 8](#lubridate)                                  |Exercise 6                                                                                        |                                                      |                                    |
|    5|Tuesday, September  9, 2025  |Introduction to R    |Manipulating data (grouping and summarizing; joins)               |[Chapter 9](#manipulation): Sections 1-4                 |Exercise 7                                                                                        |Exercises 5 (characters/numbers) & 6 (dates)          |                                    |
|    5|Thursday, September 11, 2025 |Introduction to R    |Manipulating data (wide and long)                                 |[Chapter 9](#manipulation): Section 5                    |Exercise 7                                                                                        |                                                      |                                    |
|    6|Tuesday, September 16, 2025  |Introduction to R    |Programming best practices (commenting, citing, documentation)    |[Chapter 10](#style)                                     |Continue Exercise 7, end-of-unit wrap-up                                                          |Exercise 7 (data manipulation)                        |                                    |
|    6|Thursday, September 18, 2025 |Data visualization   |Principles of data visualization                                  |[Few 2014, Data Visualization for Human Perception](https://www.interaction-design.org/literature/book/the-encyclopedia-of-human-computer-interaction-2nd-ed/data-visualization-for-human-perception)|Discussion: approaches to data visualization                                                      |                                                      |                                    |
|    7|Tuesday, September 23, 2025  |Data visualization   |Introduction to ggplot2: plotting for data exploration            |[Chapter 11](#ggplot)                                    |Exercise 8                                                                                        |                                                      |                                    |
|    7|Thursday, September 25, 2025 |Data visualization   |Plotting for data exploration                                     |                                                         |Exercise 8                                                                                        |                                                      |                                    |
|    8|Tuesday, September 30, 2025  |Data visualization   |Plotting for communication                                        |[Chapter 12](#data-presentation)                         |Exercise 9                                                                                        |Exercise 8 (data exploration)                         |                                    |
|    8|Thursday, October  2, 2025   |Data visualization   |Plotting for communication                                        |                                                         |Exercise 9                                                                                        |                                                      |                                    |
|    9|Tuesday, October  7, 2025    |Data visualization   |Plotting for communication - plot critique and discussion         |Peruse the R Graph Gallery: https://r-graph-gallery.com/ |Discussion: good graphics                                                                         |Exercise 9 (data visualization)                       |                                    |
|    9|Thursday, October  9, 2025   |Data visualization   |Tables                                                            |[Chapter 13](#tables)                                    |Exercise 10                                                                                       |                                                      |                                    |
|   10|Tuesday, October 14, 2025    |Reproducible science |Principles of reproducibility, data sharing                       |Tedersoo et al. (2021)                                   |Discussion: data sharing                                                                          |Exercise 10 (tables)                                  |                                    |
|   10|Thursday, October 16, 2025   |Reproducible science |Git                                                               |[Chapter 14](#git)                                       |Git practice & troubleshooting                                                                    |                                                      |                                    |
|   11|Tuesday, October 21, 2025    |Reproducible science |GitHub                                                            |[Chapter 15](#github)                                    |GitHub practice & troubleshooting                                                                 |                                                      |                                    |
|   11|Thursday, October 23, 2025   |Reproducible science |Quarto/R Markdown                                                 |[Chapter 16](#markdown)                                  |Exercise 11                                                                                       |                                                      |                                    |
|   12|Tuesday, October 28, 2025    |More R               |Data cleaning and QA/QC                                           |[Chapter 17](#qaqc)                                      |Exercise 12                                                                                       |Exercise 11 (Markdown)                                |                                    |
|   12|Thursday, October 30, 2025   |Advanced R           |for loops and if statements                                       |[Chapter 18](#for-if)                                    |Exercise 13                                                                                       |                                                      |                                    |
|   13|Tuesday, November  4, 2025   |Advanced R           |Functions                                                         |[Chapter 19](#functions)                                 |Exercise 14                                                                                       |Exercise 12 (data cleaning) & 13 (for/if)             |Data set approval for final project |
|   13|Thursday, November  6, 2025  |Advanced R           |Functions and error handling                                      |[Chapter 19](#functions)                                 |Exercise 14                                                                                       |                                                      |                                    |
|   14|Tuesday, November 11, 2025   |Advanced R           |Lists and nested data frames                                      |[Chapter 20](#lists)                                     |Exercise 15                                                                                       |Exercise 14 (Functions)                               |Withdrawal deadline 11/12           |
|   14|Thursday, November 13, 2025  |Advanced R           |Basics of spatial data                                            |[Chapter 21](#spatial)                                   |Exercise 16                                                                                       |                                                      |                                    |
|   15|Tuesday, November 18, 2025   |                     |Flex - bonus topics by request                                    |                                                         |                                                                                                  |Exercise 15 (lists) & 16 (spatial data)               |                                    |
|   15|Thursday, November 20, 2025  |Reproducible science |Metadata and data/code releases                                   |Gundersen (2021)                                         |                                                                                                  |                                                      |                                    |
|   16|Tuesday, November 25, 2025   |                     |In-class work day (project); attendance optional                  |                                                         |Project work                                                                                      |                                                      |Last day of class                   |
|   17|Tuesday, December  2, 2025   |                     |No class - Friday schedule in effect                              |                                                         |                                                                                                  |Final project                                         |                                    |
