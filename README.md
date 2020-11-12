# DEAfrica-CSE Senegal Flood Analysis

This repository contians Jupyter notebooks and associated data and documentation for conducting urban flood analyses in Dakar, Senegal. One of the goals is to create a model for determining flood hazard maps (flooding frequency) and flood risk maps (product of flooding frequency and estimated damages caused by floods in different areas) for the coastline of Senegal.

## Organization

The **floodareas** directory contains the citizen science data (**citizen-data-flood-zones**), EO4SD flood hazard (**eo4sd_dakar_fhazard_2018**) and risk (**eo4sd_dakar_frisk_2018**) maps, and a GeoJSON file enclosing an area of Dakar particularly vulnerable to flooding (**Dakar_flooding.geojson**).

The **landuse** directory contains a shapefile of landuse data (**landuse_DK.shp**). There is also a notebooks that loads and shows the data (**landuse_DK_test.ipynb**).

The **precipitation** directory contains precipitation data. Information on obtaining CHIRPS precipitation data is included at the bottom of this document (place in a **chirps** subdirectory of **precipitation**).

The **shapefile** directory contains population data in the **Commune_DK.shp** file. The shapes of Dakar's districts is contained in the **Dakar_districts.geojson** file.

The **tests** directory contains various testing notebooks (e.g. loading and inspecting data).

There are several methods that have been used to try to detect flooding in Dakar.

## Method 1 - Machine Learning

**Summary: A larger dataset than the current one for Dakar is needed to create an accurate model for determining a flood hazard map for Senegal. The models that can be created with the current flood hazard map for Dakar do not perform well. If you are unfamiliar with machine learning, try one of the other methods mentioned below first.**

The first attempt was to train and use machine learning models. These notebooks are in the **training_and_mapping** directory.

