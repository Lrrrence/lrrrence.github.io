---
title: 'Adding historic map overlays to elevation data with Rayshader'
date: 2024-06-22
lastmod: 2024-06-22
categories: ['Geospatial']
---
Having [dabbled with Rayshader in the past](<https://lawrenceyule.com/2022/11/08/rayshading-the-isle-of-wight/> "Rayshading the Isle of Wight") I thought now might be a good time to follow-up on that post with a (fairly) succinct tutorial on how to overlay historic maps on elevation data. This is heavily inspired by the work of [Michael Schramm](<https://michaelpaulschramm.com/posts/2020-10-08-rayshading_maps/>), but I took a slightly different approach to certain aspects. Let's get started...

* * *

Firstly, we need a beautiful map to use as an overlay. A fantastic array of these (for the US) can be found in the [USGS Historical Topographic Map Collection](<https://ngmdb.usgs.gov/topoview/viewer/#4/40.01/-99.93>). You can easily download georeferenced images (as GeoTiffs) here. Some other great options are [oldmapsonline.org](<https://www.oldmapsonline.org>) or the [David Rumsey Map Collection](<https://www.davidrumsey.com/view/georeferenced-maps>).

In this example we'll work with a map of [Ship Rock, New Mexico](<https://ngmdb.usgs.gov/topoview/viewer/#11/36.6632/-108.8680>) from 1934, in 1:62500 scale, referenced to the North American Datum of 1927 (NAD 27).

Let's to get this map into R, download the associated elevation data, and start Rayshading! To begin our code, let's load the necessary packages and set the working directory to our current folder:
    
    
```r
library(rstudioapi) # get working dir
library(terra) #working with rasters
library(tidyterra) #plotting terra objects
library(tidyverse) #utilities
library(sf) #vectors
library(elevatr) #elevation data
library(viridis) # colours
library(rayshader)

# set working directory
current_file <- rstudioapi::getActiveDocumentContext()$path
current_dir <- dirname(current_file)
setwd(current_dir)
cat("Current working directory set to:", getwd(), "\n")
```

Now let's pull in our GeoTiff, check/get the CRS, and plot it to make sure we've got the right map:
    
    
```r
# Load the GeoTIFF file
topo_map <- rast("your_file_here.tif")
cat(crs(topo_map))

# get original crs
og_crs <- crs(topo_map)

# plot
p <- ggplot()+
      geom_spatraster_rgb(data = topo_map)
p
```

![](/assets/img/wp-uploads/2024/06/original-topo.png)

Now we need to reproject our raster to EPSG:4326 (WGS 84) so that our coordinates are represented as Lat/Long, as this makes manipulating the data much easier. Then we'll get the extent, create a polygon, get the coordinates of that polygon, create a bounding box, and finally use that to get the elevation data corresponding to our map. This is achieved through the fantastic `[Elevatr](<https://github.com/jhollist/elevatr>)` package.
    
    
```r
# reproject to WGS84 (lat/long)
topo_map_latlong <- terra::project(topo_map, "EPSG:4326")
# get extent
e <- ext(topo_map_latlong)
# make polygon from extent
p <- as.polygons(e)
# get coords
c <- crds(p)
# convert to df
c_df <- data.frame(lon = c, lat = c)
# create bbox sf object
bbox_sf <- st_as_sf(c_df, coords = c("lon", "lat"), crs = 4326)

# use elevatr to get elevation data
elev <- get_elev_raster(locations = bbox_sf,
                     z = 12, prj = "EPSG:4326", clip = "locations")
elev <- rast(elev) # to terra
```

Now let's convert the retrieved raster to a `terra SpatRaster` and plot it with some contour lines to visualise the elevation.
    
    
```r
#plot
p <-  ggplot() +
  geom_spatraster_contour_filled(
    data = elev,
    breaks = seq(minmax(elev),minmax(elev),50),
    alpha = 1
  ) +
  geom_spatraster_contour(
    data = elev,
    breaks = seq(minmax(elev),minmax(elev),50),
    color = "grey30",
    linewidth = 0.1
  ) +
  scale_fill_hypso_d()+
  theme(panel.background = element_rect(fill = "white", color = NA),
        plot.background = element_rect(fill = "white", color = NA),)
p
ggsave(filename = "elev_contours.png", plot = p, width = 12, height = 8, dpi = 300)
```

![](/assets/img/wp-uploads/2024/06/elev_contours.png)

Now we need to consider how to limit our elevation data to the area shown on our map, so that it doesn't extend past the neatlines. This isn't strictly necessary, as leaving that elevation in can also make for an interesting effect, but let's do it anyway. We can do this by finding the coordinates of the neatline area and then cropping our elevation data to it. A nice way of doing this is to plot an interactive map where the coordinates at our mouse cursor are shown, so we can easily copy them.
    
    
```r
library(leaflet)
library(leafem)

topo_map_agg <- terra::aggregate(topo_map, fact=4, fun=mean) # reduce size

m <- leaflet() %>%
  addProviderTiles(providers$OpenStreetMap) %>%
  addRasterImage(topo_map_agg, opacity = 0.8, project = TRUE) %>%
  addMouseCoordinates()
m
# ctrl + Lclick to copy coordinates to clipboard

library(htmlwidgets)
saveWidget(m, file="m.html")
```

You may need to reduce the size of your raster in order to plot it efficiently.

Now let's use the coordinates we've found to crop our raster, set the elevation difference between the two parts, and plot the result to check everything lines up.
    
    
```r
# crop to neatlines

elevation_range <- minmax(elev)
min_elevation <- elevation_range

base_raster <- elev * 0 + (min_elevation-150)

# use coords from leaflet map
x <- c(-109.04536, -109.04607, -108.75043, -108.74999)
y <- c(36.50004, 36.75007, 36.49986, 36.75008)
coordinates <- data.frame(x = x, y = y)

# Convert the data frame to an sf object with the initial CRS (WGS84)
sf_points <- st_as_sf(coordinates, coords = c("x", "y"), crs = 4326)

#polygon for plot
sf_polygon <- sf_points %>%
  st_union() %>%
  st_cast("POLYGON")

# plot polygon overlay
ggplot()+
  geom_spatraster_rgb(data = topo_map)+
  geom_sf(data = sf_polygon, fill = "blue", alpha = 0.5)

# crop elevation to extent of points
interior_elevation <- terra::crop(elev, extent(sf_points))
elev_crop <- merge(interior_elevation, base_raster)

ggplot()+
  geom_spatraster(data = elev_crop) +
  scale_fill_viridis() # Use Viridis color palette
```

![](/assets/img/wp-uploads/2024/06/polygon_overlay.png)

![](/assets/img/wp-uploads/2024/06/elevation_crop-1024x683.png)

Now let's reproject the elevation raster to the CRS of the original map (NAD27).
    
    
```r
# reproject
elev_crop <- terra::project(elev_crop, og_crs)
cat(crs(elev_crop))
```

Now we need to prepare the RGB layers of our overlay by converting them into an array. We'll use the original map rather than reprojecting (again) the version we used to get the elevation data, etc. We can also get the dimensions of this array to define the final resolution of our image.
    
    
```r
# reduce res
topo_map <- aggregate(topo_map, fact=2, fun=mean)
dim(topo_map) # y*x

# breakout RGB layers
names(topo_map) <- c("r", "g", "b")
topo_r <- rayshader::raster_to_matrix(topo_map$r)
topo_g <- rayshader::raster_to_matrix(topo_map$g)
topo_b <- rayshader::raster_to_matrix(topo_map$b)
topo_rgb_array <- array(0, dim = c(nrow(topo_r), ncol(topo_r), 3))
topo_rgb_array <- topo_r/255
topo_rgb_array <- topo_g/255
topo_rgb_array <- topo_b/255
topo_rgb_array <- aperm(topo_rgb_array, c(2,1,3))

dims <- dim(topo_rgb_array)
width = dims
height = dims
```

Now it's time to combine our layers into a beautiful map with [Rayshader](<https://www.rayshader.com/index.html>). Set the zscale to control how exaggerated the elevation is (smaller number = more dramatic). We set `rescale_original = TRUE` to ensure that the original resolution of the overlay is maintained.
    
    
```r
#set scale of z
zscale = 8

# rayshade
elev_mat <- raster_to_matrix(elev_crop)
elev_mat = resize_matrix(elev_mat, scale=1)
ray_shadow <- ray_shade(elev_mat, sunaltitude = 30, zscale = zscale, multicore = TRUE)
ambient_shadow <- ambient_shade(elev_mat, zscale = zscale)

elev_mat %>%
  sphere_shade() %>%
  add_overlay(topo_rgb_array, rescale_original = TRUE) %>%
  add_shadow(ray_shadow, max_darken = 0.3) %>%
  add_shadow(ambient_shadow, 0.3) %>%
  plot_3d(heightmap = elev_mat, 
          zscale = zscale,
          solid = FALSE,
          water = FALSE, 
          fov = 0, 
          theta = 0, 
          zoom = 0.75, 
          phi = 90,
          background = "white")

render_camera(theta=0,phi=90,fov=60,zoom=0.6)

render_snapshot("snapshot.png",
                software_render = TRUE,
                width = width,
                height = height)
```

![](/assets/img/wp-uploads/2024/06/shiprock_snapshot_v2.png)

This is nice, but we can take it a step further with `[render_highquality](<https://www.rayshader.com/reference/render_highquality.html>)`. Start by disabling the `add_shadow` elements. You can use the built-in light sources, but you can get a more interesting effect by using a HDR scene, like [these from Poly Haven](<https://polyhaven.com/a/cape_hill>). If you want long, dramatic, shadows then go for sunrise/sunsets. Control how bright the scene is with `intesity_env`. This will take awhile to render so it's a good idea to do some tests with a reduced resolution, using a combination of `rayshader::resize_matrix` and `terra::aggregate`.
    
    
```r
environment_light <- "your_HDR_file.hdr"

render_highquality(
  "shiprock_highquality.png", 
  parallel = TRUE, 
  sample_method = "sobol",
  samples = 250,
  light = FALSE, 
  interactive = FALSE,
  environment_light = environment_light,
  intensity_env = 0.7,
  rotate_env = 90,
  width = width, 
  height = height
)
```

![](/assets/img/wp-uploads/2024/06/shiprock_highquality_v3.png)

Full code available on [GitHub](<https://github.com/Lrrrence/Rayshader-map-overlays>).
