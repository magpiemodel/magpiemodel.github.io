Changing MAgPIE Configuration
================
Abhijeet Mishra (<mishra@pik-potsdam.de>)
12 October, 2019

# 1 Introduction

## 1.1 What is configuration

In computing, configuration files (or config files) are files used to
configure the parameters and initial settings for some computer
programs. In the MAgPIE context, a config file applies desired settings
to the model. MAgPIE users may be expected to modify config files by
hand using a simple text editor as these files are human-editable plain
texts.

MAgPIE reads its configuration file just before starting a model run.
Editing these config files by hand maybe practical in cases where a user
is making one run at a time but more often than not, users may have to
work with a combination of different settings for the model.

We’ll discuss *writing your own start script* in the next session which
changes the model setting automatically.

## 1.2 Relevence to MAgPIE

Across Unix-like operating systems many different configuration-file
formats exist, with each application or service potentially having a
unique format, but there is a strong tradition of them being in
human-editable plain text. Filename extensions of .cnf, .conf, .cfg, .cf
or as well .ini and similar are often used. MAgPIE configuration file
has a **.cfg** extension.

MAgPIE 4.1 is designed on a *modular model structure*. This means that
the different *components* of the model interact with each other to act
as a complete model. These components are called **modules** in MAgPIE
terminology. These modules are individual entities and can be thought of
as a **switch(es)** that can be turned on or off.

## 1.3 Learning objectives

The goal of this exercise is to make changes to the *default*
configuration. After completion of this exercise, you’ll be able to:

1.  Understand how MAgPIE configuration works.

  

2.  Change the title and number of simulation steps for your runs.

  

3.  Select which outputs your model should generate.

  

4.  Run the model with updated configuration.

  

# 2 Getting started

As stated earlier, MAgPIE configuration (or settings) can be changed by
hand using a normal text editor (notepad, notepad++ etc.). It is
expected that the participants of the workshop have already set-up a
MAgPIE clone\[1\] from the latest MAgPIE version available on
GitHub\[2\].

The folder where MAgPIE is cloned should typically contain the
files\[3\] as in Figure 1.  

![Contents of folder where MAgPIE is cloneds](figures/mag-clone.png)

Make sure that you have the following files and folders:

1.  Folders : config, core, doc, modules, scripts, standalone  
2.  CITATION.cff  
3.  LICENCS  
4.  main.gms  
5.  CHANGELOG.md, README.md  
6.  output.R, scripts.R  
7.  .Rprofile, .gitignore, .travis.yml  

# 3 MAgPIE configuration

Users are requested to follow the following procedure in order to check
the config file for MAgPIE:

  
  
1\. Naviagte to the folder where you have cloned MAgPIE.  
  
2\. Look for the folder named **config**.  
  
3\. Within this **config** folder, look for the file **default.cfg**.  
  
4\. Right click on **default.cfg** file and open it with a text editor
of your choice.  
  
  
The first few lines of **deafult.cfg** file should look like the
following (with licence text on top):  

``` r
##################
#### SETTINGS ####
##################

cfg <- list()

#### Main settings ####

# short description of the actual run
cfg$title <- "default"

# path to the submodel to be used relative to main model folder
cfg$model <- "main.gms"   #def = "main.gms"

#### input settings ####

# which input data sets should be used?

cfg$input <- c("magpie4.1_default_apr19.tgz")

#Please add system or user specific repositories (such as repos with limited
#access for) through the R option "magpie_repos". Through the append command
#below it will get merged into cfg$repositories

cfg$repositories <- append(list("https://rse.pik-potsdam.de/data/magpie/public"=NULL),
                           getOption("magpie_repos"))
```

  
  
We can go through the contents of this file gradually but the
description of core components from this file can be found in table 1
(as well as in the comments preceeding the respective settings in the
config
file).  

| No | Config               | Description                                                      | Focus |
| -: | :------------------- | :--------------------------------------------------------------- | :---- |
|  1 | cfg$title            | Model title                                                      | x     |
|  2 | cfg$model            | Path to the submodel (relative to main model folder)             |       |
|  3 | cfg$input            | Input data source                                                |       |
|  4 | cfg$repositories     | Repository containing input data                                 |       |
|  5 | cfg$force\_download  | Should data be downloaded even if inputs didn’t change?          |       |
|  6 | cfg$recalibrate      | Yield calibration                                                |       |
|  7 | cfg$calib\_accuracy  | Accuracy for yield calibration                                   |       |
|  8 | cfg$calib\_maxiter   | Max. iterations if precision goal is not met                     |       |
|  9 | cfg$damping\_factor  | Factor determining new calibration factor’s influences on result |       |
| 10 | cfg$calib\_cropland  | Switch for cropland calibration                                  |       |
| 11 | cfg$recalc\_npi\_ndc | Settings for NPI/NDC recalculation                               |       |
| 12 | cfg$policyregions    | National or Sub-national mapping                                 |       |
| 13 | cfg$gms              | Module settings                                                  | x     |
| 14 | cfg$sequential       | How runs should be made                                          |       |
| 15 | cfg$logoption        | Log information                                                  |       |
| 16 | cfg$output           | Generation of output files                                       | x     |
| 17 | cfg$results\_folder  | Results folder name                                              |       |
| 18 | cfg$files2export     | Files copied to output folder                                    |       |
| 19 | cfg$runstatistics    | Folder run statistics location                                   |       |
| 20 | cfg$model\_name      | Name of the overall model                                        |       |
| 21 | cfg$model\_version   | Model version                                                    |       |
| 22 | cfg$developer\_mode  | Developer mode                                                   |       |
| 23 | cfg$debug            | Debugging mode                                                   |       |

