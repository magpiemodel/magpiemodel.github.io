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
  - Install magclass package
lessonsContent:
  - Introduction to the magclass data class
  - magclass object functionality
  - data manipulation with magclass objects
published: true
---

## magclass installation

The `magclass` package is the data class most R functionality of MAgPIE is built on. Installing it is easy:

```r
options(repos = c(CRAN = "https://cran.rstudio.com/", pik = "https://rse.pik-potsdam.de/r/packages"))
install.packages("madrat")
library(madrat)
```

Setting the `repos` option in the first line is optional and adds the PIK-hosted package repository as second repository after the official CRAN repository. Doing so will make sure that you receive the most up-to-date version of `magclass`, without this line the package version currently hosted on CRAN will be installed, which is updated less regularly. 

The package comes with some vignettes (tutorials) which introduce you to the world of `magclass` and which also serve as basis for this tutorial. To list all tutorials run:

```r
vignette(package = "magclass")
```

To open a vignette (e.g. the main vignette `magclass`) just run:

```r
vignette("magclass")
```

All vignettes are also hosted on the internet:

* [magclass main tutorial](https://pik-piam.r-universe.dev/magclass/doc/magclass.html)
* [magclass concept](https://pik-piam.r-universe.dev/magclass/doc/magclass-concept.html)
* [magclass object expansion](https://pik-piam.r-universe.dev/magclass/doc/magclass-expansion.html)
* [magclass 6](https://pik-piam.r-universe.dev/magclass/doc/magclass6.html)