The algorithm used in these notebooks is as follows:
1. Load bands from various satellites and derive relevant information from them (e.g. water indicies like MNDWI). This includes elevation data from [SRTM](https://www2.jpl.nasa.gov/srtm/) and precipitation data from CHIRPS ([info here](https://www.chc.ucsb.edu/data/chirps), [data here](https://data.chc.ucsb.edu/products/CHIRPS-2.0/)).
2. Calculate the "summary statistics" for each data variable across time (e.g. water index for a particular satellite). These statistics can be min, mean, max, and standard deviation - or just mean for binary variables. This results in collection of 2D datasets (spatial dimensions - "composites" in a sense).
3. Run a [linear discriminant analysis](https://en.wikipedia.org/wiki/Linear_discriminant_analysis) to identify the summary stats that best help predict flood hazard areas. This can provide insight into which summary statistics are the most helpful in identifying flood hazard areas (including the degree of hazard).
4. Train the machine learning models.
5. Create flood hazard maps with the trained models.

**Note that the successfulness of machine learning techniques is often determined by the amount of data used to train them - both features (summary statistics in this case) and labels (the flood hazard levels of the pixels for the training area). A model trained on a given set of training data should not be expected to perform well on very different data.**

## Method 2 - Water Indices

The second attempt was to use water indices - attempting to approximate either the flood hazard map from EO4SD or [the citizen science data](https://hess.copernicus.org/articles/24/61/2020/hess-24-61-2020.pdf). These notebooks are in the **water_indices** directory. Detecting water with water indicies is typically more useful for reasonably deep water bodies and is the simpliest option.

>#### Challenges

There are several factors that can complicate this type of analysis:
1. In torrential (rainfall) flooding, using optical data such as Landsat or Sentinel-2 may not be a good idea since clouds may obscure views of the flooding.
2. May satellites do not have a revisit time frequent enough to see a flooding event, but if water remains for several days, then this may be acceptable.
3. The exent of the flood may be hard to discern due to urban features, such as buildings and their shadows.
4. Optical data may also be unsuitable since the flooded areas are often too shallow to have a significantly water-like spectral response.

Radar data such as Sentinel-1 may help detect fairly shallow water bodies in urban environments.

## Method 3 - Hydrological Analysis with RichDEM

We can use a library called [RichDEM](https://richdem.readthedocs.io/en/latest/) to analyze torrential (precipitation) flooding such as occurs in Dakar. Modeling the flow and accumulation of water in a torrential flood is a kind of hydrological modeling.

>### Data

There are several required types of datasets:

| Data Type | Example | Description |
|-----------|---------|-------------|
| Digital Elevation Map | SRTM    | 30m spatial resolution elevation data, available on the DEAfrica sandbox as the product ‘srtm’. [Download from USGS EarthExplorer](https://earthexplorer.usgs.gov/)      |
|                       | MERIT   | 90m spatial resolution elevation data based on several data sources including SRTM – more accurate than SRTM. [See here](http://hydro.iis.u-tokyo.ac.jp/~yamadai/MERIT_DEM/)           |
| Precipitation Data    | CHIRPS  | 0.05x0.05 degree spatial resolution, daily max temporal resolution. [See here](https://data.chc.ucsb.edu/products/CHIRPS-2.0/)               |
|                       | IMERG   | 0.1x0.1 degree spatial resolution, 30 minutes max temporal resolution. [Download from NASA Giovanni](https://giovanni.gsfc.nasa.gov/giovanni/). |
| Soil Permeability     | ISRIC   | [See here](https://data.isric.org/geonetwork/srv/eng/catalog.search) (has water holding capacity, but possibly not soil permeability data for Senegal)   |
| Soil Moisture         | SMAP    | 30 km resolution, measures moisture in top 5cm of soil. [See here](https://smap.jpl.nasa.gov/) |
| Population            | GPW     | [See AidData GeoQuery](http://geo.aiddata.org/query/)                    |
| Land Cover            | In Repo | See the landuse/landuse_DK.shp file. |

Although the SRTM data is currently available on the Digital Earth Africa sandbox, we recommend using MERIT data for its higher accuracy than SRTM. High elevation accuracy is very important for this analysis since the flooding area is quite flat.

>### Process

This section is a work in progress.

The following process is known to be possible using the RichDEM library.
1. Apply depression filling (recommended for first attempt) or breaching to the DEM.
2. Apply a small gradient (either Beau-convergent  or Epsilon-constant) toward the lowest egress to avoid ambiguous flow in flat areas.
3. Use the D-infinity flow accumulation method (Tarboton, 1997) on the filled DEM to obtain an idealized flow pattern. This reveals areas of high flow as well as the hydrological units.

>### Challenges

In hydrological modeling, determining where water is flowing in urban areas is difficult for several reasons: 
1. Urban areas (such as Dakar) are often fairly flat, so the flow of water is often ambiguous.
2. The DEM data should acount for buildings (not just terrain) for an accurate analysis. It also must have very good accuracy and resolution (especially temporal).
3. Some datasets such as soil permeability may be difficult to obtain.

## Obtaining data

To obtain the flood hazard shapefile (**floodareas/eo4sd_dakar_fhazard_2018**), use [this webpage](https://datacatalog.worldbank.org/dataset/dakar-senegal-flood-hazard-map-esa-eo4sd-urban). 

To obtain the flood risk shapefile (**floodareas/eo4sd_dakar_frisk_2018**), use [this webpage](https://datacatalog.worldbank.org/dataset/dakar-senegal-flood-risk-map-esa-eo4sd-urban), 

To obtain the CHIRPS precipitation data (**precipitation/chirps**), use [this webpage to download the montly TIFFs](https://data.chc.ucsb.edu/products/CHIRPS-2.0/africa_monthly/tifs/). Place the **.tif.gz** files in **precipitation/chirps** and then extract them with **gzip -k \*** when in the **precipitation/chirps** directory. You can see the **chirps_test.ipynb** notebook in that directory for an example of how to load the data.