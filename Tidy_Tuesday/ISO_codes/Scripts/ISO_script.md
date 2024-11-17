---
title: "ISO Country Codes"
author: "Annie Deck"
date: "2024-11-16"
output: 
 html_document:
   keep_md: yes
---


# R Markdown tabs {.tabset}
* This week I learned how to make each of my subsections of code into their own tab in R Markdown

## Load libraries

``` r
library(tidyverse)
library(here)
library(maps)
library(mapdata)
library(mapproj)
library(beyonce)
```

## Bring in data

``` r
countries <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2024/2024-11-12/countries.csv')

country_subdivisions <-readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2024/2024-11-12/country_subdivisions.csv')

former_countries <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2024/2024-11-12/former_countries.csv')

##Save to data folder 
readr::write_csv(countries, here::here("Tidy_Tuesday", "ISO_codes", "Data", "countries.csv"))

readr::write_csv(country_subdivisions, here::here("Tidy_Tuesday", "ISO_codes", "Data", "country_subdivisions.csv"))
```

## Data wrangling 

``` r
## create new data set of subdivision count per country
subdivision_counts <- country_subdivisions %>%
  count(alpha_2, name = "subdivision_count") ##group by alpha_2 country code

##join data sets
country_joined <- countries %>%
  left_join(subdivision_counts, by = "alpha_2") %>% ##joining by shared column alpha 2
  mutate(subdivision_count = replace_na(subdivision_count, 0)) ##replace NA values with 0 for countries with no subdivisions

##get lat and long for countries
world <- map_data("world")

##get world data and fix country names to match
world <- map_data("world") %>%
  mutate(region = case_when(
    region == "USA" ~ "United States",
    region == "UK" ~ "United Kingdom",
     region == "Democratic Republic of the Congo" ~ "Congo, Democratic Republic Of The",
    region == "Republic of Congo" ~ "Congo",
    region == "Czech Republic" ~ "Czechia",
    region == "Ivory Coast" ~ "Côte D'Ivoire",
    region == "Russia" ~ "Russian Federation",
    region == "Bolivia" ~ "Bolivia, Plurinational State of",
    region == "Venezuela" ~ "Venezuela, Bolivarian Republic of",
    region == "Tanzania" ~ "Tanzania, United Republic of",
    TRUE ~ region
  ))

##join data sets
world_data <- world %>%
  inner_join(country_joined, by = c("region" = "name"))
```

## Plot 

``` r
ISO_plot <- world_data %>%
  ggplot(aes(x = long, 
             y = lat, 
             group = group,
             fill = subdivision_count)) +
  geom_polygon(color = "white", size = 0.1) + ##adds country shapes and puts thin white line between them 
  scale_fill_gradient2( # add color scale 
    low = "#298", #white
    mid = "#2ca25f", #light green
    high = "#006", #dark green
    midpoint = median(world_data$subdivision_count), ##center color scale at median value
    name = "Number of Subdivisions",
    labels = function(x) round(x, 0)  # Round the legend numbers
  ) +
  theme_minimal() + #add theme
  labs(title = "Number of ISO Identified Subdivisions by Country", #title
       caption = "Source: Tidy Tuesday ISO Country Codes dataset") + #caption
  coord_map(projection = "mercator") +  #Mercator projection
  theme(plot.title = element_text(size = 14, face = "bold"), #make title look nice
    axis.title = element_blank(),
    axis.text = element_blank(),
    panel.grid = element_blank()) +
  scale_y_continuous(limits = c(-60, 90)) +  #Limit latitude to reduce polar distortion
  coord_fixed(ratio = 1.3)  # Adjusts width/height proportion to reduce distortion 

ISO_plot
```

<img src="ISO_script_files/figure-html/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

## Saving

``` r
ggsave(here("Tidy_Tuesday", "ISO_codes", "Output", "ISO_plot.png"))
```
