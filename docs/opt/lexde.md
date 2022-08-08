# Solving xxx (formal name of Alexandre's model) with Lex-DE in optuna-externm

## Install Requirements

Make sure you have Python>=3.8 and install the following libraries.

- For simulation model
  - pyemu
  - flopy
  - geopandas
  - openpyxl
- For optimizers
  - optuna
  - optuna_externm

```bash
pip install pyemu flopy geopandas openpyxl optuna
pip install git+https://github.com/trhallam/optuna-externm.git
```

## Simulation Code

Download `sim.tgz` and `binaries.tgz`. The first archive contains the Python code to run the simulation and it requires binary files in the second archive.

```bash
tar -zxvf sim.tgz
tar -zxvf binaries.tgz
```

Now, you have two folders: `sim` and `binaries`.

Copy the binary files for your OS from `binaries` to the `sim` folder. Enter `sim` and change the permission of the binary files (`mf6` and `mp7`).

```bash
cp binaries/linux/* sim/
cd sim
chmod +x mf6 mp7
```

In the same directory, you can find `example_run.py`.

```python
"""example_run.py"""
import helpers
out = helpers.run()
print(out)
```

```mermaid
flowchart LR
param("par.dat")
run["helpers.run()"]
out("(Q,MR)")
param --> run --> out
```

This piece of code allows you to evaluate the fitness values of the solution in `par.dat` in the same folder.

```dat
# par.dat
pname:h_inst:0_ptype:gr_usecol:2_pstyle:d_idx0:bar 8.03316610313876
pname:h_inst:0_ptype:gr_usecol:2_pstyle:d_idx0:gal 8.360218312960171
pname:q_inst:0_ptype:gr_usecol:2_pstyle:d_idx0:r21 -0.11113363253671613
pname:q_inst:0_ptype:gr_usecol:2_pstyle:d_idx0:r20 -0.12761231301918585
```

## Use Optimizers from `optuna_externm`

### Decision Variables

We want to find the parameters that mimimize Q and MR.

`par_bounds.xlsx` provides the name and range of the decision variables of this problem.

```python
import pandas as pd
bd = pd.read_excel("par_bounds.xlsx")
name = bd["parname"]
xl = bd["min"]
xu = bd["max"]
n_var = len(xl)
# xl = [8.0 8.0 -0.139   -0.139]
# xu = [9.5 9.5 -0.0139 -0.0139]
```

### Objective Functions

To evaluate the objective values, you just need to write parameters into `par.dat` and run `helpers.run()`. This operation returns a dataframe contains two floats (Q and MR).

The simulation can get an error if an unfeasible solution is inputted. In this case, a large value of penalty (1e09) is returned as objective values for both Q and MR.

### Optimization Problem

We can write the Python code of the optimization problem follow [the examples provided by optuna]().

```python
import numpy as np
import helpers
def objective(trial):
    x = np.full(n_var, np.nan)
    for i in range(n_var):
        x[i] = trial.suggest_float( # i-th variable
            name[i].split(":")[-1], # variable name
            xl[i],                  # lower bound
            xu[i]                   # upper bound
        )
    # write to par.dat
    pd.Series(x, index=name).to_csv("par.dat", header=None, sep=" ")
    # return the objective functions as a tuple
    return tuple(helpers.run().values.flatten())
```

### Optimization Algorithms

As an example, we use `LEXDESampler` in the `optuna_externm`. Lex-DE is an evolutionary algorithm based on differential evolution and lexicase selection.

```python
import optuna
from optuna_externm.opt._lexde import LEXDESampler
study = optuna.create_study(
    sampler=LEXDESampler(),         # optimizer
    directions=[
        "minimize",                 # direction of the first objective
        "minimize"                  # direction of the second objective
    ],
    storage="sqlite:///demo.db"     # database to store trials
)
study.optimize(
    objective,                      # objective function
    n_trials=200,                   # maximum number of trials
    n_jobs=10                       # parallelism
)
```

The above code are provided in `demo.py`. You can run `demo.py` to do optimization.

```bash
python demo.py
```

After running the script, there will be a database file `demo.db` in the same folder. It contains the information of all the trials that sampled during the optimization.

Please open `result_explorer.ipynb` to see how to check the results.
