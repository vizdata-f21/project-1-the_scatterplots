animation test
================
The Scatterplots

### Analysis

``` r
#data wrangling
fivenum(parks$spend_per_resident_data)
```

    ## Warning in Ops.factor(x[floor(d)], x[ceiling(d)]): '+' not meaningful for
    ## factors

    ## [1] NA NA NA NA NA

``` r
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
glimpse(parks_q1)
```

    ## Rows: 333
    ## Columns: 3
    ## $ city               <fct> "Portland", "Portland", "Portland", "Portland", "Po…
    ## $ year               <dbl> 2020, 2019, 2018, 2017, 2016, 2015, 2014, 2013, 201…
    ## $ spend_per_resident <dbl> 250.00, 224.00, 251.00, 165.00, 154.00, 145.00, 155…

``` r
#joining based on city and year to include med_park_size_data in the dataset
parks_q1 <- parks %>% 
  select(city, year, med_park_size_data) %>% 
  right_join(parks_q1, by = c("city","year")) 

#creating quartile bins for spending per resident, ranges based on five number summary
parks_q1 <- parks_q1 %>% 
  arrange(city) %>% 
  mutate(spend_bins = case_when(
    between(spend_per_resident, 0, 59) ~ "1st quartile",
    between(spend_per_resident, 60, 84) ~ "2nd quartile",
    between(spend_per_resident, 85, 131) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  ))

#creating quartile bins for median park size, ranges based on five number summary
  parks_q1 <- parks_q1 %>% 
    mutate(size_bins = case_when(
    between(med_park_size_data, 0, 3.2) ~ "1st quartile",
    between(med_park_size_data, 3.21, 5.0) ~ "2nd quartile",
    between(med_park_size_data, 5.01, 7.7) ~ "3rd quartile",
    TRUE ~ "4th quartile"
  )) 

q1_plot<- ggplot(parks_q1, aes(x = spend_per_resident, y = med_park_size_data, 
                     size = size_bins, color = spend_bins)) + 
        geom_point() +  
        labs(title = "INSERT TITLE HERE",
            subtitle = "Year: {frame_time}",
             x = "Spending per Resident (in USD)", 
             y = "Median Park Size (in acres)", 
             size = "Size Bins", 
            #it would be great if these legends had the actual values of the bins, maybe we change observation names? 
             color = "Spend Bins") +
        scale_x_continuous(breaks = seq(from = 0, to = 400, by = 50)) + 
        transition_time(year)

animate(q1_plot)
```

    ## Warning: Using size for a discrete variable is not advised.

<img src="animation-test_files/figure-gfm/question 1-1.gif" width="90%" />
