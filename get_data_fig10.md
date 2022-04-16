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

# Get Figure 10 Data

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
#lats = (15, 20) # lat min, lat max
#lons = (25, 29) # lon min, lon max
#years = (2013, 2015) # start year, end year
##################################################################

# Thompson, MB
##################################################################
lats = (51, 57) # lat min, lat max
lons = (259, 265) # lon min, lon max
years = (1960, 2015) # start year, end year (note, no leap days)
years = (400, 500)
##################################################################

save_data = True # save as netcdf for further processing?
```

```{code-cell} ipython3
# remove problem models
#fig10_models.remove('MIROC-ES2L')
#fig10_models.remove('MIROC6')
#fig10_models.remove('SAM0-UNICON')
```

```{code-cell} ipython3
# parse each model in the list 
for source in fig10_models:
    try:
        source_id = source

        # get all the 3hr fields
        table_id = '3hr'
        %run CMIP6_lib.ipynb
        required_fields = ['tas', 'huss']
        print(f"""Fetching domain:
              {source_id = }
              {experiment_id = }
              {lats = }
              {lons = }
              {years = }""")
        print("acquiring 3hrly data")
        # grab all fields of interest and combine (3hr)
        my_fields = [get_field(field, df_in) for field in required_fields]
        small_fields = [trim_field(field, lats, lons, years) for field in my_fields]
        ds_3h = xr.combine_by_coords(small_fields, compat="override", combine_attrs="drop_conflicts")

        # filter extraneous dimensions
        for dim in ["height", "time_bounds", "depth", "depth_bounds"]:
            try:
                ds_3h = ds_3h.drop(dim)
            except:
                pass

        # take spatial average
        #ds_3h = ds_3h.mean(dim=("lat", "lon"))

        print("acquiring daily data")
        # get all the daily fields
        table_id = 'day'
        %run CMIP6_lib.ipynb
        required_fields = ['mrsos']

        # grab all fields of interest and combine (day)
        my_fields = [get_field(field, df_in) for field in required_fields]
        small_fields = [trim_field(field, lats, lons, years) for field in my_fields]
        ds_day = xr.combine_by_coords(small_fields, compat="override", combine_attrs="drop_conflicts")

        # filter extraneous dimensions
        for dim in ["height", "time_bounds", "depth", "depth_bounds", "lat_bnds", "lon_bnds"]:
            try:
                ds_day = ds_day.drop(dim)
            except:
                pass

        # take spatial average
        #ds_day = ds_day.mean(dim=("lat", "lon"))

        # interpolate daily data onto the 3hourly and merge.
        print("interpolating")
        day_interp = ds_day.interp_like(ds_3h).chunk({"time":-1})
        print("scrubbing NaN values")
        day_interp = day_interp.interpolate_na(dim="time")
        print("merging datasets")
        full_dataset = ds_3h.merge(day_interp).metpy.quantify().chunk({"time":10000})


        if save_data:
            print(f"saving {source_id} to disk as netcdf")
            full_dataset.to_netcdf(f"./data/{source_id}-{experiment_id}-fig10.nc", engine="netcdf4")

            print("success\n\n")
        else:
            print(f"successfully parsed {source_id}\n\n")
    except:
        print(f"{source} failed")
```
