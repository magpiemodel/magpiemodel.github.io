---
layout: tutorial
title:  Creating a start script
shortID: startScript
image: assets/images/generic/pic07.jpg
lastUpdated: 2025-06-15
model: MAgPIE
modelVersion: 4.10.0
authors: Abhijeet Mishra, Michael Windisch, Vartika Singh, Michael Crawford
level: 3
requirements:
  - GAMS installed, MAgPIE cloned on local machine
  - Undertaken Tutorials 4 and 5 with an understanding of how to start a model run
lessonsContent:
  - The anatomy of a MAgPIE start script
  - Creating a basic start script
  - Creating a start script with loops
  - Familiarization with the `scenario_config.csv` configuration table
published: true
---

# Introduction

MAgPIE needs a configuration file to parameterize the model scenario before starting a run. While editing these configuration files by hand is sometimes useful for testing, more complex scenario designs quickly outgrow this practicality. This motivates the need for start scripts, which are a flexible way to parameterize and run an arbitrary number and configuration of MAgPIE scenarios.

This tutorial introduces the "default" start script and then builds a few simple extensions to it, modifying several parameters by hand. It then introduces the configuration table for when you need preset configurations (e.g. SSPs).

Before writing our own start script, it's important to know the necessary components of one. In the last section of Tutorial 4, the final exercise is to start a "Default" model run. For this, we first opened a terminal within the base directory of the MAgPIE clone, running the following command:

```bash
Rscript start.R
```

This operation calls the `start.R` script from in the base directory of your MAgPIE clone. Afterwards, you can select: 

```bash
1: default | start run with default.cfg settings
``` 

This will execute a "Default" run of the MAgPIE model. This run will be parameterized entirely by the defaults contained in the `config/default.cfg`. The script itself can be found in `scripts/start/default.R`. 

> The structure of this menu mirrors `scripts/start`, and putting your new project's start scripts within `scripts/start/projects` is the most common approach.

# The anatomy of the default MAgPIE start script

Opening this script, you see the main building blocks of a run script:

```r
# |  (C) 2008-2025 Potsdam Institute for Climate Impact Research (PIK)
# |  authors, and contributors see CITATION.cff file. This file is part
# |  of MAgPIE and licensed under AGPL-3.0-or-later. Under Section 7 of
# |  AGPL-3.0, you are granted additional permissions described in the
# |  MAgPIE License Exception, version 1.0 (see LICENSE file).
# |  Contact: magpie@pik-potsdam.de

# ------------------------------------------------
# description: start run with default.cfg settings
# position: 1
# ------------------------------------------------

# Load start_run(cfg) function which is needed to start MAgPIE runs
source("scripts/start_functions.R")

#start MAgPIE run
start_run(cfg="default.cfg")
```

A MAgPIE start script will have the following basic components. First, it must have a header: 

```r
# |  (C) 2008-2025 Potsdam Institute for Climate Impact Research (PIK)
# |  authors, and contributors see CITATION.cff file. This file is part
# |  of MAgPIE and licensed under AGPL-3.0-or-later. Under Section 7 of
# |  AGPL-3.0, you are granted additional permissions described in the
# |  MAgPIE License Exception, version 1.0 (see LICENSE file).
# |  Contact: magpie@pik-potsdam.de

# ------------------------------------------------
# description: start run with default.cfg settings
# position: 1
# ------------------------------------------------
```

This contains a description of the script and (optionally) a position in the menu for where it should sit in the list of scripts. These are what you see when you source `Rscript start.R` as you actually start the run.

Second, there will have to be a line sourcing the script `start_functions.R`:

```r
# Load start_run(cfg) function which is needed to start MAgPIE runs
source("scripts/start_functions.R")
```

This tells the R environment to source `scripts/start_functions.R`. This script defines helper functions to assemble GAMS sets and update metadata from spatial and module inputs, then encapsulates the full MAgPIE startup workflow—checking dependencies, locking the model, updating inputs, snapshotting the R environment, and submitting the run.

And finally, there will be a line starting the MAgPIE run:

