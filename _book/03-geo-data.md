# Geo Data

This section is all about geographical data and visualization. 

## raster and vectors
This tutorial is by **RLadies Freiburg** Elisa Schneider. 
There are different ways to plot maps in R. The packages you use will depend on which your aim is and on the format of your data. Sometimes, geographical data can be simply in a **.txt or .csv file**, where you have one column for latitude and one for longitude and other variables or measurements corresponding to that location in the following columns. Yo can also have **shape files**, which are special files for geographical information. This files may contain points, lines or polygons. They are usually **.shp** files. Finally you can have **raster files** which are something similar to pictures or .png files because these files have pixels and (at least) a value associated with each pixel. The difference between a .png file and a shape file is that the shape-file has associated coordinates that locate this pixels on a specific surface of the world. The format is usually **.geotiff** or just **.tiff**.

In this meetup we will cover the basis of working with this formats. But this is only the starting point. There is a lot out there. 


### Simplest example

We can just use `geom_polygon` from the library `ggplot` to create our first figure. Let´s suppose we know the latitude and the longitude of all the nodes in our polygon. How can we plot it? 

```{}
require(ggplot2)
funny <- data.frame(lat = c(62, 55, 48, 48, 62), long = c(20,10.5,20,10,10)) # from top right 
#following the clock
ggplot(funny, aes(x = long, y = lat)) + # we specify the data
  geom_polygon(fill="green") + # we plot it
  geom_point(aes(x=13.25, y=50.5), color="violet", size=18) # we can also add points this was
```


### Information already availabe in R

The geometry of the polygons of the countries in the world are already loaded into R. You can access this info using the function `map_data` from the package `ggplot2`. 

How do we plot only one or a subset of countries?

We need to get the data for the geometry of the countries. To do that we use the library `maps` and `ggplot2`. 

```{}
countries <- c("Germany", "France")
# Get the data
some.maps <- map_data("world", region = countries) # This function retrieves the data. The data we get is a df with lat and long of each node. 
head(some.maps)
require(dplyr)
require(tidyr)
# Mean of lat and long to then write the labels 
lab.data <- some.maps %>%
  group_by(region) %>%
  summarise(long = mean(long), lat = mean(lat))
lab.data
```


**Themes in ggplot** In ggplot2 the display of non-data components is controlled by the theme system. The theme system of ggplot2 allows the manipulation of titles, labels, legends, grid lines and backgrounds. There are various build-in themes available that already have an all-around consistent style. A very common one for plotting data is `theme_bw()` and `theme_map()`. We use `theme_map()` which will give our plot the appearance of a map. To use `theme_map()` we need to load the package `ggthemes()`. 

```{}
require(ggthemes)
fra_and_ger <- ggplot(some.maps, aes(x = long, y = lat)) +
  geom_polygon(aes( group = group, fill = region))+ #plot polygon, use group to group all the nodes of the same country together. 
  geom_text(aes(label = region), data = lab.data,  size = 3, hjust = 0.5)
#using theme_map
fra_and_ger + #print labels
  theme_map() +
  theme(legend.position = "none")
#Using theme bw
fra_and_ger + #print labels
  theme_bw() +
  theme(legend.position = "none")+
  xlab("Longitude") + ylab("Latitude")
 
```


### the world 
From a couple of countries to the whole world: Plotting simple and nice maps of the world

`ggplot2` is becoming the standard for R graphs. However, it does not handle spatial data specifically. Handling spatial objects in R relies on Spatial classes defined in the package `sp` or `sf`. ggplot2 allows the use of spatial data through the package `sf`. `sf` elements can be added as layers in a graph. The combination of `ggplot2` and `sf` enables to create maps, using the grammar of graphics,but incorporation geographical info.

#### 1. We can just create a map from the world. 
We will do this using the library `ggplot2`. 

The function `ne_countries` also retrieves info about the countries. But now you do not get just nodes with coordinates but an `sf` object that contains much more info. 

```{}
require(rnaturalearth)
require(rnaturalearthdata)
require(sf)
world <- ne_countries(scale = "medium", returnclass = "sf") # this is another function to get polygons of countries. 
class(world)
dim(world)
head(world[, 1:5])
```

