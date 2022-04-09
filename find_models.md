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

# Find Models

This notebook searches the pangeo CMIP6 google cloud data and generates a list of fields from which we can generate figures 10 and 11 from Betts' paper. 

**Fig 10**

Looking for 3hrly or better surface temp and humidity (`tas, huss`) and daily or better soil moisture in the top layer (`mrsos`)

**Fig 11**

Looking for sensible and latent heat fluxes (`?, ?`) and precipitation rate (`?`)

**Fig 16**

Looking for precip, evap total column water, and omega

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
df_expt = df_in[(df_in.experiment_id == "historical")]# | (df_in.experiment_id == "historical")]
```

## Figure 10 Models

```{code-cell} ipython3
# we only want 3hr data or less for temp and humidity, daily or better for soil moisture
df_3hr = df_expt[(df_expt.table_id == "3hr") | (df_expt.table_id == "CF3hr")]
df_day = df_expt[(df_expt.table_id == "day")]
```

```{code-cell} ipython3
models_3h = df_3hr.groupby("source_id")
models_day = df_day.groupby("source_id")
```

```{code-cell} ipython3
# variables required to create figure 10
fig10_vars = ['tas', 'huss']

# most models have air temperature and humidity, need to get soil moisture from daily data
fig10_soil_vars = ['mrsos'] 

# which models are able to produce figure 10?
fig10_models_3hr = []
fig10_models_day = []

for model in models_3h.groups.keys():
    the_model = models_3h.get_group(model)
    model_vars = list(the_model.variable_id)
    if all(i in model_vars for i in fig10_vars) == True:
        fig10_models_3hr.append(model)
        
for model in models_day.groups.keys():
    the_model = models_day.get_group(model)
    model_vars = list(the_model.variable_id)
    if all(i in model_vars for i in fig10_soil_vars) == True:
        fig10_models_day.append(model)
```

```{code-cell} ipython3
# models that contain 3hrly temp and humidity, and daily soil moisture
fig10_models = []
print("models to use for figure 10")
print("=" *40)
for model in fig10_models_day:
    if model in fig10_models_3hr:
        fig10_models.append(model)
        print(model)


print("\n\nmodels to reject")
print("=" *40)
for model in fig10_models_day:
    if model not in fig10_models_3hr:
        print(model)
```

## Figure 11 Models

```{code-cell} ipython3
fig11_vars = ['tas', 'huss']
fig11_day_vars = ["pr", 'hfls', 'hfss']


# which models are able to produce figure 11?
fig11_models_3hr = []
fig11_models_day = []

for model in models_3h.groups.keys():
    the_model = models_3h.get_group(model)
    model_vars = list(the_model.variable_id)
    if all(i in model_vars for i in fig11_vars) == True:
        fig11_models_3hr.append(model)
        
for model in models_day.groups.keys():
    the_model = models_day.get_group(model)
    model_vars = list(the_model.variable_id)
    if all(i in model_vars for i in fig11_day_vars) == True:
        fig11_models_day.append(model)
```

```{code-cell} ipython3
# models that contain 3hrly temp and humidity, and daily soil moisture
fig11_models = []
print("models to use for figure 11")
print("=" *40)
for model in fig11_models_day:
    if model in fig11_models_3hr:
        fig11_models.append(model)
        print(model)


print("\n\nmodels to reject")
print("=" *40)
for model in fig11_models_day:
    if model not in fig11_models_3hr:
        print(model)
```