```r
#start MAgPIE run
start_run(cfg="default.cfg")
```

The `start_run` function handles everything necessary to begin the MAgPIE run itself, but needs a configuration. For the default run, it just uses the core `default.cfg`, found in `config/default.cfg`. 

These three building blocks are all that's fundamentally necessary for producing a single MAgPIE run.

# Creating a bespoke, basic start script

The simplest way to generate more than one MAgPIE scenario is to create a configuration list using the `default.cfg` as a template, modify it, starting the run, and then repeating the process. Modifying a few other parameters ensures that the resulting scenarios are easy to process once they're finished.

For the following example, we focus on a user trying to start a "business-as-usual" baseline scenario, as well as a scenario with a carbon price.

```r
# |  (C) 2008-2025 Potsdam Institute for Climate Impact Research (PIK)
# |  authors, and contributors see CITATION.cff file. This file is part
# |  of MAgPIE and licensed under AGPL-3.0-or-later. Under Section 7 of
# |  AGPL-3.0, you are granted additional permissions described in the
# |  MAgPIE License Exception, version 1.0 (see LICENSE file).
# |  Contact: magpie@pik-potsdam.de

# ----------------------------------------------------------
# description: A simple custom start script
# ----------------------------------------------------------

# Load the MAgPIE start-up functions
source("scripts/start_functions.R")

version <- "v1"

### Scenario 1: Default run ───────────────────────────────────────────────────
# Load default configuration
source("config/default.cfg")   # defines `cfg`

cfg$title <- paste(version, "NPi", sep = "_")

cfg$gms$c56_pollutant_prices <- "R34M410-SSP2-NPi2025"              # def = R34M410-SSP2-NPi2025
cfg$gms$c56_pollutant_prices_noselect <- "R34M410-SSP2-NPi2025"     # def = R34M410-SSP2-NPi2025

cfg$gms$c60_2ndgen_biodem <- "R34M410-SSP2-NPi2025"                 # def = R34M410-SSP2-NPi2025
cfg$gms$c60_2ndgen_biodem_noselect <- "R34M410-SSP2-NPi2025"        # def = R34M410-SSP2-NPi2025

# Launch the default MAgPIE run
start_run(cfg, codeCheck = FALSE)

### Scenario 2: Carbon price run ──────────────────────────────────────────────
# Re‐load default config to reset any prior modifications
source("config/default.cfg")

cfg$title <- paste(version, "PkBudg1000", sep = "_")

cfg$gms$c56_pollutant_prices <- "R34M410-SSP2-PkBudg1000"     
cfg$gms$c56_pollutant_prices_noselect <- "R34M410-SSP2-PkBudg1000"

cfg$gms$c60_2ndgen_biodem <- "R34M410-SSP2-PkBudg1000"  
cfg$gms$c60_2ndgen_biodem_noselect <- "R34M410-SSP2-PkBudg1000"

# Launch the carbon‐price MAgPIE run
start_run(cfg, codeCheck = FALSE)
```

This script essentially runs a fully default (though helpfully renamed) MAgPIE scenario, and then a second scenario with a GHG price achieving 2 °C by 2100. More information about each particular parameter can be found in the `default.cfg` itself:

