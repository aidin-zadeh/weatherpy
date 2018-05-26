
## 1. Insights

The climatic variations as a function of geograpy are being investigated through 2-d scatter and climatic geomap plots. This involves analyzing the association beween four basic climatic factors including temperature, humidity, cloudiness and wind speed with respect to latitude and latitude-longitude.

### 1.1. Climate and Latitude
### 1.1.1. Temperature (F) vs. Latitude
Investigating temperature at different latitudes shows:
- There exists a negative association between temperature and distance from equator. Temperature smoothly falls off with latitude towards the two poles. The regions close to the equator experience higher temperatures compared to those close to the North and South poles.

### 1.1.2. Humidity (F) vs. Latitude
Investigating humidity at different latitudes shows:
- There exists a weak negative association between the temperature and the distance from equator. As one moves away from the equator, the humidity decreases .

### 1.1.3. Cloudiness (F) vs. Latitude
Investigating the cloudiness at different latitude shows:
Ther is no clear association with latitude.

### 1.1.4. Wind Speed vs (Total Drivers vs Total Rides)
Investigating the wind speed at different latitudes shows:
- There exist a weak positive association between wind speed and distance from equator.

### 1.2. Climate and Latitude-Longitude
TBA

## 2. Limitations

- The current analysis has heavily focused on temperature records sampled at cities across the world that can easily influenced by the non-unfiormity of city distributions. As the land masses are distributed predominantly in the Northern Hemisphere (68%) compared to the Southern Hemisphere (32%), it is quite likely that more cities are located in Northern hemisphere compared to the Sothern Hemisphere.
- The climatic variation has only been assessed with respect to latitude while ingnorng other plusible governing factors. It is well know that any climatic phenomenon depends on other factors such as altitude and local topgrapy and clearly, an accurate climatic analysis can be achieved as any potential factor being accounted for global level climatic variations is considered.   

## 3. Implementation

### 3.1. Import Dependecies


```python
# import dependecies
%matplotlib inline
import os, sys, inspect
import time
import random
import requests
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime
from mpl_toolkits.basemap import Basemap as bm
from itertools import product
from citipy import citipy as cp
from pprint import pprint

# add parent dir to system dir
current_dir = os.path.dirname(os.path.abspath(inspect.getfile(inspect.currentframe())))
root_dir = os.path.dirname(current_dir)
sys.path.insert(0, root_dir) 
```

### 3.2. Import API keys


```python
# OpenWeatherMap api key
from config import api_key
```

### 3.3. Set global varriables


```python
DEGREE = u"\u00b0"
```

### 3.4. Define functions


