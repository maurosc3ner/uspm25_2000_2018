# Two Decades of Pollution in US 

## County-Level Annual Mean PM2.5 Concentrations, 2000-2018

![US PM2.5 ]()


## Objectives

+ Creating custom equal-area albers projection by moving AL and HI to the bottom.
+ Create an animation of the Annual Mean PM2.5 Concentrations by County in R (ggplot, magick)

## Dataset

We will use the **North American Regional Estimates (V4.NA.02.MAPLE)** for the surface pm 2.5 [1]. You can get the raw images from [here](http://fizz.phys.dal.ca/~atmos/martin/?page_id=140).  However, I have already processed 2000-2018 for you and placed as a time-wide format by U.S. FIPS,  (check [data/pm2.5byCounty.csv](https://github.com/maurosc3ner/uspm25_2000_2018/blob/master/data/pm2.5byCounty.csv) ). For the map, CDC's US-ADM2 map for my analysis but you can use also getData boundaries too.

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
library(spacetime)
library(magick)

d<-read.csv("pm2.5byCounty.csv")   

uscounty<-readOGR("../covid19/data/SVI2018","SVI2018_US_county")
uscounty@data<-uscounty@data %>%
  select(FIPS,ST,ST_ABBR) #subset columns
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


## References

[1] van Donkelaar, A., R. V. Martin, et al. (2019). Regional Estimates of Chemical Composition of Fine Particulate Matter using a Combined Geoscience-Statistical Method with Information from Satellites, Models, and Monitors. Environmental Science & Technology, 2019, [doi:10.1021/acs.est.8b06392](https://doi.org/10.1021/acs.est.8b06392).

[2] Initial custom albers projection https://github.com/hrbrmstr/rd3albers
