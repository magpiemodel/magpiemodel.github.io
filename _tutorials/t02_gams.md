---
layout: tutorial
title:  GAMS code, modules & realizations
image: assets/images/generic/pic02.jpg
shortID: gams
lastUpdated:   2025-06-14
model: MAgPIE
modelVersion: 4.10.1
author:
  - kk
  - fb
  - am
  - emb
  - iw
level: 2
requirements:
  - GAMS Installed
  - MAgPIE clone in local machine
lessonsContent:
  - Navigate through the GAMS code of MAgPIE
  - Understand the structure of modules and realizations
  - Understand naming rules and conventions
published: true
---

## Preface: General structure of the MAgPIE Model

MAgPIE (Model for Agricultural Production and its Impacts on the
Environment) is a modular open-source framework for modeling global
land systems. Before looking into the code structure, we first introduce 
the concept of ‘modules‘, the building blocks of MAgPIE.

The MAgPIE model structure is built upon the idea that every module
is autonomous and interacts through a **clearly defined interface** 
with other modules. This reflects the idea that every module represents
a separate part of the model that could be represented in a simple or more
sophisticated manner without relying on other modules. A complete description 
of module interdependencies is available in the 
[model documentation](https://rse.pik-potsdam.de/doc/magpie/4.10.1/).

Below, we briefly highlight the ‘main line’ of module interdependence:

<img src="../assets/images/tutorials/magpie_model.gif" alt="centered image" width="450"/>

- Population and GDP projections (based on scenario assumptions)
  represent the main drivers of the model.
- These drivers shape food consumption patterns, leading to demand for
  primary agricultural products.
- Demand patterns are translated into production patterns via trade flows.
- Production, combined with biophysical yield data, defines cropping patterns
  and informs land-use decisions.
  
Importantly, these are not one-way dependencies: e.g., constraints in 
land availability can influence food consumption patterns through rising 
food prices across different commodities. Associated costs of
different activities are fed into the goal function of the cost
module, which guides optimization and is thereby central for the model.

Alongside the ‘main line’ of agricultural production modules, the model
includes:
    - Impact assessments and policy interactions (upper part)
    - Detailed components of the production chain (lower part).

## The MAgPIE GAMS code: Model components

The core of the MAgPIE model is written in GAMS. For the model execution,
all parts of the code are automatically put into a single file, the `full.gms`. 

The `modules` folder stores and organizes the main code components, following 
the thematic structure defined by the modular architecture of MAgPIE.
Each subfolder within `modules` represents a specific model component and can contain
multiple realizations — different implementations of that component. Only one 
realization per module is included in the final model execution by being 
inserted into the `full.gms`. The choice of realizations is determined by the configuration 
settings, defined in `default.cfg` or specified in the run start scripts.   

###  Structure of a module

When you open the `modules` folder, you see a long list of the individual
modules and the `include.gms` file that ensures the inclusion of all modules into the
`full.gms`. All modules are built similarly and follow the same structure:

![structure of any module](../assets/images/tutorials/module_struc.png)

  - An `input` folder with shared input files used by all realizations
  - `Realization` folders containing the source code of each realization
  - The `modules.gms` file with the module description and listing of all
    realizations
  - Several `[realization_name].gms` files with the realization description
    and include statements to the specific source code

This structure makes it easy to extend the model: new realizations can be added 
by following the same template (more in [this tutorial](t10_changeCode.md)).

### Structure within each realization

Within each realization, the source code is distributed over several
gms-files to ensure the correct order of calculations during the optimization.
Moreover, the interfaces (objects used by more than one module or by a module
and the core code) for the module inputs and outputs are defined by this
structure. The following table provides an overview of the purpose of each gms-file.
Note that not all gms-files are required in every realization.

| gms.file         | function                                                                                                                             |
| :--------------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| declarations.gms | Declares all variables, equations, and parameters for the realization.                                                               |
| equations.gms    | Contains functional relationships that have to be fulfilled within the optimization.                                                 |
| input.gms        | Loads input from `any_module/input` or `any_module/a_realization/input`.                                                             |
| sets.gms         | Lists sets that are used (mainly) within this realization or are needed for interfaces defined within this realization.              |
| preloop.gms      | Includes calculations to be executed before the time-step loop starts.                                                               |
| presolve.gms     | Includes calculations to be executed for each time step before the optimization process.                                             |
| postsolve.gms    | Includes calculations to be executed for each time step after the optimization process and defines output.                           |
| nl\_fix.gms      | Fixes non-linear behaviour to linear behavior.                                                                                       |
| nl\_release.gms  | Releases restrictions imposed during non-linear fixes.                                                                               |
| scaling.gms      | Lists the expected order of magnitude of specific variables calculated in this realization to improve the efficiency of the run.     |   
| not\_used.txt    | Lists interfaces (declared in other modules) that are not used within this realization, but in other realizations of the same module. |

## Coding etiquette: Variable and parameter naming

Each MAgPIE module is designed to be self-contained with well-defined interfaces.
In this sense, realizations are replaceable within a module, since all realizations 
of a module have to deliver/interact with the same interfaces.
This is ensured by defined rules for variable and parameter naming.


The following prefixes are used within the model code:

    q_ eQuations
    v_ Variables
    s_ Scalars
    f_ File parameters - these parameters contain data read from input file
    i_ Input parameters - influence the optimization, but are not influenced by it
    p_ Processing parameters - influence optimization and are being influenced by it
    o_ Output parameters - influenced by optimization, but without effect on the optimization
    x_ eXtremely important output parameters - output parameters, that are necessary for the model to run properly (required by external postprocessing). They must not be removed.
    c_ Switches from the config.gms - parameters, that are switches to choose different scenarios
    m_ Macros

The prefixes have to be extended in some cases by a second letter:

    ?m_ module-relevant object - This object is used by at least one module and the core code or another module. (Changes related to this object have to be performed carefully).
    ?00_ (a 2-digit number) module-only object - This 2-digit number defines the module the object belongs to. This object is only used by the respective module. It makes sure that different modules do not contain the same object.

Note that sets are treated slightly differently: Instead of adding a
prefix, sets have a 2-digit number suffix (defining the number of the
module in which the set is exclusively used). If the set is used in more
than one module, it does not have any suffix.

In other cases, the prefixes are extended with a second letter to indicate details such as:

    ?c_ value for the Current timestep - necessary for constraints. Each *c_-object must have a time-dependent counterpart
    ?q_ parameter containing the values of an equation
    ?v_ parameter containing the values of a variable

Suffixes indicate the level of aggregation of an object:

    (no suffix) highest disaggregation available
    _(setname) aggregation over set
    _reg regional aggregation (exception)
    _glo global aggregation (exception)

## Workshop Material
You will find the slides used in the workshop [here](../assets/pdf/GAMScode.pdf).
