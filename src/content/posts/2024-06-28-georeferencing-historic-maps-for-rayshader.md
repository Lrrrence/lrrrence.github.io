---
title: 'Georeferencing historic maps for Rayshader'
date: 2024-06-28
lastmod: 2024-06-28
tags: ['geospatial', 'maps', 'rayshader']
categories: ['Geospatial']
---
In a recent post I explained how beautiful shadows to could be applied to old historic maps by combining elevation data with the wonderful Rayshader package. In that example the map I used came conveniently pre-georeferenced, but what if you need to do it yourself? In this post I will explain how this can be done with QGIS. 

* * *

Let's start with the map itself. I found [this gorgeous map of Mauritius](<https://www.davidrumsey.com/luna/servlet/detail/RUMSEY~8~1~360082~90127192:To-Lieut--Genl--the-honble--Sir-Cha?qvq=q:pub_list_no%3D%2214506.000%22;lc:RUMSEY~8~1&mi=10&trs=132>) via David Rumsey's Map Collection, created in 1835 by John Arrowsmith in 1:115,000 scale. This scan has a resolution of 9794x13454, so all the details are clearly visible. The file type is highly compressed .jp2 (19 MB).

![](/assets/img/wp-uploads/2024/06/og_map.png)

Let's get this into QGIS and see what we're working with. To carry out the georeferencing follow the tutorial on YouTube here:

https://www.youtube.com/watch?v=XV62QEk0Cxg 

There's a trade off here - we want an accurate match between modern and historical coordinates, but without significantly distorting the original map. [Taking a look at the QGIS documentation](<https://docs.qgis.org/3.34/en/docs/user_manual/working_with_raster/georeferencer.html> "Taking a look at the QGIS documentation"), the **Thin Plate Spline** (TPS) algorithm is technically the best for this scenario as our map has no known CRS, and is inaccurate by today's standards, but this will almost certainly distort your map. It might be better to sacrifice accuracy and use the **Linear** algorithm, just don't look too closely at the final render... 

If you're going to use **TPS** , focus on setting ground control points (GCPs) around the coast of the island, as this is where the discrepancies will be the most apparent on our final map. I also used major road junctions, and mountain peaks. 

Now we can crop down our map to remove the other pages etc. Do this by going to `Raster>Extraction>Clip Raster by Extent...` and drawing a suitable rectangle. You could also create a polygon and then use `Raster>Extraction>Clip Raster by Mask Layer...`, but make sure to set _"Assign a specified nodata value..."_ to zero.

Now let's export this as a GeoTiff and import it into R.

### Building a map in R

Let's load the necessary packages and set the working directory to our current folder:
    
    
    library(rstudioapi) # get working dir
    library(terra) #working with rasters
    library(tidyterra) #plotting terra objects
    library(tidyverse) #utilities
    library(sf) #vectors
    library(elevatr) #elevation data
    library(rayshader)
    library(viridis) # colours
    
    # set working directory
    current_file <- rstudioapi::getActiveDocumentContext()$path
    current_dir <- dirname(current_file)
    setwd(current_dir)
    cat("Current working directory set to:", getwd(), "\n")

Now let's load our map and check that it's georeferenced:
    
    
    topo_map <- rast("your_map_here.tif")
    cat(crs(topo_map))
    
    # get original crs
    og_crs <- crs(topo_map)
    
    # plot
    p <- ggplot()+
          geom_spatraster_rgb(data = topo_map)
    p

![](/assets/img/wp-uploads/2024/06/PTmgpo1Bgm.png)

Looking good! Now let's use the extent of the map to get the corresponding elevation data using `elevatr` and plot it:
    
    
    # get extent
    e <- ext(topo_map)
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
                         z = 10, prj = "EPSG:4326", clip = "locations")
    elev <- rast(elev) # to terra
    
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

![](/assets/img/wp-uploads/2024/06/xxWu7XlsFP.png)

This is good but it would be nice if we could limit the elevation data to only the island itself. Let's get a shapefile of the island and mask the elevation, then combine that with a base layer that sets the elevation around the island to zero.
    
    
    library(geodata)
    # Download GADM data for Mauritius
    mauritius <- gadm(country = "Mauritius", level = 0, path = tempdir())
    # Convert to sf object
    mauritius <- st_as_sf(mauritius)
    # Crop the sf object to the bounding box
    mauritius_cropped <- st_crop(mauritius, bbox_sf)
    plot(st_geometry(mauritius_cropped))
    # crop/mask elevation to extent
    elev_crop <- terra::crop(elev,mauritius_cropped)
    elev_masked <- mask(elev_crop, mauritius_cropped)
    # Plot the masked raster
    plot(elev_masked)
    # create base raster of map extent
    base_raster <- elev * 0
    # merge base with mask
    elev <- merge(elev_masked,base_raster)

![](/assets/img/wp-uploads/2024/06/N2e5MuiN3n.png)

