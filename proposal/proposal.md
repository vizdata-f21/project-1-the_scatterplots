Project proposal
================
The Scatterplots

``` r
library(tidyverse)
```

``` r
parks <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2021/2021-06-22/parks.csv')
```

## Dataset

Our dataset comes from The Trust for Public Land’s ParkScore Index, an
annually released report which ranks the parks of the 100 most populated
cities in the United States according to 4 key metrics: access,
investment, amenities, and acreage. There are 713 rows and 28 columns in
the data set ranging from 2012 through 2020.

The COVID-19 Pandemic has laid bare a multitude of inequities that exist
in the United States. While many people were able to flock to their
local parks in order to get out of the house, many others were not. Our
group is interested in exploring the Parks Access dataset in order to
answer questions regarding access and equity to the nation’s largest
cities’ parks. The Citylab article accompanying the dataset from
TidyTuesday discussed a number of reasons for unequal access to quality
parks and we are hoping to explore these in our project.

## Questions

1- What is the relationship between spending per resident and park size
in different U.S. regions and/or cities over time?

2- How many amenities do parks with the top 10 and bottom 10 rankings in
2020 have and how does this vary based on what proportion of the top 10
and bottom 10 cities’ land is parkland in 2020?

## Analysis plan

1- To answer the first question we are planning to use the `year`,
`spend_per_resident_data`, and `med_park_size_data` from the dataset. We
will create another variable of quartiles/quintiles of the spending per
resident in order to create “bins” that each city-year can fall into as
well as a categorical variable describing which region of the US the
cities fall into. We plan on testing various visualizations that allow
us to explore these categorical differences and then making the decision
of which best suits our research question. We will create our final
visualizations based on which relationship (spending per resident and
park size vs. region or vs. city) we find to be the most compelling and
works best on a visualization.

2- To answer the second question, we are planning to create a geographic
plot of the 20 cities that rank within the top 10 and bottom 10. By
focusing in on the top and bottom 10, we are able to get a general idea
of what the best and worst cities have to offer park-wise, without
overwhelming the plot with all 100 cities. We intend to use the
variables `rank` and `park_pct_city_data`. We will also use additional
data to plot the points geographically on a map of the United States. We
will initially attempt to do this through geocoding, using the ggmap
package and guidance found at the following link.
(<https://www.jessesadler.com/post/geocoding-with-r/>) In this map, we
intend to display which proportions of each cities’ land is parkland
through the size of the points on the map (using the variable
`park_pct_city_data`) and next to the point we will annotate the plot
with the respective ranking of that city. For a second visualization, we
will plot the points data for the various amenities provided in the data
set on a map where the points will be mapped to size (ie: more points
for `splashground_points` or `playground_points` for example will result
in a larger point) and shape will be mapped to amenity type. This will
allow us to compare the proportion of parkland in the 10 top and bottom
cities map to the amenties map and see if there is any correlation.
