##okeedokee - this is to find the closest occurences
#to your project site
##load your workspace

##clear your enviornment with this
#rm(list=ls())


setwd("C:/Users/allan/OneDrive/New_Graph/Current/Code/R/SAR")
load("sar_work_space.RData")


##inspiration from hydatr - need to load hydatr to see this 
#getAnywhere(hydat_find_stations)

#get the lat long of your site with geocode
#install.packages("ggmap")
devtools::install_github("dkahle/ggmap")

library(ggmap)

##can't gete the google api to work for me.  dsk works fine though.
register_google(key = "AIzaSyC4RsJU51ciNexchCKznbP9zawA07THzn0")


##had to add source = "dsk" to get the google api to work
loc <- geocode("Ymir,bc", source = "dsk")


#or add the coordinates manually - need to figure 
#out why this doesn't look right yet
#loc2<- c(lon = -95.3632715, lat = 29.7632836)


#convert lat long 4326 to wkid 3857 
library(sp)
library(rgdal)
#coordinates(loc)=~lon+lat
#proj4string(loc)=CRS("+init=epsg:4326")
#spTransform(loc,CRS("+init=epsg:3857")) #don't need this
#loc_df<- as.data.frame(loc)

library(gmt)
library(dplyr)

#first we need the geom columns to be in lat long
df<- as.data.frame(df)
coordinates(df)=~geometry.x+geometry.y
proj4string(df)=CRS("+init=epsg:3857")
df_lat_long <- spTransform(df,CRS("+init=epsg:4326"))

#calculate the distance from each occurence to your spot 
#and add as column
df_lat_long$dist_from_query_km <- geodist(loc$lat,loc$lon, df_lat_long$geometry.y, 
                                 df_lat_long$geometry.x, units = "km")

#flatten your df to allow filtering
df_lat_long_flat <- as.data.frame(df_lat_long)

#filter it by occurences within 50m
close_occ <- filter(df_lat_long_flat, dist_from_query_km <=20 )
unique(close_occ$value)


#select the info you want to put in table
#close_occ<- select(close_occ,"layerId", "Species.Code", 
                 #"Description","Locality", "Date", "dist_from_query_km", 
                 #"geometry.x","geometry.y")

##order by closest to longest 
close_occ <- arrange(close_occ, dist_from_query_km)
tail(close_occ)

#get bounding box of closest and furthest occurences 
#occ_bb <- c(min(close_occ$geometry.x), min(close_occ$geometry.y), 
            #max(close_occ$geometry.x), max(close_occ$geometry.y))

#map it
myMap <- get_map(location=loc, source="google", maptype="terrain", crop=FALSE, zoom = 10) 
ggmap(myMap)

ggmap(myMap)+
  geom_point(aes(x = geometry.x, y = geometry.y), data = close_occ,
             alpha = .5, color="darkred", size = 3)


