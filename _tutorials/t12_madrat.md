---
layout: tutorial
title:  Input data preparation (MADRaT)
shortID: madrat
lastUpdated:   2022-22-02
model: MADRaT
modelVersion: 1.0.0
author: dc
level: 4
requirements:
  - Basic knowledge of R programming language
  - Fork and clone https://github.com/pik-piam/mrtutorial
lessonsContent:
  - Introduction to the MADRaT framework
  - magclass object functionality
  - MADRaT-based input data preprocessing
  - Portable Unaggregated Collections (PUCs) in MADRaT
exercises:
  - task: " With the ```agg_gdp``` magpie object, answer the following questions, what is Germany’s share of Agricultural GDP in 2010? (Germany’s ISO3
    code is “DEU”)"
    solution: "``agg_gdp[“DEU”,2010,]``"
  - task: "Recalculate a tgz file with a different regional aggregation, using the OECD regionmapping found in folder XXX and the .puc file we have already created. Additionally, insert a text file with the text ''extra OECD text'' in the new output."
    solution: "`` pucAggregate(puc = ''C:/PIK/inputdata/puc/rev2_extra_tutorial_tag.puc'', regionmapping = ''regionmappingOECD.csv'', extra = ''new OECD text'') ``"

published: true
---

## Introduction

In this exercise, we will dig deeper into magclass objects and the MADRaT framework.
Libraries organized through this framework do the bulk of the processing of the
data that goes in and comes out of the MAgPIE model, and are
standardized for consistency. For this exercise, we will work with the
‘mrtutorial’ package as an example of the mr- package structure.

Please fork and locally clone your own branch of mrtutorial from
<https://github.com/pik-piam/mrtutorial>

## Package structure

You can also create new madrat-based libraries through RStudio via “New Project –\>
New Directory –\> R Package”.

Importantly, to make these libraries accessible to  MADRaT functionality,
they must include the file “madrat.R” in the R folder of the package.

## MADRaT Functions - Downloading, Reading, Calculating Model Inputs

We will look closesly into the workflow of processing new data sources
to be ready for use in the MAgPIE model. Madrat splits this workflow
into download, read, correct, convert, and calculate steps, each of
which has a specific function wrapper.

### Download function

Note that if direct download not possible, data files can be manually
created in the inputdata/sources folder. This is not the preferred
implementation, but in this case, a download function is not necessary.
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
pop_no_conv <- readSource("TutorialWDI", subtype="SP.POP.TOTL", convert=FALSE)
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
pop_conv <- readSource("TutorialWDI", subtype="SP.POP.TOTL", convert=TRUE)

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

By default, calcOutput functions will aggregate to the regional level,
otherwise ```agggregate = F``` is reqired to keep the original regional level. 
The region mapping can also be changed in setConfig().

``` r
getConfig()
```

Now try running the same function again. We can also go see where cache
files are stored. All functions are cached as ```.Rds``` files in
```/inputdata/cache\_folder``` after the first time they are run, meaning
time-consuming functions do not need to be re-run from scratch. The caching functionality catches any changes to the function content and/or arguments, the source folder, or any mappings called within the function; in such cases, a new cache file will be created. Caching can toggled with setConfig(forcecache=TRUE).

## Model Preprocessing

### fullMODEL() functions and retrieveData() wrapper

Data from mr-libraries is bundled together into a ```.tgz``` file to be used as model input. This is done via the "full" functions which calls all the ```calcOutput()``` functions needed to be included. For MAgPIE these are 3 separate calls for regional, cellular, and validation data: ```fullMAGPIE()```, ```fullCELLULARMAGPIE```, and ```fullVALIDATION```. "full" functions are called via the retrieveData(), and create a ```.tgz``` file containing the processed (aggregated) function outputs in the ```/inputdata/output/``` folder. These files are aggregated to the regional level in the case of ```fullMAGPIE()``` and the cluster level in the case of ```fullCELLULARMAGPIE```. They also create a Portable Unaggregated Collections (PUCs, see below) tgz file of the function outputs at original resolution, in the ```/inputdata/puc/``` folder. The ```dev``` argument allows for running development-phase preprocessings, with the contents of the ```dev``` argument appended to the ```.tgz``` output. If the ```dev``` flag is used, the ```PUC``` is by default not created.

```r
retrieveData(model = "tutorial", rev=1, dev = "" )
```

### Portable Unaggregated Collections (PUCs)

The ```retrieveData()``` function creates a PUC. This ```tgz``` file includes the outputs of the functions at original resolution. This allows for the easier sharing of processed input data that the user can then aggregate as they prefer, using the function ```pucAggregate```.

Extra arguments can be specified give additional instructions to the preprocessing. In this simple example case, the ```extra``` argument simply allows for the insertion of a text file into the ```.tgz```, and these can be changed when re-calculating the ```.tgz``` from the ```.puc```. In the case of MAgPIE regional pucs ```.puc```, the main argument to be changed is the ```regionmapping``` while for cellular data ```ctype``` (the number of clusters) and ```clusterweight``` (weighting for clusters) can additionally be changed. PUCs thus allow the user to locally process data flexibly for any region mapping, and number and weight of clusters. 

