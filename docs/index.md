# Optuna-ExternM

Optuna External Model (Optuna-ExternM) is a framework which provides a common API for 
models to interface with the [`optuna`](https://optuna.org/) Python library. 
The common API means that many models can become plug and play with different optimisation
algorithms. The framework also provides a `yml` based configuration file system that can be
used to create and manage many optimisation runs of a model with slightly different configurations
by optioning parts of the model setup. We think this allows for easier testing of different algorithms
and more portable models.

In addition to the framework, we also include additional Optuna sampling algorithms and tools
for performing sampling based upon Design of Experiments or Sensitivity Analysis proceedures.

## Quick Start

See the quick start example in the user guide.

## Installation

### Installing from source

Clone the repository

```
git clone http://github.com/trhallam/optuna-externm
```

and install using `pip`

```
cd optuna-externm
pip install .
```