```python
def scatter(
    x, y,
    marker="o",
    markersize=6,
    markeredgecolor="black",
    markeredgewidth=1,
    markerfacecolor="black",
    fillstyle="full",
    linestyle="",
    alpha=.7,
    xlabel="",
    ylabel="",
    label="",
    title="",
    xlim=None,
    ylim=None,
    fontsize_title=14,
    fontsize_label=13,
    fontsize_xtick=12,
    fontsize_ytick=12,
    figsize=(7, 5),
    legend=True,
    grid=True,
    ax=None,):
    """Scatter plot given input 'x' and 'y'. Wrapper function to interface 'matplotlib.pyplot.plot'
    to perform scatter plot.
    """
    
    # create figure/axis handler
    if ax is None:
        fig = plt.figure(figsize=figsize)
        ax = fig.subplots(1,1)
    
    # scatter plot
    ax.plot(
        x, y,
        marker=marker,
        markersize=markersize,
        markeredgecolor=markeredgecolor,
        markeredgewidth=markeredgewidth,
        markerfacecolor=markerfacecolor,
        fillstyle=fillstyle,
        linestyle=linestyle,
        alpha=alpha,
        label=label,
    )
    # set title
    _ = ax.set_title(
        title,
        fontsize=fontsize_title,
        fontweight="bold"
    )
    # set axis labels
    ax.set_xlabel(xlabel)
    ax.set_ylabel(ylabel)
    ax.xaxis.label.set_size(fontsize_label)
    ax.yaxis.label.set_size(fontsize_label)
    
    # set axis ticks
    [tick.label.set_fontsize(fontsize_xtick) for tick in ax.xaxis.get_major_ticks()]
    [tick.label.set_fontsize(fontsize_ytick) for tick in ax.yaxis.get_major_ticks()]
    
    # set axis range limits            
    ax.set_xlim(xlim)
    ax.set_ylim(ylim)
    
    # set legend
    if legend and (label is not ""):
        ax.legend()
    
    # set grid
    if grid:
        ax.grid(
            True,
            color="grey",
            linestyle=":",
            linewidth=1.5,
            alpha=0.5
        )
    
    # set tight layout
    plt.tight_layout()
    return ax


def scatter_weather_vs_latitude():
    """Scatter plot weather trend vs. latitude. Wrapper fucntion to interface 'scatter' to perform
    weather data scatter plot, handeling different colors for north- and south-hemisphere.
    """
    # set figure size
    fig = plt.figure(figsize=(figsize))
    ax = fig.subplots(1, 1)
    
    # get x-axes and y-axes data 
    x = df["Latitude"].values
    y = df[metric].values
    
    # set axes params 
    xlabel = "Latitude"
    ylabel = f"{metric} ({unit})"
    time_str = time.strftime("'%Y-%m-%d %H:%M'", current_time)
    title = f"{metric} variations ({unit}) vs Latitude for {time_str}"
    
    # scatter plot for north-hemisphere
    ax = scatter(
        x=x[x>=0],
        y=y[x>=0],
        markerfacecolor="red",
        markeredgecolor="red",
        xlabel=xlabel,
        ylabel=ylabel,
        title=title,
        xlim=[-90, 90],
        ylim=ylim,
        label="Northern Hemisphere",
        ax=ax,
    )
    
    # scatter plot for south-hemisphere
    ax = scatter(
        x=x[x<0],
        y=y[x<0],
        markerfacecolor="navy",
        markeredgecolor="navy",
        xlabel=xlabel,
        ylabel=ylabel,
        title=title,
        xlim=[-90, 90],
        ylim=ylim,
        label="Southern Hemisphere",
        ax=ax,
    )
    return ax, fig


def geomap_scatter(lats, lons, vals=None, vlim=None, markersize=4,
    fontsize_title=12, fontsize_label=11, fontsize_xtick=9, fontsize_ytick=9,
    alpha=0.95, title="", label="", figsize=(10, 7),
    colormap="bwr"):
    """Geomap plot weather trends. Wrapper fucntion to interface 'mpl_toolkits.basemap'to perform
    geo-map for weather data.
    """
    
    
    # create figure/axis handler
    fig = plt.figure(figsize=figsize)
    ax = fig.subplots(1, 1)
    
    # create a base map object
    bmap = bm(
        projection='merc',
        llcrnrlat=-80,
        urcrnrlat=80,
        llcrnrlon=-180,
        urcrnrlon=180,
        lat_ts=50,
        resolution="l")
    
    # draw coast lines
    bmap.drawcoastlines()
    
    # draw countries
#     bmap.drawcountries()
    
    # draw/set up parllel lines
    parallels = np.arange(-90.,91.,30.)
    bmap.drawparallels(parallels)
    bmap.drawparallels(
        parallels,
        labels=[True,False,False,False],
        fontsize=fontsize_ytick)
    
    # draw/set up meridians
    meridians = np.arange(-180.,181.,30.)
    bmap.drawmeridians(meridians)
    bmap.drawmeridians(
        meridians,
        labels=[True,False,False,True],
        fontsize=fontsize_xtick,)
    
    # draw/set up map boundaries
    bmap.drawmapboundary(fill_color='white')
    
    # draw/set up continents
    bmap.fillcontinents(color="#cc9955", lake_color="steelblue", alpha=0.2)
    
    # set title
    _ = ax.set_title(
        title,
        fontsize=fontsize_title,
        fontweight="bold"
    )
    
    # set axis ticks
    [tick.label.set_fontsize(fontsize_xtick) for tick in ax.xaxis.get_major_ticks()]
    [tick.label.set_fontsize(fontsize_ytick) for tick in ax.yaxis.get_major_ticks()]
    plt.tight_layout()    

    # tranform input lats and lons to map projections
    x, y = bmap(lons, lats)
    
    if vals is None:
        # perform 2-D scatter plot
        ax.plot(
            x, y,
            marker="o",
            linestyle="",
            linewidth=2,
            markerfacecolor="royalblue",
            markeredgecolor="mediumblue",
            markersize=markersize,
            alpha=alpha)
    else:
        # perom bubble plot
        if vlim is None:
            vlim = [min(vals), max(vals)]
        # set a colormap
        cmap = plt.cm.get_cmap(colormap)
        cax = plt.scatter(
            x=x, y=y, c=vals,
            vmin=vlim[0],
            vmax =vlim[1],
            cmap=cmap,
            s=50,
            edgecolors='none',
            alpha=.95,
            )
        # set color bar
        cbar = plt.colorbar(cax, shrink =1, pad=0.01)
        cbar.set_label(label)
        date = datetime.utcnow()
        CS = bmap.nightshade(date, alpha=0.2)
    
    # set tight layout
    plt.tight_layout()
    
    return ax, fig

```