Description of MAgPIE configuration

MAgPIE is modular in structure and the various modules can be seen in
the **gms** section (cfg$gms$\*) of the config file. These modules in
MAgPIE can be turned on to select a *module realization* (in case of
multiple *module realizations*, an off setting is also available).

# 4 Exercises

## 4.1 Changing the title

Let us try to see what is the default title of MAgPIE runs. This can be
seen by looking for the following text in default.cfg file (line 17):  

``` r
cfg$title
```

As you can see, the model is titled *default* for obvious reasons.
Usually the **title** setting helps in creating the name of the
**results** folder inside the **output** folder of the model. You can
see this setting by searching for the following test in default.cfg file
(line 619).  

``` r
cfg$results_folder
```

*output/:title::date:* tells us that the results from a MAgPIE run are
stored within the **output** folder where another sub-folder is created
automatically for each run. The name of this *results folder* is a
combination of **model title** and the **current date**. Usually you can
use any character string for the results folder name but it is important
to remember that the model execution stops if a result folder by the
same name exists.  

``` r
Question:

If you happen to start a MAgPIE run right now, can you guess the name of the 
results folder within the output folder?
```

  

In order to change the default tile, users are requested to edit the
setting so as to contain their **affiliation** within the quotations and
save the file after editing. The default setting of title:  

``` r
cfg$title <- "default"
```

Should be changed in the following format:

``` r
cfg$title <- "mag4workshop_<affiliation>"
```

For example:  

``` r
cfg$title <- "mag4workshop_PIK"
```

## 4.2 Changing the time step

MAgPIE, being a recursive dynamic model, can run with various levels of
time steps.

These time steps are usually in five year time intervals but you can
also run MAgPIE with ten year time steps (or a combination thereof).

A full set of **time-step** settings\[4\] can be found in the core sets
defined in the model code under **core/sets.gms**.

Current time-step setting can be found by looking for the following text
in deafult.cfg file (line 80):  

``` r
cfg$gms$c_timesteps
```

“*coup2100*” refers to time steps y1995, y2000, y2005, y2010,y2015,
y2020, y2025, y2030, y2035, y2040, y2045, y2050, y2055, y2060, y2070,
y2080, y2090 and y2100.

In this exercise, we’ll try to change this default time-step setting to
**5** which corresponds to the years **y1995, y2000, y2010, y2020 and
y2030**. Note that MAgPIE starts simlations in y1995.

Similar to how we updated the title, you can simply delete “coup2100”
from the line containing  

``` r
cfg$gms$c_timesteps <- "coup2100"
```

and set it to 5  

``` r
cfg$gms$c_timesteps <- 5
```

## 4.3 Changing the ouputs generated

MAgPIE is also capable of generating some stylized outputs which can be
created automatically once the model run is finished. To see the current
output generation settings, look for the following text in the
default.cfg file (line 614):  

``` r
cfg$output 
```

These ouputs\[5\] can be based on **single** run of the model or can
also be **comparative** in nature where the comparison is made between
two or more runs.

The default MAgPIE configuration runs **rds\_report** (to be used in
appResults), **validation** (as a pdf) and **interpolation** scripts
after a model run is finished. Usually, the creation of validation pdf
takes quite some time, generating a pdf file of about 1500 pages.

In this exercise, we’ll exclude the creation of validation pdf (after a
model run) due to time constraints. To do so, you can delete
“rds\_report”,“validation”,“interpolation” from  

``` r
cfg$output <- c("rds_report","validation","interpolation")
```

and replace it with “rds\_report”,“interpolation”  

``` r
cfg$output <- c("rds_report","interpolation")
```

# 5 Running the model with updated configuration

So far, we have successfully changed:

  

1.  Title of MAgPIE runs

  

2.  Time steps of simulation

  

3.  Outputs desired

  

Now, lets try to run the model with this updated configuration.

To do so, windows users can go to the folder where MAgPIE was cloned and
open a command line prompt there. For windows users, this can be done by
typing **cmd** or **powershell** at the address bar of the folder where
MAgPIE was cloned and then pressing return/enter.

For mac users (if right click -\> open terminal setting is not enabled),
Head into System Preferences and select Keyboard \> Shortcuts \>
Services. Find “New Terminal at Folder” in the settings and click the
box. Now, when you’re in Finder, just right-click a folder and you’re
shown the open to open Terminal.

In the command prompt (or powershell or terminal) you just opened, use
the following command:  

``` r
Rscript start.R
```

You’ll now see a bunch of R scripts on your command prompt, based on
which the model can be run.

We are only focusing on the default script at the moment\[6\] which uses
the **default.cfg** file as starting settings. We have edited this
**default.cfg** file during the course of this exercise and changed some
basic settings.

In order to start the run, Select option 1 (***default***) to tell the
model that we want to make a default settings run (using the updated
default.cfg) and then option 1 again (***Direct execution***).

This will start\[7\] the model run on your local machine.

1.  **Cloning a repository**:
    <https://help.github.com/en/articles/cloning-a-repository>

2.  **MAgPIE repo**: <https://github.com/magpiemodel/magpie>

3.  Users may or may not see the **.git** folder in their model folder
    depending on their *hidden files and folder* settings.

4.  The time step settings are within the **gms** component of the
    **cfg** list.

5.  Complete list of output R scripts can be found in
    **scripts/output/single** or **scripts/output/comparison** folders.

6.  Complete list of starting R scripts can also be found in the
    **scripts**/**start** folder.

7.  To stop a run you can use the key combination **ctrl** + **c** on
    windows machines.
