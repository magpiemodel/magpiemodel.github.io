---
layout: tutorial
title: Model documentation (goxygen)
image: assets/images/generic/pic03.jpg
shortID: goxygen
lastUpdated: 2022-01-05
model: MAgPIE
modelVersion: 4.4.0
author:
  - jpd
  - fb
level: 2
requirements:
  - local copy of the MAgPIE model (<https://github.com/magpiemodel/magpie>)
  - R installed (<https://www.r-project.org/>)
  - Libraries `pandoc` and `pandoc-citeproc` installed (<https://pandoc.org>)
lessonsContent:
  - Explanation of in-code documentation
  - goxygen installation guide (for documentation compilation)
  - Properly reading MAgPIE documentation
  - Updating model documentation
exercises:
  - task: Create the model documentation from code
    solution: "1. go to the main directory of the MAgPIE model folder and start R using the command line \n
               2. run `goxygen::goxygen()`\n
               3. The documentation can now be found in the \"doc\" sub-folder
                  of the model"
  - task: Check the number of interfaces and interactions of the yields module in MAgPIE 4.4.0
    solution: "1. Open the model documentation for MAgPIE 4.10.1 ([here](https://rse.pik-potsdam.de/doc/magpie/4.10.1))\n
               2. Find the yield module ([14. Yields](https://rse.pik-potsdam.de/doc/magpie/4.10.1/14_yields.htm))\n
               3. Count the number of [inputs](https://rse.pik-potsdam.de/doc/magpie/4.10.1/14_yields.htm#input) and
               [outputs](https://rse.pik-potsdam.de/doc/magpie/4.10.1/14_yields.htm#output) to the model\n
               4. Check the links to other modules in the [interface plot](https://rse.pik-potsdam.de/doc/magpie/4.10.1/14_yields.htm#interfaces)\n\n
               **Answer:** \n
               * **12 input** and **3 output** interfaces\n
               * **11 modules** exchange information with the yields module"
  - task: Change the land module to "Water" and add yourself as author
    solution: "1. go to your local MAgPIE copy and navigate to the land module @ `modules/10_land`\n
               2. open `modules.gms` in an editor\n
               3. Replace `Land` after `@title` with `Water`\n
               4. Add your name in the line starting with `@authors`\n
               5. run `goxygen::goxygen()` from the main folder of the model"
published: true
---

## In-code documentation
MAgPIE uses in-code documentation, which means that the code documentation is
not written separately, but is instead part of the code base itself. A sharable
documenation is created by running a documentation extraction tool (in our case
[goxygen]) over the code base.
This approach has several advantages over the traditional approach:

* **It reduces work**: Many parts of the documentation can be extracted directly
from the code, e.g. variable declarations or in case of GAMS code even variable
descriptions are part of the code base and can be taken from there. In a similar
fashion, code snippets or equations can be directly read from code. In a traditional
approach one would need to transfer all these information manually from code to
documentation. With [goxygen] this happens automatically.
Parts that cannot be extracted from the code itself, such as additional
explanations are inserted in the code in form of code comments that serve as 
explanatory text in the documentation.

* **It makes it easier to keep the document up-to-date**: With two separate
locations for code and documentation, it happens quite often in practice that
changes in the code are not being transferred to the documentation, often leading
to outdated model documentations. In-code documentation reduces this risk, by
1) extracting information automatically so that this information will automatically
  be up-to-date and
2) putting the comments directly next to the code so that
  it becomes much more likely that the developer also updates these, when the
  corresponding code changes.

* **Documentation can be created on demand for every state of the code base**:
With in-code documentation everyone can easily create a new model documentation
fitting a specific version of the code base. They just apply the documentation
extraction function on the code. In addition, this procedure allows to also
choose an appropriate format. For instance, MAgPIE documentation can currently
be compiled in Markdown, HTML or PDF format. While Markdown is a suitable format
for further processing, HTML is well suited for online documentations and PDF
works well as attachment to publications.

The following tutorial explains how in-code documentation is implemented in MAgPIE and
how it can be used.

## The goxygen package

MAgPIE uses the [goxygen] R package, which is based on the concept of the
[doxygen](https://www.doxygen.nl) package applied to GAMS code.

To get the most recent version of the package you should make sure
to have the PIK-CRAN repository in your repository list of your R profile:

```
options(repos = c(CRAN = "@CRAN@", pik = "https://rse.pik-potsdam.de/r/packages"))
```

After that you can install it in R via

```
install.packages("goxygen")
```

To use [goxygen] you have to make sure that `pandoc` as well as `pandoc-citeproc`
are installed on your system. Installation instructions for pandoc can be found
at <https://pandoc.org/>.

For a documentation in PDF you additionally need to make sure to have
[Latex](https://www.latex-project.org/) installed.

To extract a documentation from MAgPIE you have to navigate to the main folder
of the model, open R in the command line, load the `goxygen` package 
and run the extraction function
`goxygen()`:

```
library(goxygen)
goxygen()
```

Note that if you do this for the first time, you'll need to wait a few minutes
for renv to set up the package environment. The goxygen package should show up in the list of installed packages there.
More details on renv will follow in tutorial [Managing renv](https://magpiemodel.github.io/tutorials/t04-renv).

By default the function will create a folder `doc` in which the documentation
in markdown, HTML and PDF format will be stored.

## Documentation Structure
The model documentation for MAgPIE [4.10.1](https://github.com/magpiemodel/magpie)
can be found [here](https://rse.pik-potsdam.de/doc/magpie/4.10.1/).
Links to the model documentations of other versions can usually be found in the
`README.md` of the corresponding model version.
Besides the overview page, which gives a general introduction to the model, the
documentation is structured by modules, which are each documented on a
separated HTML page (see for instance the [TC module](https://rse.pik-potsdam.de/doc/magpie/4.10.1/13_tc.htm)).
Each module page starts with a brief introduction of the module, which we call
`module contract`. This part explains the purpose of the module, what kind of
information it computes and what kind of information it requires in order to do so
(the interfaces to other modules). After the description, the interfaces are shown
graphically as well as in table form. The input table lists all variables and
parameters the module needs to receive from other modules in order to work,
while the output table lists the variables/parameters the module provides to
the rest of the code.

After this general module overview section, which is valid for all realizations of the module,
each specific realization is documented. This part is relevant when it comes to
selecting the appropriate realization for a scenario. 
It lists the specific equations used in the given realization and sometimes also other code snippets
of the code relevant to understand the specific realization. Each realization
section ends with a statement about limitations of the given realization, which
can also help in the selection process.

The documentation is not only relevant for people new to the model, but can also
be used as examination tool to better understand certain model results or for
tracking down bugs in the code. Here, in particular the interface information as
well as the module contract can play a role: The interfaces help to better understand
how certain dynamics in the model can occur and how they can be explained, whereas
the module contract helps to determine whether a certain module behaves in line
with this contract or violates it. This helps to determine which part of the model
requires some fixing if the model shows faulty behavior.


## Modify documentation

To modify the documentation one can just change the documentation comments in
the code. To distinguish documentation comments from normal comments they
always start with a `*'` instead of just `*`.
The module contract of a module can always be found in the `module.gms` file
of the corresponding module, while the introduction to a realization is found in
the `realization.gms` files.
To better control the documentation output there are a few control commands
which all start with an `@`. For instance, `@equations` starts a section which
documents equations and makes sure that all following equations in the code
also appear in the documentation. Similarly, `@code` starts a section
for code display that shows up in the documentation. To close
a section the `@stop` command is used.
A complete and up-to-date overview of available commands can be found on the
help page of the `goxygen` function.

[goxygen]:https://github.com/pik-piam/goxygen
