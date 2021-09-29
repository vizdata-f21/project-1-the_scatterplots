Project title (DON’T FORGET THIS)
================
The Scatterplots

### Introduction

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

## What is the relationship between spending per resident and park size in different U.S. regions and/or cities over time?

### Introduction

To answer our first question (above), we used the “parks” dataset and
analyzed the relationship between the variables
“spend\_per\_resident\_data”, “city”, “med\_park\_size\_data”, and
“year”. We thought it’d very interesting to look into how the spending
on parks varies across different cities and over the course of the last
decade. We were also curious and wanted to analyze how spending per
resident and park size varies across different regions/cities. Our group
felt as though those were four of the most compelling variables in the
dataset and we were all unsure of what the relationship would be between
spending per resident and park size versus year and then region/city, so
we were excited to create visualizations based upon those variables.

### Approach

To answer our first question addressing the relationship between
spending per resident and year, we first created a new variable, called
“bins”, to break down the spending per resident for each city into
four quartiles (where the 1st quartile had the least spending per
resident and the 4th quartile had the most). This allowed us to analyze
how spending trends shifted over time for each quartile. For this
visualization, we created a line graph where each line represents a city
and the color represents the quartile (or “bin”) that the city falls
into. The line graph allowed us to illustrate the trends in different
cities’ spending per resident over time (from 2012-2020) in what we felt
was the clearest way to demonstrate that relationship.

# add second graph approach

### Analysis

``` r
#data wrangling
# to remove the $ and change from a categorical variable to a numerical variable 
# selected relevant variables, pivot wider to see what cities have data from every year
# drop na's from cities that do not have spending data from every year
# pivot longer to return dataset to a structure that can be plotted on a line plot

parks_q1 <- parks %>%
  select(year, city, spend_per_resident_data) %>% 
  mutate(across(starts_with("spend_per_resident_data"), ~gsub("\\$", "", .) 
                  %>% as.numeric)) %>% 
  pivot_wider(names_from = "year", 
              values_from = "spend_per_resident_data") %>% 
  drop_na() %>% #this is where we lose it 
  pivot_longer(cols = starts_with("20"),
               names_to = "year", 
               values_to = "spend_per_resident") 

#making the year variable numeric so we can join med_park_size_data back 
parks_q1 <- parks_q1 %>% 
  mutate(year = as.numeric(year))


#joining based on city and year to include med_park_size_data in the dataset
parks_q1 <- parks %>% 
  select(city, year, med_park_size_data) %>% 
  right_join(parks_q1, by = c("city","year")) 

fivenum(parks_q1$spend_per_resident)
```

    ## [1]  15  62  94 134 399

``` r
#creating quartile bins for spending per resident, ranges based on five number summary
parks_q1 <- parks_q1 %>% 
  arrange(city) %>% 
  mutate(spending = case_when(
    between(spend_per_resident, 0, 62) ~ "1st quartile",
    between(spend_per_resident, 63, 94) ~ "2nd quartile",
    between(spend_per_resident, 95, 134) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  ))

#creating quartile bins for median park size, ranges based on five number summary
  parks_q1 <- parks_q1 %>% 
    mutate(size = case_when(
    between(med_park_size_data, 0, 3.2) ~ "1st quartile",
    between(med_park_size_data, 3.21, 5.0) ~ "2nd quartile",
    between(med_park_size_data, 5.01, 7.7) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  )) 

q1_plot<- ggplot(parks_q1, aes(x = spend_per_resident, y = med_park_size_data, 
                               group = city)) + 
        geom_point(aes(size = size, color = spending)) +   
        labs(title = "Median Park Size vs. Spending Per Resident from 2012-2020 in 37 U.S. Cities",
             subtitle = "Year: {frame_time}",
             x = "Spending per Resident (USD)", 
             y = "Median Park Size (acres)", 
             size = "Park Size", 
             caption = "Quartiles for spending are $0-$62, $63-$94, $95-$134, and $135+ for 1st to 4th quartiles, respectively. 
Quartiles for size are 0-3.2 acres, 3.2-5.0 acres, 5.0-7.7 acres, 7.7+ acres for 1st to 4th quartiles respectively.",
             color = "Spending") +
        theme(plot.caption = element_text(size = 5, hjust = 0), 
              plot.title = element_text(size = 10)) +
        scale_x_continuous(breaks = seq(from = 0, to = 400, by = 50)) + 
        scale_y_continuous(breaks = seq(from = 0, to = 20, by = 5)) +
        scale_color_manual(values = c("#8999b0","#738148","#7c5d2d","#447aab")) +
        transition_time(as.integer(year), range = c(2012L, 2020L))

animate(q1_plot, duration = 18)
```

    ## Warning: Using size for a discrete variable is not advised.

