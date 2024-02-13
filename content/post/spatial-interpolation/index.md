---
authors:
- admin
categories:
- Spatial Analysis
- R Programming
date: "2024-02-13T00:00:00Z"
draft: false
featured: true
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/round-gray-framed-compass-on-brown-map-t11oyf1K8kA)'
  focal_point: "Center"
  placement: 1
  preview_only: false
lastmod: "2024-02-13T00:00:00Z"
projects: []
subtitle: 'Demystifying Spatial Interpolation with Ordinary Kriging in R'
summary: 'A beginner-friendly dive into creating spatially interpolated maps using R. No PhD required!'
tags:
- R
- Spatial Analysis
- Kriging
- Mapping
title: 'Mapping the Unseen: Spatial Interpolation with R'
---

# Mapping the Unseen: Spatial Interpolation with R

Ever wondered how to transform scattered data points into a coherent spatial map that tells a story? Today, we're embarking on a cartographic adventure, exploring the realms of **Ordinary Kriging** and **Variograms** with R, your trusty analytical steed. Fear not, brave explorer—while the journey sounds arduous, you won't need a PhD in Geostatistics to follow along!

## What is Ordinary Kriging anyway?

Imagine you're a treasure hunter with a map dotted with clues about where to find hidden gold. But, there's a twist: the map doesn't cover every inch of the land, leaving some treasures undiscovered. What if I told you there's a magical tool that can predict where these unseen treasures lie, using the clues you already have? Welcome to the world of **Ordinary Kriging**, your cartographic crystal ball.

Ordinary Kriging is a statistical technique used in the realm of geographical information systems (GIS) to predict unknown values across a spatial area, based on known data points. Think of it as creating a smooth, continuous surface over a map, where each point on that surface is a treasure chest, and the contents of unopened chests are estimated based on the treasures found in chests you've already opened.

Let's break Ordinary Krigging down into steps that even a pirate without a compass could follow:

1. **Gathering Data**: First, you need a map of known treasures. In spatial analysis, this means collecting data points with known values across your area of interest. These could be measurements like rainfall, temperature, or, in our whimsical analogy, spots where treasure has been found.

2. **Understanding Relationships**: Ordinary Kriging relies on understanding how these data points relate to each other across space. This relationship is modeled through something called a **variogram**, which helps us see patterns like, "The closer the treasures, the more similar their value."

3. **Predicting the Unseen**: With the variogram's guidance, Ordinary Kriging then predicts the values at locations where we haven't dug yet. It cleverly uses the pattern it sees in the known data to estimate what lies beneath the uncharted spots on our map.

