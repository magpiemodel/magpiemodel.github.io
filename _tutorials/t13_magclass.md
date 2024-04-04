---
layout: tutorial
title:  Data processing with magclass
shortID: magclass
lastUpdated:   2024-26-03
model: magclass
modelVersion: 6.15.0
author: jpd
level: 3
requirements:
  - Basic knowledge of R programming language
  - R installation
lessonsContent:
  - Introduction to the magclass data class
  - magclass object functionality
  - data manipulation with magclass objects
exercises:
  - task: "Take the \"animal\" example data set from the magclass package and compute how many rabbits there are reported in Luxembourg (LUX) in may of each available year."
    solution: "1. `library(magclass)` \n
               2. `a <- maxample(\"animal\")` \n
               3. `dimSums(a[\"LUX\",\"may\",\"rabbit\"], dim = 3)` \n
               4. Result is: 16 in 2000, 34 in 2001 and 100 in 2002"
  - task: "Write down three options to select the values for birds in Luxembourg (LUX) from the \"animal\" example data set."
    solution: "1. `library(magclass)` \n
               1. `a <- maxample(\"animal\")` \n
               2. `a[\"LUX\",,\"bird\"]` \n
               3. `mselect(a, country = \"LUX\", species = \"bird\")` \n
               4. `a[list(country = \"LUX\"),,list(species = \"bird\")]`\n
               5. Bonus: `a[\"LUX\", dim = 1][\"bird\", dim = 3]`"
  - task: "Set all values for red birds in the animal data set to 0 and all values greater than 10 to 10."
    solution: "1. `library(magclass)` \n
               1. `a <- maxample(\"animal\")` \n
               2. `a[,,\"bird\"][,,\"red\"] <- 0` \n
               3. `a[a > 10] <- 10`"
  - task: "Rename all dogs in the animal data set to cats and remove the \"type\" subdimension."
    solution: "1. `library(magclass)` \n
               1. `a <- maxample(\"animal\")` \n
               2. `getItems(a, dim = \"species\")[3] <- \"cat\"` \n
               3. `getItems(a, dim = \"type\") <- NULL`"
published: true
---

## magclass installation

The `magclass` package is the data class most R functionality of MAgPIE is built on. Installing it is easy:

```r
options(repos = c(CRAN = "https://cran.rstudio.com/",
                  pik = "https://rse.pik-potsdam.de/r/packages"))
install.packages("magclass")
library(magclass)
```

Setting the `repos` option in the first line is optional and adds the PIK-hosted package repository as second repository after the official CRAN repository. Doing so will make sure that you receive the most up-to-date version of `magclass`, without this line the package version currently hosted on CRAN will be installed, which is updated less regularly. 

## Vignettes

The package comes with some vignettes (tutorials) which introduce you to the world of `magclass` and which also serve as basis for this tutorial. To list all tutorials run:

```r
vignette(package = "magclass")
```

To open a vignette (e.g. the main vignette `magclass`) just run:

```r
vignette("magclass")
```

All vignettes are also hosted on the internet:

* [**magclass concept**](https://pik-piam.r-universe.dev/magclass/doc/magclass-concept.html): Explaining the general concept behing magclass objects in comparison to arrays and data frames
* [**magclass main tutorial**](https://pik-piam.r-universe.dev/magclass/doc/magclass.html): Central tutorial explaining the basics how to use magclass
* [**magclass 6**](https://pik-piam.r-universe.dev/magclass/doc/magclass6.html): Tutorial about some important changes which came with the complete rewrite of the package in version 6. 
* [**magclass object expansion**](https://pik-piam.r-universe.dev/magclass/doc/magclass-expansion.html): more specific tutorial going into the details of object expansion and set matching.
