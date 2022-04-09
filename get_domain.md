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

# Get Domain

**Author:** Andrew Loeppky (Lots of code stolen from Jamie Byer)

**Project:** Land-surface-atmosphere coupling - CMIP6 intercomparison 

This code grabs a climate model from the cloud, screens it for required variable fields, then selects a user specified domain and saves it to disk as a netcdf4 file.

+++

## Part I: Get a CMIP 6 Dataset and Select Domain

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
# Attributes of the model we want to analyze (put in csv later)
#source_id = 'GFDL-ESM4' # working fig 11
source_id = 'ACCESS-CM2'

#experiment_id = 'piControl'
experiment_id = 'historical'
table_id = '3hr'

# Domain we wish to study

# test domain #
##################################################################
lats = (15, 20) # lat min, lat max
lons = (25, 29) # lon min, lon max
years = (1961, 1972) # start year, end year (note, no leap days)
##################################################################

# Thompson, MB
#lats = (54, 56) # lat min, lat max
#lons = (261, 263) # lon min, lon max
#years = (100, 300) # start year, end year (note, no leap days)

save_data = False # save as netcdf for further processing?
```

```{code-cell} ipython3
%run CMIP6_lib.ipynb
```

```{code-cell} ipython3
required_fields = ['tas', 'huss']#, 'ps'] , 'mrsos', 
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
# this is how to get fields!
#df_in[df_in.source_id == source_id][df_in.table_id == table_id][df_in.experiment_id == experiment_id]
```

```{code-cell} ipython3
# check that our run has all required fields, list problem variables
#fields_of_interest = []
#missing_fields = []
#for rq in required_fields:
#    if rq not in available_fields:
#        missing_fields.append(rq)
#    else:
#        fields_of_interest.append(rq)


#print(f"Model: {source_id}\n"+"="*30)
#print("Contains required fields:")
#[print("   ", field) for field in required_fields if field in fields_of_interest]

#if fields_of_interest == required_fields:
#    model_passes = True
#    print("\nAll required fields present\n")
#else: 
#    model_passes = False
#    print("Missing required fields:")
#    [print("   ", field) for field in required_fields if field not in fields_of_interest]
        
```

```{code-cell} ipython3
print(f"""Fetching domain:
          {source_id = }
          {experiment_id = }
          {table_id = }
          {lats = }
          {lons = }
          {years = }
          dataset name: my_ds (xarray Dataset)""")
```

```{code-cell} ipython3
# grab all fields of interest and combine
my_fields = [get_field(field, df_in) for field in required_fields]
small_fields = [trim_field(field, lats, lons, years) for field in my_fields]
my_ds = xr.combine_by_coords(small_fields, compat="broadcast_equals", combine_attrs="drop_conflicts")
print("successfully acquired domain")
success = True
```

```{code-cell} ipython3
my_ds
```

```{code-cell} ipython3

```

```{code-cell} ipython3
# save as netcdf as per these recommendations:
# https://xarray.pydata.org/en/stable/user-guide/dask.html#chunking-and-performance
# netcdf cant handle cftime, so convert to ordinal, then back once the file is reopened
my_ds["time"] = date2num(my_ds.time, "minutes since 0000-01-01 00:00:00", calendar="noleap", has_year_zero=True)

# get rid of time bounds variable, if it exists
try:
    my_ds = my_ds.drop("time_bnds")
except:
    pass
```

```{code-cell} ipython3
if save_data:
    print(f"saving {source_id} to disk as netcdf")
    my_ds.to_netcdf(f"./data/{source_id}-{experiment_id}.nc", engine="netcdf4")
    print("success\n\n")
else:
    print(f"successfully parsed {source_id}\n\n")
```

re-write this from scratch. 

1) select df_in where table_id == 3hr, and all required fields exist
2) slice and save each dataset as netcdf
