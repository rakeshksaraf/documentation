---
title: County-Level Choropleth in R | Examples | Plotly
name: County Level Choropleth
permalink: r/county-level-choropleth/
description: How to create county-level choropleths in R with Plotly.
layout: base
thumbnail: thumbnail/county-level-choropleth.jpg
language: r
has_thumbnail: true
display_as: maps
order: 6
output:
  html_document:
    keep_md: true
---



### New to Plotly?

Plotly's R library is free and open source!<br>
[Get started](https://plot.ly/r/getting-started/) by downloading the client and [reading the primer](https://plot.ly/r/getting-started/).<br>
You can set up Plotly to work in [online](https://plot.ly/r/getting-started/#hosting-graphs-in-your-online-plotly-account) or [offline](https://plot.ly/r/offline/) mode.<br>
We also have a quick-reference [cheatsheet](https://images.plot.ly/plotly-documentation/images/r_cheat_sheet.pdf) (new!) to help you get started!

### Version Check

Version 4 of Plotly's R package is now [available](https://plot.ly/r/getting-started/#installation)!<br>
Check out [this post](http://moderndata.plot.ly/upgrading-to-plotly-4-0-and-above/) for more information on breaking changes and new features available in this version.

```r
library(plotly)
packageVersion('plotly')
```

```
## [1] '4.5.6.9000'
```
### Mapbox Access Token

To create mapbox maps with Plotly, you'll need a Mapbox account and a [Mapbox Access Token](https://www.mapbox.com/studio/) that you can add to your [Plotly Settings](https://plot.ly/settings/mapbox). If you're using a Chart Studio Enterprise server, please see additional instructions here: https://help.plot.ly/mapbox-atlas/.

### Creating Polygon Boundaries


```r
library(plotly)

blank_layer <- list(
  title = "",
  showgrid = F,
  showticklabels = F,
  zeroline = F)

p <- map_data("county") %>%
  filter(region == 'california') %>%
  group_by(group) %>%
  plot_ly(
    x = ~long,
    y = ~lat,
    fillcolor = 'white',
    hoverinfo = "none") %>%
  add_polygons(
    line = list(color = 'black', width = 0.5)) %>%
  layout(
    xaxis = blank_layer,
    yaxis = blank_layer)

# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/getting-started
chart_link = api_create(p, filename="county-level-create-polygons")
chart_link
```

<iframe src="https://plot.ly/~RPlotBot/4375.embed" width="800" height="600" id="igraph" scrolling="no" seamless="seamless" frameBorder="0"> </iframe>

### Add County-Level Data


```r
library(tidyverse)
library(plotly)

df <- read.csv("https://raw.githubusercontent.com/bcdunbar/datasets/master/californiaPopulation.csv")

cali <- map_data("county") %>%
  filter(region == 'california')

pop <- df %>%
  group_by(County.Name) %>%
  summarise(Pop = sum(Population))

pop$County.Name <- tolower(pop$County.Name) # matching string

cali_pop <- merge(cali, pop, by.x = "subregion", by.y = "County.Name")

cali_pop$pop_cat <- cut(cali_pop$Pop, breaks = c(seq(0, 11000000, by = 500000)), labels=1:22)

p <- cali_pop %>%
  group_by(group) %>%
  plot_ly(x = ~long, y = ~lat, color = ~pop_cat, colors = c('#ffeda0','#f03b20'),
          text = ~subregion, hoverinfo = 'text') %>%
  add_polygons(line = list(width = 0.4)) %>%
  add_polygons(
    fillcolor = 'transparent',
    line = list(color = 'black', width = 0.5),
    showlegend = FALSE, hoverinfo = 'none'
  ) %>%
  layout(
    title = "California Population by County",
    titlefont = list(size = 10),
    xaxis = list(title = "", showgrid = FALSE,
                 zeroline = FALSE, showticklabels = FALSE),
    yaxis = list(title = "", showgrid = FALSE,
                 zeroline = FALSE, showticklabels = FALSE)
  )

# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/getting-started
chart_link = api_create(p, filename="county-level-fill-polygons")
chart_link
```

<iframe src="https://plot.ly/~RPlotBot/4377.embed" width="800" height="600" id="igraph" scrolling="no" seamless="seamless" frameBorder="0"> </iframe>

### Add Polygon to a Map Projection


```r
library(plotly)

geo <- list(
  scope = 'usa',
  showland = TRUE,
  landcolor = toRGB("gray95"),
  countrycolor = toRGB("gray80")
)

p <- cali_pop %>%
  group_by(group) %>%
  plot_geo(
    x = ~long, y = ~lat, color = ~pop_cat, colors = c('#ffeda0','#f03b20'),
    text = ~subregion, hoverinfo = 'text') %>%
  add_polygons(line = list(width = 0.4)) %>%
  add_polygons(
    fillcolor = 'transparent',
    line = list(color = 'black', width = 0.5),
    showlegend = FALSE, hoverinfo = 'none'
  ) %>%
  layout(
    title = "California Population by County",
    geo = geo)

# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/getting-started
chart_link = api_create(p, filename="county-level-geo")
chart_link
```

<iframe src="https://plot.ly/~RPlotBot/4379.embed" width="800" height="600" id="igraph" scrolling="no" seamless="seamless" frameBorder="0"> </iframe>

### Add Polygon to Mapbox

To create mapbox maps with Plotly, you'll need a Mapbox account and a [Mapbox Access Token](https://www.mapbox.com/studio/) that you can add to your [Plotly Settings](https://plot.ly/settings/mapbox).


```r
library(plotly)

Sys.setenv('MAPBOX_TOKEN' = 'your_mapbox_token_here')
```


```r
library(plotly)

p <- cali_pop %>%
  group_by(group) %>%
  plot_mapbox(x = ~long, y = ~lat, color = ~pop_cat, colors = c('#ffeda0','#f03b20'),
          text = ~subregion, hoverinfo = 'text', showlegend = FALSE) %>%
  add_polygons(
    line = list(width = 0.4)
  ) %>%
  add_polygons(fillcolor = 'transparent',
    line = list(color = 'black', width = 0.5),
    showlegend = FALSE, hoverinfo = 'none'
  ) %>%
  layout(
    xaxis = list(title = "", showgrid = FALSE, showticklabels = FALSE),
    yaxis = list(title = "", showgrid = FALSE, showticklabels = FALSE),
    mapbox = list(
      style = 'light',
      zoom = 4,
      center = list(lat = ~median(lat), lon = ~median(long))),
    margin = list(l = 0, r = 0, b = 0, t = 0, pad = 0)
  )

# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/getting-started
chart_link = api_create(p, filename="county-level-mapbox")
chart_link
```

<iframe src="https://plot.ly/~RPlotBot/4381.embed" width="800" height="600" id="igraph" scrolling="no" seamless="seamless" frameBorder="0"> </iframe>
