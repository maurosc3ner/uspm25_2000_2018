# Two Decades of Pollution in US 

## County-Level Annual Mean PM2.5 Concentrations, 2000-2018

![US PM2.5 ]()


## Mini-tutorial goals

+ Creating custom equal-area albers projection by moving AL and HI to the bottom.
+ Create an animation of the Annual Mean PM2.5 Concentrations by County in R (ggplot, magick)

## Dataset:

We will use the **North American Regional Estimates (V4.NA.02.MAPLE)** for the surface pm 2.5 [1]. You can get the raw images from [here](http://fizz.phys.dal.ca/~atmos/martin/?page_id=140).  However, I have already processed 2000-2018 for you and placed as a time-wide format by U.S. FIPS (check data/pm2.5byCounty.csv)

```
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
rm(list = ls())
library(tidyverse)
library(raster)
library(rgdal)
library(sf)
library(ggthemes)
library(rgeos)
library(maptools)
library(spacetime)
library(magick)

d<-read.csv("pm2.5byCounty.csv")   

uscounty<-readOGR("../covid19/data/SVI2018","SVI2018_US_county")
uscounty@data<-uscounty@data %>%
  select(FIPS,ST,ST_ABBR) #subset columns
```

```


## References

[1] van Donkelaar, A., R. V. Martin, et al. (2019). Regional Estimates of Chemical Composition of Fine Particulate Matter using a Combined Geoscience-Statistical Method with Information from Satellites, Models, and Monitors. Environmental Science & Technology, 2019, [doi:10.1021/acs.est.8b06392](https://doi.org/10.1021/acs.est.8b06392).

[2] Initial custom albers projection https://github.com/hrbrmstr/rd3albers
