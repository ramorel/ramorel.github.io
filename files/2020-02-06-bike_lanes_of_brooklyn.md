---
title: "Brooklyn Bike Lanes"
author: "rpm"
date: 2020-02-06
output:
  html_document:
    keep_md: true
    number_sections: true
    toc: true
    toc_depth: 2
    theme: yeti
    highlight: zenburn
---



I was inspired by this [bit of code](https://github.com/deanmarchiori/culburra/blob/master/culburra.Rmd) to make a map of Brooklyn bike lanes--the lanes upon which I once biked many a mile.


```r
library(osmdata)
library(dodgr)
library(tidyverse)
library(sf)
library(ggminthemes)
library(tigris)
options(tigris_use_cache = TRUE)
```

# Importing and cleaning the street data


```r
# Boundary box that covers Brooklyn with bits of Manhattan and Queens
bbox <- st_bbox(c(xmin = -74.05, 
                  xmax = -73.81, 
                  ymax = 40.74,
                  ymin = 40.55), 
                crs = st_crs(4326)) %>% 
  st_as_sfc()

roads <- suppressMessages(roads("NY", "Kings", class = "sf"))

# Use this boundary box to remove NJ
bbnj <- st_bbox(c(xmin = -74.05, 
                  xmax = -74.02, 
                  ymax = 40.74,
                  ymin = 40.69), 
                crs = st_crs(4326)) %>% 
  st_as_sfc()

# Get shapefiles for the 5 boroughs boundaries
url <- "https://www1.nyc.gov/assets/planning/download/zip/data-maps/open-data/nybb_19d.zip"
fil <- basename(url)
if (!file.exists(fil)) download.file(url, fil)

nyc <- unzip(fil)

nyc <- st_read(nyc[1]) %>% 
  st_transform(4326) %>% 
  st_intersection(bbox)
```

```
## Reading layer `nybb' from data source `/Users/richardmorel/Documents/GitHub/ramorel.github.io/files/nybb_19d/nybb.shp' using driver `ESRI Shapefile'
## Simple feature collection with 5 features and 4 fields
## geometry type:  MULTIPOLYGON
## dimension:      XY
## bbox:           xmin: 913175.1 ymin: 120121.9 xmax: 1067383 ymax: 272844.3
## epsg (SRID):    2263
## proj4string:    +proj=lcc +lat_1=41.03333333333333 +lat_2=40.66666666666666 +lat_0=40.16666666666666 +lon_0=-74 +x_0=300000.0000000001 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=us-ft +no_defs
```

```r
bklyn <- nyc[2, ]
queens <- nyc[3, ]
mnhtn <- nyc[4, ]

# Thanks to tidycensus's creator Kyle Walker for [this handy function](https://walkerke.github.io/tidycensus/articles/spatial-data.html)
# Need it to remove bike lanes that cross the Hudson to NJ
st_erase <- function(x, y) {
  st_difference(x, st_union(y))
}

brooklyn_box <- getbb("Brooklyn, New York")
bklyn_streets <- dodgr_streetnet(brooklyn_box, expand = 1)


bike_lanes <- 
  bklyn_streets %>% 
  as_tibble() %>% 
  select(osm_id, name, contains("bicycle"), contains("cycleway")) %>% 
  pivot_longer(cols = c(-osm_id, -name), names_to = "variables") %>% 
  mutate(value = ifelse(is.na(value), "no", "yes")) %>% 
  group_by(osm_id, name) %>% 
  mutate(bike_lane = ifelse(any(value == "yes"), "yes", "no")) %>% 
  select(-variables, -value) %>% 
  distinct() %>% 
  ungroup()

bike_lanes <- 
  left_join(bike_lanes, bklyn_streets) %>% 
  st_as_sf() %>% 
  st_intersection(bbox) %>% 
  st_erase(bbnj) %>%
  select(osm_id, name, bike_lane)
```

# Creating the map

```r
ggplot() +
    geom_sf(data = bbox, fill = "lightcyan2", color = "grey20", size = 0.5) + 
    geom_sf(data = bklyn, fill = "grey70", alpha = 0.5, size = 0.43) + 
    geom_sf(data = roads, color = "grey25", size = 0.25) +
    geom_sf(data = filter(bike_lanes, bike_lane == "yes"), size = 0.4, color = "palegreen", show.legend = FALSE) +
    geom_sf(data = queens, fill = "grey50", alpha = 0.5, size = 0.2) +
    geom_sf(data = mnhtn, fill = "grey50", alpha = 0.5, size = 0.2) + 
    geom_sf(data = bbox, fill = "transparent", color = "grey20", size = 0.5) + 
    labs(title = "The Bike Lanes of Brooklyn") +
  theme_metal()
```

![](2020-02-06-bike_lanes_of_brooklyn_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