### 3.5. Define setup parameters


```python
# set number of samples
nm_samples = 700
# set (latitude, longitude) intervals
nm_lats = 100
nm_lons = 100
# set path to save figures
path_fig = os.path.join(root_dir, "reports", "figures")
path_log = os.path.join(root_dir, "reports", "logs")
# set true to save figures
save_fig = True
# set true to save results to csv
save_csv = True
# set true to log print
verbose = False
# set figure size
figsize = (10, 7)
markersize = 6
# set api params
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "imperial"
params = {
    "appid": api_key,
    "units": units
}
```


```python
# generate latitudes and longitudes
lats = np.linspace(-90, 90, nm_lats)
lons = np.linspace(-180, 180, nm_lons)
```


```python
df = pd.DataFrame()
df["City Name"] = [""]
df["Country Code"] = [""]

index = 0
for lat, lon in product(lats, lons):
    city = cp.nearest_city(lat, lon)
    city_name = city.city_name.title()
    country_code = city.country_code
    
    if not df["City Name"].isin([city_name]).any():
        if verbose:
            print("{:4d} ({:+07.2f}{:s},{:+07.2f}{:s}) {:20s} {:s}".format(
                index, lat, DEGREE, lon, DEGREE, city_name, country_code))
        df.loc[index, "City Name"] = city_name
        df.loc[index, "Country Code"] = country_code
        index += 1
```


```python
df.head(20)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City Name</th>
      <th>Country Code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Vaini</td>
      <td>to</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mataura</td>
      <td>pf</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Rikitea</td>
      <td>pf</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Punta Arenas</td>
      <td>cl</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ushuaia</td>
      <td>ar</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hermanus</td>
      <td>za</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Bredasdorp</td>
      <td>za</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Port Elizabeth</td>
      <td>za</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Port Alfred</td>
      <td>za</td>
    </tr>
    <tr>
      <th>9</th>
      <td>East London</td>
      <td>za</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Taolanaro</td>
      <td>mg</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Busselton</td>
      <td>au</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Albany</td>
      <td>au</td>
    </tr>
    <tr>
      <th>13</th>
      <td>New Norfolk</td>
      <td>au</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Hobart</td>
      <td>au</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Bluff</td>
      <td>nz</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Kaitangata</td>
      <td>nz</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Cape Town</td>
      <td>za</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Kruisfontein</td>
      <td>za</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Saint-Philippe</td>
      <td>re</td>
    </tr>
  </tbody>
</table>
</div>



### 3.6. Collect evaluation city weather data


```python
# get 'n' random samples from collected cities 
df = df.loc[
    np.random.choice(list(range(0, df.shape[0])),
                     size=nm_samples,
                     replace=False), :]
df["Latitude"] = ""
df["Longitude"] = ""
df["Temperature"] = ""
df["Humidity"] = ""
df["Cloudiness"] = ""
df["Wind-Speed"] = ""
df["Wind-Direction"] = ""

