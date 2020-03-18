setwd("C:/Users/allan/OneDrive/New Graph/Current/Code/R/SAR")
#look at what is in the directory
dir(".")

#load required packages and import data
.libPaths() #check library path
library(tidyverse)
library(rgdal)
library(jsonlite)
library(data.table)
library(sf)
library(RPostgreSQL)


##define your call url
url <- "http://142.103.209.57:6080/arcgis/rest/services/VascularWMS/MapServer/find?searchText=%s%s"

#Call the file that has already been queried within the BC Ecosystem Explorer
sar.raw <-as.data.frame(data.table::fread("bcsee_export.tsv"),fill = TRUE)

##fix the column headers to not have spaces
names(sar.raw) <- make.names(names(sar.raw), unique=TRUE)

##could do it wiht a csv but involves extra steps
#sar.raw <- read.csv('bcsee_export.csv',
                          #header = TRUE, 
                          #strip.white = TRUE,
                          #as.is = TRUE)

##remove the rows with no species data -- not sure why I did this.  prob doesn't make sense
#sar.raw <- sar.raw[!(is.na(sar.raw$Species.Code) | sar.raw$Species.Code==""), ]

##lets query for what we want listed species
unique(sar.raw$Name.Category)
sar.raw_sample <- filter(sar.raw, Name.Category == "Vascular Plant"
                         & (BC.List == "Red" 
                         | BC.List == "Blue"))

#lets take a random sample 
sar.raw_sample <- sample_n(sar.raw_sample,2,replace = FALSE)

#use sprintf to line up scientific name in url format 
#sar.raw$Scientific.Name.sub <- gsub(" ","+",sar.raw$Scientific.Name)
Scientific.Name.sub <- gsub(" ","+",sar.raw_sample$Scientific.Name)

#add the subbed scientific name into the urls and make a url column for later
#maybe don't need the first step if we want a column anyway
#map.url <- sprintf("http://142.103.209.57:6080/arcgis/rest/services/VascularWMS/MapServer/find?searchText=%s%s",Scientific.Name.sub,"&contains=true&searchFields=sciname&sr=&layers=1%2C2%2C3%2C4%2C5%2C6%2C7%2C8&layerDefs=&returnGeometry=true&maxAllowableOffset=&geometryPrecision=&dynamicLayers=&returnZ=false&returnM=false&gdbVersion=&f=pjson")
#sar.raw_sample$map.url <- sprintf(url,Scientific.Name.sub,"&contains=true&searchFields=sciname&sr=&layers=1%2C2%2C3%2C4%2C5%2C6%2C7%2C8&layerDefs=&returnGeometry=true&maxAllowableOffset=&geometryPrecision=&dynamicLayers=&returnZ=false&returnM=false&gdbVersion=&f=pjson")

##testing less verbose request
sar.raw_sample$map.url <- sprintf(url,Scientific.Name.sub,"&contains=true&searchFields=sciname&layers=1%2C2%2C3%2C4%2C5%2C6%2C7%2C8%2C9%2C10&returnGeometry=true&f=json")


#this website discussed downloading files if they exist 
#https://nicercode.github.io/guides/repeating-things/
#might come in handy at some point I guess

#this uses the jsonlite package.  
#need to flatten for write.csv because of nested data frames?
files <- sapply(sar.raw_sample$map.url, function(x) 
  fromJSON(readLines(x), flatten = TRUE), USE.NAMES = FALSE)



#convert to data frame that has binded the data frames from the list
#library(data.table)
df <- rbindlist(files, fill = TRUE)
names(df)
unique(df$value)

##we need to remove the observations that do not have geometry.x or geometry.y values
df <- df[!is.na(df$geometry.x), ]

##little test here to get decimal degrees
#df$geometry.x <- df$geometry.x/100000
#df$geometry.y <- df$geometry.y/100000

#replace x and y with a "geometry" column.  not sure why this is not in decimal degrees!!!!!!
df = st_as_sf(df, coords = c("geometry.x", "geometry.y"), 
                 crs = 3857, agr = "constant", remove = FALSE)

#rename the column names to remove "attributes"
colnames(df) <- gsub("attributes.","",colnames(df))

#make a column with the species code to tie to the original export info
sp.split <- as.data.frame(str_split_fixed(df$value, " ",2))
df$Species.Code <- toupper(paste0(strtrim(sp.split$V1, 4),strtrim(sp.split$V2, 3)))
rm(sp.split)

#tie location data to original bcee output
df <- merge(df,sar.raw, by = "Species.Code", all.x = TRUE)
names(df)

##from here you can go to 02 file to find occurences near your project and/or save to your database below


#Enter the values for you database connection
dsn_database = "postgis"            
dsn_hostname = "localhost"
dsn_port = "5432"               
dsn_uid = "postgres"        
dsn_pwd = "postgres"


#connect and see if the connection to the database is working
tryCatch({
  drv <- dbDriver("PostgreSQL")
  print("Connecting to database")
  conn <- dbConnect(drv, 
                    dbname = dsn_database,
                    host = dsn_hostname, 
                    port = dsn_port,
                    user = dsn_uid, 
                    password = dsn_pwd)
  print("Connected!")
},
error=function(cond) {
  print("Unable to connect to database.")
})
## save the layer to your database
dbWriteTable(conn, c("working", "testyballs3"), overwrite = TRUE, value = loc)


#save a RData file of work so far to avoid overdownloading
#save(df, data2, file = "my_data.RData")
save.image(file = "sar_work_space.RData") 


#load("sar_work_space.RData")

#####Make a csv 
#name <- paste0(genus,".",species1) #this was from before
name <- "SAR_locations"
file_name.csv <- paste0(name,format(Sys.time(),"_%Y%m%d_%H%M.csv"))
file_name.csv
write.csv(df,file_name.csv)




