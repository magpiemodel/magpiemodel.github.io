---
layout: tutorial
title:  Model documentation (goxygen)
shortID: goxygen
lastUpdated:   2022-01-05
model: MAgPIE
modelVersion: 4.x
author:
  - jpd
level: 2
requirements:
  - local copy of the MAgPIE model
  - R installed
  - Libraries "pandoc" and "pandoc-citeproc" installed (https://pandoc.org)
lessonsContent:
  - Explaination of in-code documentation
  - goxygen installation guide (for documentation compilation)
  - Properly reading MAgPIE documentation
  - Updating model documentation
lessonsLearned:
  - finding the model documentation for a specific version
  - understanding the module contract and realization descriptions
  - knowing how to update the documentation
published: true
---

## In-code documentation
MAgPIE uses in-doce documentation, which means that the code documentation is
not written separately, but is instead part of the code base itself. A sharable
documenation is created by running a documentation extraction tool (in our cases
[goxygen]) over the code base.
This approach has several advantages over the traditional approach:

* **It reduces work**: Many parts of the documentation can be extracted directly
from the code, e.g. Variable declarations or in case of GAMS code even Variable
descriptions are part of the code base and can be taken from there. In i similar
fashion code snippets or equations can be directly read from code. In a traditional
approach one would need to transfer all these information manually from code to
documentation. With [goxygen] this happens automatically.
Parts which cannot be extracted from the code itself, such as additional
explanations are inserted in the code in form of code comments.

* **It makes it easier to keep the document up-to-date**: With two separate
locations for code and documentation in happens in practice quite often that
changes in the code are not being transferred to the documentation, often leading
to outdated model documentations. In-code documentation reduces this risk, by
1) extracting information automatically so that this information will automatically
  be up-to-date and 2) putting the comments directly next to the code so that
  it becomes much more likely that the developer also updates these, when the
  corresponding code changes.

* **Documentation can be created on demand for every state of the code base**:
With in-code documentation everyone can easily create a new model documentation
fitting a specific version of the code base just by applying the documentation
extraction function on the code. In addition, this procedure allows to also
choose an appropriate format. For instance MAgPIE documentation can currently
be compiled in Markdown, HTML or PDF format. While Markdown is a suitable format
for further processing, HTML is well suited for online documentations and PDF
works well as attachment to publications.

In the following it is explained how all that is implemented in MAgPIE and
can be used.

## The goxygen package

MAgPIE uses the [goxygen] R package which taking the concept of the
[doxygen](https://www.doxygen.nl) package and applying it on GAMS code.

To get the most recent version of the package you should make sure
to have the PIK-CRAN repository in your repository list:

```
options(repos = c(CRAN = "@CRAN@", pik = "https://rse.pik-potsdam.de/r/packages"))
```

After that you can install it in R via

```
install.packages("goxygen")
```

To use [goxygen] you have to make sure that `pandoc` as well as `pandoc-citeproc`
are installed on your system. Installation instructions for pandoc can be found
at https://pandoc.org/.

For a documentation in PDF you additionally need to make sure to have
[Latex](https://www.latex-project.org/) installed.

To extract a documentation from MAgPIE you have to navigate to the main folder
of the model, load the `goxygen` package and run the extraction function
`goxygen()`:

```
library(goxygen)
goxygen()
```
By default the function will create a folder `doc` in which the documentation
in markdown, HTML and PDF format will be stored.

## Documentation Structure
The model documentation for MAgPIE 4.4.0 can be found at <https://rse.pik-potsdam.de/doc/magpie/4.4.0>.
Besides the Overview page, which gives a general introduction to the model, the
documentation is structured by modules, which each one of them documented on a
separated HTML page (see for instance the [TC module](https://rse.pik-potsdam.de/doc/magpie/4.4.0/13_tc.htm)).
Each module page starts with a brief introduction of the module which we call
`module contract`. This part explains the purpose of the module, what kind of
information it computes and what kind of information it requires in order to do so
(the interfaces to other modules). After the description the interfaces are shown
graphically as well as in table form. The input table lists all variables and
parameters the module needs to receive from other modules in order to work,
while the output table lists the variables/parameters the module provides to
the rest of the code.

After that general section, which is valid for all implementations of the module,
each specific realization is documented. This part is relevant when it comes to
selecting the appropriate realization for an experiment and lists the specific
equations used in the given realization and sometimes also other code snippets
of the code relevant to understand the specific realization. Each realization
section end with a statement about limitations of the given realization, which
can also help in the selection process.

The documentation is not only relevant for people new to the model, but can also
be used as examination tool to better understand certain model results or for
tracking down bugs in the code. Here, in particular the interface information as
well as the module contract can play a role: The interfaces help to better understand
how certain dynamics in the model can occur and how they can be explained, whereas
the module contract helps to determine whether a certain module behaves in line
with this contract or violates it which helps to determine which part of the model
requires some fixing if the model shows faulty behavior.





## Updating documentation


[goxygen]:https://github.com/pik-piam/goxygen
