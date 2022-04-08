Optuna History Matching (`optuna-hm`) is a library of utilities and functions
for performing reservoir flow simulation history matching with the 
`optuna-externm` library. The library currently works with Eclipse 100 and OPM Flow
decks.

**Key Features**

 - Handler to modify single values in simulation deck files.
 - Handlers for evaluating the misfit of simulation runs.
 - Handlers for optionally saving the outputs of simulation runs.
 - Different metrics for measuring the misfit of summary curves and seismic attribute maps.

The key features remove much of the boiler plate needed to setup a History Matching model with Eclipse and `optuna-externm`.

## Quick Start

See the tutorial example in the user guide.

## Installation

### Installing from source

`optuna-hm` requires `optuna-externm`. Please install `opunta-externm` first following the installation [instructions](../index.md#installation).

Clone the repository

```
git clone http://github.com/trhallam/optuna-hm
```

and install using `pip`

```
cd optuna-hm
pip install .
```