```r
# * GHG emission price scenario
# * Note: For best consistency it is recommended to use trajectories from the most recent
# *     coupled REMIND-MAgPIE runs. Currently, this is R34M410.
# * Available options:
# *     none: no GHG prices
# * R34M410: Coupled REMIND-MAgPIE runs with REMIND 3.4.0.dev619 (dev version close to 3.5) and MAgPIE 4.10
# *  Note: SSP5 is provided but is no longer actively maintained and validated by REMIND
# *  Note: See scripts/start/test_runs.R for mapping of NPi2025, PkBudg1050 and PkBudg650 to RCPs for SSP1/SSP2 and SSP3
# *   NPi2025: Current policies; limit peak warming to 3.0°C in SSP1 and SSP2, but not in SSP3.
# *   PkBudg1050: Peak Budget with 1050 GtCO2 until net-zero CO2 emissions; well-below 2.0°C in 2100
# *   PkBudg650:  Peak Budget with 650 GtCO2 until net-zero CO2 emissions; well-below 1.5°C in 2100 (not feasible for SSP3)
# *     R34M410-SSP1-NPi2025, R34M410-SSP1-PkBudg1000, R34M410-SSP1-PkBudg650,
# *     R34M410-SSP2-NPi2025, R34M410-SSP2-PkBudg1000, R34M410-SSP2-PkBudg650,
# *     R34M410-SSP3-NPi2025, R34M410-SSP3-PkBudg1000,  R34M410-SSP3-rollBack,
# *     R34M410-SSP5-NPi2025, R34M410-SSP5-PkBudg1000, R34M410-SSP5-PkBudg650
# (...)
# * Note: c56_pollutant_prices applies to countries selected in policy_countries56
# * c56_pollutant_prices_noselect applies to all other countries.
# * Available scenarios for c56_pollutant_prices_noselect are identical to c56_pollutant_prices
# * (see above) except for emulator and coupling (which can only be chosen for c56_pollutant_prices)
cfg$gms$c56_pollutant_prices <- "R34M410-SSP2-NPi2025"     # def = R34M410-SSP2-NPi2025
cfg$gms$c56_pollutant_prices_noselect <- "R34M410-SSP2-NPi2025"     # def = R34M410-SSP2-NPi2025
```

Some important facets of this simple runscript to be familiar with:

- Be sure to source (**and then re-source!**) the `cfg` list to its "factory defaults" before each scenario with `source("config/default.cfg")`. Otherwise, with more complicated scenarios you may accidentally (and quietly) mix scenario definitions. Re-sourcing this script resets the `cfg` list by running this code each time in the `default.cfg`:

```r
##################
#### SETTINGS ####
##################

cfg <- list()

#### Main settings ####

# short description of the actual run
cfg$title <- "default"
```

- Providing a **version ID** (`version`, which is here pasted to the `cfg$title`) ensures some degree of traceability for your debugging and analysis down the road. 
- To modify individual parameters, you must change them in the GAMS parameters list. This is done through the **`cfg$gms$` variable**. This method corresponds to the structure in the `default.cfg`. Other parameters, e.g. the title, are accessed in the outer `cfg` list. You can find each of the parameters available by looking at the `default.cfg` (see Tutorial 6).
- If you're out of the development process and know that your GAMS code compiles, `codeCheck = FALSE` elimates pre-run syntax checks of your `.gms` code, saving you a bit of time.

To be able to run the start script, it should be saved in `scripts/start/projects/`, and can then be found through the `Rscript start.R` menu.

# Basic looping is helpful for factorial scenario setups 

Scenario analyses will often require some degree of factorial design, as soon as a user maybe want to understand the independent and combined effects of multiple interventions simultaneously. Basic looping is a good method to ensure reproducability and reduce the chances that you misconfigure any particular scenario.

For this example, after looking at the basic impacts of a carbon price on the land system, the user decides that they're also interested in the mitigation of nitrogen-based pollutants via marginal abatement cost curves (MACCs):

