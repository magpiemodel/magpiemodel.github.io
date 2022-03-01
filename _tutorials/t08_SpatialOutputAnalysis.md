---
layout: tutorial
title:  Spatial output analysis
shortID: output2
lastUpdated:   2022-03-01
model: MAgPIE
modelVersion: 4.4.0
author:
  - pvj
level: 3
requirements:
  - Local copy of the MAgPIE model (<https://github.com/magpiemodel/magpie>)
  - Have R installed (<https://www.r-project.org/>)
  - Have R packages `madrat`, `magclass` and `luplot` installed
lessonsContent:
  - Know where to find spatially explicit output data (NetCDF & `magclass`-format)
  - Data visualisation with the Panoply NetCDF Data Viewer
  - Learn how to make animated time series with output data
  - Use the `luplot` package for spatial visualisation of magclass objects in R
exercises:
  - task: "Read the file `cell.land_0.5.mz` from the MAgPIE output folder into R and assign it to an object `x`"
    solution: "```x <- read.magpie("<path-to-output-folder>/cell.land_0.5.mz```"
  - task: "How many 0.5 degree grid cells, time steps and land use types does the object `x` contain? Also use `magclass::getComment()` to find out in which unit the data is stored."
    solution: "Find out how many 0.5 degree grid cells, time steps and land use types `x` contains by using `str(x)`. You will find out that for default runs `x` covers 59199 grid cells, 19 time steps and 7 land use types. `magclass::getComment(x)` gives `"unit: Mha per grid-cell"`"
  - task: "Calculate the difference in cropland area per grid cell between 2015 and 2050 and store the difference in an object calles `cropDiff`."
    solution: "```cropDiff <- x[,"y2050", "crop"] - x[,"y2015", "crop"]```"
  - task: "Plot the difference in cropland area per grid cell between 2015 and 2050 using `luplot::plotmap()`. See also `?plotmap` for further plotting options."
    solution: "```luplot::plotmap(cropDiff, legend_range = c(-0.25, 0.25))```"

published: true
---

## Introduction

In the previous tutorial, we have learned how to analyse MAgPIE results at the regional scale. The next step, covered by this tutorial, is the analysis of spatially explicit output data at the 0.5 degree level. We therefore build on  knowledge covered by the previous tutorial on how to access the _output_ folder of your local MAgPIE version. If you are unsure, how to access the _output_ folder or if you have not yet conducted a MAgPIE simulation, please, have a look at the previous tutorial(s).

MAgPIE itseld does not operate at the 0.5 degree level, yet many procesess like land competition, agricultural production or irrigation are based on biophysical information at the 0.5 degree grid cell level that is aggregated to spatial simulation units (clusters) in order to address computational constraints.

## Disaggregation scripts

After optimisation, some of the model outputs can be disaggregated back to the 0.5 degree level via dedicated disaggregation scripts, which can be found under `"<your MAgPIE folder>/scripts/output/extra"`. In the default MAgPIE configuration only the script `"extra/disaggregation"` is selected via ``cfg$output``:

``` r
cfg$output <- c("output_check", "rds_report", "validation_short",
                "extra/disaggregation")
```
However, other disaggregation scripts can be added by changing ``cfg$output`` _before_ conducting a model run, e.g. by adding `"extra/disaggregation_BII"`. This will add spatially explicit output data for the _biodiversity intactness index_ (BII) to the output folder.

``` r
cfg$output <- c("output_check", "rds_report", "validation_short",
                "extra/disaggregation", "extra/disaggregation_BII")
```

## Visualisation of spatially explicit MAgPIE output with the _Panoply NetCDF Data Viewer_

Spatially explicit MAgPIE output is usually stored in two different data formats in the _output_ folder, including the MAgPIE-own _magclass_-format with the ending '.mz' and the NetCDF format with the ending '.nc'. This section covers the handling and visualisation of spatial ouputs in the NetCDF format by using the NetCDF data visualisation tool _Panoply_. The _magclass_-format can be read-in and processed in R by using the R-packages `magclass` or `luplot` for visualisation, which will be covered at a later stage.

### Installling _Panoply_

In order to download _Panoply_, please, visit the website <https://www.giss.nasa.gov/tools/panoply/>. Scroll down until you find the header _Get Panoply_ and click on _Download Panoply_. This will lead you to the following page:

![Panoply download](../assets/img/t08_panoply_download.png)

Please choose the right version for your operation system, select a destination folder and proceed with the download. After the download has finished follow the instructions for installing _Panoply_ that can be found on the website for your operating system.

### Make sure Java 9 (or later) is installed

If you encounter an error while trying to execute _Panoply_ after installation, make sure you have **Java 9** (or later) available on your computer. In case you need to install/update Java, go to the website <https://www.oracle.com/java/technologies/downloads/#jdk17-windows> and download the appropriate version for your operating system.

![Panoply download](../assets/img/t08_panoply_download.png)

### Using _Panoply_ to visualise MAgPIE output

Navigate to the folder, in which _Panoply_ is stored and click on Panoply.exe to execute the software. Each time a new session is started in _Panoply_ a new window opens that allows you to select your data. Navigate to the _output_ folder of your local MAgPIE version and select a subfolder containing the results of a finished model run. _Panoply_ will display all NetCDF files that are available in this folder.

![Panoply download](../assets/img/t08_output_data.png)

Select the file 'cell.land_0.5.nc' and press _open_. We now see a window that lists all information that is stored in this file, including the names and the type of data that are available.

![Panoply download](../assets/img/t08_view_data.png)

We can now select the item 'crop' by left-clicking it. Then click _Create Plot_, accept all defaults and press _create_. This will create a plot similar to this example:

![Panoply download](../assets/img/t08_initial_plot.png)

The plot shows the cropland area in each grid cell given in Mha. _Panoply_ also allows us to do basic calculations and plot them. Now, let us compute the difference in cropland area per grid cell between the initial time step and 2050.

We can go back to our data window and click on 'crop' once again.

![Panoply download](../assets/img/t08_select_data.png)

But instead of pressing _Creat plot_, this time click on _Combine plot_. You'll now find two array tabs on the top left-hand side. Also _Panoply_ will prompt the _Arrays_-window for us. Under 'plot' select 'Array 2 - Array 1' and select the year 2050 in "Array 2: crop", as shown below, and hit 'Enter'

![Panoply download](../assets/img/t08_arrays.png)

We can also change the appearance of the plot by reprojection the map to a different coordinate reference system (CRS), or by changing the color scale.

In order to change the CRS, we can open the window _Map projection_ via

![Panoply download](../assets/img/t08_change_projection1.png)

and then, for example, change to the 'Equal Earth' projection by clicking on the _Projection_ drop-down menu in the _Map projection_ window

![Panoply download](../assets/img/t08_change_projection2.png)

and we end up with this map:

![Panoply download](../assets/img/t08_equalEarth.png)

Now let us work a bit on the color scale. For this, select the option _Scale_ under _Window_.

![Panoply download](../assets/img/t08_choose_scale.png)

This will open the following window, where we can tweak a range of map properties.

![Panoply download](../assets/img/t08_choose_scale.png)

The you can set the range of the legend to [-0.25; 0.25] and select the color bar 'CB_PiYG_09.cpt' or any color bar to your liking. However, consider that this is a change map and using a neutral color like grey or white in the center of your color range is helps with the interpretation of the data. Also set the label format to '%.2f'. This will change the legend label format two digits behind the comma. You can also add a custom caption by selecting _custom_ and entering a suitable text. Under _Window_ -> _Label_ we can also change/remove global plot labels. In the end your plot should look similar to this:

![Panoply download](../assets/img/t08_example_CroplandChange.png)

### Creating an animated time series based on MAgPIE projections using _Panoply_

_Panoply_ can also be used to create an animated time series (e.g. in MP4-format) based on spatially explicit MAgPIE projections. In the following we will create an animated time series of changes in cropland area per grid cell, based on the map that we have created in the previous steps.

To create an animation go to _Export Animation_.

![Panoply download](../assets/img/t08_export_animation.png)

This will open up the following window

![Panoply download](../assets/img/t08_animation_options.png)

In order to display relative changes, set Array 1 to 'None (leave fixed)' and select a frame rate of e.g. 1 fps, then click on _Okay_. Choose a file destination, select a file name and press _Save_. _Panoply_ will now create an MP4-animation of the complete time series.

![Panoply download](../assets/img/t08_animation_progress.png)
