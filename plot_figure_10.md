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

# Plot Figure 10

```{code-cell} ipython3
import xarray as xr
import pandas as pd
import numpy as np
import cftime
import matplotlib.pyplot as plt
from matplotlib.ticker import (MultipleLocator, AutoMinorLocator)
import netCDF4 as nc
import os

# Handy metpy tutorial working with xarray:
# https://unidata.github.io/MetPy/latest/tutorials/xarray_tutorial.html#sphx-glr-tutorials-xarray-tutorial-py
import metpy.calc as mpcalc
from metpy.cbook import get_test_data
from metpy.units import units
from metpy.plots import SkewT
```

```{code-cell} ipython3
files = os.listdir("data/")
files
```

```{code-cell} ipython3
ps = 100000 * units.Pa # temporary hack, should interpolate pressure from daily timeseries
```

```{code-cell} ipython3
for data in files:
    data='EC-Earth3-historical.nc'
    # open the data and re-convert time to cftime so xarray is happy
    data_in = xr.open_dataset(f"data/{data}", engine="netcdf4", decode_times=False).metpy.quantify()
    data_in["time"] = cftime.num2date(data_in.time, "minutes since 0000-01-01 00:00:00", calendar="noleap", has_year_zero=True)
    print("loaded data")
    
    # use metpy to convert humidity field to dew point temp
    data_in["td"] = mpcalc.dewpoint_from_specific_humidity(ps, data_in.tas, data_in.huss)

    # compute the spatial average
    spatial_average = data_in.mean(dim=("lat", "lon"))
    print("calculated spatial average")

    # separate by soil moisture by rounding to nearest kg/m3 in top soil layer
    the_max = max(spatial_average.mrsos[np.isnan(spatial_average.mrsos.values) == False])
    the_min = min(spatial_average.mrsos[np.isnan(spatial_average.mrsos.values) == False])
    the_range =  the_max - the_min
    
    spatial_average["soil_moisture_grp"] = ((spatial_average.mrsos / (the_range / 6)).round() * (the_range / 6))[np.isnan(spatial_average.mrsos.values) == False]
    
    
    gbysoil = spatial_average.groupby(spatial_average.soil_moisture_grp)
    print('generated groupings')
    # calculate and plot the average diurnal cycle of lcl height
    fig, ax = plt.subplots()
    for key in gbysoil.groups.keys():
        # group by hour
        hourly_data = gbysoil[key].groupby(gbysoil[key].time.dt.hour).mean(dim="time")  

        # find and plot the lcl
        plcl, tlcl = mpcalc.lcl(ps, hourly_data.tas, hourly_data.td)
        plcl_hpa = plcl / 100
        # HERE should add an hour 24 to each line by copying hour 0

        if key == min(gbysoil.groups.keys()):
            plot_kwargs = {"color":"darkblue"}
        elif key == max(gbysoil.groups.keys()):
            plot_kwargs = {"color":"red"}
        else:
            plot_kwargs = {"color":"black", "linestyle":"--", "linewidth":0.8}

        ax.plot(hourly_data.hour, plcl_hpa[np.isnan(plcl) == False], **plot_kwargs)
        the_label = ax.annotate(f"{round(key)} kg/m$^3$", (24, plcl_hpa[-1]))

    # make the plot match Betts fig 11
    plt.gca().invert_yaxis()
    ax.set_xlabel("UTC")
    ax.set_ylabel("P$_{LCL}$ (hPa)")
    #ax.axvline(21, color="k", linewidth=1)
    ax.xaxis.set_major_locator(MultipleLocator(6))
    ax.xaxis.set_major_formatter('{x:.0f}')
    ax.xaxis.set_minor_locator(MultipleLocator(1))
    ax.set_xticks((0,6,12,18,24))
    ax.set_xlim(0,28)
    ax.set_title(data[:-3]);
```

```{code-cell} ipython3
data='ACCESS-CM2-historical_mean.nc'
# open the data and re-convert time to cftime so xarray is happy
data_in = xr.open_dataset(f"data/{data}", engine="netcdf4", decode_times=False)#.metpy.quantify()
data_in = data_in.drop("height")
data_in["tas"] = data_in.tas * units.kelvin
data_in["time"] = cftime.num2date(data_in.time, "minutes since 0000-01-01 00:00:00", calendar="noleap", has_year_zero=True)
data_in["td"] = mpcalc.dewpoint_from_specific_humidity(ps, data_in.tas, data_in.huss)
data_in["soil_moisture_grp"] = (data_in.mrsos / 2).round() * 2
data_in
```

```{code-cell} ipython3

# compute the spatial average
#spatial_average = data_in.mean(dim=("lat", "lon"))
#print("calculated spatial average")
spatial_average = data_in

# separate by soil moisture by rounding to nearest kg/m3 in top soil layer
the_max = max(spatial_average.mrsos[np.isnan(spatial_average.mrsos.values) == False])
the_min = min(spatial_average.mrsos[np.isnan(spatial_average.mrsos.values) == False])
the_range =  the_max - the_min

spatial_average["soil_moisture_grp"] = ((spatial_average.mrsos / (the_range / 6)).round() * (the_range / 6))[np.isnan(spatial_average.mrsos.values) == False]


gbysoil = spatial_average.groupby(spatial_average.soil_moisture_grp)
print('generated groupings')
# calculate and plot the average diurnal cycle of lcl height
fig, ax = plt.subplots()
for key in gbysoil.groups.keys():
    # group by hour
    hourly_data = gbysoil[key].groupby(gbysoil[key].time.dt.hour).mean(dim="time")  

    # find and plot the lcl
    plcl, tlcl = mpcalc.lcl(ps, hourly_data.tas, hourly_data.td)
    plcl_hpa = plcl / 100
    # HERE should add an hour 24 to each line by copying hour 0

    if key == min(gbysoil.groups.keys()):
        plot_kwargs = {"color":"darkblue"}
    elif key == max(gbysoil.groups.keys()):
        plot_kwargs = {"color":"red"}
    else:
        plot_kwargs = {"color":"black", "linestyle":"--", "linewidth":0.8}

    ax.plot(hourly_data.hour, plcl_hpa[np.isnan(plcl) == False], **plot_kwargs)
    the_label = ax.annotate(f"{round(key)} kg/m$^3$", (24, plcl_hpa[-1]))

# make the plot match Betts fig 11
plt.gca().invert_yaxis()
ax.set_xlabel("UTC")
ax.set_ylabel("P$_{LCL}$ (hPa)")
#ax.axvline(21, color="k", linewidth=1)
ax.xaxis.set_major_locator(MultipleLocator(6))
ax.xaxis.set_major_formatter('{x:.0f}')
ax.xaxis.set_minor_locator(MultipleLocator(1))
ax.set_xticks((0,6,12,18,24))
ax.set_xlim(0,28)
ax.set_title(data[:-3]);
```