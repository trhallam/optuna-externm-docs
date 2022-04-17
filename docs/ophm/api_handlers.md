# Handlers

```python
from optuna_hm import handlers
```

## `optuna_hm.handlers.simh`

Generic handlers for interfacing with and evaluating Eclipse (Schlumberger) style reservoir simulation models and software. The software must have an command line interface.

::: optuna_hm.handlers.simh
    rendering:
      show_signature: true
      separate_signature: true
      heading_level: 3
      show_root_toc_entry: false

## `optuna_hm.handlers.ecl`

Handlers for interfacing with and evaluating Eclipse (Schlumberger) Reservoir Simulation models. Eclipse is a proprietary simulation software which has a command
line interface.


::: optuna_hm.handlers._ecl_eval
    rendering:
      show_signature: true
      separate_signature: true
      heading_level: 3
      show_root_toc_entry: false

## `optuna_hm.handlers.flow`

Flow is an alternative open-source simulator to Eclipse. It uses similar files as input
and output but has a slightly different call signature. This module modifies the Eclipse methods to suit flow where needed. Evaluation and metrics functions from Eclipse can be used with Flow.

::: optuna_hm.handlers.flow
    rendering:
      show_signature: true
      separate_signature: true
      heading_level: 3
      show_root_toc_entry: false
