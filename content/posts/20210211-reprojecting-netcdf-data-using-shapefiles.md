---
title: "Reprojecting netCDF Data Using Shapefiles"
date: 2021-02-11T04:00:45-07:00
draft: false
tags: ["development", "spatiotemporal"]
---

I'm part of an active multi-university research effort tagged [Project Sustain](http://urban-sustain.org). The goal of this work is to support robust analyses over myriad domains to better inform urban planning. Understandably it is a broad goal with a multi-faceted approach. My particular role is in backend development. Specifically, loading data for efficient, distributed queries. Additionally, I'm contributing to interfaces for training models (e.x. regression models to predict future temperature, clustering counties based on precipitation), computing extreme value analyses (e.x. classifying heat-waves is different for each region), and other analytics tasks.

This post relates to my recent work in reprojecting gridded, netCDF data using shapefiles. The data we have is:

- [macav2 netCDF files](http://www.climatologylab.org/maca.html): 8 daily-reported, weather-based variables (in separate files) with 3 dimensions (time, latitude, and longitude) collected from the continental United States.

- [census TIGER/line shapefiles](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.2010.html): separate files for state, county, and census tract boundaries

We are trying to store this data in a time-series format, where each observation contains a shape_id (trivially defined by state, county, or census tract) and a timestamp of the data. That way, we can quickly retrieve all data based on spatiotemporal filtering criteria. The reality -- this is a rather difficult problem in regards to computational complexity. Although our netCDF data is defined by latitude, longitude and dimensions each coordinate actually refers to a rectangular region. This means that while shape inclusion is simple for point data, where each coordinate will belong to a single county, our situation is more difficult because a polygonal region may, and typically does, overlap multiple shapes. Incremental pair-wise inclusion comparisons between netCDF coordinate polygons and shapefiles over the entire dataspace is very expensive, we need a better solution.

# Creating Shapefile Coordinate Index

We begin by creating a shapefile coordinate index. This is simply a mapping between the latitude and longitude netCDF dimensions and shape_id's. As previously stated, an iterative, pair-wise comparison is computationally expensive. Rather, for each netCDF coordinate we identify a number of shapes which "may be close" based on distances of polygon centroids. This is configurable, but we have found limiting this _close-set_ to 5 shapes produces adequate results without excessive overhead. With this operation we are effectively reducing the search space. A depiction of the algorithm is provided below using a _close-set_ size of 2 for the blue bounded query over the continental United States. Note that, while the bounding region covers 4 states, only 2 will be accessed because of the _close-set_ size. This variable is a trade-off between result accuracy and processing speed.

<p align="center">
  <img width="90%" src="/posts/20210211-reprojecting-netcdf-data-using-shapefiles/shapefile-index.png">
</p>

The result of this operation is a delimited file where each row is formatted as (longitude, latitude, shape_id). It has always bugged me that global coordinates are always presented as (latitude, longitude) but in reality that is presenting (y, x), where (x, y) is more logical. This is an issue I battled when supporting gdal 3.* (originally implemented in 2.*) in my [st-image](https://github.com/hamersaw/st-image) library. Anyways, the resulting _shapefile index_ file is formatted as shown below.

    ...
    1 561 G5300090
    1 562 G5300090
    2 423 G4100150
    2 424 G4100150
    2 425 G4100150
    2 426 G4100150
    2 427 G4100150
    2 547 G5300090
    2 547 G5300310
    2 546 G5300310
    ..

# Projecting netCDF Data

From here, computing aggregations over shape variables should be straight forward. We read the _shapefile index_ into an internal map, where the key is the shape_id and value is a vector of netCDF coordinates. This construct is depicted below. Then simply iterate over each shape and compute variable aggregations based on netCDF values.

    ...
    G5300090 -> [(1, 547),(1, 561), (1,562)]
    G5300310 -> [(2, 546), (2,547)]
    G4100150 -> [(2, 423), (2,424), (2,425), (2,426), (2,427)]
    ...

However, I quickly ran into issues with over provisioned RAM. Our test dataset was for 2005, a single year which totaled just 2.9GB. Typically the macav2 data is distributed in 5 year increments. As a result, files for our 8 daily-reported variables total ~14.5GB in netCDF format. My laptop, a Lenovo X1 Extreme Gen2, has 16GB of memory. When performed over this larger data, the process was quickly killed.

A solution is to iteratively process data windows based on dimensional slices. This paradigm should not be foreign to those familiar with data cubes and similar approaches. This idea is to partition the data based on a dimension, in our case time, and process only an individual small chunk. A visual of partitioning our 3 dimension dataset by time is provided below.

<p align="center">
  <img width="50%" src="/posts/20210211-reprojecting-netcdf-data-using-shapefiles/time-partitioning.png">
</p>

We found that this approach was able to cope with increasing dataset sizes with negligible overhead. When processing time in increments of 50 days the process memory utilization is under 2.5GB. Additionally, we are able to reproject the 8 daily-reported variable, 5 year datasets into United States counties in under 5 minutes each. In our use-case this is acceptable given each results in over 5.5 million observations. I've included a head dump of the resulting data below.

    gis_join,timestamp,min_specific_humidity,max_specific_humidity,min_precipitation,max_precipitation,min_surface_downwelling_shortwave_flux_in_air,max_surface_downwelling_shortwave_flux_in_air,min_max_air_temperature,max_max_air_temperature,min_min_air_temperature,max_min_air_temperature,min_eastward_wind,max_eastward_wind,min_northward_wind,max_northward_wind,min_vpd,max_vpd
    G0100010,1104537600,0.005,0.006,11.258,17.949,41.459,51.531,286.043,289.009,272.514,275.082,-4.489,-3.623,6.011,6.449,27.000,34.000
    G0100070,1104537600,0.005,0.005,19.261,34.422,25.584,38.209,286.417,288.424,272.374,273.330,-4.532,-4.058,5.574,6.330,29.000,38.000
    G0100050,1104537600,0.005,0.005,1.572,4.659,69.754,84.655,287.273,289.528,272.444,274.336,-3.463,-2.611,5.533,6.587,30.000,46.000
    G0100150,1104537600,0.004,0.004,25.878,38.686,32.094,37.887,282.359,286.059,270.719,273.756,-5.409,-4.380,3.925,5.026,26.000,44.000
    G0100110,1104537600,0.005,0.006,3.754,7.375,57.550,74.096,286.205,288.312,272.132,273.151,-3.576,-2.950,6.160,7.174,19.000,39.000
    G0100130,1104537600,0.006,0.006,5.336,10.419,51.815,68.422,288.829,291.417,273.865,275.652,-3.244,-2.356,6.380,7.237,31.000,43.000
    G0100090,1104537600,0.004,0.005,27.876,42.626,25.293,29.998,284.354,286.204,271.667,273.768,-5.510,-4.029,3.352,4.676,28.000,38.000
    ...

The source code for this work is publicly available under the MIT license in my [ncproj-rs github repository](https://github.com/hamersaw/ncproj-rs).