Much better. Now let's plot an interactive map so that we can extract the coordinates of the map's neatline. This way we can create a nice step around it.
    
    
    library(leaflet)
    library(leafem)
    
    topo_map_agg <- terra::aggregate(topo_map, fact=8, fun=mean) # reduce size
    
    # Create the leaflet map with mouse coordinates
    m <- leaflet() %>%
      addProviderTiles(providers$OpenStreetMap) %>%
      addRasterImage(topo_map_agg, opacity = 0.8, project = TRUE) %>%
      addMouseCoordinates()
    m
    # ctrl + Lclick to copy coordinates to clipboard
    
    library(htmlwidgets)
    saveWidget(m, file="leaflet.html")
    
    #TL-TR-BR-BL
    # " lon: 57.29144 | lat: -19.92213 | zoom: 17 "
    # " lon: 57.83505 | lat: -19.92102 | zoom: 18 "
    # " lon: 57.83283 | lat: -20.54365 | zoom: 17 "
    # " lon: 57.29144 | lat: -20.54527 | zoom: 18 "

Now let's crop to those points and plot the result to make sure everything lines up:
    
    
    elevation_range <- minmax(elev)
    min_elevation <- elevation_range
    
    base_raster <- elev * 0 + (min_elevation-200)
    
    # use coords from leaflet map
    x <- c(57.29144, 57.83505, 57.83283, 57.29144)
    y <- c(-19.92213, -19.92102, -20.54365, -20.54527)
    coordinates <- data.frame(x = x, y = y)
    
    # Convert the data frame to an sf object with the initial CRS (WGS84)
    sf_points <- st_as_sf(coordinates, coords = c("x", "y"), crs = 4326)
    
    #polygon for plot
    sf_polygon <- sf_points %>%
      st_union() %>%
      st_cast("POLYGON")
    
    # plot polygon overlay
    p <- ggplot()+
      geom_spatraster_rgb(data = topo_map)+
      geom_sf(data = sf_polygon, fill = "blue", alpha = 0.5)
    p
    #ggsave(filename = "polygon_overlay.png", plot = p, width = 12, height = 8, dpi = 300)
    
    # crop elevation to extent of points
    interior_elevation <- terra::crop(elev, extent(sf_points))
    elev_crop <- merge(interior_elevation, base_raster)
    
    p <- ggplot()+
      geom_spatraster(data = elev_crop) +
      scale_fill_viridis() # Use Viridis color palette
    p

![](/assets/img/wp-uploads/2024/06/kr70Y5qnMk.png) ![](/assets/img/wp-uploads/2024/06/ZqluuaDtDU.png)

Great, everything looks good, so let's get to rayshading...
    
    
    # reduce res
    topo_map_final <- aggregate(topo_map, fact=3, fun=mean)
    dim(topo_map_final) # y*x
    
    # breakout RGB layers
    names(topo_map_final) <- c("r", "g", "b")
    topo_r <- rayshader::raster_to_matrix(topo_map_final$r)
    topo_g <- rayshader::raster_to_matrix(topo_map_final$g)
    topo_b <- rayshader::raster_to_matrix(topo_map_final$b)
    topo_rgb_array <- array(0, dim = c(nrow(topo_r), ncol(topo_r), 3))
    topo_rgb_array <- topo_r/255
    topo_rgb_array <- topo_g/255
    topo_rgb_array <- topo_b/255
    topo_rgb_array <- aperm(topo_rgb_array, c(2,1,3))
    
    dims <- dim(topo_rgb_array)
    width = dims
    height = dims
    
    #set scale of z
    zscale = 12
    
    # rayshade
    elev_mat <- raster_to_matrix(elev_crop)
    elev_mat = resize_matrix(elev_mat, scale=0.6)
    #ray_shadow <- ray_shade(elev_mat, sunaltitude = 30, zscale = zscale, multicore = TRUE)
    #ambient_shadow <- ambient_shade(elev_mat, zscale = zscale)
    
    elev_mat %>%
      sphere_shade() %>%
      add_overlay(topo_rgb_array, rescale_original = TRUE) %>%
      #add_shadow(ray_shadow, max_darken = 0.5) %>%
      #add_shadow(ambient_shadow, 0.5) %>%
      plot_3d(heightmap = elev_mat, 
              zscale = zscale,
              solid = FALSE,
              water = FALSE, 
              fov = 0, 
              theta = 0, 
              zoom = 0.75, 
              phi = 90,
              windowsize = 1200,
              background = "white")
    
    render_camera(theta=0,phi=89,fov=60,zoom=0.6)
    
    environment_light <- "your_hdr_file_here.hdr"
    
    render_highquality(
      "mauritius_highquality.png", 
      parallel = TRUE, 
      sample_method = "sobol",
      samples = 250,
      light = FALSE, 
      interactive = FALSE,
      environment_light = environment_light,
      intensity_env = 1,
      rotate_env = 90,
      width = width, 
      height = height
    )
    

![](/assets/img/wp-uploads/2024/06/mauritius_highquality_v4.png)