We can plot this geographical object using `ggplot2` and geometry `geom_sf`

```{} 
plot_w1 <- ggplot(data = world) +
    theme_bw()+ 
    geom_sf() + 
    xlab("Longitude") + ylab("Latitude") + 
    ggtitle("World map", subtitle = paste0("(", dim(world)[1], " countries)"))
plot_w1
```


#### 2. We can change the colors, add text ... 

```{}
plot_w1 +
   geom_sf(color="blue", fill="black") +
  theme(panel.background = element_rect(fill = "black"))+# Modify the theme to change the background 
  #Where was the funny polygon located?
  geom_polygon(data=funny, aes(x = long, y = lat), color="green", fill="green") + # we plot it
  # Add points using lat-lon information
  geom_point(aes(x=13.25, y=50.5), color="violet", size=2)
```

However our nice modern-art-style polygon looks different in the map compared to the first plot. **Any idea about what is happening?**

#### Coordinate reference systems (crs). 
Going from the rear world (the earth) to the simplifyed model (the map)


#### After choosing elipsoid and datum we have to project from 3D to 2D. 

You can get a feeling of how the Mercator projection distorts our worldview at [link](https://thetruesize.com).

Google and many apps use unprojected coordinates. When the coordinates are unprojected, they are in degrees and give the position on an sphere and not in a 2D surface. But, you still need an ellipsoid and datum to make clear which reference system you are using. All this information is coded in an **EPSG** code.
The most common coordinate reference system **crs** (used by Google and most apps) is EPGS: 4326. When you have lat Lon coordinates and no more info, it is likely that the EPSG is 4326, which uses the datum WGS84. 


To get a taste: 
[EPSG: 4326](https://epsg.io/map#srs=4326&x=0.000000&y=0.000000&z=1&layer=streets)
[Coordinate Systems Worldwide](https://epsg.io)

To find the one used in your region of interest: 

[Argentina](https://spatialreference.org/ref/?search=argentina)
[Germany](https://spatialreference.org/ref/?search=germany)
[Europe](https://spatialreference.org/ref/?search=europe)


## Make Choropleth Map

What is a *Chorophlet Map*? "A choropleth map is a thematic map in which areas are shaded or patterned in proportion to the measurement of the statistical variable being displayed on the map, such as population density or per-capita income."([Wikipedia](https://en.Wikipedia.org/wiki/Choropleth_map))

#### 1. Load Data

```{}
require(readr) # to use the function read_csv()
life.exp <- read_csv("data/LifeExp.csv")
```


The data was obtained from [link](https://www.kaggle.com/kumarajarshi/life-expectancy-who), lot of cool data-sets are available for free. 

#### 2. Tidy the data set 
have one column with country and another with life expectancy

```{}
life_exp <- life.exp %>%
  filter(year == 2016 & sex == "Both sexes") %>%  # Keep data for 2016 and for both sex
  dplyr::select(country, value) %>%                      # Select the two columns of interest
  rename(name = country, lifeExp = value) %>%   # Rename columns
  #We have a very common proble when using different sources, the name of the countries not allways match
  # Replace "United States of America" by USA in the region column
  mutate(
    name = ifelse(name == "United States of America", "United States", name), 
    name = ifelse(name == "Russian Federation", "Russia", name)
    )
```


#### 3. Get data of the polygons 
of each country and merge map and life expectancy data 

```{}
#attribute names
colnames(world)
life_exp2 <- left_join(world, life_exp, by = "name")
```


##### 4. Plot with ggpoligon

```{}
life_exp_map <- ggplot(data = life_exp2) +
    geom_sf(aes(fill = lifeExp )) +
    scale_fill_viridis_c(option = "plasma", trans = "sqrt") # this allows you to choose different colour scale
life_exp_map
```

Clearly we do not have information for all the countries regarding life expectancy and/or we have some problems with non-matching country names. When we call `left_join` we only keep in the table the countries that are in the left table. 

#### 5. Add points

We can also add points to our plot. For example we could add some capitals. Points where samples were obtained, etc. We can also control the size and color of the points using another variable. For example, we could plot sample points and the size would be number of observations. We could plot one point per city and the size number of inhabitants...and so son and so forth. 

```{}
country_capitals <- read_csv("data/country-capitals.csv")
south_america_capitals <- country_capitals %>% dplyr::filter(ContinentName == "South America")
south_america_capitals$CapitalLatitude <- as.numeric(south_america_capitals$CapitalLatitude)
require(ggrepel)
life_exp_map +
  geom_point(data=south_america_capitals, 
             aes(x=CapitalLongitude, y=CapitalLatitude),
             alpha=0.5)+
  geom_text_repel(data=south_america_capitals, # geom_text_repel avoidsoverlapping
            aes( x=CapitalLongitude, y=CapitalLatitude, label=CapitalName), 
            hjust=0, vjust=0, size= 3)+ 
  theme_bw()
  
```

#### 6. Add scale and north

```{}
require(ggspatial)
life_exp_map + 
  annotation_scale(location = "bl", width_hint = 0.5) + # scale
    annotation_north_arrow(location = "bl", which_north = "true", #north arrow
        pad_x = unit(0.75, "in"), pad_y = unit(0.5, "in"),
        style = north_arrow_fancy_orienteering) +
    coord_sf(xlim = c(-20.15, 50.12), ylim = c(35.65, -45)) #crop
```

## Rasters and polygons

Usually spatial data comes as a **raster** or as a **polygon** format. We usually have shape files with the onformation of sampling sites, neighborhoods, streets, rivers, hospitals, surveys, etc. saved as .shp files.
Or we have lat-Lon data that we can transform into an sf object as we did before. 
We can also get raster files such as climate raster files or remote sensing data, land use, etc. Most of the times we want to plot all this info together (raster + one or more shape-files). 

```{}
require(rgdal)
require(raster) #Package to work with raster files
require(sf) #Package to work with shape files
```


All this libraries allow us to work with spatial data using R. 

- `rgdal` is a library that that allows us to read and write geospatial data in R. The library just translates the already existing library `gdal` into R. The link to `gdal`project is [link](https://gdal.org/). 

- `raster` is the library that we use to work with raster formats. 

- `sf` encodes spatial vector data. We already used it. 

#### 2. Load the raster files and do some calculations

Let´s suppose he have different sampling points in Germany. We are interested in the temperature difference between summer and winter in this sites. Can we do a map to display this info? How would you do that?


#### This are some usefull links to find data:

Link to the [Federal Agency for Cartography](https://www.bkg.bund.de/DE/Home/home.html)
Link to [open BW data](https://www.lgl-bw.de/lgl-internet/opencms/de/07_Produkte_und_Dienstleistungen/Open_Data_Initiative/)
Link to [WoldClim](http://worldclim.org/version2)
link to [land use data](https://land.copernicus.eu/pan-european/corine-land-cover)
Another useful [link](https://www.eea.europa.eu)

All the data used in this example was download from this sites or directly from R. 

All the data used in this example was download from this sites or directly from R. 

```{}
t_july<-raster("data/wc2.0_5m_tavg_07.tif") # loads the raster
t_december<-raster("data/wc2.0_5m_tavg_01.tif") # loads the raster
# Get temperature range
t_diff <- abs(t_july - t_december)
plot(t_diff, main = "Temperature range")
```

Raster calculation is relatively straight forward using R if the raster have the same resolution. If the raster come from different sources you usually have to re-shape one raster. 

more info [here](https://rspatial.org/spatial/4-rasterdata.html)



#### 3. Load the shape files 

```{}
sites<-st_read("data/sample_sites.shp") # loads the vector data - sites
#get germany voundaries from world R data
germany <- world %>%  dplyr::filter (name == "Germany")
plot(germany)
```

As you can see here, each polling has associated data. Geometry has the info to build the polygon. *level*, *type*, etc. is the info related to each polygon. If the Polygons have different info in this fields, the polygons will be displayed in different colors. 

```{}
#get germany voundaries from world R data
gfs <- world %>%  dplyr::filter (name %in% c("Germany", "Austria"))
plot(gfs)
```



#### 4. Check reference system

We have data coming from different sources, we have to check that the coordinate reference system match (otherwise we will have lots of problems and we will be doing everything wrong)

```{}
# The function to get the coor. system is different between shapes and rasters
st_crs(germany)#get the projection
st_crs(sites)#get the projection
crs(t_diff)#get the projection
st_crs(germany)$proj4string == st_crs(sites)$proj4string
st_crs(germany)$proj4string == crs(t_diff, asText = T)
```


The data do not have the same coord. reference system. We have to transform one. 

```{}
sites_transformed <- st_transform(sites, crs = crs(t_diff, asText = T))#transform coordinate sistem
germany_transformed <- st_transform(germany, crs = crs(t_diff, asText = T))#transform coordinate 
```


#### 5. Crop the raster to the shape file extent

We get the extent (the coordinates of the extreme points) for our shape file of Germany. 

```{}
germany_extent <- extent(germany_transformed)
germany_extent
```


```{}
# crop the land use data by the extend of BW. Crop funciton from the raster package
crop_tdiff <- crop(t_diff, germany_extent, snap = "out")
plot(crop_tdiff)
```


In order to get a raster that matches exactly with the shape, we have to mask it. 
We could also do this step without cropping first, but that would be much more computational intensive. 

```{}
mask_tdiff <- mask(crop_tdiff, germany_transformed)
plot(mask_tdiff)
```


#### 6. Plot with `tmap`


The library tmap is another grate option to build thematic maps. I find it very useful to work with raster data. Useful info can be found in the [tmap vignette](https://cran.r-project.org/web/packages/tmap/vignettes/tmap-getstarted.html). Another library to plot raster data is `rasterVis`. We will not cover it today but you can check the [vignette](https://oscarperpinan.github.io/rastervis/) 
You can plot many layers together. Each needs some reference(point, raster, text, etc.)


```{}
require(tmap)
tmap_mode("plot") # static plot or leaflet. 
tm_shape(mask_tdiff) + 
  tm_raster (col= "layer" , style = "cont", n=10,  title = "T. Difference") +
tm_shape(germany_transformed) + #add ploygon of germany
  tm_borders() +
tm_shape(sites_transformed) +
   tm_symbols(col = "gray", scale = .5)+ ##add points
      tm_text("Site", size = 0.8,  ymod=-0.6, root=1, size.lowerbound = .60, #add text
        bg.color="gray", bg.alpha = .5) +
tm_layout(inner.margins = c(0.15, 0.30, 0.10, 0.1)) + # crop the extent of the map
tm_layout(legend.position = c("left","bottom")) + # add legent
    tm_compass() + # add north
  tm_scale_bar() #add scale
```


```{}
tm <- tm_shape(mask_tdiff) + 
  tm_raster (col= "layer" , style = "cont", n=10,  title = "T. Difference") +
tm_shape(germany_transformed) + #add ploygon of germany
  tm_borders() +
tm_shape(sites_transformed) +
   tm_symbols(col = "gray", scale = .5)+ ##add points
      tm_text("Site", size = 0.8,  ymod=-0.6, root=1, size.lowerbound = .60, #add text
        bg.color="gray", bg.alpha = .5) +
tm_layout(inner.margins = c(0.15, 0.30, 0.10, 0.1)) + # crop the extent of the map
tm_layout(legend.position = c("left","bottom")) + # add legent
    tm_compass() + # add north
  tm_scale_bar() #add scale
tmap_save(tm, filename = "t_range.png")
```


With tmap is really easy to do interactive maps! 

```{}
require(tmap)
tmap_mode("view")
tm_shape(mask_tdiff) +
  tm_raster (col= "layer" , style = "cont", n=10,  title = "T. Difference") +
tm_shape(germany_transformed) +
  tm_borders() +
tm_shape(sites_transformed) +
   tm_symbols(col = "gray", scale = .5)+
      tm_text("Site", size = 0.8,  ymod=-0.6, root=1, size.lowerbound = .60, 
        bg.color="gray", bg.alpha = .5) +
tm_layout(inner.margins = c(0.1, 0.30, 0.10, 0.1)) +
tm_layout(legend.position = c("left","bottom"))
##Reclasify values in the rasta flie. Reclasify function of raster file function
```


`tmap` is a great library to make maps. Check the [vignette](https://cran.r-project.org/web/packages/tmap/vignettes/tmap-getstarted.html)