4. **Creating a Continuous Map**: The result is a treasure map filled with predictions, showing us where we might find gold (or whatever we're mapping) across the entire area. This is incredibly useful for making informed decisions without having to dig up the whole land.

Now, you might wonder why it's called "Ordinary" Kriging. It's not because it's mundane but because it assumes the mean (average value) is constant and unknown across the entire study area. This is a common scenario in many spatial analysis projects, making Ordinary Kriging a versatile and widely used approach.

Ordinary Kriging is like having a wizard in your pocket, especially when dealing with environmental data, natural resources, or any field where spatial patterns matter. It allows us to make educated guesses about areas they haven't directly observed, saving time, resources, and potentially guiding them to untold riches.

While the concept might sound complex at first, Ordinary Kriging is essentially about using what we know to intelligently predict what we don't. By understanding the spatial relationships in our data, we can fill in the blanks on our maps, making invisible patterns visible and uncovering insights that guide our decisions in the real world.


## Gathering Your Tools

Before we set sail, ensure your toolkit is equipped with R and the following magical packages: `gstat`, `sp`, `ggplot2`, `sf`, and `raster`. These are the arcane ingredients we'll use to conjure our maps from mere numbers.

```r
# Load the necessary spellbooks
library(gstat)
library(sp)
library(ggplot2)
library(sf)
library(raster)
```

## Preparing the Data Scroll

Our journey begins with a scroll of data, my_data.csv. This ancient manuscript contains the locations and attributes of various traits across different realms (in this case, national parks).

```r
# Summoning the data
my_data <- read.csv("/path/to/your/my_data.csv")
```

We'll transform this humble CSV into a SpatialPointsDataFrame, marking each point's position on the Earth's surface.

```r
coordinates(my_data) <- ~lon+lat  # Binding the spell of location
```

## Consulting the Variogram Oracle

The Variogram Oracle reveals how our traits relate across distances, guiding our interpolation quest. We seek its wisdom to craft a model that describes these spatial relationships.

In the adventurous quest of spatial analysis, where we aim to unveil the hidden treasures of data scattered across the land, the **Variogram** emerges as our mystical compass. It's not a map of where the treasures are, but rather a guide to understanding how these treasures—be they gold, raindrops, or trees—relate to each other across the distances that separate them. 

### What is in a Variogram?

Imagine you're planting flowers in your garden. You've noticed that flowers planted close together tend to grow similarly, while those further apart might not share the same fate. A variogram captures this idea of spatial correlation—how one thing relates to another thing over space—and puts it into a form we can use to predict how flowers (or any other data points) will behave across your entire garden.

At its heart, a variogram is a graph that shows the relationship between the spatial distance and the similarity (or dissimilarity) of data points. Here's how it works:

1. **Distance and Difference**: First, we measure how far apart our data points are and calculate the difference in their values (like height, temperature, or soil moisture). 

2. **Plotting the Relationship**: These differences are then squared, averaged, and plotted against the distances to produce the variogram. The result is a curve that rises as the distance increases, illustrating how the similarity between data points decreases as they move further apart.

3. **Key Features of a Variogram**:
  - *Nugget:* The value at which the variogram starts on the y-axis, indicating the variance at zero distance (often due to measurement errors or microscale variations not captured by the sampling).
  - *Range:* The distance at which the variogram levels off, indicating the limit beyond which data points do not influence each other. 
  - *Partial Sill (Psill):* The height of the variogram  plateau minus the nugget, representing the variance attributed to spatial structure.

Variograms are the backbone of spatial interpolation methods like Kriging. They allow us to model the spatial structure of our data, answering critical questions like, "How far apart do two points need to be before they're more different than alike?" With this knowledge, we can predict values at unmeasured locations with greater accuracy, creating a continuous surface or map from discrete data points.

Interpreting a variogram is both an art and a science, requiring an understanding of the underlying spatial processes that generated your data. By fitting a model to the empirical variogram, we can simulate the spatial structure and use this model for making predictions through Kriging.

Variograms offer a window into the spatial soul of our data, providing insights into how things are connected across the landscape. They are essential tools for anyone looking to map the unseen or understand the spatial dynamics at play. Whether you're a scientist, a data enthusiast, or simply curious about the patterns that shape our world, mastering variograms opens up a universe of possibilities for exploration and discovery.

So next time you look at a map or consider the distance between things, remember the variogram: your guide to decoding the mysteries of space and making the invisible, visible.


```r
variogram_modelling <-
  function(mod,
           data,
           psill_t,
           model_t = c("Sph", "Exp", "Lin", "Mat"),
           range_t,
           nugget_t) {
    variogram <- variogram(mod, data)

    variogram.model <-
      fit.variogram(variogram,
                    model = vgm(
                      psill = psill_t,
                      model = model_t,
                      range = range_t,
                      nugget =  nugget_t
                    ))

    print(plot(variogram, model = variogram.model))
    return (variogram.model)
  }
```

With our variogram model in hand, we're ready to perform Ordinary Kriging—the art of predicting unseen values across our map.


## The Kriging Incantation

Armed with our variogram's guidance, we cast the Ordinary Kriging spell. This powerful magic interpolates values across a grid, revealing the hidden patterns of our traits.

```r
ordinary_kriging <- function(...) {
    ordinary_kriging <-
  function(data,
           krig_formula,
           variog_mod,
           shp_file_path,
           var_name,
           grid_count = 200) {

    aoi <- st_read(shp_file_path)
    ext <- extent(aoi)

    # Define a grid over which to interpolate
    gridded_data <-
      expand.grid(
        lon = seq(ext[1], ext[2], length.out = grid_count),
        lat = seq(ext[3], ext[4], length.out = grid_count)
      )

    coordinates(gridded_data) <- ~ lon + lat
    slot(gridded_data, "proj4string") <- crs_wgs84

    # Ordinary kriging
    kriging_result <-
      krige(
        formula = krig_formula,
        locations = data, # knp
        newdata = gridded_data,
        model = variog_mod
      )

    # Convert to a dataframe for plotting
    kriging_df <- as.data.frame(kriging_result)

    # Convert kriging results to RasterLayer
    kriging_raster <-
      rasterFromXYZ(kriging_df[, c("lon", "lat", "var1.pred")])

    masked_raster <- mask(kriging_raster, aoi)
    # Convert masked raster back to a dataframe for ggplot2
    masked_df <- as.data.frame(rasterToPoints(masked_raster))

    p <- ggplot() +
      geom_tile(data = masked_df, aes(x = x, y = y, fill = var1.pred)) +
      #scale_fill_continuous(type = "viridis") +
      scale_fill_gradientn(colors = c("darkgreen", "yellow", "red"),
                          values = scales::rescale(c(0, 1/2, 1))) + # Adjust positions if needed
      coord_fixed() +
      coord_fixed() +
      theme_bw() +
      theme(legend.position = c(0.05, .83)) +
      labs(fill = var_name, x = "Longitude", y = "Latitude")

    print(p)

    fname <- paste0(getwd(), "/", var_name, ".png")
    ggsave(
      fname,
      width = 12,
      height = 8,
      units = "in",
      dpi = 500
    )
    message(paste0("Done saving file to: "), fname)
  }
}
```

As we chant the final words of our kriging incantation, a map materializes before our eyes. This is no ordinary map, but a detailed representation of our data's spatial patterns, rendered visible through the power of R and our analytical prowess.

```r
# Perform the kriging on our example data
ordinary_kriging(data = my_data, ...)
```

## Embarking on Your Own Adventure

Now that you've witnessed the power of spatial interpolation with R, why not embark on your own cartographic quest? With the code spells provided and your data scroll in hand, you're well-equipped to reveal the hidden patterns of your own realm.

Remember, the path to mastery is through practice and exploration. May your maps be many, and your insights profound!

Happy mapping, fellow explorers! Until our paths cross again in the realm of data and beyond.