Advanced: Change MAgPIE GAMS Code
================
Florian Humpenöder (<humpenoeder@pik-potsdam.de>)
10 December, 2020

  - [1 Introduction](#introduction)
  - [2 Learning objectives](#learning-objectives)
  - [3 Adding a new realization](#adding-a-new-realization)
  - [4 Testing a new realization](#testing-a-new-realization)

### 1 Introduction

MAgPIE has a modular concept. Each module (e.g. `pasture`) can have
several realizations (e.g. `dynamic` and `static`). The purpose of these
realization is a) to maintain the current default model behavior and b)
to keep the model operational while developing a new realization. A
typical use case is the extension of a realization by a specific
feature. In this case, one would copy the current default realization,
rename it properly, and apply the wanted changes. Then, the model
behavior can be compared between the two realizations. Eventually, the
new realization might become the new default at some point and the old
realization is deleted.

### 2 Learning objectives

This tutorial shows how to add a new realization to a module in the
MAgPIE model. To illustrate the different steps, we will expand the
`urban` land module. In the current MAgPIE master, the `urban` land
module has only a `static` realization. In this tutorial, we will add a
`dynamic` realization, which changes urban land based on population
growth.

### 3 Adding a new realization

We want to add a new realization to the `urban` land module. The `urban`
land module is located here: `modules/34_urban`.

#### Add a new realization by duplicating an existing one

Duplicate the `static` folder and rename it to `dynamic`. Now we need to
add and edit files in the `dynamic` folder. Add the following files by
duplicating the `presolve.gms` file: `declarations.gms`, `equations.gms`
and `preloop.gms`. Open each new file and delete the copied code from
`presolve.gms`, but keep the copyright header. Now add the following
content to these files, and modify `presolve.gms` and `realization.gms`.

##### declarations.gms

``` r
equations
 q34_urban(j)               urban land (mio. ha)
;

parameters
 p34_pop_growth(t_all,i)        annual population growth rate (1)
;
```

##### equations.gms

``` r
q34_urban(j2)..
 vm_land(j2,"urban") =e= 
 pcm_land(j2,"urban") * (1 + sum((ct,cell(i2,j2)), p34_pop_growth(ct,i2)) * m_timestep_length);
```

##### preloop.gms

``` r
loop(t_all$(ord(t_all) > 1),
 p34_pop_growth(t_all,i) = (im_pop(t_all,i)/im_pop(t_all-1,i) - 1) / m_yeardiff(t_all);
);
```

##### presolve.gms

``` r
*vm_land.fx(j,"urban") = pcm_land(j,"urban");
vm_carbon_stock.fx(j,"urban",c_pools) = 0;
```

##### realization.gms

``` r
*' @description In the dynamic realization, urban land expands based on population growth.
*' Carbon stocks are fixed to zero because
*' information on urban land carbon density is missing.

*' @limitations Carbon stocks are assumed zero.
```

Hint: I saved these changes in the `urban` land module in a feature
branch. Your code should agree with the code in `f_urban`:
<https://github.com/flohump/magpie/tree/f_urban/modules/34_urban/dynamic>

#### Update the code

To include the new realization `dynamic` properly into the GAMS code we
run a specific R command in the main folder. First navigate in your
command line to the MAgPIE main folder, then open a new R session (type
`R` followed by ENTER), and then copy-paste the following R commands:

``` r
gms::update_fulldataOutput()
gms::update_modules_embedding()
```

These two commands will add a `postsolve.gms` file, and update the
`realization.gms` and `declarations.gms` files. Hint: If you change, add
or delete variables/parameters always run these commands to avoid GAMS
compilation errors.

Run codeCheck to check if all module interfaces exist.

``` r
gms::codeCheck(interactive = TRUE)
```

codeCheck will detect a problem with interfaces in 34\_urban. Follow the
instructions, which will add a `not_used.txt` file.

Now you can quit the R session with `q()`.

### 4 Testing a new realization

#### Start a model run

For a quick GAMS test, we simply set the new realization in the file
`main.gms` in line 256 (replace `static` by `dynamic`). We can then
check if the model compiles correctly with this command evoked from the
command line.

``` r
gams main.gms action=C
```

If you get compilation errors, you have to resolve these first. Look
into the `main.lst` file. It will tell you what kind of error occurred.

To make our test run as fast as possible, we reduce the number of time
steps to 3 and deactivate elastic food demand. For this, we change
`$setglobal c_timesteps coup2100` in main.gms in line 224 to `$setglobal
c_timesteps quicktest`, and change the value of `s15_elastic_demand`
from 1 to 0 in line 74 in `modules/15_food/anthropometrics_jan18/input`.
<https://github.com/flohump/magpie/blob/f_urban/main.gms#L224>
<https://github.com/flohump/magpie/blob/f_urban/modules/15_food/anthropometrics_jan18/input.gms#L74>

Now we can start a test run with this command. This can take up to 10-15
minutes, depending on the resources of your machine.

``` r
gams main.gms
```

GAMS will create a `fulldata.gdx` file in the main folder.

For starting a productive model run, we would have to change the config
file `config/default.cfg` in line 489 (replace `static` by `dynamic`).
We could now start a model run with `Rscript start.R -> 1: default -> 1:
Direct execution`. Or, even better write a start script without changing
`config/default.cfg`.

#### Check the results

Start a new R session in the MAgPIE main folder, and execute these
commands.

``` r
options(digits=2)
library(magpie4)
gdx <- "fulldata.gdx"
land(gdx,level="glo",type="urban")
land(gdx,level="reg",type="urban")
```