<img src="README_files/figure-gfm/question 1-1.gif" width="90%" />

Links:
<https://github.com/thomasp85/gganimate/wiki/Animation-Composition>
<https://cran.r-project.org/web/packages/gganimate/gganimate.pdf>
<https://gganimate.com/>
<https://www.datanovia.com/en/blog/gganimate-how-to-create-plots-with-beautiful-animation-in-r/>
<http://r-statistics.co/Top50-Ggplot2-Visualizations-MasterList-R-Code.html>
<https://ropensci.org/blog/2018/07/23/gifski-release/>
<https://gif.ski/> <https://github.com/r-rust/gifski>
<https://gganimate.com/articles/gganimate.html#rendering-1>
<https://stackoverflow.com/questions/52899017/slow-down-gganimate-in-r>

*note to self: what is the relationship between spending per resident
and park size in different U.S. cities over time?*

``` r
#data wrangling
parks_regions <- parks_q1 %>% 
  mutate(region = case_when(
         city %in% c("Boston", "Long Beach", "New York", "Philadelphia") ~ "Northeast", 
        city %in% c("Atlanta", "Baltimore", "Jacksonville", "Louisville", 
                    "Memphis", "Nashville", "Virginia Beach") ~ "Southeast", 
         city %in% c("Chicago", "Columbus", "Detroit", "Kansas City", 
                     "Milwaukee") ~ "Midwest", 
         city %in% c("Albuquerque", "Austin", "Dallas", "El Paso", 
                     "Fort Worth", "Houston", "Mesa", "Oklahoma City",
                     "Phoenix", "San Antonio", "Tucson") ~ "Southwest", 
         city %in% c("Denver", "Fresno", "Las Vegas", "Los Angeles", 
                     "Portland", "Sacramento", "San Diego", "San Francisco", 
                     "San Jose", "Seattle") ~ "West"))

parks_regions <- parks_regions %>% 
  group_by(region, year) %>% 
  summarize(mean_spend = mean(spend_per_resident), mean_med_size = mean(med_park_size_data)) %>% 
  print()
```

<<<<<<< HEAD
    ##      city year med_park_size_data spend_per_resident     spending         size
    ## 1 Atlanta 2020               2.90             151.00 4th quartile 1st quartile
    ## 2 Atlanta 2019               2.90             138.00 4th quartile 1st quartile
    ## 3 Atlanta 2018               2.90             139.00 4th quartile 1st quartile
    ## 4 Atlanta 2017               3.10             134.00 3rd quartile 1st quartile
    ## 5 Atlanta 2016               3.10             120.00 3rd quartile 1st quartile
    ## 6 Atlanta 2015               3.10              98.00 3rd quartile 1st quartile
    ## 7 Atlanta 2014               3.10              87.00 2nd quartile 1st quartile
    ## 8 Atlanta 2013               2.95              90.33 2nd quartile 1st quartile
    ## 9 Atlanta 2012               3.00              99.39 3rd quartile 1st quartile
=======
    ## `summarise()` has grouped output by 'region'. You can override using the `.groups` argument.

    ## # A tibble: 45 × 4
    ## # Groups:   region [5]
    ##    region     year mean_spend mean_med_size
    ##    <chr>     <dbl>      <dbl>         <dbl>
    ##  1 Midwest    2012       84.5          4.24
    ##  2 Midwest    2013       85.5          4.33
    ##  3 Midwest    2014       87.6          5.58
    ##  4 Midwest    2015       92.8          5.58
    ##  5 Midwest    2016       97            5.58
    ##  6 Midwest    2017      106.           5.52
    ##  7 Midwest    2018      115.           5.74
    ##  8 Midwest    2019      115.           5.72
    ##  9 Midwest    2020      119.           5.72
    ## 10 Northeast  2012      114.           2.33
    ## # … with 35 more rows
