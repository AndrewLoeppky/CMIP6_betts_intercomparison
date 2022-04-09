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

# Make Fig 10

```{code-cell} ipython3
# run the model validation notebook, which creates a variable `fig10_models`,
# which is a list of models with the necessary fields
experiment_id = 'historical'
%run find_models.ipynb
```

```{code-cell} ipython3
# Domain we wish to study

# test domain 
##################################################################
lats = (15, 20) # lat min, lat max
lons = (25, 29) # lon min, lon max
years = (2013, 2015) # start year, end year (note, no leap days)
##################################################################

# Thompson, MB
##################################################################
#lats = (54, 56) # lat min, lat max
#lons = (261, 263) # lon min, lon max
#years = (100, 300) # start year, end year (note, no leap days)
##################################################################

save_data = False # save as netcdf for further processing?
```

```{code-cell} ipython3
# remove problem models
try:
    fig10_models.remove('MIROC-ES2L')
except:
    pass
try:
    fig10_models.remove('MIROC6')
except:
    pass

fig10_models = ['ACCESS-CM2', 'ACCESS-ESM1-5', 'AWI-ESM-1-1-LR']
```

```{code-cell} ipython3
# get all the 3hr fields
for source in fig10_models:
    source_id = source
    table_id = '3hr'
    %run CMIP6_lib.ipynb
    required_fields = ['tas', 'huss']
    print(f"""Fetching domain:
          {source_id = }
          {experiment_id = }
          {table_id = }
          {lats = }
          {lons = }
          {years = }
          dataset name: my_ds (xarray Dataset)""")
    
    # grab all fields of interest and combine
    my_fields = [get_field(field, df_in) for field in required_fields]
    small_fields = [trim_field(field, lats, lons, years) for field in my_fields]
    ds_3h = xr.combine_by_coords(small_fields, compat="override", combine_attrs="drop_conflicts")
    print("successfully acquired domain\n")
```

```{code-cell} ipython3
ds_3h
```

```{code-cell} ipython3
# get all the daily fields
for source in fig10_models:
    source_id = source
    table_id = 'day'
    %run CMIP6_lib.ipynb
    required_fields = ['mrsos']
    print(f"""Fetching domain:
          {source_id = }
          {experiment_id = }
          {table_id = }
          {lats = }
          {lons = }
          {years = }
          dataset name: my_ds (xarray Dataset)""")
    
    # grab all fields of interest and combine
    my_fields = [get_field(field, df_in) for field in required_fields]
    small_fields = [trim_field(field, lats, lons, years) for field in my_fields]
    ds_day = xr.combine_by_coords(small_fields, compat="override", combine_attrs="drop_conflicts")
    print("successfully acquired domain\n")
```

```{code-cell} ipython3
ds_day
```

```{code-cell} ipython3
day_interp = ds_day.interp(time=ds_3h["time"])
day_interp
```

```{code-cell} ipython3
ds_3h.merge(day_interp)
```
