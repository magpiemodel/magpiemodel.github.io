---
layout: tutorial
title:  Creating a new realization
shortID: changeCode
image: assets/images/generic/pic10.jpg
lastUpdated:   2026-06-05
model: MAgPIE
modelVersion: 4.14.0
author: Florian Humpenöder
level: 4
requirements:
  - A GitHub Codespace for MAgPIE — GAMS, R (with the `gms` and `magpie4` packages) and a MAgPIE checkout are pre-installed
  - The MAgPIE default input data — download it once via `Rscript start.R` -> "download data" (skip this if it is still present from a previous session)
lessonsContent:
  - adding a new module realization
  - changing the MAgPIE GAMS code
  - integrating and testing the new realization
published: true
---

## Introduction

MAgPIE has a modular concept. Each module can have several realizations: one holds
the current default behavior, while you develop a new one alongside it. A typical
workflow is to **copy the current default realization, rename it, and apply your
changes** — which is exactly what we do here.

We add a new realization `pop_growth` to the `urban` land module
(`modules/34_urban`). The module currently has two realizations: `static` (urban
land constant over time) and `exo_nov21` (urban land prescribed from an input
dataset — the default). Our `pop_growth` realization instead lets urban land grow
with population. This is for illustrative purposes only.

`exo_nov21` already pulls urban land towards a cellular **target** using a **soft
penalty** (urban land may deviate from the target at a high cost, which keeps the
model feasible). We reuse that machinery and only change where the target comes
from: instead of reading it from a file, we compute it from population growth. The
reasoning is explained in the [Details](#details-and-background) section at the end.

## Adding a new realization

### Copy the default realization

From the MAgPIE main folder, copy `exo_nov21` to a new `pop_growth` folder. Use the
terminal — in Codespaces you cannot duplicate a folder by clicking on it:

``` bash
cp -r modules/34_urban/exo_nov21 modules/34_urban/pop_growth
```

### Remove the input data

`pop_growth` computes its target instead of reading it from a file, so delete the
input-data parts of the copy:

``` bash
rm -rf modules/34_urban/pop_growth/input.gms \
       modules/34_urban/pop_growth/input \
       modules/34_urban/pop_growth/sets.gms
```

### Edit the files

For each file below, set its content to what is shown. Keep the license header (the
`*** |` block) at the top. In `declarations.gms`, also keep the auto-generated
`R SECTION` block at the bottom — it is updated for you by the helper commands in the
next step. The headers and `R SECTION` blocks are omitted from the code below for
brevity.

#### declarations.gms

Declares the population-growth parameter and the target, the deviation-cost scalar,
the cost and deviation variables, and the equations:

``` r
parameters
 p34_pop_growth(t_all,i)   annual population growth rate (1)
 p34_urban_target(j)       urban land target from population growth (mio. ha)
;

scalars
 s34_urban_deviation_cost  artificial cost for deviating from the urban target (USD17MER per ha)  / 1e+06 /
;

positive variables
 vm_cost_urban(j)          Technical adjustment costs
 v34_cost1(j)              Technical adjustment costs
 v34_cost2(j)              Technical adjustment costs
;

equations
 q34_urban_cell(j)         Constraint for urban land
 q34_urban_cost1(j)        Technical punishment equation
 q34_urban_cost2(j)        Technical punishment equation
 q34_bv_urban(j,potnatveg) Biodiversity value for urban land (Mha)
;
```

#### preloop.gms

Computes the annual population growth rate from the population driver `im_pop`, and
initialises the biodiversity value:

``` r
$ontext
Calculate the annual population growth rate. Since t_all has 5-year time steps, we
divide the change between time steps by the number of years between them
(m_yeardiff) to get annual values.
$offtext
loop(t_all$(ord(t_all) > 1),
 p34_pop_growth(t_all,i) = (im_pop(t_all,i)/im_pop(t_all-1,i) - 1) / m_yeardiff(t_all);
);

* Initialize biodiversity value
vm_bv.l(j2,"urban", potnatveg) =
  pcm_land(j2,"urban") * fm_bii_coeff("urban",potnatveg) * fm_luh2_side_layers(j2,potnatveg);
```

#### presolve.gms

Computes the target each time step — the previous urban land scaled by population
growth — and fixes urban land in the first time step (see
[Details](#details-and-background)):

``` r
vm_carbon_stock.fx(j,"urban",ag_pools,stockType) = 0;

if(ord(t) = 1,
  p34_urban_target(j)   = pcm_land(j,"urban");
  vm_land.fx(j,"urban") = pcm_land(j,"urban");
else
  p34_urban_target(j)   = pcm_land(j,"urban") * (1 + sum(cell(i,j), p34_pop_growth(t,i)) * m_timestep_length);
  vm_land.lo(j,"urban") = 0;
  vm_land.l(j,"urban")  = p34_urban_target(j);
  vm_land.up(j,"urban") = Inf;
);
```

#### equations.gms

Holds the soft penalty: `q34_urban_cost1` and `q34_urban_cost2` measure the
deviation of urban land below and above the target, `q34_urban_cell` charges it to
the costs, and `q34_bv_urban` sets the biodiversity value:

``` r
q34_urban_cost1(j2) ..
            v34_cost1(j2) =g= p34_urban_target(j2) - vm_land(j2,"urban");

q34_urban_cost2(j2) ..
            v34_cost2(j2) =g= vm_land(j2,"urban") - p34_urban_target(j2);

q34_urban_cell(j2) ..
            vm_cost_urban(j2) =e= (v34_cost1(j2) + v34_cost2(j2)) * s34_urban_deviation_cost;

q34_bv_urban(j2,potnatveg) ..
 vm_bv(j2,"urban", potnatveg) =e= vm_land(j2,"urban") * fm_bii_coeff("urban",potnatveg) * fm_luh2_side_layers(j2,potnatveg);
```

#### scaling.gms and realization.gms

Leave `scaling.gms` unchanged (it scales `vm_cost_urban` for the solver). In
`realization.gms`, update the `@description` to document the new behavior:

``` r
*' @description In this realization urban land changes over time with population
*' growth. Each time step urban land is pulled towards a target equal to the previous
*' urban land scaled by population growth, via a high punishment term for deviating
*' from the target. Carbon stocks are assumed zero.

*' @limitations Only for illustrative purpose.
```

### Update and check the code

In the main folder, open an R session (type `R`) and run:

``` r
gms::update_fulldataOutput()
gms::update_modules_embedding()
gms::codeCheck(interactive = TRUE)
```

The first two commands regenerate the auto-managed parts (`postsolve.gms`, the
output declarations in `declarations.gms`, and the phase list in `realization.gms`).
`codeCheck` then verifies the module interfaces. When it stops at an interface that
is not handled in every realization, type a short reason such as `not needed` (or
press Enter) and it writes the `not_used.txt` entry for you — type `n` only if the
interface should actually be handled in code (see
[Details](#details-and-background)). Then quit R with `q()` to return to the
terminal — the commands in the next section run in the shell, not in R.

## Testing the new realization

### Compile and run

Set the realization to `pop_growth` in `main.gms`: open `main.gms` and change the
line `$setglobal urban  exo_nov21` to `$setglobal urban  pop_growth`. Equivalently,
run the `sed` command below in the terminal (not in R). Then check that the model
compiles:

``` bash
sed -i 's/^$setglobal urban .*/$setglobal urban  pop_growth/' main.gms
gams main.gms action=C
```

If there are errors, look into `main.lst`. For a fast test, reduce the number of
time steps to three: change the line `$setglobal c_timesteps  coup2100` to
`$setglobal c_timesteps  quicktest`, or run the command below in the terminal. Then
run the model (this can take 10–15 minutes, depending on the Codespace):

``` bash
sed -i 's/^$setglobal c_timesteps .*/$setglobal c_timesteps  quicktest/' main.gms
gams main.gms
```

This creates a `fulldata.gdx` in the main folder. The commands above are a quick
GAMS test; for a full productive run, use a dedicated start script instead (see
[Details](#details-and-background)).

### Check the results

Start an R session in the main folder (type `R`) and run:

``` r
options(digits=2)
library(magpie4)
gdx <- "fulldata.gdx"
land(gdx,level="glo",types="urban")
land(gdx,level="reg",types="urban")
```

Global urban land now grows with population (your exact numbers may differ slightly
with the input data version):

```
y1995 y2010 y2025
   48    55    60
```

Regionally it grows fastest where population grows fastest (e.g. SSA, MEA) and stays
roughly constant where population stagnates (e.g. JPN, EUR). The base year (y1995)
reproduces the initial urban land (taken from the LUH input data), because urban land
is fixed in the first time step.

## Clean up

When you are done, reset the model to its default state so the following tutorials
are not affected. Run these commands in the terminal — if R is still open from the
previous step, quit it first with `q()`. This restores `main.gms` (in particular the
`urban` realization and the time steps), removes the new realization, and checks the
result:

``` bash
git checkout main.gms modules/
rm -rf modules/34_urban/pop_growth modules/34_urban/exo_nov21/not_used.txt
git status
```

`git status` should report no modified tracked files and no `pop_growth` folder under
`modules/34_urban/` — the model code is back to default. Any untracked run artifacts
that remain (e.g. `restart_*.g00`) are harmless; your run outputs (the `output/`
folder, `fulldata.gdx`) and the downloaded input data are kept.

## Details and background

**Why a soft penalty instead of a hard equation?** Forcing urban land to exactly
match the target with a hard `=e=` equation can make the model infeasible: in a
tight cell there may be no room left to grow urban land without violating another
land constraint. The soft penalty lets urban land deviate from the target at a high
cost (`s34_urban_deviation_cost = 1e6` USD per ha): `q34_urban_cost1` and
`q34_urban_cost2` measure the deviation below and above the target, and
`q34_urban_cell` charges it to `vm_cost_urban`. Urban land therefore follows the
target closely while the model stays feasible. This is the same pattern `exo_nov21`
uses for its input-data target.

**First time step.** Urban land is fixed to its initial value in the first time step,
so the base year keeps the input urban land (taken from the LUH land-use dataset)
rather than the growth target, which would otherwise already move urban land in the
base year. From the second time step on it is freed and follows the target.

**Productive run.** The quick test above runs GAMS directly. For a full run (complete
time horizon and output processing), do not edit `config/default.cfg`. Instead add a
start script `scripts/start/pop_growth.R` that overrides only the urban realization:

``` r
library(gms)
source("scripts/start_functions.R")
source("config/default.cfg")

cfg$title     <- "pop_growth"
cfg$gms$urban <- "pop_growth"

start_run(cfg, codeCheck = FALSE)
```

and launch it with `Rscript start.R runscripts=pop_growth submit=direct` (or run
`Rscript start.R` without arguments for an interactive menu to pick the start script
and submission type). `start_run` writes the config's settings into `main.gms`, so
running this script also resets the `$setglobal` lines edited during the quick test
back to their configured values.

**License header.** Every `.gms` file must start with the MAgPIE license header (the
`*** |` block). `codeCheck` checks for it, so copy it to the top of any new file you
create.

**Interfaces and `not_used.txt`.** `pop_growth` uses the population interface
`im_pop`, which the sibling realizations `static` and `exo_nov21` do not. Every
interface must be addressed in *all* realizations of a module, so `codeCheck` adds
`im_pop, input, not needed` to a `not_used.txt` in `static` and `exo_nov21`.
Conversely, `pop_growth` does not use `sm_fix_SSP2` (which `exo_nov21` uses), so it
gets a `not_used.txt` listing `sm_fix_SSP2`. In interactive mode `codeCheck` stops
at each such interface and asks for a reason; type a short reason (e.g. `not needed`,
or press Enter for a default) and it appends the entry to the relevant
`not_used.txt`. Type `n` only if the interface should actually be handled in the
code.

**What we changed relative to `exo_nov21`.** We removed the input dataset
(`input.gms`, the `input/` folder, `sets.gms`) and the regional-total constraint
`q34_urban_land`, and replaced the input-data target `i34_urban_area` with the
population-growth target `p34_urban_target` computed in `preloop.gms` and
`presolve.gms`. The soft-penalty equations, the biodiversity equation and the
scaling are reused unchanged.
