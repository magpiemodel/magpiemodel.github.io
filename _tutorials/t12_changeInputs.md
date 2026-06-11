---
layout: tutorial
title:  Input data repositories & Patches
shortID: changeInput
image: assets/images/generic/pic12.jpg
lastUpdated:   2026-06-11
model: MAgPIE
modelVersion: 4.14.0
author:
  - Miodrag Stevanović
level: 4
requirements:
  - Local copy of the MAgPIE model (https://github.com/magpiemodel/magpie)
  - Have R installed (https://www.r-project.org/)
  - Have R package `gms` installed
lessonsContent:
  - Create a local input data repository.
  - Package a patch file.
  - Include a patch in the model configuration
published: true
---

## Introduction

The input data for MAgPIE is prepared by a set of pre-processing
routines that take the data from original sources (e.g. FAO, LPJmL…),
execute additional calculations and convert it to the required MAgPIE
parameter format. These pre-processing routines are not accessible as
open source at the moment. A user is instead provided with ready-made
inputs that are necessary for the model execution.

The input files are setup in the config file `config/default.cfg`,
usually at the beginning of the settings. Currently, the input data is
set as:

``` r
cfg$input <- c(regional    = "rev4.131_h12_magpie.tgz",
               cellular    = "rev4.131_h12_1b5c3817_cellularmagpie_c200_MRI-ESM2-0-ssp245_lpjml-8e6c5eb1.tgz",
               validation  = "rev4.131_h12_92e02314_validation.tgz",
               additional  = "additional_data_rev4.65.tgz",
               calibration = "calibration_H12_FAO_01Apr26.tgz")
```

Once specified in the configuration as the input data, the data is
automatically downloaded (if needed) when the model run is started.

The prepared input data are compressed tar archive files "`.tgz`", which
can be opened with software such as [7-Zip](https://www.7-zip.org/), or
in terminal by `tar` and `untar` commands. The data archive files
contain the following types of data:

  - Cellular input data (e.g. land area, crop yields, water
    requirements, carbon density):
      - `cellular    = "rev4.131_h12_1b5c3817_cellularmagpie_c200_MRI-ESM2-0-ssp245_lpjml-8e6c5eb1.tgz"`
  - Regional input and validation data (e.g. food demand):
      - `regional    = "rev4.131_h12_magpie.tgz"`
      - `validation  = "rev4.131_h12_92e02314_validation.tgz"`
  - Calibration data:
      - `calibration = "calibration_H12_FAO_01Apr26.tgz"`
  - Global and other input data (e.g. conversion factors, national
    policies):
      - `additional  = "additional_data_rev4.65.tgz"`

## Patch input data

There is a specific procedure on how to handle the changing of the input
data. It will be demonstrated by the example of changing the USA NDC
policy on afforestation target at 2030.

### Example: Change national land-based NDC policies

Once the input data is downloaded to the local MAgPIE repository in
forms of different input files in designated input folders (in the core,
scripts, modules and module realizations), a user can update or change
these input data files. This can be done directly by manipulating the
files, but this approach carries a risk that such changes are not
documented and that in certain cases the made changes can be overwritten
by the repeated download of the data from the declared repositories. In
order to avoid this risk, it is recommended to create a local folder
that serves as a repository for the patch files that will apply changes
to the data by overwriting the original data.

### Create a local data repository

The folder for local input data repository can be created anywhere and
it's path must be provided to the settings in `config/default.cfg` file.

Let us assume that the patch folder (`patch_inputdata`) is created in
the main MAgPIE repository. One can do it in R:

``` r
dir.create("./patch_inputdata")
```

or in the command line:

``` cmd
mkdir patch_inputdata
```

Once the directory is created, provide its location to the configuration
file. It's important to keep the list structure of the repository
information:

``` r
cfg$repositories <- append(list("https://rse.pik-potsdam.de/data/magpie/public"=NULL,
                                "./patch_inputdata"=NULL),
                           getOption("magpie_repos"))
```

### Create a patch and package it

Create a sub-directory in the `./patch_inputdata` which is going to be
used for packaging of the patched files.

``` r
dir.create("./patch_inputdata/patch_ndc_usa")
```

Copy the original file the `policy_definition.csv` in the patch folder.
In R:

``` r
file.copy(from="./scripts/npi_ndc/policies/policy_definitions.csv",
          to="./patch_inputdata/patch_ndc_usa/.")
```

or in the command line:

``` cmd
cp scripts/npi_ndc/policies/policy_definitions.csv patch_inputdata/patch_ndc_usa/.
```

Edit the content, in this case update the existing USA afforestation NDC policy
(`USA,affore,ndc,1,1995,2015,7.1`) with a more ambitious target of 15 MHa of afforested area
starting in 2020 and reaching the target at 2030:

``` txt
USA,affore,ndc,1,2020,2030,15
```

After saving the file, package it with the tardir function in R
environment and delete the file patch folder:

``` r
gms::tardir(dir="patch_inputdata/patch_ndc_usa",
               tarfile="patch_inputdata/patch_ndc_usa.tgz")

unlink("patch_inputdata/patch_ndc_usa", recursive = TRUE)
```

### Add the patch file to the configuration

Finally, the configuration file should be informed about the change in
the input data and the existing patch file that replaces the existing
input data. For this, edit the `config/default.cfg` file from:

``` r
cfg$input <- c(regional    = "rev4.131_h12_magpie.tgz",
               cellular    = "rev4.131_h12_1b5c3817_cellularmagpie_c200_MRI-ESM2-0-ssp245_lpjml-8e6c5eb1.tgz",
               validation  = "rev4.131_h12_92e02314_validation.tgz",
               additional  = "additional_data_rev4.65.tgz",
               calibration = "calibration_H12_FAO_01Apr26.tgz")
```

to:

``` r
cfg$input <- c(regional    = "rev4.131_h12_magpie.tgz",
               cellular    = "rev4.131_h12_1b5c3817_cellularmagpie_c200_MRI-ESM2-0-ssp245_lpjml-8e6c5eb1.tgz",
               validation  = "rev4.131_h12_92e02314_validation.tgz",
               additional  = "additional_data_rev4.65.tgz",
               calibration = "calibration_H12_FAO_01Apr26.tgz",
               patch       = "patch_ndc_usa.tgz")
```

It is very important to add the patch file at the end of the listings in
the `cfg$input` listings, because every next `.tgz` archive will
overwrite the files previously imported by the files that are contained
in it.

At the next start of the model by `Rscript`, the new patch will place
the file with change inputs according to the changes in the settings.

## Alternative way of adding a local repository

As an alternative to the steps described above, one can add a local
repository to `getOption("magpie_repos")`. For this, one has to add an
entry in the `.Rprofile` file. Typically `.Rprofile` is located in the
users' home directory `(~/.Rprofile)`.

``` r
options(magpie_repos=list("~/patchdata/"=NULL))
```

The local repository `patchdata` can be located anywhere on your
filesystem. You could then add `patch_ndc_usa.tgz` in this
folder.

> **Hint:** If you can't locate your `.Rprofile`, use `usethis::edit_r_profile()`.

`cfg$repositories` in `default.cfg` can be reverted back.

``` r
cfg$repositories <- append(list("https://rse.pik-potsdam.de/data/magpie/public"=NULL),
                           getOption("magpie_repos"))
```

With this setup, the download script (`Rscript start.R -> 3 Download
data`) will first look into the public repo and secondly into your local
repo for downloading the files specified in `cfg$input`.

## Exercises

### Exercise 1: Test a changed NDC policy via a start script

Write your own starting script that tests the scenario with the changed USA NDC policy
described above. None of the changes should occur in `default.cfg`; the starting script
should instead introduce them to the loaded cfg object. Add it as
`scripts/start/projects/name_of_your_script.R`.

Solution:

``` r
# |  (C) 2008-2020 Potsdam Institute for Climate Impact Research (PIK)
# |  authors, and contributors see CITATION.cff file. This file is part
# |  of MAgPIE and licensed under AGPL-3.0-or-later. Under Section 7 of
# |  AGPL-3.0, you are granted additional permissions described in the
# |  MAgPIE License Exception, version 1.0 (see LICENSE file).
# |  Contact: magpie@pik-potsdam.de

# ----------------------------------------------------------
# description: Test USA NDC
# ----------------------------------------------------------

######################################
#### Script to start a MAgPIE run ####
######################################

library(lucode2)
library(magclass)
library(gms)

source("scripts/start_functions.R")
source("config/default.cfg")

cfg$results_folder <- "output/:title:"
cfg$output <- c("rds_report")

cfg$title <- "SSP2_NDC_default"
cfg <- gms::setScenario(cfg, c("SSP2","NDC"))
start_run(cfg, codeCheck=FALSE)

cfg$title <- "SSP2_NDC_USA"
cfg <- gms::setScenario(cfg, c("SSP2","NDC"))
cfg$input <- c(regional    = "rev4.131_h12_magpie.tgz",
               cellular    = "rev4.131_h12_1b5c3817_cellularmagpie_c200_MRI-ESM2-0-ssp245_lpjml-8e6c5eb1.tgz",
               validation  = "rev4.131_h12_92e02314_validation.tgz",
               additional  = "additional_data_rev4.65.tgz",
               calibration = "calibration_H12_FAO_01Apr26.tgz",
               patch       = "patch_ndc_usa.tgz")
start_run(cfg, codeCheck=FALSE)
```

### Exercise 2: Customize the TC cost regression parameters

The technological change (TC) cost in MAgPIE is computed as a power function of land
use intensity (tau):

```
v13_cost_tc = pc13_land × i13_tc_factor × v13_tau_core ^ i13_tc_exponent × (1 + pm_interest)^15
```

The factor and exponent are loaded from `modules/13_tc/input/f13_tc_factor.cs3` and
`f13_tc_exponent.cs3`, each providing three scenario columns (`low`, `medium`, `high`).
The factor scales the absolute TC cost (a *lower* factor makes technological change
cheaper, so the model invests more in yield growth), while the exponent controls how
steeply the cost rises with land use intensity. The active scenario is selected via
`cfg$gms$c13_tccost` in the start script (default: `"medium"`). The available scenario
names are defined in the `scen13` set in `modules/13_tc/endo_jan22/sets.gms`.

Create a patch that adds a `custom` column to both CSV files and extends the `scen13`
set in `sets.gms` to include `custom`. For the `custom` scenario:

  - **Exponent** (`f13_tc_exponent.cs3`): use a value of 3.0 for all years.
  - **Factor** (`f13_tc_factor.cs3`): keep the `medium` pathway (1672) through 2020, then
    from 2025 make technological change progressively cheaper — more ambitious than the
    `low` pathway — reaching a floor of 1144 from 2040 onward (1540, 1408, 1276 for 2025,
    2030, 2035).

Then write a start script that sets `cfg$gms$c13_tccost <- "custom"` and compares a
default run against a run using the more ambitious, steeper TC cost curve.

> **Note:** The directory structure inside the patch archive must mirror the MAgPIE
> folder layout so that files are extracted to the right locations.

Solution:

``` r
# Create staging directories that mirror the MAgPIE folder structure
dir.create("patch_inputdata/patch_tc_custom/modules/13_tc/input",
           recursive = TRUE)
dir.create("patch_inputdata/patch_tc_custom/modules/13_tc/endo_jan22",
           recursive = TRUE)

# Copy the three files to patch
file.copy("modules/13_tc/input/f13_tc_factor.cs3",
          "patch_inputdata/patch_tc_custom/modules/13_tc/input/.")
file.copy("modules/13_tc/input/f13_tc_exponent.cs3",
          "patch_inputdata/patch_tc_custom/modules/13_tc/input/.")
file.copy("modules/13_tc/endo_jan22/sets.gms",
          "patch_inputdata/patch_tc_custom/modules/13_tc/endo_jan22/.")
```

The `custom` column can be added to the two CSV files in two equivalent
ways. These files are in MAgPIE's `.cs3` format, so you can either treat
them as plain tables with base R, or read them as magclass objects.

**Option A — base R (`read.csv` / `write.csv`):**

``` r
# Exponent: curve of 3.0 for all years
df_exp <- read.csv("patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_exponent.cs3",
                   comment.char = "*")
df_exp$custom <- 3.0
write.csv(df_exp,
          "patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_exponent.cs3",
          row.names = FALSE, quote = FALSE)

# Factor: medium through 2020, then a more ambitious (cheaper) pathway from 2025
df_fac <- read.csv("patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_factor.cs3",
                   comment.char = "*")
df_fac$custom <- df_fac$medium
yrs <- as.integer(sub("y", "", df_fac$dummy))
df_fac$custom[yrs == 2025] <- 1540
df_fac$custom[yrs == 2030] <- 1408
df_fac$custom[yrs == 2035] <- 1276
df_fac$custom[yrs >= 2040] <- 1144
write.csv(df_fac,
          "patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_factor.cs3",
          row.names = FALSE, quote = FALSE)
```

**Option B — magclass (`read.magpie` / `write.magpie`):**

``` r
library(magclass)

# Exponent: curve of 3.0 for all years
m_exp <- read.magpie("patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_exponent.cs3")
m_exp <- add_columns(m_exp, addnm = "custom", dim = 3.1)
m_exp[, , "custom"] <- 3.0
write.magpie(m_exp, "patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_exponent.cs3")

# Factor: medium through 2020, then a more ambitious (cheaper) pathway from 2025
m_fac <- read.magpie("patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_factor.cs3")
m_fac <- add_columns(m_fac, addnm = "custom", dim = 3.1)
m_fac[, , "custom"] <- m_fac[, , "medium"]
m_fac[, c(2025, 2030, 2035), "custom"] <- c(1540, 1408, 1276)
m_fac[, seq(2040, 2150, 5), "custom"] <- 1144
write.magpie(m_fac, "patch_inputdata/patch_tc_custom/modules/13_tc/input/f13_tc_factor.cs3")
```

Continue (with either option) by extending the `scen13` set and packaging
the patch:

``` r
# Extend the scen13 set in sets.gms to include 'custom'
sets_lines <- readLines("patch_inputdata/patch_tc_custom/modules/13_tc/endo_jan22/sets.gms")
sets_lines <- gsub("/low, medium, high/", "/low, medium, high, custom/", sets_lines)
writeLines(sets_lines,
           "patch_inputdata/patch_tc_custom/modules/13_tc/endo_jan22/sets.gms")

# Package and clean up
gms::tardir(dir     = "patch_inputdata/patch_tc_custom",
            tarfile = "patch_inputdata/patch_tc_custom.tgz")
unlink("patch_inputdata/patch_tc_custom", recursive = TRUE)
```

Start script comparing default vs. custom TC costs:

``` r
library(lucode2)
library(magclass)
library(gms)

source("scripts/start_functions.R")
source("config/default.cfg")

cfg$results_folder <- "output/:title:"
cfg$output <- c("rds_report")

cfg$title <- "SSP2_NDC_default_tc"
cfg <- gms::setScenario(cfg, c("SSP2","NDC"))
start_run(cfg, codeCheck=FALSE)

cfg$title <- "SSP2_NDC_tc_custom"
cfg <- gms::setScenario(cfg, c("SSP2","NDC"))
cfg$gms$c13_tccost <- "custom"
cfg$input <- c(regional    = "rev4.131_h12_magpie.tgz",
               cellular    = "rev4.131_h12_1b5c3817_cellularmagpie_c200_MRI-ESM2-0-ssp245_lpjml-8e6c5eb1.tgz",
               validation  = "rev4.131_h12_92e02314_validation.tgz",
               additional  = "additional_data_rev4.65.tgz",
               calibration = "calibration_H12_FAO_01Apr26.tgz",
               patch       = "patch_tc_custom.tgz")
start_run(cfg, codeCheck=FALSE)
```

After running, restore `sets.gms` since it is git-tracked:

``` bash
git checkout modules/13_tc/endo_jan22/sets.gms
```
