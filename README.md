Project title
================
The Scatterplots

## Introduction

(1-2 paragraphs)

Brief introduction to the dataset.

You may repeat some of the information about the dataset provided in the
introduction to the dataset on the TidyTuesday repository, paraphrasing
on your own terms.

Imagine that your project is a standalone document and the grader has no
prior knowledge of the dataset.

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

(1-2 paragraphs) Introduction to the question and what parts of the
dataset are necessary to answer the question. Also discuss why you’re
interested in this question.

### Approach

(1-2 paragraphs) Describe what types of plots you are going to make to
address your question.

For each plot, provide a clear explanation as to why this plot
(e.g. boxplot, barplot, histogram, etc.) is best for providing the
information you are asking about.

The two plots should be of different types, and at least one of the two
plots needs to use either color mapping or facets.

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
  select(year, city, spend_per_resident_data, med_park_size_data) %>% 
  mutate(across(starts_with("spend_per_resident_data"), ~gsub("\\$", "", .) 
                  %>% as.numeric)) %>% 
  pivot_wider(names_from = "year", 
              values_from = "spend_per_resident_data") %>% 
  drop_na() %>% 
  pivot_longer(cols = starts_with("20"),
               names_to = "year", 
               values_to = "spend_per_resident")

  #from K: i tried to add `med_park_size_data` in the select() function, but then it only returns data on memphis, not sure why this is happening cause i never call city into a pivot function and that worked 
#posted on discussion about this, hopefully we get a response soon 

parks_q1 <- parks_q1 %>% 
  mutate(bins = case_when(
    between(spend_per_resident, 0, 59) ~ "1st quartile",
    between(spend_per_resident, 60, 84) ~ "2nd quartile",
    between(spend_per_resident, 85, 131) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  ))

# to remove the $ and change from a categorical variable to a numerical variable 
# selected relevant variables, pivot wider to see what cities have data from every year
# drop na's from cities that do not have spending data from every year
# pivot longer to return dataset to a structure that can be plotted on a line plot

ggplot(data = parks_q1, mapping = aes(x = year, y = spend_per_resident, group = city)) + 
  geom_line(mapping = aes(color = bins))
```

![](README_files/figure-gfm/question%201-1.png)<!-- -->

``` r
    #it looks like the quartile / color changes for some cities over time, interesting observation prob could draw conclusion from 

#NOTE: include med_park_size_data in parks_q1

#in our proposal we said med_park_size_data would also be part of this viz so we need to make sure thats included in the dataset parks_q1
  #from K: i tried to add it in the select() function, but then it only returns data on memphis 

#SUGGESTION FROM MINE: This is just a suggestion, in case your original plan doesn't yield as compelling a plot as you'd like: you might consider whether the park is located in a metropolitan city or not and explore relationships about other amenities in the park depending on this variable.
  #create a binary variable if in metropolitan city or not 

us_cities <- us.cities #from maps package
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

(1-2 paragraphs) Introduction to the question and what parts of the
dataset are necessary to answer the question. Also discuss why you’re
interested in this question.

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
             show.legend = F) +
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