>>>>>>> 03b9b5ef46511839ad23b7760e42639b85c630e9

``` r
ggplot(parks_regions, aes(x = year, y = mean_spend, group = region)) + 
  geom_line(aes(size = mean_med_size, color = region), lineend = "round") + 
  scale_color_manual(values = c("#738148", "#bc8a31", "#3b5c75", "#4f3e23", "#8999b0")) + 
    labs(title = "Mean Spending per Resident Over Time\n with Respect to Mean of Median Park Size", 
         subtitle = "by US Region",
         x = "Year", 
         y = "Mean Spending Per Resident (in USD)", 
         size = "Mean of Median Size (in acres)", 
         color = "Region")
```

<img src="README_files/figure-gfm/question 1 plot 2-1.png" width="90%" />

(2-3 code blocks, 2 figures, text/code comments as needed) In this
section, provide the code that generates your plots. Use scale functions
to provide nice axis labels and guides. You are welcome to use theme
functions to customize the appearance of your plot, but you are not
required to do so. All plots must be made with ggplot2. Do not use base
R or lattice plotting functions.

### Discussion

(1-3 paragraphs) In the Discussion section, interpret the results of
your analysis. Identify any trends revealed (or not revealed) by the
plots. Speculate about why the data looks the way it does.

## How many amenities do parks with the top 10 and bottom 10 rankings in 2020 have and how does this vary based on what proportion of the top 10 and bottom 10 cities’ land is parkland in 2020?

### Introduction

For our second question, we wanted to look at parks with the top and
bottom 10 rankings in 2020 and compare the percentage of their land
being parkland. We also wanted to compare the number of amenities each
of those parks have, including dog parks, playgrounds, restrooms, etc.
To address these questions, we merged the parks and cities datasets by
city, and utilized the “city”, “longitude”, and “latitude” variables
from the cities dataset, as well as the “rank” variable (to determine
the top and bottom 10 park rankings) and variable for the amenities from
the parks dataset. We thought this would be an interesting question to
address as we wanted to illustrate and determine which variables were
the most meaningful when determining the park rankings. We wanted to
compare and interpret whether the top 10 parks had much more amenities
than the bottom 10 parks, and also visualize where those parks are
geographically in the US and how much of their cities’ respective land
is parkland.

### Approach

(1-2 paragraphs) Describe what types of plots you are going to make to
address your question. For each plot, provide a clear explanation as to
why this plot (e.g. boxplot, barplot, histogram, etc.) is best for
providing the information you are asking about. The two plots should be
of different types, and at least one of the two plots needs to use
either color mapping or facets.

### Analysis

``` r
### data wrangling

#top/bottom 10 cities
parks_2020 <- parks %>%
  filter(year == 2020,
         rank <= 10 | rank >= 88)

#matching cities dataframe with parks dataframe
cities <- cities %>%
  filter(state != "Maine") %>%
  mutate(city = case_when(city == "Washington" ~ "Washington, D.C.",
                          city == "Charlotte" ~ "Charlotte/Mecklenburg County",
                          TRUE ~ city)) %>%
  select(city, latitude, longitude) %>%
  rbind(tibble(city = c("Arlington, Virginia"),
               latitude = c(38.8816),
               longitude = c(-77.0910)))

#merging cities and parks data frames
parks_2020_coords <- left_join(parks_2020, cities, by = "city")

#creating an indicator variable for rank
parks_2020_coords <- parks_2020_coords %>%
  mutate(rank_div = ifelse(rank <= 10, "top", "bottom"))

#dodging overlapping points
parks_2020_coords <- parks_2020_coords %>%
  mutate(longitude = case_when(rank == 1 ~ -93.5,
                              rank == 3 ~ -92.6,
                              rank == 2 ~ -76.6,
                              rank == 4 ~ -77.5,
                              rank == 89 ~ -96.7,
                              rank == 94 ~ -97.5,
                              TRUE ~ longitude),
         updown = ifelse(rank %in% c(3, 89, 2), "down", "up"))

### testing out mapping
ggplot() +
  geom_polygon(data = map_data("state"), aes(x = long, y = lat, group = group),
               fill = "white", color = "gray60") +
  geom_point(data = parks_2020_coords,
             aes(x = longitude, y = latitude, color = rank_div,
                 size = as.numeric(str_extract(park_pct_city_data,"\\d+"))/100)) +
  geom_text(data = parks_2020_coords %>% filter(updown == "up"),
            aes(x = longitude, y = latitude, label = paste0("#",rank)),
            size = 3.5, vjust = -.7, family = "bold") +
  geom_text(data = parks_2020_coords %>% filter(updown == "down"),
            aes(x = longitude, y = latitude, label = paste0("#",rank)),
            size = 3.5, vjust = 1.7, family = "bold") +
  scale_size_continuous(labels = scales::percent) +
  scale_color_manual(values = c("#bc8a31", "#315d1b")) + 
  labs(x = NULL, y = NULL, size = "% of city that\nis parkland",
       title = "Top and bottom 10 city rankings of parks",
       subtitle = "scaled by % of city that is parkland") +
  coord_map() + 
  theme_void() +
  guides(color = "none") + 
  theme(legend.position = c(.92,.3),
        plot.title = element_text(hjust = 0.1),
        plot.subtitle = element_text(hjust = 0.1))
```

