---
layout: post
hero_darken: true
hero_height: is-small
title:  "GPS2space: An Open-source Python Library for Working with GPS data"
author: "Shuai Zhou"
date: "2020-09-16"
excerpt: "An introduction to an open-source Python library: GPS2space"
---

**GPS2space** is an open-source Python library for:

- Compiling spatial data from raw GPS lat/long coordinates
- Constructing Buffer- and Convex hull-based activity space and shared space at different time scales
- Speeding up nearest distance query using cKDTree method (currently only support Point-Point nearest distance query)

See [documentation](https://gps2space.readthedocs.io/en/latest/index.html) for more information

## Load libraries


```python
%matplotlib inline

import pandas as pd
import geopandas as gpd
import matplotlib.pyplot as plt

from gps2space import geodf
from gps2space import space
```

## Load data


```python
df = pd.read_csv('../data/example.csv')
gdf = geodf.df_to_gdf(df, x='longitude', y='latitude')
```

## Construct person-time


```python
gdf['timestamp'] = pd.to_datetime(gdf['timestamp'], infer_datetime_format=True)
gdf['week'] = gdf['timestamp'].dt.week

gdf['person_week'] = gdf['pid'].astype(str) + '_' + gdf['week'].astype(str)
```


```python
gdf.head()
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
      <th>pid</th>
      <th>timestamp</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>geometry</th>
      <th>week</th>
      <th>person_week</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>P2</td>
      <td>2020-04-27 10:42:22.162176000</td>
      <td>40.993799</td>
      <td>-76.669419</td>
      <td>POINT (-76.66942 40.99380)</td>
      <td>18</td>
      <td>P2_18</td>
    </tr>
    <tr>
      <th>1</th>
      <td>P2</td>
      <td>2020-06-02 01:12:45.308505600</td>
      <td>39.946904</td>
      <td>-78.926234</td>
      <td>POINT (-78.92623 39.94690)</td>
      <td>23</td>
      <td>P2_23</td>
    </tr>
    <tr>
      <th>2</th>
      <td>P2</td>
      <td>2020-05-08 23:47:33.718185600</td>
      <td>41.237403</td>
      <td>-79.252317</td>
      <td>POINT (-79.25232 41.23740)</td>
      <td>19</td>
      <td>P2_19</td>
    </tr>
    <tr>
      <th>3</th>
      <td>P2</td>
      <td>2020-04-26 14:31:12.100310400</td>
      <td>41.991390</td>
      <td>-77.467769</td>
      <td>POINT (-77.46777 41.99139)</td>
      <td>17</td>
      <td>P2_17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>P2</td>
      <td>2020-03-31 15:53:27.777897600</td>
      <td>41.492674</td>
      <td>-76.542921</td>
      <td>POINT (-76.54292 41.49267)</td>
      <td>14</td>
      <td>P2_14</td>
    </tr>
  </tbody>
</table>
</div>



## Calculate convex hull-based activity space


```python
convex_space = space.convex_space(gdf, group='person_week', proj=2163)
```


```python
convex_space.head()
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
      <th>person_week</th>
      <th>geometry</th>
      <th>convx_area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>P1_1</td>
      <td>POLYGON ((-79.08352 40.61927, -78.64613 41.031...</td>
      <td>0.459119</td>
    </tr>
    <tr>
      <th>1</th>
      <td>P1_10</td>
      <td>POLYGON ((-77.29803 39.81326, -78.33096 41.824...</td>
      <td>2.980511</td>
    </tr>
    <tr>
      <th>2</th>
      <td>P1_11</td>
      <td>POINT (-80.05021 41.60526)</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>P1_12</td>
      <td>LINESTRING (-78.86727 41.79762, -75.39833 40.5...</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>P1_13</td>
      <td>POLYGON ((-79.36647 39.99230, -77.46196 41.851...</td>
      <td>1.425932</td>
    </tr>
  </tbody>
</table>
</div>



## Check P1 and P2 at week 10


```python
p1_w10 = convex_space[convex_space['person_week']=='P1_10']
p2_w10 = convex_space[convex_space['person_week']=='P2_10']
```


```python
share = gpd.overlay(p1_w10, p2_w10, how='intersection')
```


```python
ax = p1_w10.boundary.plot(edgecolor='green')
p2_w10.boundary.plot(ax=ax,edgecolor='black')
share.plot(ax=ax)
plt.show();
```


![png](/images/2020-11-26-intro-to-gps2space.png)


As shown in the figure:

- The enclosed green line is the activity space for P1 at week 10
- The enclosed black line is the activity space for P2 at week 10
- The blue area is the shared space for P1 and P2 at week 10