# get current time
current_time = time.gmtime()

# collect weather data from API
for index, row in df.iterrows():
    city_name = row["City Name"]
    country_code = row["Country Code"]
    params["q"] = f"{city_name},{country_code}"
    response_ = requests.get(url, params)
    response = response_.json()
    try:
        df.loc[index]["Latitude"] = response["coord"]["lat"]
        df.loc[index]["Longitude"] = response["coord"]["lon"]
        df.loc[index]["Temperature"]  = response["main"]["temp"]
        df.loc[index]["Humidity"] = response["main"]["humidity"]
        df.loc[index]["Cloudiness"] = response["clouds"]["all"] 
        df.loc[index]["Wind-Speed"] = response["wind"]["speed"]
        df.loc[index]["Wind-Direction"] = response["wind"]["deg"]
        if verbose:
            print(f"{index} {city_name} <{response_.url}>: OK")
    except (KeyError, IndexError):
        if verbose:
            print(f"{index} {city_name} <{response_.url}>: ERROR, skipped")
        df.drop(labels=index, inplace=True)
df = df.reset_index()
```


```python
df.head(20)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>City Name</th>
      <th>Country Code</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Temperature</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind-Speed</th>
      <th>Wind-Direction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2362</td>
      <td>Aklavik</td>
      <td>ca</td>
      <td>68.22</td>
      <td>-135.01</td>
      <td>28.4</td>
      <td>92</td>
      <td>90</td>
      <td>9.17</td>
      <td>280</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1585</td>
      <td>Kaka</td>
      <td>tm</td>
      <td>37.35</td>
      <td>59.62</td>
      <td>87.58</td>
      <td>30</td>
      <td>0</td>
      <td>3.71</td>
      <td>52.0002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103</td>
      <td>Veinticinco De Mayo</td>
      <td>ar</td>
      <td>-27.38</td>
      <td>-54.75</td>
      <td>49.69</td>
      <td>88</td>
      <td>36</td>
      <td>2.71</td>
      <td>95.5002</td>
    </tr>
    <tr>
      <th>3</th>
      <td>354</td>
      <td>Tsiroanomandidy</td>
      <td>mg</td>
      <td>-18.77</td>
      <td>46.05</td>
      <td>80.5</td>
      <td>33</td>
      <td>0</td>
      <td>5.53</td>
      <td>116.502</td>
    </tr>
    <tr>
      <th>4</th>
      <td>81</td>
      <td>Lakes Entrance</td>
      <td>au</td>
      <td>-37.88</td>
      <td>147.99</td>
      <td>53.47</td>
      <td>95</td>
      <td>0</td>
      <td>6.96</td>
      <td>80.5002</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2390</td>
      <td>Zhigansk</td>
      <td>ru</td>
      <td>66.77</td>
      <td>123.37</td>
      <td>56.44</td>
      <td>41</td>
      <td>12</td>
      <td>3.71</td>
      <td>357</td>
    </tr>
    <tr>
      <th>6</th>
      <td>742</td>
      <td>Amapa</td>
      <td>br</td>
      <td>-1.83</td>
      <td>-56.23</td>
      <td>75.16</td>
      <td>93</td>
      <td>0</td>
      <td>2.37</td>
      <td>20.0002</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1018</td>
      <td>Lahij</td>
      <td>ye</td>
      <td>13.06</td>
      <td>44.88</td>
      <td>106.75</td>
      <td>21</td>
      <td>0</td>
      <td>8.63</td>
      <td>147</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2356</td>
      <td>Sangar</td>
      <td>ru</td>
      <td>63.92</td>
      <td>127.47</td>
      <td>63.64</td>
      <td>31</td>
      <td>0</td>
      <td>6.85</td>
      <td>302</td>
    </tr>
    <tr>
      <th>9</th>
      <td>745</td>
      <td>Jardim</td>
      <td>br</td>
      <td>-21.48</td>
      <td>-56.15</td>
      <td>64.99</td>
      <td>78</td>
      <td>12</td>
      <td>10.09</td>
      <td>81.0002</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1681</td>
      <td>Sitges</td>
      <td>es</td>
      <td>41.24</td>
      <td>1.82</td>
      <td>68.58</td>
      <td>77</td>
      <td>75</td>
      <td>17.22</td>
      <td>60</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1984</td>
      <td>Zavitinsk</td>
      <td>ru</td>
      <td>50.11</td>
      <td>129.44</td>
      <td>61.03</td>
      <td>56</td>
      <td>44</td>
      <td>2.37</td>
      <td>230.5</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1603</td>
      <td>Ishinomaki</td>
      <td>jp</td>
      <td>38.42</td>
      <td>141.3</td>
      <td>64.4</td>
      <td>63</td>
      <td>40</td>
      <td>11.41</td>
      <td>120</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1427</td>
      <td>Yafran</td>
      <td>ly</td>
      <td>32.06</td>
      <td>12.53</td>
      <td>77.68</td>
      <td>46</td>
      <td>0</td>
      <td>3.94</td>
      <td>123</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2313</td>
      <td>Troitsko-Pechorsk</td>
      <td>ru</td>
      <td>62.71</td>
      <td>56.19</td>
      <td>53.74</td>
      <td>56</td>
      <td>0</td>
      <td>7.85</td>
      <td>268.5</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1459</td>
      <td>Portales</td>
      <td>us</td>
      <td>34.19</td>
      <td>-103.33</td>
      <td>67.55</td>
      <td>51</td>
      <td>1</td>
      <td>4.7</td>
      <td>90</td>
    </tr>
    <tr>
      <th>16</th>
      <td>853</td>
      <td>Labuan</td>
      <td>my</td>
      <td>5.33</td>
      <td>115.2</td>
      <td>88.79</td>
      <td>74</td>
      <td>75</td>
      <td>8.05</td>
      <td>280</td>
    </tr>
    <tr>
      <th>17</th>
      <td>529</td>
      <td>Jatiroto</td>
      <td>id</td>
      <td>-7.61</td>
      <td>109.46</td>
      <td>82.63</td>
      <td>88</td>
      <td>44</td>
      <td>15.79</td>
      <td>126.5</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1827</td>
      <td>Merrill</td>
      <td>us</td>
      <td>42.03</td>
      <td>-121.6</td>
      <td>48.2</td>
      <td>87</td>
      <td>90</td>
      <td>11.41</td>
      <td>250</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1007</td>
      <td>Sokoto</td>
      <td>ng</td>
      <td>13.06</td>
      <td>5.24</td>
      <td>85.42</td>
      <td>76</td>
      <td>48</td>
      <td>12.44</td>
      <td>206</td>
    </tr>
  </tbody>