<img src="README_files/figure-gfm/question-2-1.png" width="90%" />

``` r
parks_2020_coords <- parks_2020_coords %>%
  mutate(total_amenities = playground_data + restroom_data + basketball_data)


parks_amenities <- parks_2020_coords %>%
  pivot_longer(cols = c(playground_data, restroom_data, basketball_data), names_to = "amenity", values_to = "value")





#parks_2020_coords <- parks_2020_coords %>%
 # mutate(rank_quartile = case_when(rank <= 5 ~ "1st quartile",
  #                        rank > 5 & rank <= 10 ~ "2nd quartile",
   #                       rank >= 88 & rank < 93 ~ "3rd quartile",
    #                      rank >= 93 ~ "4th quartile"))

#ggplot(data = parks_amenities, mapping = aes(x = city, y = total_amenities, fill = total_amenities)) + 
  #geom_bar(stat = "identity") +
  #geom_text(data = parks_2020_coords, aes(label = rank), hjust = -.5, color = "black", family = "bold") +
  #coord_flip() +
  #labs(y = "Total Amenities", x = NULL)

ggplot(data = parks_amenities, mapping = aes(x = reorder(city, -rank))) + 
  geom_bar(stat = "identity", mapping = aes(y = value, fill = amenity)) +
  geom_text(data = parks_2020_coords, mapping = aes(label = paste0("#",rank), y = total_amenities), hjust = -.1, 
            color = "black",
            family = "bold",
            size = 2.5) +
  coord_flip() +
  scale_fill_discrete(labels = c("Basketball Courts", "Playgrounds", "Restrooms")) +
  guides(fill = guide_legend(reverse = TRUE)) +
  labs(title = "Top and bottom 10 city rankings by amenities",
       y = "Total Amenities per 10K  Residents", x = NULL, fill = "Amenities") +
  scale_fill_manual(values = c("#bc8a31", "#738148", "#3b5c75")) + 
  theme(plot.title = element_text(hjust = 0),
        plot.subtitle = element_text(hjust = 0)) +
  theme_minimal()
```

    ## Scale for 'fill' is already present. Adding another scale for 'fill', which
    ## will replace the existing scale.

<img src="README_files/figure-gfm/question-2-vis-2-1.png" width="90%" />

\=======

(2-3 code blocks, 2 figures, text/code comments as needed) In this
section, provide the code that generates your plots. Use scale functions
to provide nice axis labels and guides. You are welcome to use theme
functions to customize the appearance of your plot, but you are not
required to do so. All plots must be made with ggplot2. Do not use base
R or lattice plotting functions.

### Discussion

(1-3 paragraphs) In the Discussion section, interpret the results of
your analysis. Identify any trends revealed (or not revealed) by the
plots. Speculate about why the data looks the way it does.

For the second question, our first visualization

## Presentation

Our presentation can be found [here](presentation/presentation.html).

## Data

Include a citation for your data here. See
<http://libraryguides.vu.edu.au/c.php?g=386501&p=4347840> for guidance
on proper citation for datasets. If you got your data off the web, make
sure to note the retrieval date.

## References

List any references here. You should, at a minimum, list your data
source.
