Running history matching with `optuna-externm` requires that you have an interface to the flow simulator and are able to modify it's input and output data.

The inputs to a flow simulator are usually termed a deck, which comprises a number of ascii text files. The primary file will have the extension `.DATA` and this contains the top level instructions to the flow simulator. Other ascii files can have any extension but are usually included in the `.DATA` file with an `INCLUDE` statement. Typical additional files are the `.GRDECL` grid property files or `.INC` include files which might have additional simulation directives or property tables in them.

Understanding how to modify and build flow simulation decks is beyond the scope of this documentation, but you should consult your flow simulators manual for more detail and help on setting up a flow simulation model.

The outputs of a flow simulation model are usually the binary or text files which contain information relating to well performance (summary curves), and grid properties at predefined report dates. Again, the types of summary curves and grid properties how put and how often is related to the setup of your simulation model and not covered here.

`optuna-hm` whilst not dealing with the setup of a model, does provide useful interfaces to both modify, run and recover data from a flow simulation model. This is the fundamental three step process an optimisation algorithm needs to achieve.

 1. **Modify**: Modification of the model relates to perturbation of the model parameters by the optimisation algorithm. These are the parameters we want to change to improve the fit of our model to the history. Parameters must be changed by the overarching optimisation process before the simulator is run for each trial.
 2. **Run**: In the run phase, the modified simulation model must be run using the external flow simulation software. This requires `optuna-externm` to invoke the simulator on the modified simulation deck. The external process must then be monitored until completion.
 3. **Recover**: When the simulation is complete, it creates a lot of data. If we are going to run many 100s or 1000s of these models, it is not practical to save everything we create. Therefore we need to choose what we recover from the model outputs before throwing away the rest. Don't worry, because we know the parameters we can always rerun the model again at a later date if we need to. The recovered information is critical to calculating the misfit with the history, and may include well summary curves, grid properties at specific report dates or simulation deck files.

## Modify

The modify process is handled in `optuna-hm` by the `EclDatHandler` class.
This class initiates with a filename, where the file is a single text file from within
a deck such as the ".DATA" file.

```python
datfile = EclDatHandler("SIM.DATA")
```

The `update` method is then used to modify the provided file in-place. Where update takes a pandas `DataFrame` where the index is a flag and the 


The update method knows where to make changes by looking in files for the flags after any `/`. In flow simulation decks the `/` usually indicates the end of a record. Anything after this is treated like a comment so we are safe to put our directives here.

In this example, we have two flags for the `update` method, `PERMXL1` and `PERMXL2`.
The number after each flag tells `update` the index of which value we are to change (starting at 0). So in this case `PERMXL1 0` will change the value `-999` to the new value we pass to `update`. While `PERMXL2 2`, will change the value of the `11` (the index `2`).

```
EQUALS
PERMX
  -999 1 11 1 12 1  1 / PERMXL1 0 PERMXL2 2
```

There can be as many flags as you want, but each represents a parameter which will be optimised for in your model.

`optuna-hm` is geared towards meta-heuristic optimisation, and therefore the number of parameters you can easily change should not be very large. Grid properties are best modified through multipliers, or on a zone by zone basis. Smoothing of grid properties is better handled by optimisation algorithms which use filters, such as EnKF.

## Run

Running of the modified deck is handled by the `EclBaseHandler` class. This class is the primary workhorse in the management of a trial deck. It can copy the deck to a new location (temporary by default). Modify all the files in the deck that need to be modifed using a set of `EclDatHandler`. Run the model in an external process using the desired simulator and then copy back files that should be saved to a specified location for each trial run.

By default there are versions of `EclBaseHandler` for Eclipse 100 `Ecl100Handler` and OPM Flow `FlowHandler`. Both of these simulator specific classes make modifications to `EclBaseHandler` for their specific simulator. If you wish to use a different simulator then you will need to inherit `EclBaseHandler` to a new simulator specific class.

The `EclBaseHandler` is initialised with information about the model that will be copied, modified and run. Decks are usually held within a folder containing a single `.DATA` file and support data. For example

```
- SIM/
 ├─ SIM.DATA
 ├─ SIM_GRID.GRDECL
 ├─ SIM_SCHED.INC
```

Then, for an Eclipse simulator

```python
from optuna_hm.handlers.ecl import Ecl100Handler

sim = Ecl100Handler("SIM", "path_to_dir/SIM", popt)
```

## Recover