</table>
</div>



### 3.7. Plot city distribution map


```python
time_str = time.strftime("'%Y-%m-%d %H:%M'", current_time)
title = f"City Weather Data Distribution for {time_str}"

ax, fig = geomap_scatter(
    df["Latitude"].values,
    df["Longitude"].values,
    vals=None,
    vlim=None,
    markersize=markersize,
    title=title,
    figsize=figsize)
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"city-distribution-{time_str}"),
                transparent=False, bbox_inches="tight")
```

    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1708: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      limb = ax.axesPatch
    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1711: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      if limb is not ax.axesPatch:



![png](../images/output_22_1.png)


### 3.8. Scatter plots

### 3.8.1. Temperature vs. Latitude


```python
metric = "Temperature"
unit = f"{DEGREE}F"
ylim = None

ax , fig = scatter_weather_vs_latitude()
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-vs-latitude-{time_str}"),
                transparent=False, bbox_inches="tight")
```


![png](../images/output_25_0.png)


### 3.8.2. Humidity vs. Latitude


```python
metric = "Humidity"
unit = "%"
ylim = [0, 100]

ax , fig = scatter_weather_vs_latitude()
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-vs-latitude-{time_str}"),
                transparent=False, bbox_inches="tight")
```


![png](../images/output_27_0.png)


### 3.8.3. Cloudiness vs. Latitude