```r
# |  (C) 2008-2025 Potsdam Institute for Climate Impact Research (PIK)
# |  authors, and contributors see CITATION.cff file. This file is part
# |  of MAgPIE and licensed under AGPL-3.0-or-later. Under Section 7 of
# |  AGPL-3.0, you are granted additional permissions described in the
# |  MAgPIE License Exception, version 1.0 (see LICENSE file).
# |  Contact: magpie@pik-potsdam.de

# ----------------------------------------------------------
# description: A custom start script using loops
# ----------------------------------------------------------

# Load the MAgPIE start‐up functions
source("scripts/start_functions.R")

version <- "v1"

GHG_settings <- c(
  NPi        = "R34M410-SSP2-NPi2025",
  PkBudg1000 = "R34M410-SSP2-PkBudg1000"
)

MACC_settings <- c(
  noMACC = 0,
  low    = 10,
  med    = 50,
  high   = 100
)

for (ghg in names(GHG_settings)) {
  for (macc in names(MACC_settings)) {
    source("config/default.cfg")    # reset cfg
    
    # title: version_GHGsetting_MACCsetting
    cfg$title <- paste(version, ghg, macc, sep = "_")
    
    # GHG settings
    cfg$gms$c56_pollutant_prices          <- GHG_settings[ghg]
    cfg$gms$c56_pollutant_prices_noselect <- GHG_settings[ghg]
    cfg$gms$c60_2ndgen_biodem             <- GHG_settings[ghg]
    cfg$gms$c60_2ndgen_biodem_noselect    <- GHG_settings[ghg]
    
    # MACC settings (all to same level)
    lvl <- MACC_settings[macc]
    cfg$gms$s57_maxmac_n_soil      <- lvl
    cfg$gms$s57_maxmac_n_awms      <- lvl
    cfg$gms$s57_maxmac_ch4_rice    <- lvl
    cfg$gms$s57_maxmac_ch4_entferm <- lvl
    cfg$gms$s57_maxmac_ch4_awms    <- lvl
    
    # outputs
    cfg$output <- c("output_check", "extra/disaggregation", "rds_report", "projects/nitrogenBoundaries.R")
    
    start_run(cfg, codeCheck = FALSE)
  }
}
```

This script creates two vectors of user-provided settings, and uses a nested for-loop to cycle through each. 

**Output scripts**: It's also possible to write your own output scripts and start them automatically after your has run finish. To do so, add your output script to the `cfg$output` vector. This vector can be found at the bottom of the `default.cfg`. By default, it is:

```r
cfg$output <- c("output_check", "extra/disaggregation", "rds_report")
```

These are the most important output scripts and many workflows will rely on their results. They check for common issues in completed scenarios, disaggregate some core model outputs to the grid cell level (e.g. land pools, BII), and produce a data frame version of the `report.mif` for convenience, saved as `report.RDS`. If you have your own script to add, either write out the entire vector as I do above, or append it to the existing list, i.e. `cfg$output <- c(cfg$output, "projects/myNewOutputScript.R")`.

# Running consistent scenarios using the `scenario_config.csv`

For even more complicated analyses, users may often need to test internally-consistent scenarios featuring different e.g. SSPs, RCPs, diet shifts, or levels of climate change.

This is particularly tricky because, for instance, changing SSPs will mean modifying dozens of parameters simultaneously. The best way to do this is with a **scenario configuration table**. The default `scenario_config.csv` can be found alongside the `default.cfg` in the `config` directory. Different projects can use their own custom tables by storing them in `config/projects`.

A small portion of the standard `scenario_config.csv` looks like this, to give you an idea:

| Parameter                          | cc | nocc | nocc_hist | SSP1  | SSP2  | SSP3  | SSP4  | SSP5  |
|:-----------------------------------|:--:|:----:|:---------:|:-----:|:-----:|:-----:|:-----:|:-----:|
| gms$c_timesteps                    |    |      |           |       |       |       |       |       |
| gms$c09_pop_scenario               |    |      |           | SSP1  | SSP2  | SSP3  | SSP4  | SSP5  |
| gms$c09_gdp_scenario               |    |      |           | SSP1  | SSP2  | SSP3  | SSP4  | SSP5  |
| gms$c09_pal_scenario               |    |      |           | SSP1  | SSP2  | SSP3  | SSP4  | SSP5  |
| gms$c14_yields_scenario            | cc | nocc | nocc_hist |       |       |       |       |       |
| gms$c15_food_scenario              |    |      |           | SSP1  | SSP2  | SSP3  | SSP4  | SSP5  |
| gms$s15_rumdairy_scp_substitution  |    |      |           | 0     | 0     | 0     | 0     | 0     |
| gms$s15_food_subst_functional_form |    |      |           | 1     | 1     | 1     | 1     | 1     |
| gms$s15_food_substitution_start    |    |      |           |       |       |       |       |       |

Extrapolating this to over 80 settings demonstrates that configuring different SSPs or dietary shifts demands more than a loop or two.

