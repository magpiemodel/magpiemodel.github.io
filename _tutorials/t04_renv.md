---
layout: tutorial
title: Managing renv
shortID: renv
lastUpdated: 2022-11-24
model: MAgPIE
modelVersion: 4.6.0
author: pf
level: 3
requirements:
  - R installed (<https://www.r-project.org/>)
  - MAgPIE model folder cloned
lessonsContent:
  - Understand what renv is, how it works and why and how MAgPIE uses it.
  - Update and restore renvs via piamenv functions.
  - Get an overview of renv functions to get more control.
categories: tutorial
published: true
---
# basics
MAgPIE uses [renv](https://rstudio.github.io/renv/) for managing required R packages. When starting R from the MAgPIE folder the corresponding renv is automatically activated. This means that packages are installed to and loaded from the renv subfolder instead of the usual package library. Thus updates in the global package library do not affect MAgPIE which is especially important on multi-user systems where packages may be loaded by some users while others are updating these packages. In such a situation the loaded packages are no longer valid and R crashes. Another advantage of renv is that it can document exactly which packages are currently installed, and it makes it easy to go back to a previous package configuration via renv.lock files.

## MAgPIE renv structure
### main renv
- used when starting R in main folder, e.g. `Rscript start.R` or `Rscript output.R`
- should not have renv.lock to prevent confusion when that gets out of sync with the package library
- modify via `piamenv` functions, see below

### run renv
- each run has its own run renv and renv.lock documenting the package environment
- when starting a run the main renv is copied to the run folder
- the run renv is used for the run itself and automatic post-processing
- run renvs should ensure reproducibility, so they must never change

## modifying renv
- to modify your renv use `piamenv` functions described below
- alternatively run `make <target>` on the command line
- this requires `make` which is not pre-installed on Windows
- `make help` will print a list of available targets

### update renv
- updates all pik-piam packages in main renv
- writes updated renv.lock to renv/archive
- user is asked to run this when starting a new run
- run manually with `piamenv::updateRenv()` (`make update-renv`)

### restore renv
- resets main renv to state recorded in renv.lock file from renv/archive or from existing run folder
- run `piamenv::restoreRenv('path/to/renv.lock')`
- or choose renv.lock interactively with `piamenv::restoreRenv()` (`make restore-renv`)

# advanced
## renv files
### renv.lock
- text file which lists for each installed package:
	- version
	- source repo (CRAN, GitHub, ...)
- any renv can be reset to state described in renv.lock

### .Rprofile
- sourced whenever R is started in the same folder as the .Rprofile
- activates the corresponding renv
- one time only: installs all dependencies into renv

### renv folder
- auto-generated and managed by renv
- renv settings
- actual package library
- archive folder for {timestamp}_renv.lock files

## renv functions
The piamenv functions explained earlier should cover all common tasks, use the following for more control.
- `renv::install("package@2.3.4")` install specific package version
- `renv::install("githubuser/package", ref = "<commit hash>")` install package from GitHub, optionally provide commit hash
- `renv::install("/p/tmp/username/yourpackagefolder")` install package from sources
- `renv::remove("package")` uninstall package
- `renv::update(exclude = "renv")` (`make update-all-renv`) update all packages except renv (please do not update renv itself)
- `renv::update("package")` update package
- `renv::status()` show differences between library and renv.lock
- `renv::snapshot()` write renv.lock
- `piamenv::archiveRenv()` (`make archive-renv`) write renv.lock to renv/archive
- renv documentation: https://rstudio.github.io/renv/

## package development
When testing packages in development use `renv::install("githubuser/package")` to install the package from your fork. Be careful with updates, your custom package might get overwritten.

# legacy snapshots
Before MAgPIE started using renv it was using so-called "snapshots" to get a stable package environment. You can restore this snapshot machinery (and disable renv) by renaming `.snapshot.Rprofile` -> `.Rprofile`. If you do, please make sure to *not* commit your changes to `.Rprofile`. For coupled model runs you need to use snapshots, renv does not cover that use case yet. Snapshot support will be removed when coupled model runs are possible with renv.
