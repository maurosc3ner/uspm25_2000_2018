# Two Decades of Pollution in US 

## County-Level Annual Mean PM2.5 Concentrations, 2000-2018

![US PM2.5 ](test1.gif)


## Objectives

+ Creating custom equal-area albers projection by moving AL and HI to the bottom.
+ Create an animation of the Annual Mean PM2.5 Concentrations by County in R (ggplot, magick)

## Dataset

We will use the **North American Regional Estimates (V4.NA.02.MAPLE)** for the surface pm 2.5 [1]. You can get the raw images from [here](http://fizz.phys.dal.ca/~atmos/martin/?page_id=140).  However, I have already processed 2000-2018 for you and placed as a time-wide format by U.S. FIPS,  (check [data/pm2.5byCounty.csv](https://github.com/maurosc3ner/uspm25_2000_2018/blob/master/data/pm2.5byCounty.csv) ). For the county-level spatial data, I am using the CDC's US-ADM2 map due to my analysis [2], but you can use **getData boundaries** too as long as it has fips codes for the join.

## Code 

### Load libraries and datasets
```
rm(list = ls())
library(tidyverse)
library(raster)
library(rgdal)
library(sf)
library(ggthemes)
library(rgeos)
library(maptools)
library(magick)

d<-read.csv("pm2.5byCounty.csv")   

uscounty<-readOGR("../covid19/data/SVI2018","SVI2018_US_county")
uscounty@data<-uscounty@data %>%
  select(FIPS,ST,ST_ABBR) #subset columns
```

# Custom albers-prj

```
# taken from https://github.com/hrbrmstr/rd3albers
crsAlbers<-CRS("+proj=laea +lat_0=45 +lon_0=-100 +x_0=0 +y_0=0 +a=6370997 +b=6370997 +units=m +no_defs")
us_aea <- spTransform(uscounty,crsAlbers )
us_aea@data$id <- rownames(us_aea@data)

# extract, then rotate, shrink & move alaska (and reset projection)
alaska <- us_aea[us_aea$ST=="02",]
# plot(alaska)
# avoid 'row.names of data and Polygons IDs do not match' error
# then convert it to SPDF back again
alaskaDF <- as.data.frame(alaska)
# and set rowname=ID from polygon@ID  https://stat.ethz.ch/pipermail/r-help/2005-December/085175.html
row.names(alaskaDF) <- sapply(slot(alaska, "polygons"), function(x) slot(x, "ID"))
alaska <- SpatialPolygonsDataFrame(alaska, alaskaDF)
#geometric operations
alaska <- elide(alaska, rotate=-50)
alaska <- elide(alaska, scale=max(apply(bbox(alaska), 1, diff)) / 2.3)
alaska <- elide(alaska, shift=c(-2200000, -2600000))
proj4string(alaska) <- proj4string(us_aea)
# plot(alaska)

## Repeat the same process for Hawaii
# extract, then rotate & shift hawaii
hawaii <- us_aea[us_aea$ST=="15",]
hawaiiDF <- as.data.frame(hawaii)
# and set rowname=ID from polygon  https://stat.ethz.ch/pipermail/r-help/2005-December/085175.html
row.names(hawaiiDF) <- sapply(slot(hawaii, "polygons"), function(x) slot(x, "ID"))
hawaii <- SpatialPolygonsDataFrame(hawaii, hawaiiDF)
# continue rotating
hawaii <- elide(hawaii, rotate=-35)
hawaii <- elide(hawaii, shift=c(5550000, -1800000))
proj4string(hawaii) <- proj4string(us_aea)
us_aea <- us_aea[!us_aea$ST %in% c("02", "15", "72"),]
us_aea <- rbind(us_aea, alaska, hawaii)

#Plot final results
plot(us_aea)
# Generate State level map
usStates<-gUnaryUnion(us_aea, id = us_aea@data$ST)
plot(usStates)
```

### Mapping

You can tweak ggplot according to your needs (more dpi) but it will take more time for the animation.

```
# color range fix!
brks<-seq(floor(min(d[,-1],na.rm = T)),ceiling(max(d[,-1],na.rm = T)),by = 3)
for(idx in 2000:2018){
  g1<-ggplot() +
    geom_sf(data=st_as_sf(us_aea),aes(fill = eval(as.name(paste0("pm25_",idx)))),size=0.01) +
    geom_sf(data=st_as_sf(usStates),colour="black",fill=NA,size=0.4) +
    ggtitle(paste0(" U.S. Annual Mean PM2.5 Concentrations by County, ",idx))+
    labs(caption = "data: Surface PM2.5 [Donkelaar et al.], viz: @maurosc3ner")+
    coord_sf(crs = crsAlbers)+
    scale_fill_viridis_c(name="PM 2.5",breaks=brks,
                         limits=c(ceiling(min(d[,-1],na.rm = T)), 
                                          ceiling(max(d[,-1],na.rm = T))))+
    theme_map()+
    theme(legend.position = "right",legend.key.size = unit(0.8, "cm"),legend.box.margin = margin(1,1,1,1))+
    guides(colour = guide_legend(nrow = 1))+
    theme(plot.title=element_text(size=16))
  
  ggsave(filename = paste0(idx,"_pm25",".png"), g1,# en cm mejor para mantener el tamanho
       width = 25, dpi = 200, units = "cm", device='png')

}
# plot the last to be sure
g1
```

## Animation

```
list <- list.files(productPath, '*_pm25.png')
images <- image_read(list)
animation <- image_animate(images, fps = 2,optimize=T)
image_write(animation, 'test1.gif')
```

## Everything is better in black!

![US PM2.5 ](testCividis.gif)


If you have any question to the pre-processing, please you can reach me at:
[@maurosc3ner](https://twitter.com/maurosc3ner)
[e-mail](correaem@mail.uc.edu)
https://maurosc3ner.github.io/blog/

Thanks.

## References

[1] van Donkelaar, A., R. V. Martin, et al. (2019). Regional Estimates of Chemical Composition of Fine Particulate Matter using a Combined Geoscience-Statistical Method with Information from Satellites, Models, and Monitors. Environmental Science & Technology, 2019, [doi:10.1021/acs.est.8b06392](https://doi.org/10.1021/acs.est.8b06392).

[2] CDC's Social Vulnerability Index (SVI) https://svi.cdc.gov/data-and-tools-download.html

[3] Initial custom albers projection https://github.com/hrbrmstr/rd3albers