So, in this next example, the user decides that they're interested in running benchmarks for each SSP in MAgPIE. 

```r
# |  (C) 2008-2025 Potsdam Institute for Climate Impact Research (PIK)
# |  authors, and contributors see CITATION.cff file. This file is part
# |  of MAgPIE and licensed under AGPL-3.0-or-later. Under Section 7 of
# |  AGPL-3.0, you are granted additional permissions described in the
# |  MAgPIE License Exception, version 1.0 (see LICENSE file).
# |  Contact: magpie@pik-potsdam.de

# ----------------------------------------------------------
# description: Baseline SSP scenarios using the scenario_config table
# ----------------------------------------------------------

library(gms)
source("scripts/start_functions.R")

version   <- "v1"

### Scenario 1: SSP1 ─────────────────────────────────────────────────────────
source("config/default.cfg")
cfg <- setScenario(cfg, c("cc", "SSP1", "rcp4p5"))
cfg$title <- paste(version, "SSP1", sep = "_")
start_run(cfg = cfg, codeCheck = FALSE)

### Scenario 2: SSP2 ─────────────────────────────────────────────────────────
source("config/default.cfg")
cfg <- setScenario(cfg, c("cc", "SSP2", "rcp4p5"))
cfg$title <- paste(version, "SSP2", sep = "_")
start_run(cfg = cfg, codeCheck = FALSE)

### Scenario 3: SSP3 ─────────────────────────────────────────────────────────
source("config/default.cfg")
cfg <- setScenario(cfg, c("cc", "SSP3", "rcp4p5"))
cfg$title <- paste(version, "SSP3", sep = "_")
start_run(cfg = cfg, codeCheck = FALSE)

### Scenario 4: SSP4 ─────────────────────────────────────────────────────────
source("config/default.cfg")
cfg <- setScenario(cfg, c("cc", "SSP4", "rcp4p5"))
cfg$title <- paste(version, "SSP4", sep = "_")
start_run(cfg = cfg, codeCheck = FALSE)

### Scenario 5: SSP5 ─────────────────────────────────────────────────────────
source("config/default.cfg")
cfg <- setScenario(cfg, c("cc", "SSP5", "rcp4p5"))
cfg$title <- paste(version, "SSP5", sep = "_")
start_run(cfg = cfg, codeCheck = FALSE)
``` 

The core changes to this run script are first calling `library(gms)` to gain access to the function `setScenario`. The default use case of this function takes two arguments: The first is your particular configuration list, which will almost always be called `cfg`, and the second is a list of scenario flags (columns) the user would like to "switch on". 

> **Use caution when with `setScenario`!** Each column flag writes over the last, so be sure that the ordering of your various scenario flags is either entirely non-overlapping or, if necessary, very carefully ordered. 

Finally, it's also possible to create your own custom scenario configuration tables, storing them in `config/projects`. To read them, supply `setScenario` with a third `scenario_config` argument:

```r
cfg <- setScenario(cfg, myNewScenarioFlag, scenario_config = "config/projects/myNewScenarioConfigurationTable.csv")
```

This is out of the scope of this tutorial, but can be seen throughout the scripts stored in `start/projects`.

# Exercise

1.  Copy any of these MAgPIE start scripts into your `scripts/start/projects` directory.
2.  Add another parameter from the `config/default.cfg`, or otherwise modify the parameterization in some way. One could also add new switches from the scenario_config.csv.
3.  Save and run the script with Rscript start.R, choosing your modified entry.
4.  Confirm in the run log or output metadata that your change was applied. You can look at these particular settings within that scenario's output directory, in the `config.yaml`. This contains all the configurations from your MAgPIE run. Looking into it, you will see, e.g.:

```yaml
  s56_fader_end: 2050.0
  s56_fader_target: 1.0
  s56_fader_functional_form: 1.0
  c56_pollutant_prices: R32M46-SSP2EU-NPi
  c56_pollutant_prices_noselect: R32M46-SSP2EU-NPi
  c56_mute_ghgprices_until: y2
```
