---
title: 'A "Joy plot" style map of the Isle of Wight'
date: 2022-10-03
lastmod: 2022-10-10
tags: ['geospatial', 'maps', 'qgis', 'R']
categories: ['Geospatial']
---
This is a relatively shameless copy of the wonderful [Helen McKenzie tutorial](<https://www.helenmakesmaps.com/post/how-to-joy-plot>) on how to produce "joy plot" style maps, which is in turn a simplified version of [Travis White's tutorial](<https://cartographicperspectives.org/index.php/journal/article/view/1536/1725>). I've included a little more detail on some of the finer points of the process in areas that tripped me up a bit. I'm going to apply this style to my beloved Isle of Wight, an island just off the south coast of England. We'll be using a combination of QGIS and R to create the map, with Adobe Illustrator to add the finishing touches. 

* * *

##### Elevation data

There are a huge number of sources for this, if you're considering a small area it might be worth looking for some LiDAR data, but that resolution (1-2 m) isn't necessary for this. I went with SRTM30m.

  * <https://earthexplorer.usgs.gov>
  * <https://apps.sentinel-hub.com/eo-browser>
  * <https://dwtkns.com/srtm30m/>
  * UK LiDAR: <https://environment.data.gov.uk/DefraDataDownload/?Mode=survey>

![](/assets/img/wp-uploads/2022/10/Vgq5DUethQ.png)SRTM30m downloader

##### QGIS

  * Import into [QGIS](<https://qgis.org/en/site/>).
    * We need to set the [CRS](<https://docs.qgis.org/2.8/en/docs/gentle_gis_introduction/coordinate_reference_systems.html>) of both our project and our raster layer. I like to use OSGB 1936 (EPSG:7405) for the UK as it gives less of a squished (technical term) perspective in comparison to the commonly used WGS 84 (EPSG:4326).
    * Set the project CRS in **Project>Properties...>CRS**
    * QGIS will convert a layer CRS to the project CRS on-the-fly if we double-click the layer and assign a CRS in "Source", but we need to permanently reproject the layer.
    * Use the **Warp (reproject)** tool from the processing toolbox to set a target CRS of OSGB 1936 (EPSG:7405) and run. 
    * Now you have the layer reprojected the units should be given in meters, which will be useful when creating a grid.
  * Create a polygon and crop the raster to that extent.
    * Create a new shapefile layer and draw a polygon (Add polygon feature). You don't have to be super precise with this as we can set the minimum height at which a line is drawn in R. You can check the elevation at your mouse pointer with the [value tool](<https://plugins.qgis.org/plugins/valuetool/>). 
    * Use **Raster>Extraction>Clip Raster by mask layer...** to crop your raster down to the shape of the polygon. Make sure to finish editing the polygon before trying to clip the raster or the process will throw an error. 
  * Create a grid and assign elevation data to lines.
    * Create a grid using **Vector>Research tools>Create grid**. Try spacings of around 200 m.
    * Clip the lines to the raster layer (**Vector>Geoprocessing Tools>Clip**).
    * Delete the vertical lines.
    * Run the tool "**Generate Points (Pixel Centroids) Along Line** " from the processing toolbox.
    * Install the point sampling tool. Plugins > Manage and Install Plugins... > search for **"Point Sampling Tool"**. Go to Plugins > Analyses and open the tool, now select "Points along lines: id (source point)" and your raster layer by ctrl+click. Select where to save the output and run the tool.
    * Assign elevation data from the raster to each of the points using **"Add X/Y fields to layer"** from the processing toolbox.
    * Export as .csv by right-clicking the layer and using **Export>Save As...**

![](/assets/img/wp-uploads/2022/10/vuUVgKk0A2.png)A cropped Isle of Wight in QGIS

##### RStudio

You'll need to install the packages `ggplot2`, `ggridges`, and `dplyr` for this. Play around with the `scale` value to exaggerate the elevation. Set the `rel_min_height` based on the height of your water level, so that only the land is drawn.
    
    
    library(ggplot2)
    library(ggridges)
    library(dplyr)
    
    transects <- read.csv("data.csv")
    
    head(transects)
    
    names(transects)<-"ID"
    names(transects)<-"Elevation"
    names(transects)<-"Lon"
    names(transects)<-"Lat"
    
    joy <- ggplot(transects,
                  aes(x = Lon, y = Lat, group =Lat, height = Elevation))+
      geom_density_ridges(stat = "identity",
                          scale = 25,
                          fill = "black",
                          color = "white",
                          rel_min_height = 0.005)+
      theme(panel.grid.major = element_blank(),
            panel.grid.minor = element_blank(),
            panel.background = element_rect(fill="black"),
            axis.line = element_blank(),
            axis.text.x = element_blank(),
            plot.background = element_rect("black"),
            axis.title.x = element_blank(),
            axis.text.y = element_blank(),
            axis.title.y = element_blank())
    joy
    
    ggsave("joy_iow.pdf", width = 1920, height = 1080, units = "px")
    

##### Final touches

Import into your choice of image editor to add a border or a title. I went with [Trajan Pro](<https://fontsgeek.com/fonts/Trajan-Pro-Regular>) for the font, as used on Joy Division's "Closer" cover. I think it's a bit more interesting than Helvetica, as used on "Unknown Pleasures".

[![](/assets/img/wp-uploads/2022/10/fIu5yvJFlD.png)](</assets/img/wp-uploads/2022/10/fIu5yvJFlD.png>)Joy plot of the Isle of Wight