```python
metric = "Cloudiness"
unit = "%"
ylim = [0, 100]


ax , fig = scatter_weather_vs_latitude()
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-vs-latitude-{time_str}"),
                transparent=False, bbox_inches="tight")
```


![png](../images/output_29_0.png)


### 3.8.4. Wind-Speed vs. Latitude


```python
metric = "Wind-Speed"
unit = "mph"
ylim = None

ax, fig = scatter_weather_vs_latitude()
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-vs-latitude-{time_str}"),
                transparent=False, bbox_inches="tight")
```


![png](../images/output_31_0.png)


### 3.9. Geomaps

### 3.9.1. Temperature


```python
metric = "Temperature"
unit = f"{DEGREE}F"
colormap = "bwr"
vlim = None

time_str = time.strftime("'%Y-%m-%d %H:%M'", current_time)
title = f"{metric} ({unit}) Geomap for {time_str}"
ax , fig = geomap_scatter(df["Latitude"].values,
               df["Longitude"].values,
               df[metric].values,
               vlim=vlim,
               markersize=markersize,
               title=title,
               figsize=figsize,
               colormap=colormap)
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-geomap-{time_str}"),
                transparent=False, bbox_inches="tight")
```

    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1708: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      limb = ax.axesPatch
    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1711: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      if limb is not ax.axesPatch:



![png](../images/output_34_1.png)


### 3.9.2. Humidity


```python
metric = "Humidity"
unit = "%"
colormap = "BrBG"
vlim = None

time_str = time.strftime("'%Y-%m-%d %H:%M'", current_time)
title = f"{metric} ({unit}) Geomap for {time_str}"
ax , fig = geomap_scatter(df["Latitude"].values,
               df["Longitude"].values,
               df[metric].values,
               vlim=vlim,
               markersize=markersize,
               title=title,
               figsize=figsize,
               colormap=colormap)
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-geomap-{time_str}"),
                transparent=False, bbox_inches="tight")
```

    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1708: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      limb = ax.axesPatch
    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1711: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      if limb is not ax.axesPatch:



![png](../images/output_36_1.png)


### 3.9.3. Cloudiness


```python
metric = "Cloudiness"
unit = "%"
colormap = "hot"
vlim = None

time_str = time.strftime("'%Y-%m-%d %H:%M'", current_time)
title = f"{metric} ({unit}) Geomap for {time_str}"
ax , fig = geomap_scatter(df["Latitude"].values,
               df["Longitude"].values,
               df[metric].values,
               vlim=vlim,
               markersize=markersize,
               title=title,
               figsize=figsize,
               colormap=colormap)
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-geomap-{time_str}"),
                transparent=False, bbox_inches="tight")
```

    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1708: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      limb = ax.axesPatch
    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1711: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      if limb is not ax.axesPatch:



![png](../images/output_38_1.png)


### 3.9.4. Wind-Speed


```python
metric = "Wind-Speed"
unit = "%"
colormap = "seismic"
vlim = None

time_str = time.strftime("'%Y-%m-%d %H:%M'", current_time)
title = f"{metric} ({unit}) Geomap for {time_str}"
ax , fig = geomap_scatter(df["Latitude"].values,
               df["Longitude"].values,
               df[metric].values,
               vlim=vlim,
               markersize=markersize,
               title=title,
               figsize=figsize,
               colormap=colormap)
if save_fig:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    fig.savefig(os.path.join(path_fig, f"{metric}".lower() + f"-geomap-{time_str}"),
                transparent=False, bbox_inches="tight")
```

    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1708: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      limb = ax.axesPatch
    /home/h8147/miniconda3/envs/data-bootcamp/lib/python3.6/site-packages/mpl_toolkits/basemap/__init__.py:1711: MatplotlibDeprecationWarning: The axesPatch function was deprecated in version 2.1. Use Axes.patch instead.
      if limb is not ax.axesPatch:



![png](../images/output_40_1.png)


### 3.10. Save city eval data


```python
if save_csv:
    time_str = time.strftime("%Y-%m-%d-%H-%M", current_time)
    df.to_csv(os.path.join(path_log, f"city-weather-{time_str}.csv"))
```
