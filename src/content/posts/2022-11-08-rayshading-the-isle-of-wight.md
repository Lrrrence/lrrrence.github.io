---
title: 'Rayshading the Isle of Wight'
date: 2022-11-08
lastmod: 2022-11-09
categories: ['Geospatial']
---
A few years ago I came across the wonderful [@geospatialist](<https://twitter.com/geo_spatialist>) on twitter, and I was blown away by his vintage relief maps. For a long time I wondered how I could replicate these myself, specifically for my beloved Isle of Wight. This post is going to be a (hopefully) helpful introduction to pulling this off, coming from a total beginner in the geospatial world. Thankfully there are already countless other tutorials and guides out there that can explain the intricacies of this better than me, so I'll be including lots of links to help you along. Let's get started.

I should first point out that [@geospatialist](<https://twitter.com/geo_spatialist>) uses Blender to create his maps, which is a different method than I am going to describe here. For a tutorial on that, see [this comprehensive guide by Daniel Huffman](<https://somethingaboutmaps.wordpress.com/2017/11/16/creating-shaded-relief-in-blender/>).

I have used a combination of [QGIS](<https://www.qgis.org/en/site/>) and [R](<https://cran.r-project.org/bin/windows/base/>) \+ [RStudio](<https://www.rstudio.com/products/rstudio/>) to produce my map. Specifically using the awesome [`rayshader`](<https://www.rayshader.com>) package by Tyler Morgan-Wall, which is going to take elevation data and produce 2D/3D maps with raytraced/hillshaded shadows. I then overlayed an old map on to this to complete the scene, but you can also overlay a satellite image. 

#### Elevation data

The first thing we need is some elevation data in the form of a Digital Elevation Model (DEM). There are (what seems like) an infinite number of sources for this, and it can be difficult to know which is the most appropriate for you. From within R there are a number of packages that can be used to pull data in programmatically, such as `[geowiz](<https://github.com/neilcharles/geoviz/>)`, which pulls from [Mapzen DEM](<https://sentinelshare.page.link/wDL1>). The problem with using this method (for this case) is that we can't then edit the data to remove any ugly boundaries etc. and we can't align our nice map (more on that later) with the DEM. We're going to find and download our DEM manually and then tweak it in QGIS before using it in R.

I used the [DEFRA survey data map](<https://environment.data.gov.uk/DefraDataDownload/?Mode=survey>) to draw a bounding box around the Isle of Wight and then download the LIDAR Composite DTM 1M data. This is going to give you a number of tiles which we'll combine using QGIS. The resolution of this is overkill, but it's nice to have. 

Another nice element to add in is bathymetry data (where appropriate), although combining it with the elevation data can be a little tricky. I used the [EMODnet Bathymetry Viewing and Download service](<https://portal.emodnet-bathymetry.eu>) to select the area that I was interested in and download the RGB GeoTiff.

#### QGIS

I used QGIS to combine my DTM tiles with bathymetry data, crop down the data set to the extent of the map that I planned to overlay, and georeference my overlay so that the elevation data would correctly align with it. I won't go into the detail as what you need to do here will be quite specific to your dataset, but be safe in the knowledge that the answers are only a Google search away...

#### Map overlays

Now to find a map to overlay over the elevation data. It can be quite difficult to find suitable options, but one great source (for the UK) is the [British Geological Survey](<https://webapps.bgs.ac.uk/data/maps/home.html#QuickLinks>). There you can find old maps with the aesthetic that fits this kind of thing nicely. Another nice method is to use the [National Library of Scotland's map finder](<https://maps.nls.uk/geo/explore/>) to compare different map series. A slightly more time consuming method is to search [The National Archives](<https://discovery.nationalarchives.gov.uk>) for maps of your area of interest, but you'll find that a lot of them haven't been digitised. For this post I used the Geological Survey of England and Wales 1:63,360/1:50,000 geological map series.

#### Using rayshader

For getting started I suggest checking out [this great guide by Will Bishop](<https://wcmbishop.github.io/rayshader-demo/>). Also check out [these](<https://github.com/Pecners/rayshader_portraits>) great uses of `rayshader` by [Spencer Schien](<https://spencerschien.info/#about>), who also provides excellent documentation for reproducing them. For anyone on Twitter, consider following [flotsam](<https://twitter.com/researchremora>), who has a really great catalog of renders.

Below you can find the R code I used to generate the map. The "raster.tif" is the vintage map to be overlayed, while "elevation_data.tif" is (obviously) the elevation data. The coordinates for cropping the elevation data can be found by mousing over the four corners of the map in QGIS. Adjust the numeric value of `resize_matrix` to reduce the size of the matrix while you're tinkering with it.
    
    
    library(sp)
    library(raster)
    library(scales)
    library(sf)
    library(ggplot2)
    library(dplyr)
    library(imager)
    #load satellite data
    iow_1965r = raster::raster("raster.tif", band = 1)
    iow_1965g = raster::raster("raster.tif", band = 2)
    iow_1965b = raster::raster("raster.tif", band = 3)
    #stack RGB layers
    RGB_stack <- stack(iow_1965r,iow_1965g,iow_1965b)
    raster::plotRGB(RGB_stack)
    #show RGB CRS
    crs(iow_1965r)
    extent(RGB_stack)
    dim(RGB_stack)
    #load elevation data
    iow_elevation = raster::raster("elevation_data.tif")
    crs(iow_elevation)
    plot(iow_elevation)
    ## crop elevation to the full map extent (past neatline)
    elevation_fullcrop <- raster::crop(iow_elevation, extent(iow_1965r))
    plot(elevation_fullcrop)
    dim(elevation_fullcrop)
    ##this raster will help knockdown the elevation outside the
    ## neatline in the physical map
    base_raster <- elevation_fullcrop * 0 - 200
    plot(base_raster)
    #coordinates for cropping TL-TR-BR-BL
    x <- c(428417.735, 466743.112, 466844.374, 428247.855)
    y <- c(100111.466, 100026.692, 74368.682, 74378.837)
    xy <- cbind(x,y)
    S <- SpatialPoints(xy)
    #convert from points to spatial point data frame
    xy2 = as.data.frame(cbind(x,y))
    spdf <- SpatialPointsDataFrame(S, xy2)
    (Sr1 = Polygon(spdf))
    (Srs1 = Polygons(list(Sr1), "s1"))
    (SpP = SpatialPolygons(list(Srs1), 1:1))
    #plot(SpP, col = 3:3, pbg="white", add=T)
    SpP ### can not write as shapefile
    ### Convert the SpatialPolygons to SpatialPolygonsDataFrame
    shape_pol <- SpatialPolygonsDataFrame(SpP, match.ID=F, data= data.frame(x=spdf, y=spdf))
    shape_pol ### can be writen as shapefile
    #plot(shape_pol, col = 4, add=T)
    #make mask and discard remaining data
    mask =mask(elevation_fullcrop,shape_pol)
    mask =trim(mask, values = NA)
    extent(mask)
    plot(mask)
    #merge base with crop
    elevation <- merge(mask, base_raster)
    plot(elevation)
    extent(elevation)
    res(elevation)
    #raster to matrix for RGB
    iow_1965_matrix_r = rayshader::raster_to_matrix(iow_1965r)
    iow_1965_matrix_g = rayshader::raster_to_matrix(iow_1965g)
    iow_1965_matrix_b = rayshader::raster_to_matrix(iow_1965b)
    #build array fro matrix
    iow_rgb_array = array(0,dim=c(nrow(iow_1965_matrix_r),ncol(iow_1965_matrix_r),3))
    #convert each layer
    iow_rgb_array = iow_1965_matrix_r/255 #Red layer
    iow_rgb_array = iow_1965_matrix_g/255 #Blue layer
    iow_rgb_array = iow_1965_matrix_b/255 #Green layer
    #transpose array
    iow_rgb_array = aperm(iow_rgb_array, c(2,1,3))
    #elevation data to matrix
    iow_elevation_matrix = rayshader::raster_to_matrix(elevation)
    #resize to speed up render
    elev_mat = resize_matrix(iow_elevation_matrix,0.5)
    #rayshade
    ray_shadow      <- ray_shade(elev_mat, sunaltitude = 40, zscale = 1, multicore = TRUE)
    ambient_shadow  <- ambient_shade(elev_mat, zscale = 1, multicore = TRUE)
    lamb_shadow     <- lamb_shade(elev_mat, zscale=50, sunaltitude = 40)
    elev_mat %>%
      sphere_shade() %>%
      add_overlay(iow_rgb_array, rescale_original=TRUE) %>%
      add_shadow(ray_shadow, max_darken = 0.3) %>%
      add_shadow(ambient_shadow, max_darken = 0.3) %>%
      add_shadow(lamb_shadow, max_darken = 0.4) %>%
      #save_png("IOW1965_map.png")
      plot_map()

![](/assets/img/wp-uploads/2022/11/IOW1965A1aspect.jpg)The Isle of Wight - Rayshaded
