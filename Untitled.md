---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.13.7
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

```{code-cell} ipython3
import xarray as xr
import pooch
import pandas as pd
import fsspec
from pathlib import Path
import time
import numpy as np
import json
import cftime
import matplotlib.pyplot as plt
import netCDF4 as nc
from cftime import date2num


# Handy metpy tutorial working with xarray:
# https://unidata.github.io/MetPy/latest/tutorials/xarray_tutorial.html#sphx-glr-tutorials-xarray-tutorial-py
import metpy.calc as mpcalc
from metpy.cbook import get_test_data
from metpy.units import units
from metpy.plots import SkewT
```

```{code-cell} ipython3
# get esm datastore
odie = pooch.create(
    path="./.cache",
    base_url="https://storage.googleapis.com/cmip6/",
    registry={
        "pangeo-cmip6.csv": None
    },
)
file_path = odie.fetch("pangeo-cmip6.csv")
df_in = pd.read_csv(file_path)
```

```{code-cell} ipython3
df_in[df_in.table_id == "3hr"]
```

```{code-cell} ipython3
table_id = "3hr"
```

```{code-cell} ipython3
correct_table = df_in[df_in.table_id == table_id][df_in.experiment_id == "historical"]
```

```{code-cell} ipython3
correct_table
```

```{code-cell} ipython3
grouped = correct_table.groupby("source_id")
```

```{code-cell} ipython3
grouped.groups.keys()
```

```{code-cell} ipython3

```
