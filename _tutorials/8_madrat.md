---
layout: tutorial
title:  Input data preparation (MADRaT)
shortID: madrat
lastUpdated:   2020-12-03
author: David M Chen
level: 4
requirements:
  - Requirement A
  - Requirement B
lessonsContent:
  - Introduction to the MADRaT framework
  - Create madrat-based data processing
lessonsLearned:
  - lesson A
  - lesson B
  - (Maybe) lesson C
  - lesson D
published: true
---

## Introduction

In this exercise, we will look into the MADRaT framework; Libraries
organized throught this framework do the bulk of the processing of the
data that goes in and comes out of the MAgPIE model, and are
standardized for consistency. For this exercise, we will work with the
‘mrtutorial’ package as an example of the mr- package structure.

Please fork your own branch of mrtutorial from
<https://github.com/pik-piam/mrtutorial>

Then clone this branch locally.

## Package structure

Directories in the package are automatically generated for the most
part.

You can also create new libraries through RStudio via “New Project –\>
New Directory –\> R Package”.

Important: To make new libraries accessible through the madrat world,
they must include the file “madrat.R” in the R folder of the package.

## MADRaT Functions - Exercise

We will look closesly into the workflow of processing new data sources
to be ready for use in the MAgPIE model. Madrat splits this workflow
into download, read, correct, convert, and calculate steps, each of
which has a specific function wrapper.

### Download function

Note that if direct download not possible, data files can be manually
created in the inputdata/sources folder. This is not the preferred
implemntation, but in this case, a download function is not necessary.
Naming of the source folder must however match the read functions.
Metadata on where the data was obtained, how it was downloaded, etc.
should ideally be documented in the download script. For an ideal case,
please see downloadTau in the madrat package.

Please open the downloadTutorialWDI R script.

``` r
library(madrat)

downloadSource("TutorialWDI")
```

### Read function

Read functions are the first step in transforming input data into
magclass objects. They should be as simple as possible, with most steps
of data cleaning, filling in, and transforming reserved for the correct
function. The Read function should be able to specify between indicators
(subtypes).

Remember that magclass objects are an array with the the spatial
dimension in the first dimension, the temporal dimension in the second,
and data values in the third dimemsion(s) (3.1, 3.2…).

Now let’s look at readTutorialWDI.R. Note here that because of the way
WDI downloads it’s data, the naming of the data is assumed by madrat to
be multiple subdimensions, due to the use of the “.”. This is generally
to be avoided, to avoid confusing names and dimensions. Let’s rena

``` r
x <- readSource("TutorialWDI", subtype="SP.POP.TOTL", convert=FALSE)
```

### Convert Function

The convert function will complete the magclass object: For
country-level data, all 249 countries represented in MAGPie need to have
a value and be in ISO3 country code. toolCountryFill() in the convert
function also removesany codes that it can not match.

Note that here we have omitted a correctSource() function, it is by
default OFF but can be implemented via convert=“onlycorrect”

Please open convertTutorialWDI.R

``` r
x <- readSource("TutorialWDI", subtype="SP.POP.TOTL", convert=TRUE)

```

### calcOutput

Magclass objects are consistent in structure, making calculations easy.
The calcOutput wrapper function calls the functions used to transform
input data, called as calcOutput(“type”, “option1”, “option2”, …).

ag\_gdp() will calculate agricultural gdp as share of total gdp. Let’s
open this function.

``` r
ag_gdp <- calcOutput("AgGDP")
```

By default, calcOutput functions will aggregate to the regional level.
We can also turn the cache on via:

``` r
setConfig(forcecache=T)
```

Now try running the same function again. We can also go see where cache
files are stored. All functions are cached as .Rda in
/inputdata/cache/cache\_folder after the first run, meaning
time-consuming functions do not need to be re-run. This means however
that for any updates to functions, the older cached function needs to be
deleted. Cache is toggled with setConfig(forcecache=TRUE).

## Exercise

Using our new ag\_gdp magpie object, answer the following questions in
break out rooms:

1.  What is Germany’s share of Agricultural GDP in 2010? (Germany’s ISO3
    code is “DEU”)

Let’s imagine that agricultural GDP comes from only 3 sectors:
c(“grains”, “vegetables”, “livestock”), for which each contributes
20%, 15%, 65% of ag. GDP respectively.

2.  Modify the calcAgGDP function so that it includes a 2nd
    sub-dimension in the 3rd dimension (dim 3.2) detailing the split
    into the 3 sectors, while keeping Ag\_GDP\_share in dimension 3.1

Use new.magpie() to get started.

3.  Once the function is modified, you will have to rebuild the
    mrtutorial library in order for these changes to take place. This
    can be done with the Rstudio “Install and Restart” during the
    testing phase, but when ready for commit, lucode2::buildLibrary()
    needs to be run. This requires installing the lucode2 package. Only
    when all Errors, Warnings, and Notes are taken care of can the
    package be correctly committed.
