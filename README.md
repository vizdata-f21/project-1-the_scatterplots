Project title
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
“year”. We thought it would be interesting to look into how the
spending on parks varies across different cities and over the course of
the last decade. We were also curious and wanted to analyze how spending
per resident and park size varies across different regions/cities. Our
group felt as though those were four of the most compelling variables in
the dataset and we were all unsure of what the relationship would be
between spending per resident and park size versus year and then
region/city, so we were excited to create visualizations based upon
those variables.

### Approach

(1-2 paragraphs) Describe what types of plots you are going to make to
address your question.

For each plot, provide a clear explanation as to why this plot
(e.g. boxplot, barplot, histogram, etc.) is best for providing the
information you are asking about.

The two plots should be of different types, and at least one of the two
plots needs to use either color mapping or facets.

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
fivenum(parks$spend_per_resident_data)
```

    ## Warning in Ops.factor(x[floor(d)], x[ceiling(d)]): '+' not meaningful for
    ## factors

    ## [1] NA NA NA NA NA

``` r
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

# to remove the $ and change from a categorical variable to a numerical variable 
# selected relevant variables, pivot wider to see what cities have data from every year
# drop na's from cities that do not have spending data from every year
# pivot longer to return dataset to a structure that can be plotted on a line plot

parks_q1 <- parks_q1 %>% 
  mutate(year = as.numeric(year))
glimpse(parks_q1)
```

    ## Rows: 333
    ## Columns: 3
    ## $ city               <fct> "Portland", "Portland", "Portland", "Portland", "Po…
    ## $ year               <dbl> 2020, 2019, 2018, 2017, 2016, 2015, 2014, 2013, 201…
    ## $ spend_per_resident <dbl> 250.00, 224.00, 251.00, 165.00, 154.00, 145.00, 155…

``` r
parks_q1 <- parks %>% 
  select(city, year, med_park_size_data) %>% 
  right_join(parks_q1, by = c("city","year")) 

parks_q1 <- parks_q1 %>% 
  arrange(city) %>% 
  mutate(spend_bins = case_when(
    between(spend_per_resident, 0, 59) ~ "1st quartile",
    between(spend_per_resident, 60, 84) ~ "2nd quartile",
    between(spend_per_resident, 85, 131) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  ))

  parks_q1 <- parks_q1 %>% 
    mutate(size_bins = case_when(
    between(med_park_size_data, 0, 3.2) ~ "1st quartile",
    between(med_park_size_data, 3.21, 5.0) ~ "2nd quartile",
    between(med_park_size_data, 5.01, 7.7) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  ))

ggplot(data = parks_q1, mapping = aes(x = year, y = spend_per_resident, group = city)) + 
  geom_line() 
```

![](README_files/figure-gfm/question%201-1.png)<!-- -->

``` r
ggplot(parks_q1, aes(x = spend_per_resident, y = med_park_size_data, 
                     size = size_bins)) + 
        geom_point() +  
        labs(title = "Year: {frame_time}", x = "Spend per Resident", y = "Median Park Size") +
        transition_time(year)
```

    ## Warning: No renderer available. Please install the gifski, av, or magick package
    ## to create animated output

    ## NULL

``` r
#figure this out 

#anim_save(filename = question1_plot, animation = last_animation, path = NULL)

#animate(q1_plot, renderer = ffmpeg_renderer())


#ggplot2::ggsave

#SUGGESTION FROM MINE: This is just a suggestion, in case your original plan doesn't yield as compelling a plot as you'd like: you might consider whether the park is located in a metropolitan city or not and explore relationships about other amenities in the park depending on this variable.
  #create a binary variable if in metropolitan city or not 
```

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

#merging cities and parks dataframes
parks_2020_coords <- left_join(parks_2020, cities, by = "city")

#creating an indicator variable for rank
parks_2020_coords <- parks_2020_coords %>%
  mutate(rank_div = ifelse(rank <= 10, "top", "bottom"))

### testing out mapping
ggplot() +
  geom_polygon(data = map_data("state"), aes(x = long, y = lat, group = group),
               fill = "white", color = "#3c3b6e") +
  geom_point(data = parks_2020_coords,
             aes(x = longitude, y = latitude, color = rank_div,
                 size = park_pct_city_points),
             show.legend = FALSE) +
  labs(x = NULL, y = NULL) +
  coord_map() + 
  theme_void()
```

![](README_files/figure-gfm/question-2-1.png)<!-- -->

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
