
<!-- README.md is generated from README.Rmd. Please edit that file -->

# gnumaker

Version: 0.0.9000

## Overview

**gnumaker** makes if easy to create and use GNU Makefiles to aid a
reproducible work flow for data analysis projects.

GNU Make is the defacto standard for efficiently rerunning appropriate
steps in the data analysis or reporting process if a particular file is
changed. Only the necessary steps are rerun.

Rather than creating a new system for setting up and building output
from statistical software syntax files, **gnumaker** leverages off
existing GNU Make rules. These rules, for R, Sweave, R Markdown, Stata,
SAS and other syntax files are available at [r-makefile-definitions on
Github](https://github.com/petebaker/r-makefile-definitions). These are
described in P Baker (2019) Using GNU Make to Manage the Workflow of
Data Analysis Projects, *Journal of Statistical Software (Accepted)*.

For those not familiar with GNU Make, **gnumaker** allows simple
dependencies between files to be specified to produce a working Makefile
and the associated directed acyclic graph (DAG). I’d welcome Github
issues containing error reports or feature requests. Alternatively, you
can email the package maintainer at drpetebaker at gmail dot com.

## Installation

<!--
Install the latest CRAN version of **gnumaker** with:


```r
##install.packages("gnumaker")
```
-->

You can install the development version of **gnumaker** from GitHub
with:

``` r
## if you don't have devtools installed, first run:
## install.packages("devtools")
devtools::install_github("petebaker/gnumaker")
```

## Usage

There are three key functions in **gnumaker**. These are:

  - `create_makefile()` creates a gnu\_makefile object given
    dependencies between syntax, data and output files,
  - `plot_makefile()` plots a DAG for a gnu\_makefile object and
  - `write_makefile()` writes a Makfile to disk.

## Examples

Suppose we have a data file `simple.csv` and use `read.R` to read and
clean the data. After storing the cleaned data in a .RData file, we then
employ `linmod.R` to plot and analyse the data. Next, using the stored
results two reports, `report1.pdf` and `report2.docx` are produced from
`report1.Rmd` and `report2.Rmd`. The workflow may be encapsulated in a
Makefile which is then employed to manage the process and generate or
regenerate any intermediate files when the data or syntax changes.

``` r
library(gnumaker)

## Here we assume the default extension for running R file is Rout
## (default) but instead specify the output types for the .Rmd files
## targets 'read' depends on read.r & simple.csv, 'linmod' on linmod.R
## and 'read' etc
gm1 <-
  create_makefile(targets = list(read = c("read.R", "simple.csv"),
                                 linmod = c("linmod.R", "read"),
                                 rep1 = c("report1.Rmd", "linmod"),
                                 rep2 = c("report2.Rmd", "linmod")),
                  phony = list(all = c("rep1", "rep2")),
                  comments = list(linmod = "plots and analysis using 'linmod.R'")
                  file.exts = list(Rmd = list(rep1 = "pdf", rep2 = "docx")))
write_makefile(gm1)
plot_makefile(gm1)                  
```

The Makefile produced with `write_makefile(gm1)` is

``` bash
.PHONY: all
all: report1.pdf report2.docx

## produce report(s) from .Rmd: depends on linmod.Rout
report1.pdf: ${@:.pdf=.Rmd} linmod.Rout

## produce report(s) from .Rmd: depends on linmod.Rout
report2.docx: ${@:.docx=.Rmd} linmod.Rout

## plots and analysis using 'linmod.R'
linmod.Rout: ${@:.Rout=.R} read.Rout

## read.Rout: depends on read.R and simple.csv
read.Rout: ${@:.Rout=.R} simple.csv

include r-rules.mk
```

The DAG of the Makefile produced with `plot_makefile(gm1)` is

![DAG for simple demo Makefile. Using the minimal set of files (shown in
green rectangles), then GNU Make allows us to (re)generate all other
files shown as wheat coloured circles))](images/simple-demo.png)

For more examples, see the gnumaker vignette (under construction).
