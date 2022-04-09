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

## Useful Docs

https://docs.google.com/document/d/1yUx6jr9EdedCOLd--CPdTfGDwEwzPpCF6p1jRmqx-0Q/edit#

https://agupubs.onlinelibrary.wiley.com/doi/epdf/10.3894/JAMES.2009.1.4

https://www.carbonbrief.org/cmip6-the-next-generation-of-climate-models-explained

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
# get all the data from google's datastore
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
# only want table id "piControl" or "historical"
df_expt = df_in[(df_in.experiment_id == "piControl") | (df_in.experiment_id == "historical")]
```

```{code-cell} ipython3
# we only want 3hr data or less
df_3hr = df_expt[(df_expt.table_id == "3hr") | (df_expt.table_id == "CF3hr")]
df_3hr
```

```{code-cell} ipython3
models = df_3hr.groupby("source_id")
```

```{code-cell} ipython3
# variables required to create figure 10
fig10_vars = ['tas', 'mrsos', 'huss']

# which models are able to produce figure 10?
fig10_models = []
```

```{code-cell} ipython3
for model in models.groups.keys():
    the_model = models.get_group(model) #df_3hr[df_3hr.source_id == model]
    model_vars = list(the_model.variable_id)
    if all(i in model_vars for i in fig10_vars) == True:
        fig10_models.append(model)
```

```{code-cell} ipython3
#for model in models.groups.keys():
    #print(model)
fig10_models
```

```{code-cell} ipython3
grouped = correct_table.groupby("source_id")
```

```{code-cell} ipython3
my_model = grouped.get_group("GFDL-ESM4")
my_model
```

```{code-cell} ipython3
a = list(my_model.variable_id)
print(all(i in a for i in ["vas", "huss"]))
```
